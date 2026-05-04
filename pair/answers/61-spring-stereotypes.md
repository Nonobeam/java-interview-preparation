# 64. Spring stereotype annotations: `@Component`, `@Service`, `@Repository`, `@Controller`

## Short answer

All four register a class as a Spring bean (component-scanned). They are **specializations of `@Component`** that document layer intent and, in two cases, add real behavior.

| Annotation     | Layer            | Adds behavior beyond `@Component`? |
|----------------|------------------|-------------------------------------|
| `@Component`   | Generic / utility| No |
| `@Service`     | Service / use-case layer | No — pure semantic marker |
| `@Repository`  | Persistence layer | **Yes** — exception translation |
| `@Controller`  | Web MVC layer    | **Yes** — recognized by Spring MVC dispatcher |
| `@RestController` | REST web layer | `@Controller` + `@ResponseBody` (see Q65) |

## What each one is for

### `@Component`
The base. Use for utilities, helpers, anything that doesn't fit the other three.

```java
@Component
public class PriceFormatter { ... }
```

### `@Service`
Marks a class that holds **business logic / use cases**. Behaviorally identical to `@Component` — no extra magic. The value is documentation: "this is a service, not a controller or repository."

```java
@Service
public class TransferService {
    public void transfer(Long from, Long to, BigDecimal amount) { ... }
}
```

### `@Repository` — adds exception translation
Marks a DAO. The real bonus: Spring wraps it with a `PersistenceExceptionTranslationPostProcessor` that converts vendor-specific `SQLException` / `HibernateException` into Spring's unified `DataAccessException` hierarchy (e.g. `DataIntegrityViolationException`, `DuplicateKeyException`).

```java
@Repository
public class JdbcUserDao {
    public User find(Long id) {
        // a raw SQLException would be translated to a DataAccessException
    }
}
```

> Note: Spring Data JPA `JpaRepository` interfaces don't need `@Repository` — the proxy already does translation. But on hand-written JDBC/JPA DAOs, `@Repository` is what gets you the translation.

### `@Controller` — recognized by Spring MVC
Marks a web controller. The `DispatcherServlet` only routes requests to classes annotated `@Controller` (or `@RestController`). A plain `@Component` method with `@GetMapping` would not get picked up.

```java
@Controller
public class HomeController {
    @GetMapping("/")
    public String home(Model model) {
        model.addAttribute("name", "world");
        return "home";   // view name → resolved by ViewResolver (Thymeleaf, JSP, ...)
    }
}
```

## Component scanning

`@SpringBootApplication` (which includes `@ComponentScan`) discovers all four by scanning the base package and registering them as singleton beans. You don't need separate config for each — they all flow through the same machinery.

## Practical "which one do I use?"

```
Web request arrives  →  @Controller / @RestController
                        ↓
Business logic       →  @Service
                        ↓
Database access      →  @Repository (or Spring Data JpaRepository)
                        ↓
Anything else        →  @Component
```

Wrong choice doesn't break the app — `@Component` everywhere works — but you lose the documentation value, the MVC routing for controllers, and the exception translation for repositories.

## Interview one-liner

> "All four are `@Component` specializations that get bean-registered by component scan. `@Service` and `@Component` are semantic markers only. `@Repository` adds JDBC/JPA exception translation into Spring's unified `DataAccessException`. `@Controller` is what `DispatcherServlet` actually looks for to route web requests."
