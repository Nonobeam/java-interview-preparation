# 45. How do you use `EXPLAIN` and `ANALYZE` in SQL?

`EXPLAIN` and `ANALYZE` are the **two primary tools for understanding query performance**. They answer two different questions:

- **`EXPLAIN`** → *"What plan is the optimizer **going to** use for this query?"* (estimates, no execution)
- **`EXPLAIN ANALYZE`** / Oracle's `DBMS_XPLAN` + `GATHER_PLAN_STATISTICS`  → *"What **actually happened** when the query ran?"* (real execution, real row counts, real time)

Plus a third statement:

- **`ANALYZE <table>`** (Postgres) / **`DBMS_STATS.GATHER_TABLE_STATS`** (Oracle) → *"Recompute statistics so the optimizer can plan better."*

These three are often confused because "ANALYZE" means different things in different engines. The distinction below is essential.

---

## Part 1 — `EXPLAIN` (the plan estimate)

### PostgreSQL

```sql
EXPLAIN
SELECT * FROM orders
WHERE customer_id = 42 AND status = 'PAID';
```

Output:
```
Index Scan using idx_orders_cust_stat on orders  (cost=0.42..8.44 rows=2 width=78)
  Index Cond: ((customer_id = 42) AND (status = 'PAID'::text))
```

Key things to read:
- **Node type** — `Index Scan`, `Seq Scan`, `Bitmap Heap Scan`, `Hash Join`, `Nested Loop`, `Sort`, `Aggregate`…
- **`cost=startup..total`** — planner's abstract cost units. Only compare costs **within the same plan**; they're not milliseconds.
- **`rows=…`** — estimated rows the node will emit.
- **`width=…`** — average row width in bytes.
- **`Index Cond` / `Filter`** — Index Cond uses the index; Filter is applied after reading (more expensive).

### Oracle

```sql
EXPLAIN PLAN FOR
SELECT * FROM orders
WHERE customer_id = 42 AND status = 'PAID';

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY());
```

Key columns: `Operation`, `Name`, `Rows`, `Bytes`, `Cost (%CPU)`, `Time`, `Predicate Information`.

### MySQL

```sql
EXPLAIN SELECT * FROM orders
WHERE customer_id = 42 AND status = 'PAID';
```

Key columns: `type` (`const`, `ref`, `range`, `ALL`…), `possible_keys`, `key`, `rows`, `Extra`. `type=ALL` means full table scan.

---

## Part 2 — `EXPLAIN ANALYZE` (actual execution)

This is the one you care about most when diagnosing a slow query.

### PostgreSQL

```sql
EXPLAIN (ANALYZE, BUFFERS, VERBOSE)
SELECT * FROM orders
WHERE customer_id = 42 AND status = 'PAID';
```

Output:
```
Index Scan using idx_orders_cust_stat on orders
    (cost=0.42..8.44 rows=2 width=78)
    (actual time=0.032..0.041 rows=3 loops=1)
  Index Cond: ((customer_id = 42) AND (status = 'PAID'::text))
  Buffers: shared hit=5
Planning Time: 0.234 ms
Execution Time: 0.067 ms
```

Reading tips:
- **`actual time=<startup>..<total>`** — milliseconds **per loop**.
- **`actual rows` vs estimated `rows`** — if they differ by **>10×**, stats are stale or the planner is mis-estimating. This is the #1 cause of bad plans.
- **`loops=N`** — node executed N times (inner side of a nested loop). Total time = `actual total × loops`.
- **`Buffers: shared hit=X read=Y`** — `hit` = served from cache, `read` = fetched from disk. A query with lots of `read` is I/O-bound.
- **`Rows Removed by Filter`** — rows fetched and then thrown away. Big numbers here mean a missing index or a poor predicate.

> ⚠️ `EXPLAIN ANALYZE` **actually executes** the query — `INSERT`, `UPDATE`, `DELETE` will run for real. Wrap them in a transaction you `ROLLBACK` if you need to profile DML without side effects.

### Oracle (real execution statistics)

```sql
-- 1. Mark the session so execution stats are captured
ALTER SESSION SET STATISTICS_LEVEL = ALL;

-- or hint the query:
SELECT /*+ GATHER_PLAN_STATISTICS */ *
FROM orders WHERE customer_id = 42 AND status = 'PAID';

-- 2. After running the query, pull the plan + real stats
SELECT * FROM TABLE(
  DBMS_XPLAN.DISPLAY_CURSOR(FORMAT => 'ALLSTATS LAST')
);
```

