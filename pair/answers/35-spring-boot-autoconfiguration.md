# 38. How Spring Boot AutoConfiguration Works

## The elevator pitch
`@SpringBootApplication` enables component scanning + auto-configuration. Boot scans classpath + your config, conditionally registers a curated set of `@Configuration` classes (from starter JARs), which create the beans you'd otherwise write by hand (DataSource, DispatcherServlet, Jackson, etc.). **Convention over configuration.**

---

## Step-by-step mechanics

### 1. `@SpringBootApplication` = 3 things
```java
@SpringBootApplication
// ≡
@SpringBootConfiguration
@ComponentScan
@EnableAutoConfiguration   // ← the magic one
```

### 2. `@EnableAutoConfiguration` imports a selector
It meta-imports `AutoConfigurationImportSelector`, which asks: *"Which auto-config classes should I import?"*

### 3. The selector reads a registration file
- **Boot 2.7+ and all of Boot 3.x** read:
  `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`
- **Older Boot (pre-2.7)** read `META-INF/spring.factories` under the key
  `org.springframework.boot.autoconfigure.EnableAutoConfiguration`.

Every starter JAR (e.g., `spring-boot-autoconfigure`, `spring-boot-starter-data-jpa`) bundles one of these files listing its auto-configuration classes.

### 4. Each candidate class is gated by `@Conditional...`
The classes are **not** unconditionally applied. They use `@Conditional*` annotations to activate only when it makes sense:

| Annotation | Applies when |
|---|---|
| `@ConditionalOnClass` | Given class is on the classpath |
| `@ConditionalOnMissingClass` | Given class is **not** on the classpath |
| `@ConditionalOnBean` | A bean of that type already exists |
| `@ConditionalOnMissingBean` | **No** bean of that type exists — lets you override |
| `@ConditionalOnProperty` | A property has a given value (e.g., `spring.datasource.url`) |
| `@ConditionalOnWebApplication` | Running in a web context |
| `@ConditionalOnResource` | A classpath resource is present |

### 5. Ordered evaluation
`@AutoConfigureBefore`, `@AutoConfigureAfter`, and `@AutoConfigureOrder` let one auto-config run before/after another, so things like `DataSourceAutoConfiguration` can run before `JpaRepositoriesAutoConfiguration`.

---

## Walk-through: `DataSourceAutoConfiguration`

Simplified shape:
```java
@AutoConfiguration
@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })
@EnableConfigurationProperties(DataSourceProperties.class)
public class DataSourceAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public DataSource dataSource(DataSourceProperties properties) {
        return properties.initializeDataSourceBuilder().build();
    }
}
```
Meaning:
- Only fires if `javax.sql.DataSource` is on the classpath (you added `spring-boot-starter-jdbc`/`jpa`).
- Reads `spring.datasource.*` properties into `DataSourceProperties`.
- Creates a `DataSource` bean **only if you didn't define one yourself** — `@ConditionalOnMissingBean` is the override hook.

That's why adding `spring-boot-starter-data-jpa` + setting `spring.datasource.url=...` in `application.yml` is enough to get a working DataSource + EntityManager + transaction manager.

---

## How to override or tweak
1. **Define your own bean** of the same type — `@ConditionalOnMissingBean` on the auto-config steps aside.
2. **Set properties** in `application.yml` / `application.properties` — bound to `@ConfigurationProperties`.
3. **Exclude an auto-config**:
   ```java
   @SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
   ```
   or `spring.autoconfigure.exclude=...` in properties.
4. **Add `@AutoConfigureBefore`/`After`** on your own auto-config to control ordering.

---

## Debugging: why is (or isn't) a bean there?
Run with `--debug` (or set `debug=true`) to print the **Auto-Configuration Report**:
```
============================
CONDITIONS EVALUATION REPORT
============================

Positive matches:
-----------------
   DataSourceAutoConfiguration matched:
      - @ConditionalOnClass found required class 'javax.sql.DataSource' (OnClassCondition)

Negative matches:
-----------------
   MongoAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required class 'com.mongodb.client.MongoClient'
```
This is the single best tool to understand exactly what Boot did and why.

---

## Writing your own auto-configuration (library authors)
1. Create a `@AutoConfiguration` class with `@Conditional*` gates.
2. Expose properties via a `@ConfigurationProperties`-annotated POJO.
3. Register the class in
   `src/main/resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`
   (one FQCN per line).
4. Ship as a JAR — any downstream app that puts it on the classpath gets the config automatically.

---

## Interview one-liner
> "`@SpringBootApplication` enables `@EnableAutoConfiguration`, which uses `AutoConfigurationImportSelector` to load auto-config classes listed in `META-INF/spring/...AutoConfiguration.imports` from every starter on the classpath. Each class is guarded by `@ConditionalOn...` annotations, so it only creates beans when they make sense — and `@ConditionalOnMissingBean` lets the app override anything Boot provides."
