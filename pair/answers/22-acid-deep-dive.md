## A26: ACID Deep Dive

ACID defines what a "transaction" actually guarantees. These are not abstract concepts — each has concrete implementation mechanisms in Oracle and other databases.

---

## A — Atomicity

**Guarantee:** All operations in a transaction succeed together, or none do. "All or nothing."

### Example
```sql
BEGIN
  UPDATE account SET balance = balance - 100 WHERE id = 1;  -- debit
  UPDATE account SET balance = balance + 100 WHERE id = 2;  -- credit
COMMIT;
```
If the credit fails, the debit must be undone. The DB cannot leave account 1 debited without account 2 credited.

### How Oracle Implements It

**UNDO tablespace (rollback segments):**
- Before every change, Oracle writes the OLD value to the UNDO tablespace
- If the transaction rolls back, Oracle reads UNDO and restores old values
- If the transaction commits, UNDO is kept briefly for other TXs' read consistency, then reclaimed

```
Transaction modifies row:
  1. Read current row
  2. Write old value → UNDO tablespace
  3. Write new value → data file (via buffer cache)
  4. Write change record → REDO log
```

**Rollback flow:**
```
User issues ROLLBACK
  → Oracle reads UNDO entries for this TX (in reverse order)
  → Applies old values back to data blocks
  → Releases locks
  → Transaction gone as if never happened
```

### What Can Break Atomicity

- **App-side bugs** — catching exceptions and continuing instead of rolling back
- **Wrong `@Transactional` propagation** — e.g., REQUIRES_NEW inner TX commits independently
- **Non-transactional operations mixed in** — calling external APIs, sending messages, file writes (no way to undo these)
- **`@Transactional` on private methods** — Spring AOP proxy can't intercept, so transaction doesn't actually apply

### Spring Connection
```java
@Transactional
public void transfer(Long from, Long to, BigDecimal amount) {
    accountRepo.debit(from, amount);
    accountRepo.credit(to, amount);  // if this throws, debit rolls back
}
```
Spring wraps the method in a proxy that calls `commit()` on success, `rollback()` on exception.

**Gotcha:** By default Spring only rolls back on `RuntimeException` and `Error`. Checked exceptions **commit** the TX. Use `@Transactional(rollbackFor = Exception.class)` to change this.

---

## C — Consistency

**Guarantee:** The database moves from one valid state to another. All declared rules (constraints, triggers, referential integrity) remain satisfied after the transaction.

### This Is the Most Misunderstood Property

