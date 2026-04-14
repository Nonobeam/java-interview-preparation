# A30 — JPA Pessimistic Locking

## 1. The three pessimistic modes

| Mode | Generated SQL (Oracle) | Prevents |
|------|------------------------|----------|
| `PESSIMISTIC_READ` | `SELECT ... FOR UPDATE` on most dialects — Oracle has no `FOR SHARE`, so Hibernate degrades to `FOR UPDATE`. PostgreSQL uses `FOR SHARE`. | Dirty reads and non-repeatable reads. Other transactions may still read. |
| `PESSIMISTIC_WRITE` | `SELECT ... FOR UPDATE` | Dirty read, non-repeatable read, and lost update. Other transactions reading the same row will block if they also request `FOR UPDATE`. |
| `PESSIMISTIC_FORCE_INCREMENT` | `SELECT ... FOR UPDATE` + an `UPDATE` that bumps `@Version` at flush time, even if no business field changed | Same as `PESSIMISTIC_WRITE`, plus forces optimistic lock failure in any *other* concurrent tx that read the old version. Requires `@Version` on the entity. |

Oracle-specific notes:
- Oracle readers never block writers and vice versa (MVCC). `FOR UPDATE` takes a row lock that blocks other writers / other `FOR UPDATE` selects, but non-locking selects still see the last committed snapshot.
- `FOR UPDATE NOWAIT` (JPA hint: `jakarta.persistence.lock.timeout = 0`) returns `ORA-00054` instead of blocking. `FOR UPDATE WAIT n` uses a bounded wait.
- `FOR UPDATE SKIP LOCKED` is the classic pattern for queue-style table polling — not a JPA standard mode, use a native query.

## 2. When to prefer pessimistic over `@Version`

Pick pessimistic when **the cost of a retry is high** or **contention is guaranteed**.

- **Long critical sections with side effects.** If the transaction calls an external API (payment gateway, message broker) after reading, optimistic-retry means you either double-charge or need an idempotency key per retry. Pessimistic locking avoids the retry entirely.
- **Hot row with high contention.** An inventory counter that 200 requests per second try to decrement will fail optimistic locking almost every time — you spend more CPU on retries than on real work. Pessimistic serialises the access; for extreme cases, split the counter (see A21 capacity sharding) or move it to Redis.
- **Multi-row critical sections where retries are expensive.** E.g. a month-end accounting run that touches 50 ledger rows — a single optimistic failure means redoing the whole batch.

Pick optimistic when the write rate is low relative to reads, the retry is cheap, and you don't want lock wait queues.

## 3. The deadlock in `transfer` and how to fix it

**Interleaving:**
1. Thread T1: `transfer(A, B, 100)` — locks row A (`FOR UPDATE`).
2. Thread T2: `transfer(B, A, 50)`  — locks row B.
3. T1: tries to lock B — blocks on T2.
4. T2: tries to lock A — blocks on T1.

Oracle detects the cycle and kills one session with `ORA-00060: deadlock detected`. Result: one transaction fails at random, the application sees `CannotAcquireLockException`.

**Fix: deterministic lock ordering.** Always lock rows in a canonical order (e.g. ascending ID). Restructure to acquire both locks in one shot:

```java
@Transactional
public void transfer(Long fromId, Long toId, BigDecimal amount) {
    Long first  = Math.min(fromId, toId);
    Long second = Math.max(fromId, toId);
    Account a1 = repo.findForUpdate(first);
    Account a2 = repo.findForUpdate(second);

    Account from = fromId.equals(first) ? a1 : a2;
    Account to   = toId.equals(first)   ? a1 : a2;
    // proceed
}
```

Or issue a single query that locks both rows in a defined order:
```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("select a from Account a where a.id in (:ids) order by a.id")
List<Account> lockAll(@Param("ids") List<Long> ids);
```

With deterministic ordering, T1 and T2 both request A first, then B — one waits politely instead of deadlocking.

