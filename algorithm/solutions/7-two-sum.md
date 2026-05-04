# S7: Two Sum

## Approach — single-pass HashMap, O(n) time

For each element `nums[i]`, the complement we need is `target - nums[i]`. If we've already seen the complement at some earlier index `j`, we're done. Otherwise, record `nums[i] → i` and continue.

The trick to handle duplicates correctly is to **check first, then insert** — that way, on `[3, 3]` with `target = 6`, the second `3` looks up the first and we return `[0, 1]`.

```java
import java.util.HashMap;
import java.util.Map;

public class TwoSum {

    public static int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> seen = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];
            Integer j = seen.get(complement);
            if (j != null) return new int[]{j, i};
            seen.put(nums[i], i);
        }
        throw new IllegalArgumentException("No two-sum solution");
    }
}
```

**Time:** O(n). **Space:** O(n).

## Walkthrough on `[2, 7, 11, 15]`, target = 9

| i | nums[i] | complement | seen lookup | seen after |
|---|---------|------------|-------------|------------|
| 0 | 2       | 7          | miss        | {2→0}      |
| 1 | 7       | 2          | **hit @ 0** | return [0, 1] |

## Brute force — O(n²) (mention then improve)

```java
for (int i = 0; i < nums.length; i++)
    for (int j = i + 1; j < nums.length; j++)
        if (nums[i] + nums[j] == target) return new int[]{i, j};
```

State this baseline so the interviewer sees you understand the trade-off — then move to the HashMap version.

## Why "check then insert" matters

If you insert first then check, you'd find the same element as its own complement when `target = 2 * nums[i]`. Example: `[3, 4, 5]`, target = 6. At `i = 0`, `nums[i] = 3`, complement = 3; if 3 is already in the map (from this same iteration), you'd return `[0, 0]` — using one element twice. Check-then-insert avoids this.

## Variant — sorted array, two-pointer (no extra space)

If the input is sorted, you can do O(1) extra space:

```java
public int[] twoSumSorted(int[] sorted, int target) {
    int lo = 0, hi = sorted.length - 1;
    while (lo < hi) {
        int sum = sorted[lo] + sorted[hi];
        if (sum == target) return new int[]{lo, hi};
        if (sum < target) lo++;
        else hi--;
    }
    throw new IllegalArgumentException();
}
```

The HashMap solution doesn't require sorted input, which is the usual question constraint.

## Edge cases

- Two equal values both adding up to target → handled by check-then-insert.
- No pair exists → spec says one always exists; in real code throw or return `null`/`Optional`.
- Negative numbers and zero → no special handling, math is the same.
- Overflow on `target - nums[i]` → only an issue at extremes of `int`. Use `long` if relevant.

## Interview one-liner

> "Single pass with a HashMap of value→index. For each element, look up its complement before inserting — the look-up-before-insert order handles duplicates and avoids using the same index twice. O(n) time, O(n) space. Brute force is O(n²); the sorted-input variant is two-pointer O(1) extra space."
