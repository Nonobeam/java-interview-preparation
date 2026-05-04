# 72. Database normalization: 1NF, 2NF, 3NF

## Why normalize

Eliminate **redundancy** and **update anomalies** — situations where the same fact lives in multiple rows and a partial update creates inconsistency.

## 1NF — atomic columns, no repeating groups

Each cell holds **one value** of an atomic type. No comma-separated lists, no JSON-as-table, no `phone1`/`phone2`/`phone3` columns.

**Bad (violates 1NF):**

| user_id | name  | phones                  |
|---------|-------|-------------------------|
| 1       | Alice | "555-1, 555-2"          |

**1NF:**

`users(id, name)` and `phones(user_id, phone)`.

| user_id | phone  |
|---------|--------|
| 1       | 555-1  |
| 1       | 555-2  |

> JSON columns relax this rule deliberately in modern DBs — used wisely they're fine; used as a way to dodge proper modeling they bring back all the original problems.

## 2NF — no partial dependency on a composite key

In force only when the primary key is **composite**. Every non-key column must depend on the **whole** key, not part of it.

**Bad:** `order_items(order_id, product_id, product_name, quantity)`
- PK is `(order_id, product_id)`.
- `product_name` depends only on `product_id` — it's repeated for every order line of that product. Renaming the product means updating every row.

**2NF:** split.
- `order_items(order_id, product_id, quantity)`
- `products(product_id, product_name, ...)`

If your PK is a single column (e.g. `id`), 2NF is automatically satisfied — there's no "part" of the key to partially depend on.

## 3NF — no transitive dependency on the key

Non-key columns depend on the key directly, **not** on another non-key column.

**Bad:** `employees(id, name, dept_id, dept_name, dept_budget)`
- `id → dept_id → dept_name`. `dept_name` is a fact about `dept_id`, not about the employee.
- Update anomaly: rename a dept → update every employee row.

**3NF:** split.
- `employees(id, name, dept_id)`
- `departments(dept_id, dept_name, dept_budget)`

## BCNF — stricter 3NF

Boyce–Codd: every determinant must be a candidate key. Edge cases where 3NF passes but BCNF fails are rare in practice; if the schema feels normalized, it's almost always BCNF.

## What normalization buys you

- **No update anomalies** — change a fact in one place.
- **Smaller storage** — no redundant copies.
- **Cleaner constraints** — FKs make invalid state impossible.

## What it costs

- More joins to assemble a "view" of an entity.
- More tables to maintain in migrations.
- Read-heavy workloads can suffer if joins get expensive.

## When to denormalize intentionally

You denormalize **for performance** after measuring, not preemptively.

Patterns:

1. **Cached aggregates** — store `users.order_count` updated on insert/delete instead of `COUNT(*)` every read. Tradeoff: must keep it in sync.
2. **Materialized views** — Postgres `MATERIALIZED VIEW`, refreshed periodically; the database manages the cache.
3. **Read-side projections (CQRS)** — separate denormalized read models in NoSQL/Elasticsearch fed by the normalized OLTP database via change events.
4. **JSON columns for "schemaless tail"** — when a small set of attributes varies per record (product specs, audit metadata) and you almost never query into them. Postgres `jsonb` with GIN indexes handles this well.
5. **Wide reporting tables / star schema** — analytical workloads (data warehouse) deliberately denormalize into facts + dimensions because queries are read-only and joins on huge tables are expensive.

## Decision rule of thumb

> Normalize until it hurts; denormalize until it works — and only after you have data showing it hurts.

## Interview one-liner

> "1NF: atomic columns, no repeating groups. 2NF: no non-key column depends on only part of a composite key. 3NF: no non-key column depends on another non-key column. Normalize for correctness; denormalize selectively (cached counters, materialized views, CQRS read models, JSON columns) when reads measurably hurt."
