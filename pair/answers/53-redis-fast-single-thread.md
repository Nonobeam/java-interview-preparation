# 57. Why is Redis Fast, and How Does It Handle Concurrency with a Single Thread?

Redis routinely handles **100k–1M ops/second** on a single instance. The reasons come from its architecture, not hardware.

---

## Reason 1: Everything lives in memory

No disk I/O on the hot path. Memory access is ~100 ns vs ~10 ms for SSD. Operations like `GET`, `SET`, `INCR` are pure memory reads/writes.

---

## Reason 2: Simple, efficient data structures

Redis doesn't use a general-purpose storage engine. Each data type (`String`, `Hash`, `List`, `Set`, `ZSet`) is implemented with a hand-tuned C structure:

- `String` → raw buffer or integer encoding
- `Hash` → ziplist (compact) or hashtable depending on size
- `ZSet` → skiplist + hashtable (O(log N) range queries)

No ORM overhead, no row parsing, no schema validation.

---

## Reason 3: Single-threaded command execution

Redis uses **one thread** to execute all commands. This sounds slow — it's actually a design advantage:

- **Zero lock contention** — no mutex, no deadlock, no context switch between threads
- **Atomic by default** — every Redis command is atomic; no `COMPARE-AND-SWAP` needed
- **Predictable latency** — one command runs to completion before the next starts

```
Thread 1: GET key1   ──────────►
Thread 2: SET key2           ──────────►   (queued, not parallel)
```

Operations are serialized, but each one is so fast (microseconds) that queuing is not the bottleneck.

---

## Reason 4: I/O multiplexing (the event loop)

The single thread handles **many network connections simultaneously** using `epoll` (Linux) / `kqueue` (macOS):

```
epoll monitors 10,000 sockets
  → socket A has data   → read command → execute → write response
  → socket B has data   → read command → execute → write response
  → ... (no thread-per-connection overhead)
```

This is the same model as Nginx. The event loop picks up all ready sockets in one system call, avoiding the cost of spawning/scheduling threads.

---

## Redis 6+ — Threaded I/O (but not command execution)

Redis 6 introduced **I/O threads** for reading/writing network data in parallel, while command execution remains single-threaded:

```
Network I/O threads (multiple):   read bytes → parse command
Command execution thread (one):   execute command → produce result
Network I/O threads (multiple):   serialize → write bytes to socket
```

This removes the I/O bottleneck for high-throughput workloads (millions of small requests) without sacrificing the atomicity guarantee.

---

## Summary: why it's fast

| Factor | Why it helps |
|---|---|
| In-memory | No disk I/O |
| Efficient data structures | Minimal CPU per operation |
| Single-threaded commands | No lock overhead |
| epoll event loop | Handles thousands of connections without thread overhead |
| Threaded I/O (Redis 6+) | Parallelizes network read/write |

---

## Interview one-liner
> "Redis is fast because it stores everything in memory, uses efficient hand-tuned data structures, and executes commands in a single thread — which eliminates lock contention and makes every command inherently atomic. It handles many concurrent clients through an epoll-based event loop (I/O multiplexing), not one-thread-per-connection. Redis 6 added threaded network I/O while keeping command execution single-threaded."
