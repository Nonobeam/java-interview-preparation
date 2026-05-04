# A29 — @Transactional Self-Invocation Pitfall

## 1. Why `this.audit(o)` silently ignores `REQUIRES_NEW`

Spring AOP is **proxy-based**. When you inject `OrderService`, what the caller actually holds is a proxy object (JDK dynamic proxy if the bean implements an interface, CGLIB subclass otherwise). The proxy wraps every public method with the transactional interceptor:

```
caller --> proxy.placeOrder() --> TransactionInterceptor
                                    .invoke() { beginTx; target.placeOrder(); commit/rollback }
                                  --> target.placeOrder()
                                        --> this.audit(o)   <-- direct call on `this`
```

Inside `placeOrder`, `this` refers to the **raw target bean**, not the proxy. So `this.audit(o)` is a plain Java method call — the `TransactionInterceptor` never runs, and `@Transactional(REQUIRES_NEW)` is simply not evaluated. The audit write joins the outer transaction and rolls back with it. This is why the PM sees the audit disappear: there is no second transaction.

Same for any annotation whose meaning is enforced by an AOP interceptor — `this.notifyAsync(o)` bypasses the async executor and runs on the caller thread.

## 2. The underlying mechanism

All of these rely on **proxy interception**: `@Transactional`, `@Async`, `@Cacheable`/`@CacheEvict`, `@PreAuthorize`/`@Secured`, `@Retryable`, `@Validated`, `@Observed`. Spring weaves behaviour by inserting an interceptor *between the caller and the target*. Self-calls do not pass through any interceptor because there is no proxy in the call path.

Rule of thumb: if the annotation requires Spring to "do something around" your method, self-invocation breaks it.

## 3. Fixes ranked

### (a) Extract the annotated method to another bean — **preferred**

```java
@Service @RequiredArgsConstructor
public class OrderService {
    private final AuditService audit;
    @Transactional
    public void placeOrder(Order o) { ...; audit.record(o); }
}

@Service
public class AuditService {
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void record(Order o) { ... }
}
```

Why preferred: the split reflects reality. "Audit must survive the main tx rollback" is a separate responsibility. The proxy chain is always traversed because the call crosses bean boundaries. This is also the only fix that plays well with testing (you can mock `AuditService`).

### (b) Inject self — via constructor parameter or `ObjectProvider`

```java
@Service
public class OrderService {
    private final OrderService self;   // <-- circular, needs a lazy proxy
    public OrderService(@Lazy OrderService self) { this.self = self; }

    @Transactional
    public void placeOrder(Order o) { ...; self.audit(o); }
}
```

Works, but it's a code smell: you're telling readers "I know this looks like I'm calling myself, but trust me it's really going through the proxy." Use only when splitting the class would be artificial (e.g. same entity, same repository, genuinely one concern).

### (c) `ApplicationContext.getBean(OrderService.class).audit(o)`

Same effect as (b) but worse: ties code to the context, harder to test, slower lookup. Avoid.

### (d) `AopContext.currentProxy()` — see question 4.

### (e) Switch to AspectJ weaving — see question 5.

## 4. `AopContext.currentProxy()` + `exposeProxy = true`

Spring can stash the current proxy in a ThreadLocal so self-calls can route through it:

```java
@EnableAspectJAutoProxy(exposeProxy = true)
// or <aop:aspectj-autoproxy expose-proxy="true"/>

@Transactional
public void placeOrder(Order o) {
    ...
    ((OrderService) AopContext.currentProxy()).audit(o);
}
```

**What it solves:** self-invocation without extracting a bean or injecting self.

**Why it's still not preferred:**
- Couples your code to Spring's AOP internals via a static call. Unit tests that instantiate `OrderService` directly crash with `IllegalStateException: Cannot find current proxy`.
- The cast is unchecked — refactors that rename the class don't catch it.
- The ThreadLocal adds a small cost on every intercepted call and breaks if someone dispatches work to another thread.
- Reads worse than `self.audit(o)` with no real benefit.

Use it only when you cannot restructure (e.g. legacy code, sealed class).

## 5. AspectJ weaving

Spring AOP is *proxy-based*; AspectJ is a true AOP compiler that **rewrites the bytecode of the target class itself** (either at compile time via `aspectjtools` / `ajc`, or at class-load time via the `-javaagent:aspectjweaver.jar` agent plus `<context:load-time-weaver/>` / `@EnableLoadTimeWeaving`).

After weaving, `this.audit(o)` *is* intercepted, because the interception is inlined into the method body — there is no separate proxy object. `private` methods and internal calls work transparently.

**Cost:**
- Build and runtime complexity: extra Maven/Gradle plugin (`aspectj-maven-plugin`) or a `-javaagent` flag in every JVM (dev, CI, prod, Docker, K8s sidecars). Getting it wrong means silent skipped weaving.
- Startup slowdown from class-load-time weaving.
- Stack traces and debuggers show synthetic `$AjcClosure` frames.
- Diverges from the default Spring experience — onboarding cost for new engineers.
- Some IDE tooling (hot reload, Spring Boot DevTools) doesn't cooperate cleanly.

**When it's worth it:** large legacy codebases with pervasive self-invocation that can't be restructured, or when you need to advise `final`/`private` methods, constructors, or field accesses that proxies cannot touch. In greenfield services, always prefer proxy AOP + clean method boundaries.

## Interview-ready answer

> "Spring's `@Transactional` works via a proxy that wraps your bean. When you call `this.audit()` inside the same class, you bypass the proxy — the call goes straight to the target instance, and the `TransactionInterceptor` never runs. That's why `REQUIRES_NEW` is ignored and the audit row rolls back with the outer transaction. Same mechanism affects `@Async`, `@Cacheable`, `@Retryable` — anything AOP-driven. The cleanest fix is to move the annotated method to another bean so the call crosses a proxy boundary; that also reflects the real responsibility split. Injecting `@Lazy OrderService self` works as a tactical fix. `AopContext.currentProxy()` exists but couples you to Spring internals. AspectJ weaving sidesteps the issue entirely by rewriting bytecode, but the build and ops cost is rarely worth it."
