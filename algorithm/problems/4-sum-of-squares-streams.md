# P4: Sum of squares using Java Streams

Given a list of integers, return the sum of their squares.

```java
int sumOfSquares(List<Integer> nums)
```

Solve it using the **Stream API** with a lambda — that's the form KMS-style fresher interviewers like to see when probing Java 8 fluency.

## Examples

```
sumOfSquares([1, 2, 3])         → 14    (1 + 4 + 9)
sumOfSquares([])                → 0
sumOfSquares([-2, 3])           → 13    (4 + 9)
sumOfSquares([1000, 1000])      → 2_000_000
```

## Constraints

- `nums` may be empty.
- Watch overflow: with large or many elements, the result can exceed `Integer.MAX_VALUE`.

## What's being tested

- Comfort with `mapToInt` / `IntStream.sum()`.
- Recognising the `int` vs `long` overflow trap.
- Knowing the difference between `Stream<Integer>` (boxed) and `IntStream` (primitive).