Key columns in the output:
- **`Starts`** — how many times the operation executed.
- **`E-Rows`** — estimated rows (the planner's guess).
- **`A-Rows`** — actual rows produced.
- **`A-Time`** — actual time.
- **`Buffers`** — logical reads.
- **`Reads`** — physical reads (disk).

Same rule as Postgres: big divergence between `E-Rows` and `A-Rows` = stats problem or bad cardinality estimate.

### MySQL (8.0+)

```sql
EXPLAIN ANALYZE
SELECT * FROM orders WHERE customer_id = 42;
```

Produces a tree of actual timings per node — similar to Postgres.

---

## Part 3 — `ANALYZE <table>` (statistics refresh, Postgres) vs `EXPLAIN ANALYZE` (very different!)

In **PostgreSQL** there are **two unrelated meanings** of "ANALYZE":

| Statement | What it does |
|---|---|
| `ANALYZE users;` | Recomputes **planner statistics** for the table. No query runs. |
| `EXPLAIN ANALYZE SELECT …;` | Runs the query **and** shows the plan with real timings. |

Stale statistics are one of the most common causes of bad plans. If a query looks reasonable in `EXPLAIN` but runs slow, run:

```sql
ANALYZE orders;              -- PostgreSQL
```

or in Oracle:

```sql
BEGIN
  DBMS_STATS.GATHER_TABLE_STATS(USER, 'ORDERS');
END;
/
```

Autovacuum / auto-stats runs these on a schedule, but after a **large bulk load, truncate-and-reload, or index creation**, you should run it manually before relying on plans.

---

## A realistic workflow for diagnosing a slow query

```sql
-- 1. Confirm it's slow in the real workload.
\timing
SELECT … (slow query) …;

-- 2. Look at the current plan and actual behavior.
EXPLAIN (ANALYZE, BUFFERS) SELECT … ;

-- 3. Compare estimated vs actual rows.
--    If off by >10×, refresh statistics and replan:
ANALYZE orders;
EXPLAIN (ANALYZE, BUFFERS) SELECT … ;

-- 4. Still slow? Look for:
--    - Seq Scan on a big table        → consider an index
--    - "Rows Removed by Filter: N"    → missing/wrong index
--    - Nested Loop with loops=large   → maybe better as Hash Join
--    - Sort: Sort Method: external    → raise work_mem or add index for ORDER BY
--    - Buffers: read >> hit           → cold cache or cache pressure

-- 5. Propose an index (or query rewrite), test in staging:
CREATE INDEX CONCURRENTLY idx_orders_cust_stat
  ON orders(customer_id, status);

-- 6. Re-run EXPLAIN ANALYZE, confirm the plan changed and timing dropped.
```

---

## Red flags to spot quickly

| Sign in the plan | What it usually means |
|---|---|
| `Seq Scan` on a big table with a selective predicate | Missing index. |
| Estimated `rows=1` but actual `rows=1_000_000` | Stale statistics — run `ANALYZE`. |
| `Rows Removed by Filter:` is very high | Index filters the wrong columns. |
| `Sort Method: external merge Disk` | Sort spilled to disk — need more `work_mem` or an ordered index. |
| `Buffers: read=many, hit=few` | Cold cache — first-run cost; re-run to confirm. |
| Nested Loop with `loops=100000` and inner Seq Scan | Inner side needs an index. |
| Oracle `A-Rows` >> `E-Rows` on a join | Optimizer under-estimating selectivity → join order wrong. |

---

## Interview one-liner

> "`EXPLAIN` shows the plan the optimizer intends to use — node types, estimated rows, cost. `EXPLAIN ANALYZE` actually runs the query and reports real row counts, per-node time, loops, and buffer hits/reads. Postgres also has a separate `ANALYZE <table>` that refreshes statistics — which is totally different. My debugging loop is: `EXPLAIN ANALYZE` the slow query, compare estimated vs actual rows, refresh stats if they're off by >10×, then look for seq scans on big tables, nested loops with huge `loops`, rows removed by filter, or sorts spilling to disk — and fix with an index, a rewrite, or better statistics."
