# P8: Valid anagram

Given two strings `s` and `t`, return `true` if `t` is an anagram of `s` (same characters with the same frequencies, possibly in different order), `false` otherwise.

```java
boolean isAnagram(String s, String t)
```

## Examples

```
isAnagram("listen", "silent")     → true
isAnagram("anagram", "nagaram")   → true
isAnagram("rat", "car")           → false
isAnagram("a", "ab")              → false   (different lengths)
isAnagram("", "")                 → true
```

## Constraints

- Strings can be empty.
- Decide and document: case-sensitive? Whitespace? Unicode?
  Default for the interview: lowercase ASCII only, no whitespace handling.
- Stretch: handle full Unicode (multi-byte code points).

## What's being tested

- Quick win: length check first.
- Pick between **sort-and-compare** (O(n log n), no extra structure) and **frequency count** (O(n) time, O(k) space where k = alphabet size).
- Awareness of Unicode caveats with `char[]`.
