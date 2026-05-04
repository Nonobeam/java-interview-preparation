# 70. `GROUP BY`, `HAVING`, aggregate functions

## Aggregate functions

Functions that collapse many rows into one value:

| Function | Counts |
|----------|--------|
| `COUNT(*)`        | All rows in the group, including NULLs |
| `COUNT(col)`      | Non-NULL values of `col` |
| `COUNT(DISTINCT col)` | Distinct non-NULL values |
| `SUM(col)`        | Sum (NULLs ignored) |
| `AVG(col)`        | Average (NULLs ignored) |
| `MIN(col)` / `MAX(col)` | Min / max |
| `STRING_AGG(col, ',')` (Postgres) / `LISTAGG` (Oracle) / `GROUP_CONCAT` (MySQL) | Concatenate strings |

## `GROUP BY`

Splits rows into groups by the specified columns; each aggregate then runs **per group**.

```sql
SELECT user_id, COUNT(*) AS order_count, SUM(total) AS revenue
FROM orders
GROUP BY user_id;
```

```
user_id │ order_count │ revenue
────────┼─────────────┼─────────
   1    │      2      │  80.00
   2    │      1      │  20.00
```

### Rule: every non-aggregated column in `SELECT` must appear in `GROUP BY`

```sql
-- Wrong (in strict SQL): name is not aggregated and not in GROUP BY
SELECT u.id, u.name, COUNT(o.id)
FROM users u JOIN orders o ON o.user_id = u.id
GROUP BY u.id;

-- Correct
SELECT u.id, u.name, COUNT(o.id)
FROM users u JOIN orders o ON o.user_id = u.id
GROUP BY u.id, u.name;
```

(MySQL with `ONLY_FULL_GROUP_BY` off is permissive but the result is non-deterministic — don't rely on it.)

## `HAVING` — filter on aggregate results

`WHERE` filters individual rows **before** aggregation. `HAVING` filters **groups** after aggregation.

```sql
SELECT user_id, SUM(total) AS revenue
FROM orders
WHERE status = 'PAID'      -- filter rows BEFORE grouping
GROUP BY user_id
HAVING SUM(total) > 100;   -- filter groups AFTER aggregating
```

You cannot use an aggregate in `WHERE`:

```sql
-- ERROR: aggregate functions are not allowed in WHERE
SELECT user_id FROM orders
WHERE SUM(total) > 100
GROUP BY user_id;
```

## Logical query order

This is the mental model for why `WHERE` runs before `HAVING`:

```
1. FROM      → which tables (and joins)
2. WHERE     → row-level filter
3. GROUP BY  → bucket rows into groups
4. HAVING    → group-level filter
5. SELECT    → compute output columns (aggregates evaluated here)
6. ORDER BY  → sort output
7. LIMIT     → cut to N rows
```

So `WHERE` can't see aggregates because they don't exist yet.

## Common patterns

### Top N per group (use a window function — see Q71)

```sql
SELECT user_id, MAX(total) AS biggest_order
FROM orders
GROUP BY user_id
HAVING COUNT(*) >= 3;       -- only users with ≥ 3 orders
```

### Counting with a condition — `COUNT(... FILTER ...)` or `SUM(CASE)`

```sql
-- PostgreSQL: FILTER clause
SELECT user_id,
       COUNT(*) AS total_orders,
       COUNT(*) FILTER (WHERE status = 'CANCELLED') AS cancelled
FROM orders
GROUP BY user_id;

-- Portable: SUM(CASE)
SELECT user_id,
       COUNT(*) AS total_orders,
       SUM(CASE WHEN status = 'CANCELLED' THEN 1 ELSE 0 END) AS cancelled
FROM orders
GROUP BY user_id;
```

### NULL behavior

- `COUNT(*)` includes NULL rows.
- `COUNT(col)`, `SUM`, `AVG`, `MIN`, `MAX` all **ignore NULLs**.
- A group on a NULL column collects all NULL rows into a single group (NULL = NULL for grouping purposes, even though `NULL = NULL` is `UNKNOWN` in `WHERE`).

## Interview one-liner

> "`GROUP BY` buckets rows; aggregates collapse each bucket. `WHERE` filters rows before grouping, `HAVING` filters groups after. Logical order is FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT, which is why aggregates can appear in HAVING and SELECT but never in WHERE."