Complementary mitigations: set a short `jakarta.persistence.lock.timeout` so a deadlock-prone call fails fast rather than hanging, and instrument the service to detect and retry on `LockTimeoutException` / `CannotAcquireLockException`.

## 4. `jakarta.persistence.lock.timeout` behaviour

```java
@QueryHints(@QueryHint(name = "jakarta.persistence.lock.timeout", value = "3000"))
```

- **Oracle** — translates to `FOR UPDATE WAIT 3` (seconds). If another tx holds the lock longer than that, the session raises `ORA-30006` and JPA throws `LockTimeoutException`. Value `0` → `NOWAIT` (`ORA-00054`). Value `-1` (or `LockModeType` without hint) → `FOR UPDATE` with no timeout.
- **PostgreSQL** — no native `FOR UPDATE WAIT n`. Hibernate falls back to setting `statement_timeout` or uses `FOR UPDATE NOWAIT` only for timeout `0`. For finite waits, you typically rely on `lock_timeout` session parameter.
- **MySQL / InnoDB** — respects `innodb_lock_wait_timeout` (default 50s) globally. The hint isn't applied per-query by default; Hibernate may log a warning. Some versions support `FOR UPDATE NOWAIT` / `SKIP LOCKED` (8.0+).

The portability of the hint is weak — treat it as "please be fast", and always have a retry/fallback path.

## 5. Calling `findForUpdate` outside a transaction

- **No active transaction**: JPA spec says pessimistic locking requires a transaction context. Hibernate throws `TransactionRequiredException` (or silently returns without the lock depending on version — never rely on this). Spring's `@Transactional` on the caller is mandatory.
- **`@Transactional(readOnly = true)`**: `readOnly` is a hint — it sets `Connection.setReadOnly(true)` (which some drivers use to route to read replicas) and puts Hibernate into `FlushMode.MANUAL` to skip dirty checking. It does **not** forbid locking statements. Oracle will happily issue `FOR UPDATE` inside a "read only" JDBC connection. However, mixing `readOnly = true` with a lock is a design smell: you're declaring "I won't modify" then asking for a write lock. Either drop `readOnly` or split the operation.

## 6. Pessimistic lock + second-level cache

- **Entity L2 cache (`@Cacheable` on entity)**: Hibernate's pessimistic lock acquisition bypasses the L2 cache for the locked load — it always issues the `SELECT ... FOR UPDATE`. On commit, the cache entry is invalidated (or updated, depending on `CacheConcurrencyStrategy`). You do **not** get stale reads for the locked row.
- **Other readers hitting L2** during the lock window: they can still see the *old* cached value, because they're using non-locking reads. This is usually fine — if they needed the latest, they'd be taking a lock themselves. Use `TRANSACTIONAL` or `READ_WRITE` concurrency strategy if you want tighter coordination.
- **Query cache**: a plain `@Query` with `@Lock` invalidates the query cache for affected entities on commit. But any non-locking query that hit the query cache during the lock window can still see pre-lock results. The query cache is stamped against a table's update timestamp — the timestamp is bumped at commit.

Rule: L2 cache + pessimistic locking works, but do not assume the cache is "immediately consistent" with the locked row for *other* readers. It's eventually consistent on commit.

## Interview-ready answer

> "`PESSIMISTIC_WRITE` issues `SELECT ... FOR UPDATE` and blocks other writers; `PESSIMISTIC_READ` is the shared variant, though on Oracle it degrades to `FOR UPDATE` because there's no `FOR SHARE`. `PESSIMISTIC_FORCE_INCREMENT` additionally bumps `@Version` so other optimistic readers fail. I use pessimistic when contention is guaranteed or when a retry would cause side-effect duplication — e.g. transfer operations with external calls. The classic trap is deadlock from non-deterministic lock order; fix it by always locking rows in ascending ID. Control wait behaviour with `jakarta.persistence.lock.timeout`, though Oracle is the only mainstream DB that fully honours it per-query via `FOR UPDATE WAIT n`."
