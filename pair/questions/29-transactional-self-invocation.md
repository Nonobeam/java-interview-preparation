# Q29 — @Transactional Self-Invocation Pitfall

```java
@Service
public class OrderService {

    @Transactional
    public void placeOrder(Order o) {
        validate(o);
        persist(o);
        sendConfirmation(o);     // async email send, calls notifyAsync
        this.audit(o);           // <-- self-invocation
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void audit(Order o) {
        auditRepo.save(AuditEntry.of("ORDER_PLACED", o));
    }

    @Async
    public void notifyAsync(Order o) {
        mailer.send(o);
    }
}
```

A PM complains: "when `placeOrder` fails after `audit`, the audit row is gone too — I thought `REQUIRES_NEW` guaranteed it would stick?"

## Questions

1. Explain, mechanically, why `this.audit(o)` does not start a new transaction. Walk through what the Spring proxy actually does.
2. The same bug affects `@Async`, `@Cacheable`, `@PreAuthorize`, `@Retryable`. What is the single underlying mechanism they all share?
3. List the ways to fix self-invocation. Rank them by preference and explain why.
4. Spring offers `AopContext.currentProxy()` and also the `@EnableAspectJAutoProxy(exposeProxy = true)` flag. What problem does that solve, and why is it still not the preferred fix?
5. When would you switch from Spring AOP (proxy-based) to AspectJ compile-time or load-time weaving to sidestep this entirely? What's the cost?
