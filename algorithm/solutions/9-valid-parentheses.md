# S9: Valid parentheses

## Approach — stack of expected closers

Walk the string. For each character:
- **Open bracket** → push the **expected closing bracket** onto the stack.
- **Close bracket** → it must match the top of the stack; if the stack is empty or the top doesn't match, fail.

At the end, the string is valid iff the stack is empty.

This "push the expected closer" trick removes the need for a `Map<Character, Character>` lookup at close-time.

```java
import java.util.ArrayDeque;
import java.util.Deque;

public class ValidParentheses {

    public static boolean isValid(String s) {
        Deque<Character> stack = new ArrayDeque<>();
        for (int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);
            switch (c) {
                case '(' -> stack.push(')');
                case '[' -> stack.push(']');
                case '{' -> stack.push('}');
                default -> {
                    // c is a closer
                    if (stack.isEmpty() || stack.pop() != c) return false;
                }
            }
        }
        return stack.isEmpty();
    }
}
```

**Time:** O(n). **Space:** O(n) worst case (e.g. `((((((`).

> Use `ArrayDeque`, not `Stack`. `java.util.Stack` extends `Vector` (synchronized, slow); `Deque` is the modern recommendation.

## Walkthrough on `"{[()]}"`

| i | char | action               | stack (top → bottom) |
|---|------|----------------------|----------------------|
| 0 | {    | push }               | }                    |
| 1 | [    | push ]               | ] }                  |
| 2 | (    | push )               | ) ] }                |
| 3 | )    | pop ), matches       | ] }                  |
| 4 | ]    | pop ], matches       | }                    |
| 5 | }    | pop }, matches       | (empty)              |

Stack empty → **true** ✅

## Walkthrough on `"([)]"`

| i | char | action          | stack |
|---|------|-----------------|-------|
| 0 | (    | push )          | )     |
| 1 | [    | push ]          | ] )   |
| 2 | )    | pop ], no match | **return false** |

## Common bugs to avoid

- Using `Stack<Character>` instead of `ArrayDeque<Character>` — works but is the legacy choice.
- Returning `true` without checking the stack is empty at the end → fails on `"((("`.
- Not handling close-when-empty → `NoSuchElementException` from `pop()` on an empty stack. The `stack.isEmpty()` check before `pop` prevents it.
- Comparing `Character` objects with `==` instead of `.equals` — works here because the JVM caches the small-bracket characters in the `Character` cache, but it's fragile. Using `char` (primitive) avoids the question entirely.

## Variant — also allow other characters

If the input may contain non-bracket characters (e.g. `"a + (b * [c - d])"`):

```java
default -> {
    if (c == ')' || c == ']' || c == '}') {
        if (stack.isEmpty() || stack.pop() != c) return false;
    }
    // else: ignore non-bracket characters
}
```

## Interview one-liner

> "Stack of expected closers: push `)` when you see `(`, etc.; on a close bracket, pop and compare. Valid iff every close finds its match and the stack ends empty. O(n) time, O(n) space. Use `ArrayDeque`, not `java.util.Stack`."
