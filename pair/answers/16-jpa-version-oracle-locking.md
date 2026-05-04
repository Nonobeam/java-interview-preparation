## A20: How does JPA `@Version` work with Oracle? Does Oracle have its own row versioning mechanism?

### `@Version` is NOT a Database Feature

It's a **JPA/Hibernate-level** mechanism — works identically across Postgres, Oracle, MySQL, H2. Hibernate implements it as a plain column + a conditional UPDATE:

```sql
CREATE TABLE slot (
    id NUMBER PRIMARY KEY,
    available NUMBER,
    version NUMBER  -- just a normal column, nothing special
);

-- Hibernate generates on save:
UPDATE slot SET available = ?, version = version + 1
WHERE id = ? AND version = ?
```

If 0 rows updated → `OptimisticLockException` thrown.

### So Why Does Postgres Seem Special?

Postgres has **MVCC** with internal `xmin`/`xmax` columns — every row carries transaction metadata. But **JPA ignores all that.** `@Version` is implemented at the application layer to stay database-agnostic.

### Oracle's Native Mechanisms

**1. `ORA_ROWSCN` — Row System Change Number**

Oracle tracks when each row was last modified via an implicit SCN:
```sql
SELECT id, available, ORA_ROWSCN FROM slot WHERE id = 1;
-- returns: 1, 5, 12345678

UPDATE slot SET available = available - 1
WHERE id = 1 AND ORA_ROWSCN = 12345678;
```

**Catch:** By default Oracle stores SCN at the **block level**, not row level. For row-level tracking the table must be created with:
```sql
CREATE TABLE slot (...) ROWDEPENDENCIES;
```
Most teams don't use this — less portable, requires schema changes, and `@Version` already solves the problem.

**2. `SELECT ... FOR UPDATE` (Pessimistic locking)**

Works in both Oracle and Postgres:
```sql
SELECT * FROM slot WHERE id = 1 FOR UPDATE;
```

**3. Oracle-specific `FOR UPDATE` variants**

Oracle has richer options:
```sql
SELECT * FROM slot WHERE id = 1 FOR UPDATE NOWAIT;      -- fail immediately if locked
SELECT * FROM slot WHERE id = 1 FOR UPDATE WAIT 5;       -- wait up to 5 seconds
SELECT * FROM slot WHERE id = 1 FOR UPDATE SKIP LOCKED; -- skip rows that are locked
```

- **`NOWAIT`** — immediate fail, no blocking. Good for UX ("slot busy, try again").
- **`WAIT 5`** — wait up to N seconds, then fail. Prevents indefinite hangs.
- **`SKIP LOCKED`** — skip locked rows entirely. Ideal for **queue processing / worker pools** where multiple workers pull pending bookings concurrently.

Postgres has `NOWAIT` and `SKIP LOCKED` too (Postgres 9.5+), but not `WAIT n`.

### JPA Hints for Oracle Lock Variants

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@QueryHints({
    @QueryHint(name = "javax.persistence.lock.timeout", value = "0")  // NOWAIT
})
@Query("SELECT s FROM Slot s WHERE s.id = :id")
Slot findForUpdateNowait(@Param("id") Long id);
```

Timeout values:
- `0` → NOWAIT
- `-2` → SKIP LOCKED (Hibernate extension)
- `>0` → WAIT milliseconds

### Comparison Table

| Feature | Postgres | Oracle | JPA `@Version` |
|---------|----------|--------|---------------|
| Application-level versioning | ✓ (just a column) | ✓ (just a column) | ✓ Works on both |
| Native row-level SCN | No | `ORA_ROWSCN` with `ROWDEPENDENCIES` | N/A (ignored) |
| `SELECT FOR UPDATE` | ✓ | ✓ | ✓ via `@Lock(PESSIMISTIC_WRITE)` |
| `NOWAIT` | ✓ | ✓ | ✓ via `lock.timeout = 0` |
| `WAIT n` | ✗ | ✓ | ✓ via `lock.timeout = N ms` |
| `SKIP LOCKED` | ✓ (9.5+) | ✓ | ✓ via Hibernate extension |

### Practical Recommendation

- **Default to `@Version`** — portable, simple, no schema changes, works across all databases
- **Use `SELECT FOR UPDATE` (pessimistic)** when you must read multiple fields, validate complex logic, then write atomically
- **Use `SKIP LOCKED` (Oracle)** for worker queues — multiple threads pulling pending bookings without fighting over the same row
- **Use `ORA_ROWSCN`** only in legacy Oracle apps where JPA isn't used

### Interview-Ready Answer

> "`@Version` is DB-agnostic — Hibernate implements it as a plain column with a conditional `UPDATE ... WHERE version = ?`, so Postgres and Oracle behave identically. Oracle does have `ORA_ROWSCN` for native row-level change tracking, but it requires `ROWDEPENDENCIES` on the table and most teams stick with `@Version` for portability. Where Oracle shines is pessimistic locking — `SELECT FOR UPDATE SKIP LOCKED` is useful for queue-style processing where multiple workers pull pending records concurrently without blocking each other."
