# S8: Valid anagram

## Approach 1 — frequency count, O(n) time / O(1) space (lowercase ASCII)

Maintain a count of each character in `s` (increment) and `t` (decrement). If all counts end at zero, they're anagrams.

```java
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    int[] counts = new int[26];          // a-z
    for (int i = 0; i < s.length(); i++) {
        counts[s.charAt(i) - 'a']++;
        counts[t.charAt(i) - 'a']--;
    }
    for (int c : counts) if (c != 0) return false;
    return true;
}
```

**Time:** O(n). **Space:** O(1) (fixed 26-int array).

## Approach 2 — sort and compare, O(n log n) / O(n) space

Reads in two lines but allocates and sorts:

```java
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    char[] a = s.toCharArray();
    char[] b = t.toCharArray();
    Arrays.sort(a);
    Arrays.sort(b);
    return Arrays.equals(a, b);
}
```

Use this if you want a quick first pass; mention you'd switch to frequency count for production.

## Approach 3 — full Unicode (HashMap of code points)

`char` is 16-bit; emoji and supplementary-plane characters are surrogate pairs. The fixed 26-int array breaks immediately for non-ASCII input.

```java
public boolean isAnagram(String s, String t) {
    if (s.codePointCount(0, s.length()) !=
        t.codePointCount(0, t.length())) return false;

    Map<Integer, Integer> counts = new HashMap<>();
    s.codePoints().forEach(cp -> counts.merge(cp, 1, Integer::sum));
    t.codePoints().forEach(cp -> counts.merge(cp, -1, Integer::sum));

    return counts.values().stream().allMatch(v -> v == 0);
}
```

**Time:** O(n). **Space:** O(k) where k = distinct code points.

## Worked example: `"listen"` vs `"silent"`

Both have length 6.
After processing:

| char | s adds | t subtracts | net |
|------|--------|-------------|-----|
| l    | +1 (i=0) | -1 (i=4) | 0 |
| s    | +1     | -1 (i=0)    | 0 |
| i    | +1     | -1 (i=1)    | 0 |
| e    | +1     | -1 (i=3)    | 0 |
| n    | +1     | -1 (i=5)    | 0 |
| t    | +1     | -1 (i=2)    | 0 |

All zero → `true`.

## Edge cases

- Different lengths → `false` immediately. Cheapest check, do it first.
- Both empty → `true` (length check passes, count loop never runs).
- Mixed case (`"Listen"` vs `"silent"`) → `false` with the ASCII version. Lowercase both first if the spec wants case-insensitive: `s.toLowerCase()`.
- Whitespace and punctuation → not handled in the basic versions. For phrase-anagrams ("conversation" / "voices rant on"), strip non-letters first.
- Surrogate pairs / emoji → use Approach 3.

## Interview one-liner

> "Length-check first, then count chars: `+1` from `s`, `-1` from `t` into a 26-int array; if all zero, anagram. O(n) time, O(1) space for ASCII. Sort-and-compare is shorter but O(n log n). For Unicode beyond the BMP, use `codePoints()` into a HashMap to handle surrogate pairs correctly."
