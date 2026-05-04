# 71. Subqueries vs CTEs (`WITH`)

## Subquery

A query nested inside another. Three flavors:

### 1. Scalar subquery — returns one value

```sql
SELECT name,
       (SELECT COUNT(*) FROM orders o WHERE o.user_id = u.id) AS order_count
FROM users u;
```

### 2. Subquery in `FROM` (derived table) — returns a table

```sql
SELECT t.user_id, t.revenue
FROM (
    SELECT user_id, SUM(total) AS revenue
    FROM orders
    GROUP BY user_id
) t
WHERE t.revenue > 100;
```

### 3. Correlated subquery — references the outer query

```sql
SELECT u.id, u.name
FROM users u
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.user_id = u.id AND o.total > 1000
);
```

The inner query re-runs **logically once per outer row** — though optimizers often rewrite to a join.

## CTE — Common Table Expression (`WITH` clause)

Names a query you can reference later, like a temporary view scoped to the statement.

```sql
WITH high_value_orders AS (
    SELECT user_id, total
    FROM orders
    WHERE total > 1000
),
top_users AS (
    SELECT user_id, COUNT(*) AS big_count
    FROM high_value_orders
    GROUP BY user_id
    HAVING COUNT(*) >= 3
)
SELECT u.name, t.big_count
FROM top_users t
JOIN users u ON u.id = t.user_id
ORDER BY t.big_count DESC;
```

You can chain multiple CTEs and reuse each one inside the final SELECT.

## CTE vs subquery — practical differences

| | Subquery | CTE |
|---|---|---|
| Readability | Nested, harder to read past 2 levels | Top-down, named, reads like a pipeline |
| Reuse | Must repeat the same subquery if you need it twice | Reference the CTE name multiple times |
| Performance | Often inlined and optimized into the main plan | Postgres ≥12 inlines by default; older versions materialized always (could hurt plans) |
| Recursion | Not possible | Recursive CTEs supported (`WITH RECURSIVE`) |

## Recursive CTE — the killer feature

For hierarchies, graphs, sequences. Find an org chart from a manager:

```sql
WITH RECURSIVE subordinates AS (
    -- anchor
    SELECT id, name, manager_id, 1 AS level
    FROM employees
    WHERE id = :rootId

    UNION ALL

    -- recursive step
    SELECT e.id, e.name, e.manager_id, s.level + 1
    FROM employees e
    JOIN subordinates s ON e.manager_id = s.id
)
SELECT * FROM subordinates ORDER BY level, name;
```

Subqueries cannot do this.

## When to choose

**Use a CTE when:**
- The query has multiple logical steps and a CTE-per-step makes intent clear.
- You reference the same intermediate result multiple times in the final query.
- You need recursion.

**Use a subquery when:**
- It's a one-liner scalar value.
- You're using `EXISTS` / `IN` to test membership.
- Inlining keeps the query short and the plan obvious.

## A comparable rewrite

Subquery version:

```sql
SELECT user_id
FROM (
    SELECT user_id, SUM(total) AS revenue
    FROM orders
    GROUP BY user_id
) t
WHERE t.revenue > 1000;
```

CTE version:

```sql
WITH revenue_per_user AS (
    SELECT user_id, SUM(total) AS revenue
    FROM orders
    GROUP BY user_id
)
SELECT user_id FROM revenue_per_user WHERE revenue > 1000;
```

Same plan in modern Postgres; the CTE version is easier to extend (add a step, reuse the CTE).

## `EXISTS` vs `IN` (related subquery question)

```sql
-- EXISTS: short-circuits; great with correlated subqueries
SELECT * FROM users u
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);

-- IN: works with a list or non-correlated subquery
SELECT * FROM users
WHERE id IN (SELECT user_id FROM orders WHERE total > 100);
```

`EXISTS` is often faster on large tables because it stops at the first match. `IN` with NULLs has trap behavior — `id NOT IN (NULL, 1)` returns no rows — so prefer `NOT EXISTS` for negation.

## Interview one-liner

> "Subqueries are nested SELECTs — fine for scalar values, EXISTS/IN, or one-off derived tables. CTEs (`WITH`) name and chain intermediate results, which makes multi-step queries readable and reusable, and they're the only way to do recursion. In modern Postgres the optimizer inlines them, so the choice is mostly about clarity, not performance."
