# P7: Two Sum

Given an array of integers `nums` and an integer `target`, return the indices of the two numbers that add up to `target`.

```java
int[] twoSum(int[] nums, int target)
```

Assume each input has **exactly one solution**, and you may not use the same element twice.

## Examples

```
twoSum([2, 7, 11, 15], 9)   → [0, 1]    (2 + 7)
twoSum([3, 2, 4],     6)    → [1, 2]    (2 + 4)
twoSum([3, 3],        6)    → [0, 1]    (duplicates allowed across different indices)
```

## Constraints

- `2 <= nums.length <= 10^4`
- Values fit in `int`. Negative values allowed.
- Return order doesn't matter (`[0,1]` and `[1,0]` are both fine unless interviewer specifies).
- Each element can be used at most once (you can't pick the same index twice).

## What's being tested

- Recognizing the brute-force O(n²) and improving to **O(n) with a HashMap**.
- Single-pass elegance: while iterating, look up the complement before inserting.
- Spotting the duplicate trap (`[3, 3]` with `target = 6`).
