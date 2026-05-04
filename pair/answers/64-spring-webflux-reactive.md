# 68. Spring WebFlux & reactive basics

## The core idea

**Reactive = non-blocking, asynchronous, backpressure-aware** stream processing.

- **Spring MVC** uses the servlet model: 1 request → 1 thread, blocked while waiting on I/O. With ~200 threads in the Tomcat pool, you can serve ~200 concurrent slow requests before queuing.
- **Spring WebFlux** is built on Project Reactor + Netty. A small event-loop thread pool (one per CPU core) handles thousands of concurrent connections — threads are never blocked waiting for I/O.

Trade-off: WebFlux only delivers if the **whole stack is non-blocking**. One blocking JDBC call inside a reactive handler ruins the model.

## `Mono` vs `Flux`

Reactor's two publisher types:

| Type     | Emits                       | HTTP analog                |
|----------|-----------------------------|----------------------------|
| `Mono<T>`| 0 or 1 item, then complete  | Single response, optional  |
| `Flux<T>`| 0 to N items, then complete | Stream / collection / SSE  |

```java
Mono<User> findById(Long id);             // one user (or empty)
Flux<Order> streamOrders(Long userId);    // many orders over time
```

Both are **lazy** — nothing runs until something subscribes. In Spring WebFlux, the framework subscribes for you when the handler returns.

## A reactive controller

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    private final ReactiveUserRepository repo;   // R2DBC or reactive Mongo

    @GetMapping("/{id}")
    public Mono<User> get(@PathVariable Long id) {
        return repo.findById(id)
            .switchIfEmpty(Mono.error(new EntityNotFoundException("user " + id)));
    }

    @GetMapping(produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<User> stream() {
        return repo.findAll().delayElements(Duration.ofSeconds(1));
    }

    @PostMapping
    public Mono<User> create(@RequestBody Mono<CreateUser> body) {
        return body.flatMap(repo::save);
    }
}
```

## Reactor operators (the basics)

```java
Mono.just("alice")
    .map(String::toUpperCase)              // sync transform: "ALICE"
    .flatMap(name -> repo.findByName(name)) // async transform → Mono<User>
    .filter(User::isActive)
    .switchIfEmpty(Mono.error(new NotFoundException()))
    .doOnNext(u -> log.info("found {}", u))
    .onErrorResume(e -> fallback(e));
```

- `map` — sync 1:1 transform.
- `flatMap` — async 1:1, when the transform returns `Mono`/`Flux`.
- `filter`, `take`, `skip`, `distinct` — same as Stream.
- `zip`, `merge`, `concat` — combine publishers.
- `onErrorResume`, `retry`, `timeout` — error / resilience.

## Backpressure

If a producer emits faster than the consumer can handle, Reactor signals upstream to slow down (`request(n)` semantics). This is the part that protects WebFlux services from being overwhelmed by fast upstream sources.

## When to choose WebFlux over MVC

**Use WebFlux when:**
- High-concurrency I/O-bound workloads (API gateway, fan-out aggregator, SSE / WebSockets).
- Streaming responses or pub/sub.
- The whole stack is non-blocking: reactive driver (R2DBC, reactive Mongo, reactive Redis), `WebClient` for outbound HTTP.

**Stick with MVC when:**
- You depend on JDBC (still the universe of DB drivers).
- Your team isn't fluent in reactive — debugging, stack traces, and operator chains have a real learning cost.
- Throughput is fine on a thread-per-request model. Most CRUD apps are.

## Common traps

- **Calling blocking JDBC inside `Mono.flatMap`** — pins the event-loop thread, defeating WebFlux entirely. If unavoidable, isolate it with `subscribeOn(Schedulers.boundedElastic())`.
- **Forgetting to subscribe** — outside Spring, a Mono with no subscriber does nothing. (Inside controllers Spring subscribes for you.)
- **Mixing Mono with imperative code** — `.block()` is forbidden on the event loop; only safe in tests or hard CLI boundaries.

## Virtual threads (Java 21+) — alternative direction

`Thread.startVirtualThread(...)` and Tomcat's virtual-thread executor make blocking-JDBC-but-cheap-threads viable again. For many teams, **MVC + virtual threads** in Java 21 hits the same scaling sweet spot WebFlux promised, with simpler code.

## Interview one-liner

> "WebFlux is Spring's non-blocking reactive stack on Netty + Project Reactor. `Mono<T>` is 0–1 items, `Flux<T>` is 0–N — both lazy and backpressure-aware. Use it for high-concurrency I/O-bound work where the entire chain is non-blocking. For typical CRUD-on-JDBC, MVC (especially MVC + Java 21 virtual threads) is simpler and just as fast."
