# 43. What are the trade-offs of using indexes?

Indexes are the single biggest lever for query performance — **and** one of the easiest features to misuse. The trade-off is simple to state and painful to forget:

> **An index speeds up some reads at the cost of slowing down every write and consuming storage + memory.**

Here are the concrete trade-offs in both directions.

---

## Wins (what you get)

| Win | How it works |
|---|---|
| **Faster equality lookups** | O(log n) B-tree lookup instead of O(n) full scan. |
| **Faster range scans** | Leaf pages are linked and sorted → walk the range sequentially. |
| **Avoids sorts** | `ORDER BY indexed_col` can skip the sort step entirely. |
| **Fast joins** | Nested-loop joins become cheap when the inner side is indexed on the join key. |
| **Enforces uniqueness** | `UNIQUE` constraints are implemented as unique indexes. |
| **Foreign-key checks** | Indexes on child FK columns prevent locking issues on parent updates. |
| **Index-only scans** | A covering index can answer the query without touching the table at all. |

---

## Costs (what you pay)

### 1. Slower writes
Every `INSERT`, `DELETE`, and indexed-column `UPDATE` must update every affected index. On a 10-index table, a single insert is effectively 11 writes.

### 2. Storage
An index is often **20–60%** of the table's size. Five indexes can easily **double** the on-disk footprint. Storage is cheap, but…

### 3. Memory pressure
Index pages compete with table pages for the **buffer cache / shared buffers**. A big, rarely-used index evicts hot OLTP pages → cache hit rate drops → everything slows down.

### 4. Maintenance cost
- Every `VACUUM` (Postgres), `REBUILD` (Oracle), `OPTIMIZE TABLE` (MySQL) touches indexes.
- Index bloat from heavy UPDATE/DELETE traffic eventually requires a rebuild.
- Backups and replication carry the index weight (Oracle RMAN, Postgres WAL, MySQL binlog).

### 5. Optimizer complexity
More indexes = more plan alternatives. The planner may pick the wrong one if statistics drift, producing **non-deterministic latency** on the same query.

### 6. Build cost
Adding an index to a big live table is hours of I/O + potential write-blocking (see Q42).

### 7. Locking subtleties
Long-running index rebuilds and DDL can stall the application unless you explicitly use `CONCURRENTLY` / `ONLINE`.

---

## Context-dependent trade-offs

### Low-cardinality columns
An index on `gender`, `status`, `is_active` often **hurts** rather than helps — the optimizer won't use it, but writes still pay the tax.
**Remedy:** use a **partial index** (`WHERE status = 'ACTIVE'`) or put the low-cardinality column **after** a selective column in a composite index.

### Heavily updated columns
Indexing `updated_at` means every row update rewrites that index key. If nothing ever filters by `updated_at`, drop the index.

### OLTP vs OLAP
- **OLTP** (orders, transactions, high write rate) → keep indexes **lean**; every write tax is visible.
- **OLAP / reporting** (batch loads, few writes, heavy scans) → indexes are **far less costly**; consider bitmap indexes, columnstores, or materialized views.

### Composite vs multiple single-column
A composite `(a, b, c)` serves queries on `a`, `(a, b)`, `(a, b, c)` — but NOT on `b` alone. Plan the **leading column** around your most common filter. Stacking one composite often beats three single-column indexes.

### Write-heavy queues / event tables
These can end up with **more index bytes than data bytes**. Sometimes the right answer is **no secondary indexes** — just the PK — and serve reads from a downstream read model (CQRS, replicas).

---

## A concrete comparison

```sql
CREATE TABLE events (
    id          BIGSERIAL PRIMARY KEY,
    user_id     BIGINT,
    event_type  VARCHAR(50),
    payload     JSONB,
    created_at  TIMESTAMP
);
```

| Indexing strategy | INSERT /s | Read `WHERE user_id=? AND created_at>?` | Storage |
|---|---|---|---|
| PK only | 50,000 | ~2 s (scan) | 1× |
| + `(user_id)` + `(created_at)` | 22,000 | ~500 ms | 1.6× |
| + `(user_id, created_at)` composite | 28,000 | ~5 ms | 1.3× |
| + `(user_id, created_at)` + covering `INCLUDE(event_type)` | 26,000 | ~2 ms | 1.5× |

One well-chosen composite beats two single-column indexes on both **write throughput** and **read latency**.

---

## Rules of thumb

1. **Measure first** — use `EXPLAIN ANALYZE` to justify every index.
2. **Prefer composite indexes** over many single-column ones, ordered by selectivity / common filter.
3. **Always index FK columns** (especially in Oracle).
4. **Avoid low-cardinality standalone indexes** — use partial indexes.
5. **Consider a covering index** (`INCLUDE`) when a query is hot and reads few extra columns.
6. **Audit unused indexes** quarterly and drop them.
7. **Rebuild / reindex periodically** on heavily churned tables to control bloat.

---

## Interview one-liner

> "Indexes are a classic space-for-time trade: they give you O(log n) lookups, range scans, ordered reads, and uniqueness enforcement, but you pay with slower writes, more storage, buffer-cache pressure, maintenance overhead, and a more complex optimizer. The right discipline is to **add indexes driven by measured query patterns**, prefer composite indexes, avoid low-cardinality standalone indexes, and audit usage so you drop the ones that never pay back the tax."
