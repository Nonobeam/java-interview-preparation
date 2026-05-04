# 73. `@Component` vs `@Configuration`

## Short answer

`@Configuration` **is** a `@Component` (meta-annotated with it), so both get picked up by component scanning. The difference is what Spring does *after* registering the class:

| Aspect | `@Component` | `@Configuration` |
|---|---|---|
| Purpose | Mark a regular bean (service, helper, DAO) | Mark a class whose job is to **declare other beans** via `@Bean` methods |
| Component-scanned? | Yes | Yes (it's a specialization of `@Component`) |
| Spring proxies the class? | No (used as-is) | **Yes** — CGLIB subclass enforces singleton semantics across `@Bean` calls |
| `@Bean` methods allowed? | Yes, but in **"lite" mode** — inter-method calls create new instances | Yes, in **"full" mode** — inter-method calls return the cached singleton |
| Typical use | Business class you own | Wiring class that creates third-party / configured beans |

## Why the proxy matters — the one thing that actually breaks

```java
@Configuration                          // ← change to @Component to see the bug
public class AppConfig {

    @Bean
    public DataSource dataSource() {
        return new HikariDataSource(...);
    }

    @Bean
    public JdbcTemplate jdbcTemplate() {
        return new JdbcTemplate(dataSource());   // direct method call
    }

    @Bean
    public TransactionManager txManager() {
        return new DataSourceTransactionManager(dataSource());  // again
    }
}
```

- With **`@Configuration`**: Spring wraps `AppConfig` in a CGLIB proxy. The two `dataSource()` calls are intercepted and return the **same** singleton from the container. One pool, one tx manager bound to it. Correct.
- With **`@Component`** (lite mode): no proxy. `dataSource()` is a plain Java call → **two separate `HikariDataSource` instances**. `JdbcTemplate` and `TransactionManager` end up bound to *different* connection pools. Transactions silently don't cover your queries. Subtle, nasty bug.

The proxy is the whole reason `@Configuration` exists.

## When lite-mode `@Bean` (inside `@Component`) is fine

If `@Bean` methods don't call each other — or you inject dependencies as method parameters instead of calling sibling methods — there's no singleton hazard:

```java
@Component
public class MetricsConfig {

    @Bean
    public MeterRegistry meterRegistry() {
        return new SimpleMeterRegistry();   // standalone, no inter-bean call
    }

    @Bean
    public Timer requestTimer(MeterRegistry registry) {  // injected, not via this.meterRegistry()
        return Timer.builder("http.requests").register(registry);
    }
}
```

This works in any `@Component`. But the moment you write `meterRegistry()` instead of accepting `MeterRegistry` as a parameter, you need `@Configuration`.

## Mental model

- **`@Component`** = "I am a bean."
- **`@Configuration`** = "I am a bean *factory*. Treat my `@Bean` methods as singleton-aware."

Annotating with `@Configuration` flips on the CGLIB proxy, which is exactly what makes inter-method `@Bean` calls safe.

## Practical rule

```
Class holds business logic / state                  → @Service / @Component
Class wires up beans via @Bean methods              → @Configuration
@Bean methods that call each other                  → @Configuration is REQUIRED
@Bean methods that take dependencies as parameters  → either works, prefer @Configuration for clarity
```

## Gotchas

- `@Configuration` classes are CGLIB-proxied → they **cannot be `final`**, and `@Bean` methods cannot be `private` or `final`. (Spring Boot 2.2+ supports `proxyBeanMethods = false` to skip proxying when you don't need it — slightly faster startup, but you lose the singleton guarantee for inter-method calls.)
- `@SpringBootApplication` is itself meta-annotated with `@Configuration`, which is why you can drop `@Bean` methods directly into your main application class.
- See Q56 (`@Component` vs `@Bean`) for the class-vs-method axis, and Q64 (Spring stereotypes) for `@Component` family specializations.

## Interview one-liner

> "`@Configuration` is a specialized `@Component` that Spring wraps in a CGLIB proxy so that `@Bean` methods calling each other return the cached singleton instead of constructing new instances. Put `@Bean` methods in a plain `@Component` and inter-method calls bypass the container — you get duplicate beans and broken wiring. `@Component` is for your regular classes; `@Configuration` is for bean-factory classes."
