# 56. `@Component` vs `@Bean`

Both register beans in the Spring ApplicationContext, but they work differently and serve different purposes.

---

## `@Component` — class-level, classpath scanning

```java
@Component          // generic
@Service            // business logic layer (alias of @Component)
@Repository         // DAO layer (adds exception translation)
@Controller         // MVC controller
@RestController     // MVC + @ResponseBody
public class PaymentService {
    // Spring scans the classpath and registers this automatically
}
```

- Applied **directly on your class**
- Spring detects it via `@ComponentScan` (enabled automatically in Spring Boot)
- You own the class source code

---

## `@Bean` — method-level, inside `@Configuration`

```java
@Configuration
public class AppConfig {

    @Bean
    public RestTemplate restTemplate() {
        RestTemplate rt = new RestTemplate();
        rt.setRequestFactory(new HttpComponentsClientHttpRequestFactory());
        return rt;
    }

    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        return mapper;
    }
}
```

- Applied on a **method** that returns the bean instance
- Required when:
  - The class is from a **third-party library** (you can't add `@Component` to `RestTemplate`)
  - You need **custom construction logic** (conditional wiring, factory methods)
  - You need **multiple beans of the same type** with different configurations

---

## Side-by-side comparison

| Aspect | `@Component` | `@Bean` |
|---|---|---|
| Where | On the class | On a method in `@Configuration` |
| Discovery | Classpath scan | Explicit declaration |
| Source control | You own the class | Any class, including 3rd-party |
| Custom init | Limited | Full control |
| Multiple instances | One per class | One method = one bean, define multiple methods |

---

## When to use which

```java
// Use @Component — you own the class, no special construction
@Service
public class TransferService { ... }

// Use @Bean — third-party or complex construction
@Configuration
public class InfraConfig {

    // Can't annotate HikariDataSource with @Component — use @Bean
    @Bean
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:postgresql://...");
        config.setMaximumPoolSize(20);
        return new HikariDataSource(config);
    }

    // Two beans of the same type with different configs
    @Bean("kafkaProducer")
    public KafkaTemplate<String, String> kafkaProducer() { ... }

    @Bean("kafkaConsumer")
    public KafkaTemplate<String, String> kafkaConsumer() { ... }
}
```

---

## Interview one-liner
> "`@Component` is a class-level annotation for your own classes — Spring discovers them via classpath scanning. `@Bean` is a method-level annotation inside `@Configuration` used when you need to create beans from third-party classes or need full control over construction. Both result in beans in the ApplicationContext."
