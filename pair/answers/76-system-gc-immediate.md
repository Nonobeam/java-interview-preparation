# 76. Does `System.gc()` Trigger Immediate Garbage Collection?

## Short answer: No

`System.gc()` is a **hint** to the JVM. The JVM may:
- Run a full GC shortly after.
- Run a GC at some later time.
- Ignore the call entirely.

The JVM specification explicitly says: "Calling the `gc` method suggests that the Java Virtual Machine expend effort toward recycling unused objects… when control returns from the method call, the Java Virtual Machine has made a best effort to reclaim space from all discarded objects."

**"Best effort"** = not guaranteed, not immediate.

## The correct syntax

```java
System.gc();           // correct
Runtime.getRuntime().gc();   // equivalent — System.gc() delegates here
```

## The flag that disables it entirely

```
-XX:+DisableExplicitGC
```

When this JVM flag is set, `System.gc()` becomes a no-op. Commonly set in production to prevent application code from accidentally triggering expensive full GCs.

Also useful: `-XX:+ExplicitGCInvokesConcurrent` — makes `System.gc()` trigger a concurrent GC cycle (G1/ZGC) instead of a stop-the-world full GC.

## GC triggers — what actually causes GC

| GC type | Trigger |
|---------|---------|
| **Minor GC** (Young gen) | Eden space fills up |
| **Major GC** (Old gen) | Old generation fills up |
| **Full GC** | Old gen + Metaspace fill, or explicit `System.gc()`, or heap ergonomics decide |
| **Concurrent cycle (G1/ZGC)** | Heap occupancy threshold (default 45% for G1 initiating heap occupancy) |

You do not and should not control GC timing from application code.

## `System.runFinalization()`

```java
System.runFinalization();
```

Suggests that the JVM run the `finalize()` methods of objects pending finalization. Also just a hint. Also effectively deprecated — `finalize()` itself is deprecated since Java 9 and removed in Java 18.

Calling `System.runFinalization()` before `System.gc()` is **not** a good idea. It encourages reliance on finalizers for resource cleanup, which is unreliable and can cause object resurrection bugs. Use `try-with-resources` instead.

## When a developer is tempted to call `System.gc()`

Typical scenario: "I just released a large cache / closed a connection pool / finished a batch job — I want to free memory now."

**What to do instead:**
- Explicitly null out references so the GC can collect them on its own schedule.
- Use `WeakReference` / `SoftReference` for caches — the GC can collect them under memory pressure without being asked.
- Profile with a heap dump (`jmap -dump` or `-XX:+HeapDumpOnOutOfMemoryError`) to find real leaks.
- Tune GC thresholds (`-XX:InitiatingHeapOccupancyPercent`, `-Xmx`) if GC is running too infrequently.

Explicit `System.gc()` in production code is almost always a mistake and a red flag in code review.

## `GC overhead limit exceeded` vs regular `OutOfMemoryError`

| Error | Meaning |
|-------|---------|
| `OutOfMemoryError: Java heap space` | Heap is full — couldn't allocate a new object. Heap leak or `-Xmx` too low. |
| `OutOfMemoryError: GC overhead limit exceeded` | GC is running > 98% of the time but reclaiming < 2% of heap. JVM gives up. This usually means a slow memory leak — the heap isn't technically full yet, but it's almost useless. |
| `OutOfMemoryError: Metaspace` | Class metadata space exhausted — too many loaded classes. |

## Interview one-liner

> "`System.gc()` is only a hint — the JVM can ignore it. It is not guaranteed to run immediately or at all. In production, `-XX:+DisableExplicitGC` is often set to neutralise any explicit GC calls. Real GC is triggered by the JVM internally: minor GC when Eden fills, major/full GC when old gen fills. Calling `System.gc()` in application code is almost always wrong — tune heap sizing and GC thresholds instead."
