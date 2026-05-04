# P2: Pair-sum data structure (KMS-reported)

> Reported asked at KMS Technology, fresher Java interview.

Implement a data structure that supports two operations:

1. `void insert(int x)` — insert an integer.
2. `List<int[]> pairsWithSum(int target)` — return **all pairs** of integers in the structure whose sum equals `target`.

## Examples

```
ds.insert(1);
ds.insert(5);
ds.insert(3);
ds.insert(3);
ds.insert(7);
ds.pairsWithSum(6);
// → [[1, 5], [3, 3]]   (order doesn't matter)

ds.pairsWithSum(10);
// → [[3, 7], [3, 7]]   (two distinct 3s each pair with the 7)

ds.pairsWithSum(100);
// → []
```

## Constraints

- Values fit in `int`. Don't worry about overflow on `target` for this exercise.
- The structure may contain **duplicates**.
- A pair `(a, b)` is the same as `(b, a)` — don't return both.
- A value can pair with **itself** only if it appears at least twice in the structure (i.e. two different elements that share the value).

## What's being tested

- Picking the right data structure: `HashMap<Integer, Integer>` (value → count) for O(1) inserts and O(n) pair retrieval.
- Handling duplicates and self-pairs cleanly.
- Avoiding double-counting `(a, b)` and `(b, a)`.

## Stretch goal

If `pairsWithSum` is called frequently, can you keep an incremental cache so most queries are O(1)? Discuss the tradeoff vs the simple O(n) scan.
