# P3: Smallest distance between two strings in a list (KMS-reported)

> Reported asked at KMS Technology, fresher Java interview.

Given a `List<String>`, write a function:

```java
int dist(List<String> xs, String a, String b)
```

That returns the **smallest absolute index distance** between any occurrence of `a` and any occurrence of `b` in the list. If either `a` or `b` is missing entirely, return `-1`.

## Examples

```
xs = ["cat","dog","bird","fish","cat","duck","chicken","dog"]

dist(xs, "dog", "duck") → 2     (dog@1, duck@5 → 4; dog@7, duck@5 → 2; min = 2)
dist(xs, "cat", "dog")  → 1     (cat@0, dog@1)
dist(xs, "dog", "dog")  → 0     (a == b — convention: return 0 when same string)
dist(xs, "dog", "snake")→ -1
```

## Constraints

- `1 <= xs.size() <= 10^6`
- Strings are non-null.
- `a` and `b` may be the same string (decide and document the convention; here: return 0 if both occur).
- Multiple occurrences of `a` and `b` are possible — return the **smallest** distance across all pairings.

## What's being tested

- Avoiding the brute O(n²) "for each a, find nearest b".
- One-pass O(n) using last-seen indices.
- Edge-case handling: missing values, equal strings, empty list.
