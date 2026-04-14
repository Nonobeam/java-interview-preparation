# 42. What makes it hard to add a new index to a production database?

On a toy table, `CREATE INDEX` is instant. On a **production table with hundreds of millions of rows** and **continuous traffic**, adding an index is a **long, locking, I/O-heavy operation** that can take hours and, done naively, can take the application **offline**. That's what makes it "hard".

Four concrete reasons:

---

## 1. Building the index is expensive

To create `idx_orders_customer ON orders(customer_id)`, the engine must:
1. **Read every row** of `orders` — full table scan of potentially billions of rows.
2. **Extract** the indexed columns + row locator.
3. **Sort** them (external merge sort if they don't fit in memory).
4. **Build the B+ tree** bottom-up into new pages.
5. **Write** all those pages to disk, plus WAL/redo log entries.

For a 500 GB table this is **hours of sustained I/O** competing with live OLTP traffic. Buffer cache gets evicted, live query latency spikes, disks saturate, and replication lag balloons.

---

## 2. Locking — the silent killer

### Oracle
```sql
CREATE INDEX idx_orders_customer ON orders(customer_id);
```
Takes an **exclusive DDL lock** on the table → **no concurrent DML** (inserts/updates/deletes block) for the entire build. On a busy table this is a production incident.

Fix:
```sql
CREATE INDEX idx_orders_customer ON orders(customer_id) ONLINE;
```
`ONLINE` lets DML proceed during the build but requires **Oracle Enterprise Edition**, uses more temp space, and is roughly **2–4× slower** than offline.

### PostgreSQL
```sql
CREATE INDEX idx_orders_customer ON orders(customer_id);
```
Takes a **SHARE lock** — allows reads, **blocks writes**. Minutes of write stall on a busy table.

Fix:
```sql
CREATE INDEX CONCURRENTLY idx_orders_customer ON orders(customer_id);
```
`CONCURRENTLY` does **two passes** over the table, uses only a brief lock at start/end. Trade-offs:
- **Cannot run inside a transaction**.
- Can **fail and leave an INVALID index** that must be dropped and retried.
- Roughly **2× the duration** of a normal build.
- Still consumes heavy I/O.

### MySQL (InnoDB)
`ALGORITHM=INPLACE, LOCK=NONE` for most indexes since 5.6, but some combinations still require a full table rebuild.

---

## 3. Resource contention during the build

Even an "online" build competes for:
- **I/O bandwidth** → live queries slow down.
- **Temp space / sort buffers** → can fill up and abort the build.
- **WAL / redo** → replication lag grows (hot standbys fall behind).
- **Buffer cache** → the index scan warms up cold pages, evicting hot OLTP pages.

Teams often run large index builds during **low-traffic maintenance windows** for this reason.

---

## 4. Getting it wrong is painful to undo

- Index takes **hours to build** → rolling back takes almost as long, and a bad plan might already have caused an incident.
- The optimizer can start picking the **new index when it shouldn't** (bad stats, correlated columns) → plans regress.
- If the build **fails midway** in PostgreSQL's CONCURRENTLY path, you're left with an `INVALID` index that still takes write overhead but can't serve reads.
- Adding an index on a **replicated table** means every replica rebuilds it too — extra lag on every standby.

---

## Practical process I follow for production index additions

1. **Prove the need** with `EXPLAIN` / `EXPLAIN ANALYZE` on a slow query — not a hunch.
2. **Validate the shape** of the index in staging against **realistic data volume** (a 10 M-row staging dataset won't show the pain that 2 B rows will).
3. **Pick the online variant** (`CREATE INDEX CONCURRENTLY` in Postgres, `ONLINE` in Oracle, `ALGORITHM=INPLACE, LOCK=NONE` in MySQL).
4. **Schedule low-traffic window** if possible; monitor replication lag, disk I/O, buffer cache hit rate during the build.
5. **Have a drop plan ready** (`DROP INDEX CONCURRENTLY …`) if the new index causes plan regression.
6. **Re-run `ANALYZE`** (or Oracle `DBMS_STATS.GATHER_TABLE_STATS`) after creation so the optimizer sees it with fresh statistics.
7. **Verify usage** a week later via `pg_stat_user_indexes` / Oracle `V$OBJECT_USAGE` — if `idx_scan = 0`, drop it.

---

## Mini war-story example

> Adding a single 3-column index on an **orders** table (1.8 B rows, ~400 GB) in Postgres:
> - Tested in staging (~80 M rows) → built in 12 minutes, looked fine.
> - Production `CREATE INDEX CONCURRENTLY` took **6 hours 40 minutes**.
> - Replication lag peaked at **52 minutes** on the hot standby.
> - Disk I/O saturated at 1.2 GB/s for most of the build, causing p99 latency on unrelated queries to **2–3×**.
> - Lesson: stage with real-scale data, do it during a known maintenance window, communicate to downstream consumers, and have a `DROP INDEX CONCURRENTLY` ready.

---

## Interview one-liner

> "Adding an index to a big, live table means scanning every row, sorting, building a B-tree, and writing WAL — typically hours — while competing with live traffic. Without `CREATE INDEX CONCURRENTLY` (Postgres) or `ONLINE` (Oracle) it also takes a lock that blocks writes. On top of that, it bloats replication lag, evicts hot buffer-cache pages, and — if the new index causes plan regressions — takes nearly as long to drop. That's why adding an index in production is a **scheduled, measured, reversible operation**, not a casual DDL."
