# 63. JVM memory layout & garbage collection

## JVM memory regions

```
┌──────────────────────────────────────────────────────┐
│ Heap (shared across threads)                         │
│   ┌─────────────────────┐  ┌──────────────────────┐  │
│   │ Young generation    │  │ Old generation       │  │
│   │  Eden | S0 | S1     │  │ (long-lived objects) │  │
│   └─────────────────────┘  └──────────────────────┘  │
└──────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────┐
│ Metaspace (class metadata) — native memory           │
└──────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────┐
│ Stack (one per thread)                               │
│   Frames: local vars, operand stack, return addr     │
└──────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────┐
│ PC register (per thread) | Native method stack       │
└──────────────────────────────────────────────────────┘
```

### Heap
- Where **all objects** and arrays live.
- Shared across threads → not thread-safe by itself.
- Garbage-collected.
- Tuned with `-Xms` (initial), `-Xmx` (max).

### Stack
- One per thread.
- Holds **stack frames**: each method call pushes a frame containing local variables, operand stack, and the return address.
- Local primitives live here directly; local object references live here but **point to** heap objects.
- Bounded — recurse too deep → `StackOverflowError`.
- Tuned with `-Xss` per thread.

### Metaspace (Java 8+; replaced PermGen)
- Class metadata: bytecode of loaded classes, method info, static fields, runtime constant pool.
- In **native memory** (off-heap), unbounded by default. Tuned with `-XX:MaxMetaspaceSize`.

### Other regions
- **PC register**: per-thread current instruction pointer.
- **Native method stack**: for JNI calls.

## Where does what live?

| Item                       | Where |
|----------------------------|-------|
| `int x = 5;` inside method | Stack (the value) |
| `Object o = new Object();` | Reference on stack, object on heap |
| `static int counter;`      | Metaspace (the variable slot); the int value lives there too |
| `private String name;` (instance field) | On the heap, inside the object |
| String literal `"hi"`      | Heap (String pool, since Java 7) |

## Garbage collection — the big idea

1. The GC starts from **GC roots** (active stack frame locals, static fields, JNI handles, thread objects).
2. It marks every object **reachable** from a root.
3. Anything unmarked is **garbage** and gets reclaimed.

You don't `free()` in Java — you just stop holding references and the GC eventually collects.

## Generational hypothesis

> Most objects die young; a few live forever.

So the heap is split into **young** (Eden + 2 survivors) and **old**.

### Young-gen collection (Minor GC) — fast, frequent
1. New objects allocated in **Eden**.
2. Eden fills → minor GC: live objects copied into a survivor space (`S0` or `S1`); Eden is wiped.
3. Each minor GC bumps the survival count. After N survivals, the object is **promoted** to the old generation.

### Old-gen collection (Major / Full GC) — slow, rare
- Triggered when old gen fills up.
- Touches the whole heap (or large parts), so it's the main cause of long pauses.

### Stop-the-world
Most GC phases pause **all** application threads. Modern collectors (G1, ZGC, Shenandoah) shrink these pauses but can't fully eliminate them.

## Common collectors

| Collector  | Default in   | Goal                              |
|------------|--------------|-----------------------------------|
| Parallel   | Java 8       | Throughput (batch jobs)           |
| **G1**     | **Java 9+**  | Balanced, predictable pause times |
| ZGC / Shenandoah | Java 11+ (production-ready Java 15+) | Sub-ms pauses on huge heaps |

`-XX:+UseG1GC` etc. to choose explicitly.

## OutOfMemoryError flavors

- `Java heap space` — too many live objects on the heap. Heap leak or too small `-Xmx`.
- `Metaspace` — loading too many classes (common with reflection, dynamic proxies, classloader leaks in app servers).
- `unable to create new native thread` — OS thread limit hit.
- `GC overhead limit exceeded` — GC running constantly but reclaiming almost nothing.

## Quick debugging cues
- Heap usage growing over time, never falling back → heap leak. Capture a heap dump (`jmap -dump` or `-XX:+HeapDumpOnOutOfMemoryError`) and analyze in Eclipse MAT.
- Long Full GC pauses → check old-gen sizing or switch to G1/ZGC.
- StackOverflowError → unbounded recursion or too-deep call chain (rare in normal services).

## Interview one-liner

> "Heap holds all objects and is GC-managed; each thread has its own stack for frames and primitives. The young/old generational split exploits the fact that most objects die young — minor GCs are fast, full GCs are rare and pause-heavy. Modern G1 or ZGC keeps pauses predictable even on large heaps."
