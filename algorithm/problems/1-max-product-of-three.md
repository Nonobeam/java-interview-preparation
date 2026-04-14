# P1: Maximum product of three numbers in an array

Given an integer array `nums`, find three numbers whose product is **maximum** and return that maximum product.

## Examples

```
Input:  [1, 2, 3]
Output: 6          (1 * 2 * 3)

Input:  [1, 2, 3, 4]
Output: 24         (2 * 3 * 4)

Input:  [-10, -10, 5, 2]
Output: 500        (-10 * -10 * 5)   ← two large negatives flip to positive

Input:  [-4, -3, -2, -1, 60]
Output: 720        (-4 * -3 * 60)
```

## Constraints

- `3 <= nums.length <= 10^4`
- `-1000 <= nums[i] <= 1000`
- Array may contain **negatives**, **zeros**, and **duplicates**.

## What's being tested

- Recognise that with negatives, the maximum can come from **two smallest (most negative)** multiplied by the **largest positive**.
- Do it in **O(n)** time with constant extra memory, not the naive O(n log n) sort.
