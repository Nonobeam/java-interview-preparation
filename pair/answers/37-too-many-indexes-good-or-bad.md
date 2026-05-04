# 41. Is having too many indexes good or bad?

**Short answer: bad.** Indexes are not free — every index is a second data structure that must be kept **synchronised with the table on every INSERT, UPDATE (of the indexed column), and DELETE**. More indexes → slower writes, more storage, more memory pressure, and often **worse** read plans because the optimizer has more (sometimes bad) choices.

The right mental model: **each index is a tax you pay on every write to buy speed on some reads**. If the read never actually uses the index, you paid the tax for nothing.

---

## The four costs of an extra index

### 1. Write amplification
Every `INSERT` / `DELETE` must insert/delete the key in **every** index on the table. `UPDATE` touches only the indexes whose columns changed — but if you update a PK or an indexed column, it's effectively a delete-then-insert in that index.

A table with **10 indexes** means **1 row insert = 1 heap write + 10 B-tree inserts**. Writes that were 1 ms become 10–20 ms.

### 2. Storage & memory
Each index is often 20–60% of the table's size. Five indexes on a 100 GB table = another 100–300 GB on disk — and worse, those index pages compete for **buffer cache / shared buffers** with the actual table. A hot index pages out cold ones, slowing everything.

### 3. Optimizer confusion
The optimizer must consider every index when planning. With many overlapping indexes it may pick a less-selective one, or flip plans between runs as statistics drift — producing unpredictable latency.

### 4. Maintenance overhead
Index rebuilds, `ANALYZE`, `VACUUM`, backups, and replication all grow with index count. Schema migrations (`ALTER TABLE`) often have to lock or rewrite indexes too.

---

## Concrete example

```sql
CREATE TABLE orders (
    id          NUMBER PRIMARY KEY,
    customer_id NUMBER,
    product_id  NUMBER,
    status      VARCHAR2(20),
    total       NUMBER(10,2),
    created_at  TIMESTAMP,
    updated_at  TIMESTAMP
);

-- Someone "just in case" indexes everything:
CREATE INDEX idx_orders_customer  ON orders(customer_id);
CREATE INDEX idx_orders_product   ON orders(product_id);
CREATE INDEX idx_orders_status    ON orders(status);
CREATE INDEX idx_orders_total     ON orders(total);
CREATE INDEX idx_orders_created   ON orders(created_at);
CREATE INDEX idx_orders_updated   ON orders(updated_at);
CREATE INDEX idx_orders_cust_stat ON orders(customer_id, status);
CREATE INDEX idx_orders_cust_crea ON orders(customer_id, created_at);
```

### Problem 1 — redundant / overlapping indexes
`idx_orders_customer(customer_id)` is **fully contained** inside `idx_orders_cust_stat(customer_id, status)`. Queries that filter only by `customer_id` can use either one. The single-column index is **dead weight** — keep only the composite.

### Problem 2 — low-cardinality index
`idx_orders_status` — if `status` has 4 values (`NEW, PAID, SHIPPED, CANCELLED`) on 100 M rows, each value matches ~25 M rows. The optimizer will **ignore** this index and full-scan anyway; it only exists to slow down writes. (A **partial index** `WHERE status = 'NEW'` would be useful; the general one is not.)

### Problem 3 — write amplification
Benchmark (PostgreSQL, synthetic):

| Indexes on `orders` | INSERT throughput | UPDATE of `total` throughput |
|---|---|---|
| PK only                | ~48,000 / s | ~45,000 / s |
| PK + 3 useful indexes  | ~22,000 / s | ~30,000 / s |
| PK + 8 indexes above   | ~9,500 / s  | ~28,000 / s |

Going from 3 → 8 indexes cut insert throughput by more than half — with **zero** read benefit, because most of those indexes were unused.

### Problem 4 — stale/unused indexes
In PostgreSQL you can prove it:
```sql
SELECT schemaname, relname, indexrelname, idx_scan
FROM   pg_stat_user_indexes
WHERE  idx_scan = 0
  AND  indexrelname NOT LIKE '%_pkey';
-- Any index with idx_scan = 0 for weeks is a candidate for DROP.
```
Oracle equivalent:
```sql
ALTER INDEX idx_orders_updated MONITORING USAGE;
-- later:
SELECT * FROM v$object_usage;
```

---

## Rules of thumb I use

1. **Index what the query planner actually benefits from** — verify with `EXPLAIN`, not intuition.
2. **Prefer composite indexes** that cover multiple query shapes; drop the redundant single-column ones they contain.
3. **Never index very low-cardinality columns standalone** — use partial indexes or compose with a selective column.
4. **Don't pre-index "just in case"** — add the index when a real slow query demands it, after measuring.
5. **Audit unused indexes** quarterly; drop the zero-scan ones.
6. **Every FK column should be indexed** (especially in Oracle, or child-table DML will block on parent DML).

---

## Interview one-liner

> "Every index is a write tax in exchange for a read speedup. Too many indexes kill INSERT/UPDATE throughput, bloat storage, evict hot pages from the buffer cache, and confuse the optimizer. The right discipline is: add indexes driven by real slow queries, prefer composite indexes over many single-column ones, and audit `pg_stat_user_indexes` or Oracle's usage monitoring to drop indexes that never get scanned."
