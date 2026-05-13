# Q76 — Does `System.gc()` Trigger Immediate Garbage Collection?

Correct answer: **No** — `System.gc()` is a *hint* to the JVM; it is not guaranteed to run immediately, or at all.

## Questions

1. What does `System.gc()` actually do? Why is it only a hint?
2. Which JVM flag disables `System.gc()` entirely? When would you set it in production?
3. Describe the JVM garbage collection process at a high level: minor GC (Eden → Survivor), major GC, full GC. What triggers each?
4. What is `System.runFinalization()`? Is calling it before `System.gc()` a good idea?
5. In what real scenario would a developer be tempted to call `System.gc()`? What should they do instead?
6. How does `GC overhead limit exceeded` OOM differ from a regular `OutOfMemoryError`?
