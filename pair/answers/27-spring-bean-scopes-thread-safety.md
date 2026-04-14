# A27 — Spring Bean Scopes & Thread Safety

## 1. Default scope & the bug

**Default scope is `singleton`** — one instance per Spring container, shared across all threads.

`RequestTracker` has mutable instance fields (`currentUserId`, `requestStartTime`) but is a singleton. Under concurrent traffic, request threads race on the same instance.

**Concrete scenario:** Thread A calls `tracker.start("alice")` at t=0. Thread B calls `tracker.start("bob")` at t=10ms. Thread A then calls `tracker.getUserId()` and gets `"bob"`. Thread A logs `"user bob took 5ms"` — wrong user, wrong duration. Worse, `userService.find()` is called with the wrong userId, so Alice's HTTP response may contain Bob's profile. This is a **data-leak bug**, not just a logging glitch.

`UserController` itself is also a singleton, which is fine because apart from its (supposedly immutable) collaborators it has no mutable state. Singletons must be stateless or internally synchronized.

## 2. The five standard scopes

1. **`singleton`** (default) — one per container
2. **`prototype`** — new instance per injection/lookup; Spring does not manage its full lifecycle (no destruction callbacks)
3. **`request`** — one per HTTP request (web-aware)
4. **`session`** — one per HTTP session (web-aware)
5. **`application`** — one per `ServletContext` (web-aware)

(Spring also has `websocket` and supports custom scopes, but the above five are the standard set.)

**Change `RequestTracker` to `request` scope.** Each HTTP request gets its own instance, so the fields are naturally isolated per thread. No synchronization needed, and Spring calls destruction callbacks at request end.

## 3. Why injecting a request bean into a singleton fails, and how to fix it

**Why it fails:** `UserController` is a singleton, instantiated and wired once at container startup. Spring tries to resolve `requestTracker` at that moment — but there's no active HTTP request at startup, so the `request`-scoped bean cannot be created. You get a `BeanCreationException` / `ScopeNotActiveException`. Even if you bypassed that, injecting a short-lived request bean directly would permanently bind the singleton controller to one request's tracker — defeating the point.

**Fix — inject a scoped proxy, not the real bean.** The proxy is itself a singleton; on each method call it delegates to the real `RequestTracker` for the current request, looked up via `RequestContextHolder`.

**XML way** — `aop:scoped-proxy`:
```xml
<bean id="requestTracker" class="com.acme.RequestTracker" scope="request">
    <aop:scoped-proxy/>
</bean>
```

**Annotation way:**
```java
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestTracker { ... }
```

Use `TARGET_CLASS` for CGLIB proxies on concrete classes; `INTERFACES` if you inject through an interface type.

## 4. `ThreadLocal` as an alternative

Technically valid: put the state in a `ThreadLocal<TrackerState>` inside a singleton, and it isolates per thread.

**Downsides vs. request scope:**

- **Memory leaks in thread pools.** Servlet containers pool threads, so a `ThreadLocal` not explicitly `remove()`d at end of request lingers on the pooled thread and is visible to the next request on that thread. You need a `Filter`/`HandlerInterceptor` with a `finally { threadLocal.remove(); }` block — easy to forget, impossible to enforce at compile time. Classloader leaks on redeploy too.
- **Doesn't survive async boundaries.** If the request dispatches to a `@Async` method, a worker pool, a `CompletableFuture.supplyAsync`, or a reactive pipeline, the `ThreadLocal` is lost. Request scope integrates with Spring's `RequestContextHolder` and can be propagated (e.g. via `RequestContextFilter` with `threadContextInheritable=true`, or explicit context capture).
- **Manual lifecycle.** You must explicitly initialize and clean up. Request scope does this for free — Spring creates the bean on first access within the request and destroys it (calling `@PreDestroy`) when the request ends.
- **Testing pain.** Swapping implementations for tests is harder — `ThreadLocal` is static-flavoured state, whereas a request-scoped bean is just a bean and can be mocked/overridden normally.
- **Visibility in the type system.** `@Scope("request")` declares intent at the bean definition. A `ThreadLocal` buried in a field hides the per-request semantics from anyone reading the class.

**Rule of thumb:** `ThreadLocal` is the right tool for framework-level infrastructure (Spring's own `TransactionSynchronizationManager`, SLF4J `MDC`, security context propagation). For application-level per-request state, use request scope with a scoped proxy.
