# S3: Smallest distance between two strings in a list

## Key insight

For each index `i`, you only care about the **last seen** index of `a` and the **last seen** index of `b`. Whenever you see one, the distance to the other (if seen at all) can only get larger as you move forward — so check it immediately, then update.

This collapses the search to a single pass.

## Java — O(n) one-pass

```java
import java.util.List;

public class StringDistance {

    public static int dist(List<String> xs, String a, String b) {
        if (xs == null || xs.isEmpty()) return -1;
        if (a.equals(b)) {
            // convention: same string → 0 if it appears at all, else -1
            return xs.contains(a) ? 0 : -1;
        }

        int lastA = -1, lastB = -1;
        int best = Integer.MAX_VALUE;

        for (int i = 0; i < xs.size(); i++) {
            String s = xs.get(i);
            if (s.equals(a)) {
                lastA = i;
                if (lastB != -1) best = Math.min(best, lastA - lastB);
            } else if (s.equals(b)) {
                lastB = i;
                if (lastA != -1) best = Math.min(best, lastB - lastA);
            }
        }

        return best == Integer.MAX_VALUE ? -1 : best;
    }
}
```

**Time:** O(n). **Space:** O(1).

## Walkthrough on `["cat","dog","bird","fish","cat","duck","chicken","dog"]`, `dist("dog","duck")`

| i | s        | lastA (dog) | lastB (duck) | candidate | best |
|---|----------|-------------|--------------|-----------|------|
| 0 | cat      | -1          | -1           | —         | ∞    |
| 1 | dog      | 1           | -1           | none      | ∞    |
| 2 | bird     | 1           | -1           | —         | ∞    |
| 3 | fish     | 1           | -1           | —         | ∞    |
| 4 | cat      | 1           | -1           | —         | ∞    |
| 5 | duck     | 1           | 5            | 5-1=4     | 4    |
| 6 | chicken  | 1           | 5            | —         | 4    |
| 7 | dog      | 7           | 5            | 7-5=2     | 2    |

Result: **2** ✅

## Why the one-pass is correct

Claim: for any pair `(i, j)` of occurrences (`xs[i]=a`, `xs[j]=b`), at the moment we process `max(i, j)`, the other index is exactly `min(i, j)` recorded as `lastA` or `lastB`. Because the loop checks the candidate distance every time it advances either index, the smallest such pairing distance is observed.

## Edge cases

- Empty list → `-1`.
- Only one of `a`, `b` appears → loop never enters the "both seen" branch → return `-1`.
- `a.equals(b)` → convention chosen above (return 0). Interviewer may prefer "return min distance between two distinct occurrences" — clarify before coding. If that's the spec:
  ```java
  if (a.equals(b)) {
      int last = -1, best = Integer.MAX_VALUE;
      for (int i = 0; i < xs.size(); i++) {
          if (xs.get(i).equals(a)) {
              if (last != -1) best = Math.min(best, i - last);
              last = i;
          }
      }
      return best == Integer.MAX_VALUE ? -1 : best;
  }
  ```
- Very long list → `xs.get(i)` on `LinkedList` is O(n); use `Iterator` if list type is unknown:
  ```java
  int i = 0;
  for (String s : xs) { ... i++; }
  ```

## Naive O(n × m) for contrast

```java
// Don't write this in the interview — call it out as the baseline you're improving on.
int best = Integer.MAX_VALUE;
for (int i = 0; i < xs.size(); i++) {
    if (!xs.get(i).equals(a)) continue;
    for (int j = 0; j < xs.size(); j++) {
        if (xs.get(j).equals(b)) best = Math.min(best, Math.abs(i - j));
    }
}
```

`O(n × m)` where `m` is occurrences of `a` — degenerate to `O(n²)` if `a` is everywhere.

## Interview one-liner

> "Single pass with two `lastSeen` indices — every time you see `a` or `b`, update its index and check the distance to the other one. The smallest distance between any pair must occur at the later of the two indices, so one scan finds it. O(n) time, O(1) space."
