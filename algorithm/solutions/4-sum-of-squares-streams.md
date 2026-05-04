# S4: Sum of squares using Streams

## Idiomatic — `IntStream` + `sum()`

```java
import java.util.List;

public class SumOfSquares {
    public static int sumOfSquares(List<Integer> nums) {
        return nums.stream()
                   .mapToInt(n -> n * n)
                   .sum();
    }
}
```

- `mapToInt` unboxes to `IntStream`, avoiding boxed `Integer` arithmetic.
- `IntStream.sum()` returns `int`. Empty stream → 0 (no `Optional` for `sum`).

## Overflow-safe — return `long`

For inputs that might overflow `int`:

```java
public static long sumOfSquaresLong(List<Integer> nums) {
    return nums.stream()
               .mapToLong(n -> (long) n * n)   // cast BEFORE multiply
               .sum();
}
```

Critical detail: `(long) (n * n)` would multiply as `int` first and overflow. Cast the operand, not the result.

## Variants worth knowing

### Using `reduce`

```java
int total = nums.stream()
                .mapToInt(n -> n * n)
                .reduce(0, Integer::sum);
```

Equivalent to `.sum()` but the form generalizes to other binary operations (e.g. `Math::max`).

### Using a method reference + helper

```java
int total = nums.stream()
                .mapToInt(this::square)
                .sum();

private int square(int n) { return n * n; }
```

### `Stream<Integer>` instead of `IntStream` — works but boxes

```java
// Less efficient: boxed addition, allocates Integers
int total = nums.stream()
                .map(n -> n * n)
                .reduce(0, Integer::sum);
```

Mention this in the interview to show you know **why** `mapToInt` is preferred.

## Imperative comparison (interviewer may ask)

```java
int total = 0;
for (int n : nums) total += n * n;
```

Identical performance after JIT. Streams win on readability for chains; loops win when you're mutating multiple things.

## Edge cases

- Empty list → 0.
- `null` list → NullPointerException. Add `Objects.requireNonNull(nums)` if you want to fail fast, or guard `if (nums == null || nums.isEmpty()) return 0;`.
- Null elements inside the list → NPE on `n * n` (auto-unbox of null Integer). Filter first if relevant: `.filter(Objects::nonNull)`.

## Interview one-liner

> "`nums.stream().mapToInt(n -> n * n).sum()`. Use `mapToInt` to avoid boxing, and switch to `mapToLong` with `(long) n * n` if overflow is on the table. Cast the operand, not the multiplication result, or you've already overflowed."
