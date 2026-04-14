## A23: Oracle MERGE Statement / UPSERT

### What is MERGE?

MERGE is Oracle's **atomic upsert** — insert if not exists, update if exists — all in a single statement. Introduced in Oracle 9i. Also called "upsert" (update or insert).

### Basic Syntax

```sql
MERGE INTO target_table t
USING source_table s
  ON (t.key = s.key)
WHEN MATCHED THEN
  UPDATE SET t.col1 = s.col1, t.col2 = s.col2
WHEN NOT MATCHED THEN
  INSERT (key, col1, col2) VALUES (s.key, s.col1, s.col2);
```

### Real Example — Booking Status Sync

Imagine you receive booking updates from a carrier via EDI and need to sync them to your DB:

```sql
MERGE INTO bookings b
USING (SELECT :bookingRef AS ref,
              :status AS status,
              :vessel AS vessel
       FROM dual) incoming
  ON (b.booking_ref = incoming.ref)
WHEN MATCHED THEN
  UPDATE SET b.status = incoming.status,
             b.vessel = incoming.vessel,
             b.updated_at = SYSDATE
WHEN NOT MATCHED THEN
  INSERT (booking_ref, status, vessel, created_at)
  VALUES (incoming.ref, incoming.status, incoming.vessel, SYSDATE);
```

The `FROM dual` trick creates an inline row for single-record merges.

### Multi-Row Sync from Staging

```sql
MERGE INTO bookings b
USING staging_bookings s
  ON (b.booking_ref = s.booking_ref)
WHEN MATCHED THEN
  UPDATE SET b.status = s.status,
             b.updated_at = SYSDATE
  WHERE s.status != b.status  -- only update if actually different
WHEN NOT MATCHED THEN
  INSERT (booking_ref, status, created_at)
  VALUES (s.booking_ref, s.status, SYSDATE);
```

### DELETE Clause (Oracle extension)

You can delete matching rows too:
```sql
MERGE INTO bookings b
USING staging s
  ON (b.booking_ref = s.booking_ref)
WHEN MATCHED THEN
  UPDATE SET b.status = s.status
  DELETE WHERE s.status = 'CANCELLED';  -- delete after update if cancelled
```

### Oracle MERGE vs Postgres ON CONFLICT

**Postgres:**
```sql
INSERT INTO bookings (booking_ref, status)
VALUES ('B123', 'CONFIRMED')
ON CONFLICT (booking_ref)
DO UPDATE SET status = EXCLUDED.status, updated_at = NOW();
```

**Oracle has nothing like ON CONFLICT** — only MERGE. For simple cases, MERGE is more verbose. For complex cases, MERGE is more powerful.

### Using MERGE from Spring Boot

**Option 1 — Native query:**
```java
@Modifying
@Query(value = """
    MERGE INTO bookings b
    USING (SELECT :ref AS ref, :status AS status FROM dual) s
      ON (b.booking_ref = s.ref)
    WHEN MATCHED THEN
      UPDATE SET b.status = s.status
    WHEN NOT MATCHED THEN
      INSERT (booking_ref, status) VALUES (s.ref, s.status)
    """, nativeQuery = true)
void upsertBooking(@Param("ref") String ref, @Param("status") String status);
```

**Option 2 — JPA `saveOrUpdate` via `save()`:**
If the entity has an `@Id`, `repository.save()` will do an UPDATE if the ID exists, INSERT otherwise. But this requires a round-trip (SELECT to check) — not atomic, not as efficient as MERGE.

### Common Pitfalls

**1. NULL comparison in ON clause**
```sql
ON (t.key = s.key)  -- NULL != NULL in SQL, so NULL keys never match!
```
Use `ON (NVL(t.key, -1) = NVL(s.key, -1))` if keys can be NULL.

**2. Deadlocks with concurrent MERGE**
Two concurrent MERGEs on different rows of the same table can deadlock due to block-level locks. Consider adding `ORDER BY` to the USING subquery or processing in consistent order.

**3. Triggers fire twice**
If the row is matched, UPDATE triggers fire. If not matched, INSERT triggers fire. Both sets of triggers exist on the same table — easy to double-process.

**4. Cannot update the ON columns**
The columns used in the ON clause cannot be updated in the WHEN MATCHED UPDATE clause. Oracle blocks this — it would change what "matched" means.

### Use Cases

| Scenario | Why MERGE |
|----------|-----------|
| Syncing carrier EDI updates to booking table | Don't know if row exists; MERGE handles both |
| Idempotent event processing | Safely re-process events without duplicating |
| Nightly ETL from staging to final table | One statement vs select + insert/update loop |
| Backfilling data from one table to another | Efficient bulk upsert |

### Performance Note

MERGE performs a **single pass** over the source and target. Compared to `SELECT → IF exists UPDATE ELSE INSERT` from application code, MERGE saves network round-trips and locks. For bulk operations it's dramatically faster.

### Interview-Ready Answer

> "MERGE is Oracle's upsert — it atomically inserts or updates based on whether a row matches. Syntax is `MERGE INTO ... USING ... ON (...) WHEN MATCHED UPDATE ... WHEN NOT MATCHED INSERT`. Postgres has `INSERT ... ON CONFLICT` for simple cases, but Oracle only has MERGE.
>
> I use it for syncing external data — like receiving carrier status updates via EDI, I MERGE them into the bookings table. It's atomic, saves a round-trip, and handles both the 'new record' and 'existing record' cases in one statement.
>
> Main pitfall is NULL comparison in the ON clause — NULL doesn't equal NULL in SQL — so I wrap keys in NVL if they can be NULL."