Consistency is **not** about concurrency (that's Isolation). It's about **data validity** according to rules YOU defined.

Example rules:
- Foreign key: `booking.shipper_id` must reference a valid `shipper.id`
- Unique constraint: `booking_ref` must be unique
- Check constraint: `balance >= 0`
- Custom trigger logic: `audit_log` row must exist for every `booking` row

### How Oracle Enforces It

- **Constraints** — Oracle rejects any DML that violates declared constraints (FK, PK, CHECK, UNIQUE, NOT NULL)
- **Triggers** — execute custom validation/business logic on DML
- **Deferred constraints** — checked at COMMIT time instead of DML time:
  ```sql
  ALTER TABLE booking ADD CONSTRAINT fk_shipper
    FOREIGN KEY (shipper_id) REFERENCES shipper(id)
    DEFERRABLE INITIALLY DEFERRED;
  ```
  Useful for circular FK or batch operations.

### The Key Insight

The DB cannot guarantee your business logic is correct. It only enforces the rules you declared. If your app does:
```sql
UPDATE account SET balance = balance - 100;  -- forgot the WHERE!
```
The DB sees no constraint violation — but your data is wrong. That's an app bug, not a DB failure.

### What Can Break Consistency

- **Missing constraints** — app relies on code to enforce rules that should be in DB
- **Bypassing constraints with native queries** — `@Query(nativeQuery = true)` with `DELETE` can bypass JPA cascade logic
- **Disabled constraints** (`ALTER TABLE ... DISABLE CONSTRAINT`)
- **Triggers that depend on wrong assumptions** — e.g., trigger that updates aggregate table, but misses some paths

### Interview Tip
If asked "what is C in ACID?" — don't just say "consistency." Say:
> "It means the transaction leaves the DB in a state that satisfies all declared constraints and rules. It's different from 'eventual consistency' in distributed systems. In Oracle, this is enforced by constraints, triggers, and referential integrity — the DB will reject a TX that violates them."

---

## I — Isolation

**Guarantee:** Concurrent transactions don't see each other's partial work. Each TX runs as if it were alone (at the highest isolation level).

This is the property with the most trade-offs. Higher isolation = less concurrency.

See Q22 for the full detail. Quick recap:

| Level | Dirty | Non-Repeatable | Phantom |
|-------|:----:|:----:|:----:|
| READ_UNCOMMITTED | ❌ | ❌ | ❌ |
| READ_COMMITTED | ✓ | ❌ | ❌ |
| REPEATABLE_READ | ✓ | ✓ | ❌ |
| SERIALIZABLE | ✓ | ✓ | ✓ |

### How Oracle Implements Isolation (MVCC)

Oracle uses **Multi-Version Concurrency Control**:
- Every row has a **SCN (System Change Number)** indicating when it was last modified
- Every SELECT runs against a **consistent snapshot** — it sees the data as it was at a specific SCN
- Writers don't block readers; readers don't block writers
- Readers see old versions via UNDO (the same UNDO used for rollback)

**The magic:** When TX A does `SELECT balance FROM account WHERE id = 1` and TX B is concurrently updating, Oracle serves A the old version from UNDO — no lock wait, no dirty read.

### What Can Break Isolation

- **Using wrong isolation level** — some apps require REPEATABLE_READ, some only need READ_COMMITTED
- **Long-running transactions** — hold locks/UNDO longer, can cause `ORA-01555 snapshot too old`
- **Application-level joins** — reading from multiple tables in separate queries without a single snapshot
- **Read-modify-write races** without explicit locking

---

## D — Durability

**Guarantee:** Once a transaction commits, its changes survive crashes, power loss, and OS restarts. Forever.

### How Oracle Implements It — Write-Ahead Logging (WAL)

**The sequence on COMMIT:**
```
1. Changes are made in memory (buffer cache) — fast but volatile
2. A record of the change is written to the REDO log buffer — in memory
3. On COMMIT: Oracle forces the REDO log buffer to disk via LGWR (Log Writer)
4. LGWR uses fsync — OS is told to flush to physical disk
5. Only when fsync returns does Oracle tell the client "commit successful"
6. The actual data block write to data files happens LATER, asynchronously (DBWR process)
```

**The key insight:** COMMIT does NOT wait for data files to be written. It only waits for the **REDO log** to be flushed. That's enough because after a crash:
- Oracle reads the REDO log
- Replays all committed but unwritten changes
- The DB ends up consistent

**Crash recovery:**
```
Crash happens
  → DB restarts
  → Reads REDO log from last checkpoint
  → Re-applies committed changes that hadn't flushed to data files
  → Rolls back uncommitted changes (using UNDO)
  → DB is open
```

### What Can Break Durability

- **Disabling fsync** — huge performance gain but a power loss = data loss
- **Storage lying about fsync** — some cheap disks ack writes without actually persisting
- **`NOLOGGING` operations** — certain bulk operations skip redo for speed; not crash-safe
- **Replication lag** — primary DB crashes after commit but before replicating to standby → potential loss
- **`COMMIT WRITE NOWAIT`** (Oracle 10g+) — returns without waiting for LGWR. Faster but durability compromised.

### The CAP Angle

In distributed systems, Durability gets tricky:
- Durable on primary, but not yet replicated → failover loses data
- Synchronous replication = strong durability but slower commits
- This is why regulated industries (finance) use synchronous multi-region replication

---

## ACID in Spring Boot

Spring `@Transactional` provides ACID semantics for the duration of the annotated method:

```java
@Transactional(
    isolation = Isolation.SERIALIZABLE,        // I — pick isolation level
    propagation = Propagation.REQUIRED,        // how to handle nested TXs
    rollbackFor = Exception.class,             // A — when to rollback
    timeout = 30                                // safety: max duration
)
public void bookContainer(BookingRequest req) {
    // everything here is one atomic, consistent, isolated, durable TX
    capacityRepo.reserve(req.getVoyageId());
    bookingRepo.save(new Booking(req));
    ledgerRepo.logDebit(req.getAccountId(), req.getFee());
}
```

- **A** — if any exception bubbles up, everything rolls back
- **C** — constraints on tables still enforced
- **I** — controlled by `isolation` param
- **D** — guaranteed by DB's WAL/redo log after COMMIT

### Common Spring ACID Mistakes

**1. `@Transactional` on private method** — ignored. Spring proxies only intercept public methods.

**2. Self-invocation** — calling `this.methodB()` from within `methodA()` bypasses the proxy:
```java
@Service
public class BookingService {
    public void methodA() {
        this.methodB();  // proxy NOT applied, no TX started for B
    }

    @Transactional
    public void methodB() { ... }
}
```
Fix: inject self, or restructure.

**3. Catching exceptions and not rethrowing** — Spring rolls back only when an exception propagates out of the method:
```java
@Transactional
public void book(...) {
    try {
        repo.save(...);
    } catch (Exception e) {
        log.error("oops");  // TX commits normally — exception swallowed!
    }
}
```

**4. Using checked exceptions and expecting rollback** — Spring rolls back on RuntimeException by default. Checked exceptions commit the TX unless you specify `rollbackFor`.

---

## Summary Table

| Property | Guarantee | Oracle Mechanism | Can Break If... |
|----------|-----------|------------------|-----------------|
| **Atomicity** | All or nothing | UNDO tablespace, rollback | Bad exception handling, non-TX side effects |
| **Consistency** | Valid state → valid state | Constraints, triggers, FK | Missing constraints, bypassing with raw SQL |
| **Isolation** | Concurrent TXs don't interfere | MVCC via SCN + UNDO | Wrong isolation level, long TX |
| **Durability** | Committed data survives crashes | Write-ahead log (redo), LGWR, fsync | Disabled fsync, NOLOGGING, async replication |

---

## Interview-Ready Answer

> "ACID defines what a transaction guarantees. Atomicity means all-or-nothing — in Oracle this is done via UNDO tablespace, which stores old values so rollback can restore them. Consistency means declared rules — constraints, FKs, triggers — still hold after commit; it's different from 'eventual consistency' in distributed systems. Isolation controls what concurrent transactions see from each other; Oracle implements it via MVCC snapshots so readers don't block writers. Durability means committed data survives crashes — Oracle uses a write-ahead log written via LGWR with fsync on commit; the actual data file write can happen later because crash recovery replays the redo log.
>
> In Spring, `@Transactional` gives you all four: rollback on exception (A), constraint enforcement (C), configurable isolation (I), and the DB's durability guarantee (D). The common mistakes are catching exceptions silently (breaks A), not specifying `rollbackFor` for checked exceptions, and calling `@Transactional` methods via `this.` which bypasses the Spring proxy."
