# 40. What is a database index?

A **database index** is an auxiliary data structure that stores a **sorted (or hashed) copy of selected column values together with a pointer to the row**, so the engine can find matching rows **without scanning the whole table**.

Think of it exactly like the index at the back of a book: instead of reading every page to find the word "mitochondria", you look it up alphabetically and jump straight to page 287.

---

## The storage reality

A table is stored as **data pages** (typically 8 KB in PostgreSQL, 8 KB default in Oracle). Without an index, finding `WHERE email = 'x@y.com'` means the engine loads **every page** and scans — this is a **full table scan**. Cost grows linearly with the table: 10 M rows → read 10 M rows.

An index keeps a second, smaller structure (usually a **B+ tree**) where keys are sorted. Looking up a key is O(log n) page reads — for 10 M rows, ~4–5 page reads instead of millions.

---

## What's actually inside an index

Take `CREATE INDEX idx_email ON users(email);`:

```
B+ tree (sorted by email):

         [ m..., s... ]                    ← root
        /      |      \
   [a..][d..][h..]  [n..][p..]   [t..][w..][z..]   ← internal
    |    |    |      |    |       |    |    |
  leaves: (email, rowid) pairs, sorted, doubly-linked
```

Each **leaf entry** is `(indexed column(s), row locator)`:
- **Oracle / MySQL InnoDB (non-clustered secondary index)** → leaf holds the **primary key** of the row; the engine then uses the PK to find the actual row in the clustered index.
- **PostgreSQL** → leaf holds a **CTID** (physical page + offset pointing to the heap row).

Leaf pages are linked into a doubly-linked list so **range scans** (`BETWEEN`, `>`, `ORDER BY`) can walk sequentially without going back up the tree.

---

## What an index enables

1. **Equality lookup** — `WHERE email = ?` → O(log n).
2. **Range scan** — `WHERE created_at BETWEEN ? AND ?` → find the start, walk the leaf list.
3. **Sort avoidance** — `ORDER BY created_at` on an indexed column can skip the sort step.
4. **Index-only scan (covering index)** — if every column needed by the query is in the index, the engine never touches the table at all.
5. **Uniqueness enforcement** — a `UNIQUE` constraint is implemented as a unique index.
6. **Foreign-key checks** — joining `orders → customers` is fast because the PK side is indexed.

---

## Types of indexes (quick tour)

| Type | Use case | Example |
|---|---|---|
| **B-tree / B+ tree** | Default. Equality + range + ORDER BY. | `CREATE INDEX ON orders(customer_id);` |
| **Hash** | Equality only, no range. Rare in practice. | Postgres `USING HASH` |
| **Composite** | Multi-column filter/sort. Order matters. | `(customer_id, created_at)` |
| **Unique** | Enforces uniqueness. | `UNIQUE(email)` |
| **Partial / filtered** | Index only some rows. | `WHERE status = 'ACTIVE'` |
| **Covering / INCLUDE** | Extra columns stored in leaf to enable index-only scans. | Postgres `INCLUDE (name, total)` |
| **Bitmap** | Low-cardinality columns in OLAP. | Oracle bitmap index on `gender` |
| **Full-text / GIN / GiST** | Text search, arrays, JSONB. | Postgres `USING GIN (tags)` |
| **Function-based** | Index an expression. | `CREATE INDEX ON users(LOWER(email));` |

---

## Mini worked example

```sql
CREATE TABLE users (
    id    NUMBER PRIMARY KEY,
    email VARCHAR2(200),
    name  VARCHAR2(100)
);
-- 10,000,000 rows

-- Without index:
SELECT * FROM users WHERE email = 'alice@example.com';
-- → FULL TABLE SCAN, reads every page, ~seconds on a cold cache

CREATE INDEX idx_users_email ON users(email);

SELECT * FROM users WHERE email = 'alice@example.com';
-- → INDEX RANGE SCAN on idx_users_email → TABLE ACCESS BY INDEX ROWID
-- → ~4 logical reads, sub-millisecond
```

---

## Clustered vs non-clustered (one-liner)

- **Clustered index** = the table **is** the index; rows are physically stored in key order. There can only be one per table (because rows can only be sorted one way). InnoDB and SQL Server use the PK as the clustered index.
- **Non-clustered (secondary) index** = separate structure pointing back to the row. You can have many.

Oracle is a slight outlier — it stores tables as heaps by default, so "clustered" in the SQL Server sense doesn't apply; instead Oracle has **index-organized tables (IOTs)** as the explicit opt-in.

---

## Interview one-liner

> "An index is a sorted, auxiliary structure — almost always a B+ tree — that stores indexed column values plus a row locator, so lookups go from O(n) full scans to O(log n) page reads. It speeds up reads that match the leading column(s), supports range scans and ordered results, and enforces uniqueness — at the cost of extra storage and extra work on every write."
