# A28 — Java Memory Model: happens-before, volatile, synchronized

## 1. Two concrete bugs the request thread can observe

**Bug A — stale `ready` flag.** The JMM permits the request thread's CPU cache to hold `ready == false` forever. There is no happens-before edge forcing the loader thread's write of `ready = true` to become visible to other threads. `get()` throws `IllegalStateException` long after the loader finished.

**Bug B — partial publication / reorderings.** Even if the request thread sees `ready == true`, the JMM allows the compiler/CPU to reorder the writes inside `load()`. The request thread may observe:
- `ready = true` before `config` is assigned → `config.get(key)` throws `NullPointerException`.
- `config` reference visible but the `HashMap`'s internal arrays not fully initialised → reads return `null` for a key that is clearly present, or the iterator sees a corrupt bucket array and loops forever (this has actually happened in production — pre-Java 8 HashMap under concurrent publication famously caused CPU-pegging infinite loops in `get`).
- On older CPUs you can even see the entries `put` into `m` but not yet flushed — `m.put("region", ...)` is sequenced before `this.config = m`, but without a fence the puts can be retired after the reference assignment.

The issue is not "races on the map" (only one writer) — it's **unsafe publication**.

## 2. happens-before

*A happens-before B* means every memory write ordered before A is guaranteed visible to a read ordered after B, and the JIT/CPU is forbidden from reordering across the edge.

Inter-thread edges that create happens-before:

1. **Program order** within a single thread.
2. **Monitor lock**: unlock of monitor `M` happens-before every subsequent lock of `M`.
3. **Volatile**: write to volatile `v` happens-before every subsequent read of `v`.
4. **Thread start**: `Thread.start()` in the caller happens-before any action in the started thread.
5. **Thread join**: all actions in thread `T` happen-before `T.join()` returns.
6. **Final fields**: a correctly constructed object's final-field writes happen-before any thread reads the reference (after construction completes).
7. **Concurrency utilities**: releasing a semaphore permit, putting into a `BlockingQueue`, `CountDownLatch.countDown`, `CompletableFuture.complete`, etc. — all documented as happens-before the matching acquire.
8. **Transitivity**: if A hb B and B hb C, then A hb C.

## 3. Three fixes

### (a) `volatile`

```java
private volatile Map<String, String> config;
public void load() {
    Map<String, String> m = new HashMap<>();
    m.put("region", "eu-west-1");
    this.config = m;   // volatile write publishes everything written before it
}
public String get(String k) {
    Map<String, String> c = config;   // volatile read
    if (c == null) throw new IllegalStateException("not ready");
    return c.get(k);
}
```

Trade-off: cheapest (no locks). `ready` flag disappears — use `config == null` as the readiness signal. You must not mutate the map after publication; treat it as immutable.

### (b) `synchronized`

```java
public synchronized void load() { ... }
public synchronized String get(String k) { ... }
```

Trade-off: correct, but every read takes a monitor — serialises all readers. Use a `ReadWriteLock` if read-heavy, or prefer (a) / (c).

### (c) `final` + safe publication via constructor

```java
public final class ConfigCache {
    private final Map<String, String> config;   // final

    public ConfigCache(Map<String, String> src) {
        this.config = Map.copyOf(src);          // immutable snapshot
    }
    public String get(String k) { return config.get(k); }
}
```

Once the constructor finishes, the `final` field is safely publishable to any thread that obtains the reference — even via a non-volatile field. Trade-off: the cache is immutable, so reloading means swapping the whole instance (e.g. store the instance behind a `volatile ConfigCache current`). Best option when config is rarely refreshed.

## 4. `volatile` vs `AtomicReference` / CAS

- `volatile` guarantees **visibility and ordering** but not atomicity of compound operations. Fine for *publish once, read many* or *single-writer* updates.
- Use `AtomicReference` (CAS) when multiple writers race and you need `compareAndSet` — e.g. lock-free stacks, copy-on-write reloads where a loser must retry. Under the hood, CAS provides the same happens-before as a volatile write plus atomic check-and-swap.
- `volatile long`/`double` are atomic for read/write on all modern JVMs, but `x++` is still three steps — use `AtomicLong` / `LongAdder` (the latter scales better under contention).

## 5. `HashMap` vs `ConcurrentHashMap` after volatile publication

- **`HashMap` via `volatile` reference, never mutated after publication** — safe. The volatile write establishes happens-before, so the map's internal state (arrays, entries) is fully visible. Many production caches rely on this. `Collections.unmodifiableMap(new HashMap<>(...))` or `Map.copyOf` reinforces the immutability contract.
- **`HashMap` mutated after publication** — unsafe, full stop. Readers can see torn resizes, `NullPointerException`, or infinite loops. The volatile reference only covers the initial publication, not subsequent writes.
- **`ConcurrentHashMap`** — safe to mutate concurrently by design; internal synchronisation establishes happens-before for each put/get. Use it when the map is live-updated, not when it's write-once config.

## Interview-ready answer

> "Without a happens-before edge, a reader can see `ready=true` before the map reference is assigned, or see the reference but a half-built HashMap — classic unsafe publication. The JMM gives you several ways to create that edge: volatile write/read, monitor unlock/lock, `Thread.start/join`, and final-field semantics. For a load-once config, I'd publish via a volatile reference to an immutable `Map.copyOf` snapshot — cheapest, and treating the map as immutable after publication removes the need for any further synchronisation. `synchronized` works but serialises readers; `AtomicReference` is only needed when multiple threads race to update."
