# 49. Spring AOP

AOP (Aspect-Oriented Programming) separates cross-cutting concerns — logging, security, transactions, metrics — from business logic. Spring implements it via **proxies**, not bytecode weaving (unless you use AspectJ load-time weaving).

---

## Core vocabulary

| Term | Meaning |
|---|---|
| **Aspect** | The class that contains cross-cutting logic (e.g. `LoggingAspect`) |
| **Advice** | The actual code that runs — *when* and *what* to do |
| **JoinPoint** | A point in execution where advice can be applied (in Spring: always a method call) |
| **Pointcut** | An expression that selects which JoinPoints to intercept |
| **Weaving** | The process of linking aspects to the target object |
| **Proxy** | The wrapper Spring creates around the target bean |

---

## Types of advice

```java
@Aspect
@Component
public class LoggingAspect {

    // Runs BEFORE the method
    @Before("execution(* com.example.service.*.*(..))")
    public void logBefore(JoinPoint jp) {
        System.out.println("Calling: " + jp.getSignature().getName());
    }

    // Runs AFTER the method (regardless of outcome)
    @After("execution(* com.example.service.*.*(..))")
    public void logAfter(JoinPoint jp) {
        System.out.println("Done: " + jp.getSignature().getName());
    }

    // Runs only on successful return
    @AfterReturning(pointcut = "execution(* com.example.service.*.*(..))",
                    returning  = "result")
    public void logReturn(JoinPoint jp, Object result) {
        System.out.println("Returned: " + result);
    }

    // Runs only when an exception is thrown
    @AfterThrowing(pointcut = "execution(* com.example.service.*.*(..))",
                   throwing  = "ex")
    public void logException(JoinPoint jp, Exception ex) {
        System.out.println("Exception in " + jp.getSignature() + ": " + ex.getMessage());
    }

    // Wraps the method — most powerful, can skip execution or modify return value
    @Around("execution(* com.example.service.*.*(..))")
    public Object logAround(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = pjp.proceed();           // invoke the real method
        long elapsed = System.currentTimeMillis() - start;
        System.out.println(pjp.getSignature() + " took " + elapsed + "ms");
        return result;
    }
}
```

---

## Pointcut expression syntax

```java
execution([visibility] returnType [class].method([params]) [throws])

// All methods in a package
"execution(* com.example.service.*.*(..))"

// Specific method
"execution(public void com.example.service.OrderService.placeOrder(Long, int))"

// Any method annotated with @Transactional
"@annotation(org.springframework.transaction.annotation.Transactional)"

// Reusable pointcut
@Pointcut("execution(* com.example.service.*.*(..))")
public void serviceLayer() {}

@Before("serviceLayer()")
public void logBefore(JoinPoint jp) { ... }
```

---

## How Spring AOP works (proxy mechanism)

```
Bean A calls Bean B method
         ↓
Spring sees AOP pointcut matches
         ↓
Spring wraps Bean B in a proxy (JDK dynamic proxy or CGLIB)
         ↓
Call → proxy → advice → real method → advice → caller
```

- **JDK dynamic proxy** — used when the bean implements an interface
- **CGLIB proxy** — used when the bean is a plain class (subclasses it at runtime)

---

## Self-invocation limitation (critical!)

```java
@Service
public class OrderService {

    public void outer() {
        this.inner();   // ← calls through 'this', bypasses the proxy!
    }

    @Transactional
    public void inner() { ... }   // @Transactional will NOT apply here
}
```

When `outer()` calls `inner()` via `this`, the call goes directly to the target object — not through the proxy — so no advice runs. Same issue affects `@Transactional`, `@Async`, `@Cacheable`. (See Q29.)

---

## Real-world uses in Spring

| Feature | AOP mechanism |
|---|---|
| `@Transactional` | `TransactionInterceptor` via `BeanPostProcessor` |
| `@Cacheable` | `CacheInterceptor` |
| `@Async` | `AsyncExecutionInterceptor` |
| `@Secured` / `@PreAuthorize` | Spring Security method interceptor |

---

## Interview one-liner
> "Spring AOP intercepts method calls by wrapping beans in proxies. You define an Aspect with Advice (Before/After/Around) and a Pointcut expression to select which methods to intercept. Because it's proxy-based, internal `this.method()` calls bypass AOP — which is why `@Transactional` self-invocation doesn't work."
