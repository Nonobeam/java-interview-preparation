# 69. SQL JOIN types

## Setup tables for examples

```
users                 orders
┌────┬───────┐       ┌─────┬─────────┬────────┐
│ id │ name  │       │ id  │ user_id │ total  │
├────┼───────┤       ├─────┼─────────┼────────┤
│ 1  │ Alice │       │ 100 │ 1       │ 50.00  │
│ 2  │ Bob   │       │ 101 │ 1       │ 30.00  │
│ 3  │ Carol │       │ 102 │ 2       │ 20.00  │
└────┴───────┘       │ 103 │ 9       │ 99.00  │  ← orphan: no user 9
                     └─────┴─────────┴────────┘
```

## INNER JOIN — only matching rows on both sides

```sql
SELECT u.name, o.total
FROM users u
INNER JOIN orders o ON o.user_id = u.id;
```

Result: Alice/50, Alice/30, Bob/20.
- Carol (no orders) is excluded.
- Order 103 (no matching user) is excluded.

Use when you only care about rows that exist on both sides.

## LEFT (OUTER) JOIN — all left rows, matching right or NULL

```sql
SELECT u.name, o.total
FROM users u
LEFT JOIN orders o ON o.user_id = u.id;
```

Result: Alice/50, Alice/30, Bob/20, **Carol/NULL**.
- Carol kept; her order columns are NULL.
- Order 103 still excluded (it's on the right).

Use when the **left table is the driver** and you want to report unmatched rows: "every user, with their orders if any."

### "Anti-join" pattern with LEFT JOIN

Find users with **no** orders:

```sql
SELECT u.*
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE o.id IS NULL;
```

## RIGHT (OUTER) JOIN — mirror of LEFT

```sql
SELECT u.name, o.total
FROM users u
RIGHT JOIN orders o ON o.user_id = u.id;
```

Result: Alice/50, Alice/30, Bob/20, **NULL/99** (orphan order 103).
- Every order kept; users may be NULL.

Almost no one writes RIGHT JOIN — flip the table order and use LEFT JOIN, which reads more naturally. Code style guides at most companies prefer LEFT.

## FULL (OUTER) JOIN — every row from both sides

```sql
SELECT u.name, o.total
FROM users u
FULL OUTER JOIN orders o ON o.user_id = u.id;
```

Result: Alice/50, Alice/30, Bob/20, **Carol/NULL**, **NULL/99**.

Use for reconciliation: "show me every record from both, marked when it exists only on one side."

> **MySQL doesn't support FULL OUTER JOIN.** Emulate with `LEFT JOIN ... UNION ... RIGHT JOIN`. PostgreSQL and Oracle support it natively.

## CROSS JOIN — Cartesian product

```sql
SELECT u.name, p.code
FROM users u
CROSS JOIN promotions p;
```

3 users × N promotions = `3 × N` rows. Use intentionally for things like generating combinations or date-spine joins. Accidental cross joins (e.g. forgetting the `ON` clause) are a classic bug — every row from one side multiplied by every row from the other.

## Visual summary

```
INNER:  A ∩ B
LEFT:   A   (matches from B, or NULL)
RIGHT:  B   (matches from A, or NULL)
FULL:   A ∪ B
CROSS:  A × B  (every combination)
```

## Performance notes

- The optimizer picks the join algorithm (nested loop, hash join, merge join). You don't pick directly — but indexes on the join columns (`orders.user_id`) make a huge difference.
- `LEFT JOIN ... WHERE right.col = X` accidentally turns into an INNER JOIN logically (the WHERE filters out the NULLs). Move the predicate into the `ON` clause if you want to keep unmatched left rows.

```sql
-- Buggy: drops Carol because of the WHERE
SELECT u.name, o.total
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE o.total > 25;

-- Correct: predicate in ON, Carol kept with NULL total
SELECT u.name, o.total
FROM users u
LEFT JOIN orders o ON o.user_id = u.id AND o.total > 25;
```

## Interview one-liner

> "INNER returns only matching rows on both sides. LEFT keeps all left rows with NULLs for unmatched right; RIGHT is the mirror — usually rewritten as LEFT for readability. FULL keeps everything from both sides. CROSS is the Cartesian product — used intentionally for combinations, accidentally when you forget the ON clause."
