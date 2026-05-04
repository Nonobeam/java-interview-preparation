# 62. Exceptions: checked vs unchecked + try-with-resources

## The hierarchy

```
Throwable
├── Error                  (JVM problems: OutOfMemoryError, StackOverflowError) — don't catch
└── Exception
    ├── RuntimeException   (unchecked)
    │   ├── NullPointerException
    │   ├── IllegalArgumentException
    │   ├── IllegalStateException
    │   └── ...
    └── (other Exception)  (checked: IOException, SQLException, ...)
```

## Checked vs unchecked

| | Checked | Unchecked (`RuntimeException`) |
|---|---|---|
| Compiler enforced | Yes — must `throws` or `try/catch` | No |
| Intent | Recoverable conditions the caller might handle (file not found, network down) | Programmer errors (null deref, bad argument, broken invariant) |
| Examples | `IOException`, `SQLException`, `InterruptedException` | `NullPointerException`, `IllegalArgumentException`, `IllegalStateException`, `ClassCastException` |

```java
// checked — won't compile without throws or try/catch
public String read(Path p) throws IOException {
    return Files.readString(p);
}

// unchecked — compiles fine
public int divide(int a, int b) {
    if (b == 0) throw new IllegalArgumentException("b must be non-zero");
    return a / b;
}
```

## Modern preference

Most modern Java code (and Spring) leans toward **unchecked** exceptions because:
- Checked exceptions don't compose with lambdas / Streams.
- Most "recoverable" exceptions in practice get rethrown anyway.
- They leak implementation details up the call stack.

Spring deliberately wraps `SQLException` (checked) into `DataAccessException` (unchecked) so service layers don't need to declare it.

## try-with-resources

Auto-closes anything implementing `AutoCloseable` (or `Closeable`), even on exception.

```java
try (BufferedReader r = Files.newBufferedReader(path)) {
    return r.readLine();
}
// r.close() called automatically — even if readLine() throws
```

Equivalent without try-with-resources:

```java
BufferedReader r = Files.newBufferedReader(path);
try {
    return r.readLine();
} finally {
    r.close();  // but if r.close() ALSO throws, the original exception is lost
}
```

### Multiple resources

Closed in **reverse order** of opening.

```java
try (Connection c = ds.getConnection();
     PreparedStatement ps = c.prepareStatement(sql);
     ResultSet rs = ps.executeQuery()) {
    while (rs.next()) { ... }
}
// closes: rs, then ps, then c
```

### Suppressed exceptions

If both the body AND `close()` throw, the body's exception wins and `close()`'s exception is attached as **suppressed** — visible via `e.getSuppressed()`. No exception is silently lost.

## try / catch / finally semantics

- `finally` runs even if you `return` from `try` or `catch`.
- A `return` in `finally` **overrides** any return or exception from `try`/`catch` — almost always a bug. Don't do it.
- Catch the most specific type first, broadest last. Multi-catch (`catch (IOException | SQLException e)`) when the handling is identical.

## Common pitfalls

- Catching `Exception` (or worse, `Throwable`) and swallowing it. Always rethrow or log meaningfully.
- Logging AND throwing — pick one (usually throw; let the boundary log).
- Re-wrapping without preserving the cause: use `throw new MyException("msg", e)`, never `new MyException("msg")`.
- Catching `InterruptedException` without re-interrupting:
  ```java
  try { Thread.sleep(100); }
  catch (InterruptedException e) {
      Thread.currentThread().interrupt(); // restore the flag
      throw new RuntimeException(e);
  }
  ```

## Interview one-liner

> "Checked exceptions are compiler-enforced for recoverable conditions; unchecked extend `RuntimeException` for programmer errors. Modern code leans unchecked because checked don't compose with lambdas. Always use try-with-resources for `AutoCloseable` — it closes in reverse order and surfaces close-time exceptions as suppressed instead of losing the original."
