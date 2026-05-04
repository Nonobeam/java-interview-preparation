# 44. What happens under the hood when you add an index on a column?

When you run:

```sql
CREATE INDEX idx_users_email ON users(email);
```

…the database engine goes through a sequence of well-defined steps. Understanding this sequence is what makes the answers to Q41–43 click (why indexes cost writes, why they take so long to build, why `CONCURRENTLY` exists).

---

## Step-by-step lifecycle of `CREATE INDEX`

### 1. Acquire a lock on the table
- **Oracle (default):** exclusive DDL lock → **blocks all DML** on the table for the whole build.
- **Postgres (default):** `SHARE` lock → reads OK, **writes block**.
- **Postgres `CONCURRENTLY`:** brief lock at start and end; two full passes in between.
- **MySQL/InnoDB (`ALGORITHM=INPLACE, LOCK=NONE`):** minimal locking, online.

Without this step the tree would see mid-flight inserts and end up inconsistent.

### 2. Allocate index metadata
The engine creates a new entry in its catalog (`pg_class`, `dba_indexes`, `information_schema.statistics`) describing:
- index name, owner, table
- indexed column(s), data type, collation
- index type (B-tree, hash, GIN…), storage parameters, tablespace

At this point the index **exists logically** but is empty.

### 3. Full scan of the table
The engine reads **every page** of `users` (heap in Postgres, data segment in Oracle). For each live row it extracts:
- the indexed value(s) — e.g. `email`
- the **row locator** — `CTID` in Postgres, `ROWID` in Oracle, the PK in InnoDB secondary indexes

Cost: O(table size). On a 500 GB table this alone is gigabytes of I/O.

### 4. External sort
Entries are sorted by key. If they fit in `maintenance_work_mem` (Postgres) / `SORT_AREA_SIZE` (Oracle) the sort is in-memory; otherwise it spills to **temp files** (`base/pgsql_tmp/`, Oracle TEMP tablespace) and does an external merge sort.

Cost: O(n log n), plus disk writes for temp. This step is often the I/O peak of the whole build.

### 5. Bulk-load the B+ tree bottom-up
Sorted entries are packed into **leaf pages** (each ~fill-factor full — typically 90% for B-trees), then internal nodes are built on top from leaf boundaries, recursively up to a single root. Pages are written sequentially — much faster than inserting one at a time.

The result: a height-3 or height-4 tree even for billions of rows, because each internal node fans out 100–500 children.

### 6. Persist durably
All new index pages are written to the data files and recorded in the **WAL** (Postgres) / **redo log** (Oracle) so the index survives a crash. Replication streams these changes to replicas, which rebuild the same index locally.

### 7. Second pass (only for `CONCURRENTLY` in Postgres)
While step 3–6 ran, writes kept happening to the table. Pass 2 scans the heap again, picks up rows changed during pass 1, and inserts their index entries. Then the planner starts using the index.

### 8. Update the catalog to mark the index "ready"
Now the optimizer can see it and consider it in plans.

### 9. Update statistics
The engine runs `ANALYZE` (explicitly or automatically) to collect:
- number of distinct values
- most common values + frequencies
- histogram of values
- correlation with physical order

Without this, the optimizer may **ignore the new index** because it has no stats to estimate selectivity.

---

## What happens to **subsequent writes** once the index exists

This is where the write cost comes from.

### INSERT
```
INSERT INTO users (id, email, name) VALUES (42, 'x@y.com', 'Alice');
```
1. Insert the row into the heap/table → get a new CTID/ROWID.
2. For **each index** on the table: navigate the B-tree to the correct leaf, insert `(key, locator)`. If the leaf is full, **split** it (propagating up potentially to the root).
3. Write WAL/redo entries for the heap insert + each index insert.

### UPDATE
- If the updated columns are **not** indexed → only the heap is touched; indexes untouched.
- If an indexed column changes → the engine logically **deletes the old index entry and inserts the new one**. In Postgres, unless HOT (Heap-Only Tuple) applies, *every* index gets a new entry pointing to the new row version.
- Oracle updates in place if the column stays within the original block; otherwise it's delete+insert in the index.

### DELETE
- Mark the heap row dead.
- In Postgres, index entries pointing at dead tuples are cleaned up later by `VACUUM`.
- In Oracle/InnoDB, index entries are removed as part of the transaction.

### Commit
- All WAL/redo entries flushed to disk (or according to `synchronous_commit` / `COMMIT_WAIT` settings).
- Replicas receive the index changes and apply them locally.

---

## What the optimizer does on a query once the index exists

```sql
SELECT * FROM users WHERE email = 'x@y.com';
```

1. Parse & bind.
2. Planner evaluates candidate plans:
   - **Seq Scan** (read every heap page).
   - **Index Scan** on `idx_users_email` (B-tree lookup → fetch heap row via CTID/ROWID).
   - **Index-Only Scan** if all needed columns are in the index (and visibility map allows).
3. Cost model uses statistics:
   - `n_distinct(email)` → ~1 (unique).
   - Estimated rows returned: 1.
   - Index scan cost ≈ height of tree × random-page cost + 1 heap fetch.
   - Seq scan cost ≈ (table pages) × seq-page cost.
4. Index scan wins by orders of magnitude → plan chosen.
5. Executor walks the B-tree, gets the locator, fetches the heap row, returns it.

---

## Physical side-effects you can observe

```sql
-- Postgres: size of the new index
SELECT pg_size_pretty(pg_relation_size('idx_users_email'));

-- Was the index used?
SELECT idx_scan, idx_tup_read, idx_tup_fetch
FROM pg_stat_user_indexes
WHERE indexrelname = 'idx_users_email';

-- Oracle: structure of the built index
SELECT index_name, blevel, leaf_blocks, distinct_keys, clustering_factor
FROM user_indexes
WHERE index_name = 'IDX_USERS_EMAIL';
```

- `blevel` — height of the B-tree (typically 2–4).
- `leaf_blocks` — number of leaf pages.
- `clustering_factor` — how well the index order matches the physical row order; close to the block count = excellent (great for range scans), close to the row count = poor.

---

## Tiny ASCII picture of the result

```
BEFORE: users table = heap pages only
  [page 1: rows…] [page 2: rows…] … [page N: rows…]
  Find email = 'x@y.com' → scan all N pages.

AFTER CREATE INDEX idx_users_email:
                       [root]
                      /  |  \
                  [int][int][int]
                  /  \      \
              [leaf][leaf][leaf][leaf][leaf]   ← sorted email → (CTID)
                                 |
                                 └──→ heap page 734, offset 12
  Find email = 'x@y.com' → 3–4 page reads, done.
```

---

## Interview one-liner

> "`CREATE INDEX` locks the table (or uses an online variant), full-scans every row, extracts `(indexed_values, row_locator)`, sorts them — spilling to disk if needed — bulk-loads a B+ tree bottom-up, writes WAL/redo, updates the catalog, and runs `ANALYZE` so the optimizer picks it up. After that, every INSERT/DELETE and every UPDATE of the indexed column walks and mutates this tree, which is exactly where the write cost comes from."
