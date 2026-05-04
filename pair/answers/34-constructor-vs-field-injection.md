# 39. Constructor Injection vs Field Injection

Spring supports three injection styles — **constructor**, **setter**, and **field**. This question is really about the first and the third, and why the Spring team officially recommends **constructor injection**.

---

## The three styles at a glance

### Field injection (discouraged)
```java
@Service
public class OrderService {
    @Autowired
    private PaymentGateway gateway;
    @Autowired
    private OrderRepository repo;
}
```

### Setter injection
```java
@Service
public class OrderService {
    private PaymentGateway gateway;

    @Autowired
    public void setGateway(PaymentGateway gateway) { this.gateway = gateway; }
}
```

### Constructor injection (recommended)
```java
@Service
public class OrderService {
    private final PaymentGateway gateway;
    private final OrderRepository repo;

    // @Autowired is optional on a single constructor since Spring 4.3
    public OrderService(PaymentGateway gateway, OrderRepository repo) {
        this.gateway = gateway;
        this.repo = repo;
    }
}
```
With Lombok:
```java
@Service
@RequiredArgsConstructor
public class OrderService {
    private final PaymentGateway gateway;
    private final OrderRepository repo;
}
```

---

## Side-by-side comparison

| Aspect | Constructor | Field |
|---|---|---|
| Immutability (`final` fields) | ✅ yes | ❌ no — fields set via reflection after construction |
| Required dependencies enforced at compile time | ✅ can't construct without them | ❌ can forget one; fails at startup (or later) |
| Unit testing without Spring | ✅ `new OrderService(mock1, mock2)` | ❌ need reflection or a Spring context to set fields |
| Detects circular dependencies | ✅ fails fast at startup | ❌ may hide them, or cause subtle bugs |
| Reveals "too many dependencies" smell | ✅ giant constructor is a clear code smell | ❌ fields hide the smell |
| Works with frameworks outside Spring (e.g., CDI) | ✅ plain Java | ❌ depends on the DI framework |
| Brevity | ⚠️ more boilerplate (solve with Lombok) | ✅ shortest |
| Inheritance friendly | ✅ explicit via `super(...)` | ⚠️ subclasses don't see the injection |

---

## Why constructor injection is recommended
Spring's own reference docs and most architects (including Pivotal/Spring team engineers) recommend **constructor injection** for four reasons:

1. **Immutability.** Fields can be `final` → object is fully initialized and safe to publish to other threads. No "half-built" instance.
2. **Mandatory dependencies are explicit.** If something is required to do the job, it belongs in the constructor. The class **cannot exist** in an invalid state.
3. **Testable without the container.** A plain `new` call with mocks is all you need — no `@SpringBootTest`, no reflection, no `ReflectionTestUtils.setField`.
4. **Circular dependencies surface at startup** as a clear error, instead of being silently tolerated (field injection can mask them).

---

## When setter injection still has a place
- **Optional** dependencies that might legitimately be absent (feature toggles).
- **Re-configurable** components where the dependency can change at runtime (rare).
- Breaking a genuinely unavoidable circular dependency (usually a design smell — fix the design instead).

---

## Why field injection is discouraged
- Uses **reflection** → fields can't be `final`, breaks immutability.
- Hides dependencies — the class looks like it has none from the outside.
- Hard to test without Spring: must use `ReflectionTestUtils` or start a context.
- Makes "class is doing too much" invisible: you can add a 10th `@Autowired` field without anyone noticing, where a 10-argument constructor screams "split me up."
- Not portable outside a DI container.

The Spring team explicitly tags `@Autowired` on fields as discouraged in the reference manual, and IDEs like IntelliJ flag it with a warning.

---

## Pitfall: self-invocation + field injection
If a bean injects itself (or collaborators) via field and then calls `this.someMethod()`, AOP proxies like `@Transactional` are **bypassed**. Constructor injection of a self-reference or splitting the class is the fix. (See Q29.)

---

## Interview one-liner
> "Constructor injection produces immutable, fully initialized beans whose required dependencies are explicit and testable with plain `new`. Field injection uses reflection, can't be `final`, hides dependencies, and forces you to start a Spring context or use `ReflectionTestUtils` to test. Setter injection is fine for genuinely optional deps. Since Spring 4.3 a single constructor doesn't even need `@Autowired`, so with Lombok's `@RequiredArgsConstructor` constructor injection is as terse as field injection — with none of the downsides."
