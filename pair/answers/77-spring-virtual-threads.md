# 77. Spring Virtual Threads (Project Loom)

## Platform thread vs virtual thread

| | Platform thread (OS thread) | Virtual thread |
|---|---|---|
| Created by | JVM maps 1:1 to an OS thread | JVM — not mapped to a dedicated OS thread |
| Cost | ~1–2 MB stack, expensive to create | ~few KB, very cheap |
| Typical max per JVM | ~thousands (OS limit) | **millions** |
| Managed by | OS scheduler | JVM scheduler (mounts to carrier threads) |
| Available since | Always | Java 21 (GA) |

## The problem virtual threads solve

In traditional Spring MVC:

```
Request arrives → assigned to a Tomcat thread → thread calls DB → thread BLOCKS (waiting for I/O)
                                                                              ↑
                                                              Thread is parked but alive,
                                                              consuming 1MB of stack,
                                                              doing nothing useful
```

With 200 Tomcat threads and 100ms DB queries, you can handle ~2,000 requests/sec max. The rest queue up.

Virtual threads solve this: when a virtual thread blocks on I/O, the JVM **unmounts** it from the carrier thread. The carrier thread picks up another virtual thread to run. The blocked virtual thread is parked with only its stack (~KB), not holding an OS thread.

## How cheap are virtual threads?

```java
// This is fine with virtual threads — would OOM with platform threads
for (int i = 0; i < 1_000_000; i++) {
    Thread.ofVirtual().start(() -> {
        Thread.sleep(Duration.ofSeconds(1));
    });
}
```

A million blocking virtual threads use ~few GB (stack per thread is tiny). A million platform threads would need ~1 TB.

## Enabling virtual threads in Spring Boot 3.2+

One property in `application.yml`:

```yaml
spring:
  threads:
    virtual:
      enabled: true
```

That's it. Spring Boot replaces Tomcat's executor with a virtual-thread executor. Every HTTP request now runs on a virtual thread.

Or manually:

```java
@Bean
TomcatProtocolHandlerCustomizer<?> virtualThreadCustomizer() {
    return handler -> handler.setExecutor(Executors.newVirtualThreadPerTaskExecutor());
}
```

## Pinning — the main gotcha

**Pinning** = a virtual thread is stuck to its carrier thread and cannot be unmounted, even when blocking.

This defeats the purpose — the carrier thread is occupied the whole time.

### When pinning happens

1. **`synchronized` blocks or methods** — the JVM cannot unmount inside a `synchronized` block.
2. **Native method calls (JNI)** — native frames prevent unmounting.

```java
// This causes pinning:
synchronized (lock) {
    Thread.sleep(1000);  // virtual thread is pinned for 1 second
}

// Fix: use java.util.concurrent.locks.ReentrantLock instead:
lock.lock();
try {
    Thread.sleep(1000);  // virtual thread can unmount here
} finally {
    lock.unlock();
}
```

**Common real-world pinning source:** JDBC drivers using `synchronized` internally (e.g., older versions of PostgreSQL JDBC driver). Check with `-Djdk.tracePinnedThreads=full`.

## Virtual threads vs reactive (WebFlux)

| | Virtual threads | Spring WebFlux (Reactor) |
|---|---|---|
| Programming model | Blocking (looks like normal code) | Non-blocking (Mono/Flux, functional chains) |
| Learning curve | Low — same as traditional Spring MVC | High — reactive operators, backpressure, debugging is harder |
| When to use | Most services doing I/O-heavy work | Truly event-driven, streaming, complex async pipelines |
| Works with JPA/JDBC? | Yes — blocking drivers work fine | Needs R2DBC (reactive DB driver) |

**For a new project doing standard CRUD + DB + REST calls:** virtual threads are the right choice. You get the scalability benefit with familiar blocking code.

WebFlux shines for: streaming responses, WebSocket, or when you need fine-grained backpressure control.

## Does your code change?

Mostly **no**. Virtual threads are transparent:
- `@Async`, `ExecutorService`, `CompletableFuture` all still work.
- The main thing to change: replace `synchronized` blocks with `ReentrantLock` in hot paths.
- JDBC drivers may need updating.
- No reactive operators, no `Mono`/`Flux` — you write standard blocking code.

```java
// Before virtual threads — this blocks the platform thread:
@GetMapping("/orders")
public List<Order> getOrders() {
    return orderRepo.findAll();   // JDBC call, blocks
}

// After enabling virtual threads — SAME CODE, but now the virtual thread
// unmounts while waiting for the DB, and the carrier thread serves others:
@GetMapping("/orders")
public List<Order> getOrders() {
    return orderRepo.findAll();   // identical code, much better throughput
}
```

## Interview one-liner

> "Virtual threads (Project Loom, GA in Java 21) are JVM-managed lightweight threads — millions can run simultaneously. When a virtual thread blocks on I/O, the JVM unmounts it from the carrier OS thread, freeing that carrier to run another virtual thread. In Spring Boot 3.2+ you enable them with one property. The main gotcha is pinning: `synchronized` blocks and JNI prevent unmounting, so replace `synchronized` with `ReentrantLock` in blocking hot paths. For most services, virtual threads give you reactive-level scalability with plain blocking code, making WebFlux unnecessary unless you need streaming or backpressure."
