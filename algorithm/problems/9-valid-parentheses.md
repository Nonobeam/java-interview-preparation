# P9: Valid parentheses

Given a string containing only the characters `(`, `)`, `{`, `}`, `[`, `]`, determine whether the input string is **valid**.

A string is valid if:
1. Open brackets are closed by the **same type** of brackets.
2. Open brackets are closed in the **correct order**.
3. Every close bracket has a matching open bracket.

```java
boolean isValid(String s)
```

## Examples

```
isValid("()")          → true
isValid("()[]{}")      → true
isValid("(]")          → false
isValid("([)]")        → false   (interleaved, not nested)
isValid("{[()]}")      → true
isValid("")            → true
isValid("(")           → false   (unmatched open)
isValid(")")           → false   (unmatched close)
```

## Constraints

- `0 <= s.length <= 10^4`
- `s` only contains the six bracket characters.
- Empty string is valid by convention.

## What's being tested

- Recognising this as a **stack** problem — last-opened-first-closed = LIFO.
- Pair mapping: `)` → `(`, `]` → `[`, `}` → `{`.
- Boundary conditions: unmatched closes (stack empty when popping) and leftover opens (stack non-empty at end).
