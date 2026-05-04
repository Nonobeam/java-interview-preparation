# 60. `String` immutability, String pool, and `StringBuilder` / `StringBuffer`

## Why `String` is immutable

`String` is a `final` class wrapping a `final` `char[]` (or `byte[]` since Java 9). Once created, the value never changes. Reasons:

1. **Security** — strings are used as classloader names, file paths, network hosts, SQL parts. Immutability prevents one piece of code from mutating a string another piece is relying on.
2. **Thread-safety** — immutable objects are safe to share across threads with no synchronization.
3. **Hashcode caching** — `String.hashCode()` is computed once and cached; safe because the value can never change. This makes `String` excellent as a `HashMap` key.
4. **String pool / interning** — the JVM can safely share a single instance for the same literal, because no caller can mutate it.

## The String pool

A region of the heap (since Java 7; previously PermGen) where the JVM keeps a unique copy of each **string literal**.

```java
String a = "hello";
String b = "hello";
a == b; // true — same interned object

String c = new String("hello"); // forces a NEW heap object
a == c;          // false
a.equals(c);     // true
a == c.intern(); // true — intern() returns the pool's copy
```

- Literals are auto-interned at class-load time.
- `new String("...")` always creates a new object outside the pool.
- `intern()` lets you put a runtime-built string into the pool manually (rare; pool is shared so mass-interning user input causes memory pressure).

## `String` vs `StringBuilder` vs `StringBuffer`

| Class           | Mutable? | Thread-safe?          | Use when |
|-----------------|----------|-----------------------|----------|
| `String`        | No       | Yes (immutable)       | Storing or passing text. |
| `StringBuilder` | Yes      | **No**                | Building text in a single thread (the common case). |
| `StringBuffer`  | Yes      | Yes (synchronized)    | Building text shared across threads (rare; usually a design smell). |

### Why concatenating `String` in a loop is bad

```java
String s = "";
for (int i = 0; i < 10_000; i++) {
    s += i;  // each iteration: new String, copy, throw away the old one
}
```

Each `+=` allocates a fresh `String`. O(n²) total work and lots of garbage.

```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 10_000; i++) {
    sb.append(i);  // mutates the same internal buffer
}
String s = sb.toString();
```

O(n) total work.

> Compiler note: a single `a + b + c` is often rewritten by `javac` to use `StringBuilder` automatically. The trap is concatenation **inside loops**, where the compiler can't lift the builder out.

## When the pool helps in real code

Map keys with repeated values benefit from interning, e.g. parsing log lines with a small set of distinct level strings. But measure first — the pool is hashmap-backed and not free.

## Interview one-liner

> "String is immutable so it can be safely shared across threads and used as a hash key — and the JVM exploits that by interning literals in the String pool. For mutable text, use `StringBuilder` (single-threaded) or `StringBuffer` (synchronized, rarely needed). Never `+=` strings inside a loop."
