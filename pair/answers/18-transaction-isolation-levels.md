## A22: Transaction Isolation Levels & Concurrency Anomalies

### ACID Refresher
- **Atomicity** — all operations succeed or none do
- **Consistency** — DB moves from one valid state to another
- **Isolation** — concurrent transactions don't see each other's partial work
- **Durability** — once committed, data survives crashes

Isolation is the one we can tune — the others are always on.

### The 3 Concurrency Anomalies

**1. Dirty Read** — TX A reads TX B's uncommitted changes, then B rolls back → A has garbage data
```
TX B: UPDATE account SET balance = 100 WHERE id = 1; (not yet committed)
TX A: SELECT balance FROM account WHERE id = 1; → reads 100
TX B: ROLLBACK;
TX A has read a value that never truly existed
```

**2. Non-Repeatable Read** — TX A reads row X twice. Between reads, TX B updates X and commits. A gets different values.
```
TX A: SELECT balance FROM account WHERE id = 1; → 100
TX B: UPDATE account SET balance = 200 WHERE id = 1; COMMIT;
TX A: SELECT balance FROM account WHERE id = 1; → 200  (different!)
```

**3. Phantom Read** — TX A runs same query twice. Between runs, TX B inserts/deletes rows matching the criteria. A sees new/missing rows.
```
TX A: SELECT COUNT(*) FROM booking WHERE voyage='V1'; → 10
TX B: INSERT INTO booking (voyage='V1', ...); COMMIT;
TX A: SELECT COUNT(*) FROM booking WHERE voyage='V1'; → 11 (phantom!)
```

### The 4 Isolation Levels

| Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|-------|:----------:|:-------------------:|:------------:|
| **READ_UNCOMMITTED** | ❌ possible | ❌ possible | ❌ possible |
| **READ_COMMITTED** | ✓ prevented | ❌ possible | ❌ possible |
| **REPEATABLE_READ** | ✓ prevented | ✓ prevented | ❌ possible |
| **SERIALIZABLE** | ✓ prevented | ✓ prevented | ✓ prevented |

As isolation increases → consistency increases, concurrency (throughput) decreases.

### Oracle Specifics (Important!)

**Oracle's default: READ COMMITTED** (via MVCC snapshots — not locks)

**Oracle does NOT support READ_UNCOMMITTED** — any attempt is silently treated as READ_COMMITTED. Oracle uses multi-version concurrency control (MVCC), so dirty reads are structurally impossible.

**Oracle does NOT support true REPEATABLE_READ** — only `READ_COMMITTED` and `SERIALIZABLE`. If Spring asks for REPEATABLE_READ on Oracle, it throws an exception.

**Oracle's SERIALIZABLE** uses snapshot isolation — it's actually weaker than SQL standard serializable (subject to write-skew anomaly), but prevents the 3 named anomalies.

Oracle also has **READ ONLY** mode — read-only snapshot transaction, no modifications allowed, ensures repeatable reads for the entire transaction.

### Postgres vs Oracle
| Feature | Postgres | Oracle |
|---------|----------|--------|
| READ_UNCOMMITTED | Treated as READ_COMMITTED | Treated as READ_COMMITTED |
| READ_COMMITTED | ✓ (default) | ✓ (default) |
| REPEATABLE_READ | ✓ | ✗ (throws exception) |
| SERIALIZABLE | ✓ (true SSI) | ✓ (snapshot isolation) |

### Spring @Transactional Syntax

```java
@Transactional(isolation = Isolation.SERIALIZABLE)
public void bookContainer(Booking b) {
    // runs at SERIALIZABLE level
}
```

Spring isolation levels:
- `Isolation.DEFAULT` — use the database default (for Oracle = READ_COMMITTED)
- `Isolation.READ_UNCOMMITTED`
- `Isolation.READ_COMMITTED`
- `Isolation.REPEATABLE_READ`
- `Isolation.SERIALIZABLE`

### When to Use Which

| Scenario | Level |
|----------|-------|
| Read-heavy reports, slight staleness OK | `READ_COMMITTED` (default) |
| Financial calculations, balance checks, multiple reads of same data | `REPEATABLE_READ` (Postgres) or `SERIALIZABLE` (Oracle) |
| Complex workflows with multiple queries, must be consistent | `SERIALIZABLE` |
| Dashboard counts, tolerance for inaccuracy | `READ_UNCOMMITTED` if supported |

### Interview-Ready Answer

> "The 4 isolation levels map to the 3 anomalies they prevent. READ_COMMITTED prevents dirty reads, REPEATABLE_READ also prevents non-repeatable reads, and SERIALIZABLE prevents phantoms too.
>
> Oracle's default is READ_COMMITTED implemented via MVCC snapshots — so dirty reads are impossible regardless. Oracle doesn't support true REPEATABLE_READ — only READ_COMMITTED and SERIALIZABLE. Postgres supports all four properly.
>
> In Spring Boot, I use `@Transactional(isolation = Isolation.X)` — default is DEFAULT, which delegates to the DB's default. I only raise to SERIALIZABLE for specific critical paths like financial posting or booking confirmation, because higher isolation means more locking and less throughput."
