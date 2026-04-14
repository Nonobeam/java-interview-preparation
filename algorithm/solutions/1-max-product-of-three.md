# S1: Maximum product of three numbers

## Key insight

The maximum product of three numbers is always one of two candidates:

1. **Top three largest** — `max1 * max2 * max3`
2. **Two smallest (most negative) × largest** — `min1 * min2 * max1`

Why? Because `negative × negative = positive`. Two very negative numbers produce a large positive that, combined with the biggest positive, can beat the top-three-positives product.

All other combinations are dominated by these two:
- Three negatives → negative (lose to any positive × positive × positive).
- One negative × two positives → smaller than three positives.

So the answer is: `max(max1*max2*max3, min1*min2*max1)`.

---

## Approach 1 — Sort, O(n log n)

Simple and reads well; fine when the interviewer wants a first pass.

```java
public int maxProductOfThree(int[] nums) {
    Arrays.sort(nums);
    int n = nums.length;
    int topThree = nums[n - 1] * nums[n - 2] * nums[n - 3];
    int twoNegsTopOne = nums[0] * nums[1] * nums[n - 1];
    return Math.max(topThree, twoNegsTopOne);
}
```

**Time:** O(n log n). **Space:** O(1) (in-place sort).

---

## Approach 2 — Single pass, O(n)

Track the **three largest** and the **two smallest** in one scan.

```java
public int maxProductOfThree(int[] nums) {
    int max1 = Integer.MIN_VALUE;  // largest
    int max2 = Integer.MIN_VALUE;  // 2nd largest
    int max3 = Integer.MIN_VALUE;  // 3rd largest
    int min1 = Integer.MAX_VALUE;  // smallest
    int min2 = Integer.MAX_VALUE;  // 2nd smallest

    for (int x : nums) {
        // update top 3
        if (x >= max1) {
            max3 = max2; max2 = max1; max1 = x;
        } else if (x >= max2) {
            max3 = max2; max2 = x;
        } else if (x > max3) {
            max3 = x;
        }

        // update bottom 2
        if (x <= min1) {
            min2 = min1; min1 = x;
        } else if (x < min2) {
            min2 = x;
        }
    }

    return Math.max(max1 * max2 * max3, min1 * min2 * max1);
}
```

**Time:** O(n). **Space:** O(1).

---

## Walkthrough on `[-10, -10, 5, 2]`

| step | x   | max1 | max2 | max3 | min1 | min2 |
|------|-----|------|------|------|------|------|
| init |     | -∞   | -∞   | -∞   | +∞   | +∞   |
| 1    | -10 | -10  | -∞   | -∞   | -10  | +∞   |
| 2    | -10 | -10  | -10  | -∞   | -10  | -10  |
| 3    | 5   | 5    | -10  | -10  | -10  | -10  |
| 4    | 2   | 5    | 2    | -10  | -10  | -10  |

- topThree = `5 * 2 * -10 = -100`
- twoNegsTopOne = `-10 * -10 * 5 = 500`
- **answer = 500** ✅

---

## Edge cases to call out in the interview

- **All negatives**, e.g. `[-5, -4, -3, -2, -1]` → answer is `-1 * -2 * -3 = -6` (top-three path; the two-mins-×-max path would give `-5 * -4 * -1 = -20`).
- **Zeros** mixed in: the formula still holds; zero just produces `0` which loses unless all other candidates are negative.
- **Exactly three elements**: both formulas yield the same product — return it.
- **Overflow**: if inputs can reach `±10^9`, use `long` for the product (`1000³ = 10⁹` fits in `int`, but bigger ranges won't).

---

## Interview one-liner

> "Maximum product of three is either the top three numbers, or the two most negative times the single largest. One pass tracking `max1, max2, max3, min1, min2` gives O(n) time, O(1) space, and handles negatives cleanly."
