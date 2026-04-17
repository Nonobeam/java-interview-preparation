# 50. Circuit Breaker Pattern

Prevents cascading failures when a downstream service is slow or unavailable. Inspired by an electrical circuit breaker: trip it to stop the damage, then reset when things recover.

---

## The problem it solves

```
Service A → Service B (payment gateway — down or slow)
```

Without a circuit breaker:
- Every request to A blocks waiting for B to timeout
- Thread pool fills up
- Service A also becomes unresponsive
- Cascading failure takes down the whole system

---

## States

```
         failure threshold reached
CLOSED ─────────────────────────→ OPEN
  ↑                                 │
  │       half-open timeout         │
  └──── HALF-OPEN ←─────────────────┘
              │
    success? → CLOSED
    failure? → OPEN
```

| State | Behavior |
|---|---|
| **Closed** | Normal — requests flow through; failures counted |
| **Open** | Short-circuit — requests fail immediately (fallback), no calls to downstream |
| **Half-Open** | Trial — lets a few requests through to test recovery |

---

## Resilience4j in Spring Boot

```groovy
// build.gradle
implementation 'io.github.resilience4j:resilience4j-spring-boot3:2.2.0'
implementation 'org.springframework.boot:spring-boot-starter-aop'
```

```yaml
# application.yml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        slidingWindowSize: 10           # last 10 calls counted
        failureRateThreshold: 50        # trip if 50%+ fail
        waitDurationInOpenState: 10s    # stay OPEN for 10s before going HALF-OPEN
        permittedNumberOfCallsInHalfOpenState: 3
        minimumNumberOfCalls: 5         # don't trip until at least 5 calls recorded
```

```java
@Service
public class OrderService {

    @CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
    public PaymentResult processPayment(Long orderId, BigDecimal amount) {
        return paymentClient.charge(orderId, amount);   // remote call
    }

    // Fallback — same signature + Throwable
    public PaymentResult paymentFallback(Long orderId, BigDecimal amount, Throwable ex) {
        log.warn("Payment service unavailable for order {}: {}", orderId, ex.getMessage());
        return PaymentResult.pending("QUEUED_FOR_RETRY");
    }
}
```

---

## Combining with Retry and TimeLimiter

```yaml
resilience4j:
  retry:
    instances:
      paymentService:
        maxAttempts: 3
        waitDuration: 500ms
        retryExceptions:
          - java.net.ConnectException
  timelimiter:
    instances:
      paymentService:
        timeoutDuration: 2s
```

```java
@TimeLimiter(name = "paymentService")
@Retry(name = "paymentService")
@CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
public CompletableFuture<PaymentResult> processPayment(...) {
    return CompletableFuture.supplyAsync(() -> paymentClient.charge(...));
}
```

Order of decorators (inner to outer): `CircuitBreaker → Retry → TimeLimiter`.

---

## Monitoring

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health, circuitbreakers
  health:
    circuitbreakers:
      enabled: true
```

`GET /actuator/health` shows the state of each circuit breaker.

---

## When to use

- Calling any **external/remote service** (payment, SMS, third-party API)
- Service B has **higher latency or lower reliability** than acceptable
- You can define a **meaningful fallback** (cached response, queue for retry, degraded mode)

Do not use for internal in-memory calls — the overhead is not worth it.

---

## Interview one-liner
> "A circuit breaker wraps remote calls and trips OPEN when the failure rate exceeds a threshold, returning a fallback immediately instead of waiting for timeouts. After a cooldown it goes HALF-OPEN to test recovery. In Spring Boot, Resilience4j implements this with `@CircuitBreaker` — preventing cascading failures when a downstream service is degraded."
