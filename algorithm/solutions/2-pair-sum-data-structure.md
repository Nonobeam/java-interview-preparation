# S2: Pair-sum data structure

## Approach — value→count HashMap

Store each inserted integer with how many times it has been seen. To answer `pairsWithSum(target)`:

For each distinct value `a` in the map, look up its complement `b = target - a`:

- If `a < b`: emit `count(a) * count(b)` copies of `(a, b)` (each occurrence of `a` pairs with each occurrence of `b`).
- If `a == b`: pairs come from picking two **different** elements with value `a`. That's `C(count(a), 2) = count(a) * (count(a) - 1) / 2`.
- If `a > b`: skip — already handled when we processed `b`.

Iterating only when `a <= b` is what prevents `(1,5)` AND `(5,1)` from both appearing.

## Java implementation

```java
import java.util.*;

public class PairSumDS {
    private final Map<Integer, Integer> counts = new HashMap<>();

    public void insert(int x) {
        counts.merge(x, 1, Integer::sum);
    }

    public List<int[]> pairsWithSum(int target) {
        List<int[]> result = new ArrayList<>();
        for (Map.Entry<Integer, Integer> e : counts.entrySet()) {
            int a = e.getKey();
            int countA = e.getValue();
            int b = target - a;

            if (a > b) continue;   // skip; handled when we processed b

            if (a == b) {
                // pick 2 distinct elements with value a
                int pairs = countA * (countA - 1) / 2;
                for (int i = 0; i < pairs; i++) result.add(new int[]{a, a});
            } else {
                Integer countB = counts.get(b);
                if (countB == null) continue;
                int pairs = countA * countB;
                for (int i = 0; i < pairs; i++) result.add(new int[]{a, b});
            }
        }
        return result;
    }
}
```

**Complexity:**
- `insert` — O(1) amortized.
- `pairsWithSum` — O(d) distinct values to iterate + O(P) to materialize the P returned pairs.

## Worked example: `[1, 5, 3, 3, 7]`, target = 6

Map: `{1→1, 5→1, 3→2, 7→1}`.

| a | b = 6-a | skip? | pairs to emit |
|---|---------|-------|---------------|
| 1 | 5       | a<b   | `1 * 1 = 1` → `[1,5]` |
| 5 | 1       | a>b, skip | — |
| 3 | 3       | a==b  | `C(2,2) = 1` → `[3,3]` |
| 7 | -1      | not in map | — |

Result: `[[1,5], [3,3]]` ✅

## Variation 1 — return only **distinct** pairs

Drop the `count * count` multiplication: emit each `(a, b)` once if both exist.

```java
if (a == b) {
    if (counts.get(a) >= 2) result.add(new int[]{a, a});
} else {
    if (counts.containsKey(b)) result.add(new int[]{a, b});
}
```

## Variation 2 — incremental cache (stretch goal)

Maintain `Map<Integer, Set<int[]>> cachedByTarget`. On `insert(x)`:

- For each existing key `y` in `counts`, compute `target = x + y` and add the pair to `cachedByTarget.get(target)`.
- Insert cost becomes O(d) instead of O(1), but `pairsWithSum` becomes O(P).

Worth it only if reads vastly outnumber writes; otherwise the simple version wins.

## Edge cases to call out

- Empty structure → empty list.
- `target = 2x` and only one `x` inserted → no pair (need two distinct elements).
- `Integer.MAX_VALUE` / `MIN_VALUE` → `target - a` can overflow. Use `long` arithmetic if the interviewer cares.
- Negative values, zero — no special handling required, the math holds.

## Interview one-liner

> "Use a HashMap<value, count>. For each distinct value `a`, look up `target - a`. Process only when `a <= b` to avoid duplicate `(a,b)` and `(b,a)`. Self-pair case `a == b` requires count ≥ 2 and contributes `C(count, 2)` pairs. O(1) inserts, O(distinct + P output) queries."
