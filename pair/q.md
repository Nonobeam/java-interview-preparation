# Pair — Questions Index

Tech interview questions (Java, Spring, Oracle, JPA, Redis, concurrency).

## Spring fundamentals
- [1. Spring IoC container](questions/1-spring-ioc-container.md)
- [2. `@Transactional` deep dive](questions/2-transactional-deep-dive.md)
- [3. `@Transactional` propagation levels](questions/3-transactional-propagation-levels.md)
- [4. NESTED propagation & rollback](questions/4-nested-propagation-rollback.md)
- [5. Propagation — real-world usages](questions/5-propagation-example-usages.md)
- [21. `CrudRepository` vs `JpaRepository`](questions/21-crudrepo-vs-jparepo.md)
- [23. Spring bean scopes & thread-safety](questions/23-spring-bean-scopes-thread-safety.md)
- [25. `@Transactional` self-invocation](questions/25-transactional-self-invocation.md)
- [33. Spring bean lifecycle](questions/33-spring-bean-lifecycle.md)
- [34. Constructor vs field injection](questions/34-constructor-vs-field-injection.md)
- [35. Spring Boot auto-configuration](questions/35-spring-boot-autoconfiguration.md)
- [42. Spring MVC request lifecycle](questions/42-spring-mvc-request-lifecycle.md)
- [52. `@Component` vs `@Bean`](questions/52-component-vs-bean.md)
- [60. `@RestController` vs `@Controller`](questions/60-restcontroller-vs-controller.md)
- [61. Spring stereotypes — `@Component` / `@Service` / `@Repository` / `@Controller`](questions/61-spring-stereotypes.md)
- [62. Spring Data JPA — derived queries, `@Query`, pagination](questions/62-spring-data-queries-pagination.md)
- [63. `@ControllerAdvice` & `@ExceptionHandler`](questions/63-controller-advice-exception-handler.md)
- [64. Spring WebFlux — Mono/Flux, reactive](questions/64-spring-webflux-reactive.md)
- [68. `@Component` vs `@Configuration`](questions/68-component-vs-configuration.md)

## JPA / Hibernate
- [6. N+1 query problem](questions/6-n-plus-1-query-problem.md)
- [7. `JOIN` vs `JOIN FETCH`](questions/7-join-vs-join-fetch.md)
- [14. Oracle ↔ Spring integration differences](questions/14-oracle-spring-integration-differences.md)
- [16. JPA `@Version` & Oracle locking](questions/16-jpa-version-oracle-locking.md)
- [26. JPA pessimistic locking](questions/26-jpa-pessimistic-locking.md)

## Oracle / SQL
- [8. `SYSDATE` vs `CURRENT_DATE`](questions/8-oracle-sysdate-vs-current-date.md)
- [9. Oracle pagination](questions/9-oracle-pagination.md)
- [10. Oracle composite index](questions/10-oracle-composite-index.md)
- [11. Oracle `EXPLAIN PLAN`](questions/11-oracle-explain-plan.md)
- [12. `DELETE` vs `TRUNCATE` vs `DROP`](questions/12-delete-truncate-drop.md)
- [13. Oracle PL/SQL procedures](questions/13-oracle-plsql-procedures.md)
- [15. `OFFSET` vs page pagination](questions/15-offset-vs-page-pagination.md)
- [19. Oracle `MERGE` / upsert](questions/19-oracle-merge-upsert.md)
- [20. Oracle `CONNECT BY` hierarchical](questions/20-oracle-connect-by-hierarchical.md)
- [36. What is a DB index](questions/36-what-is-a-db-index.md)
- [37. Too many indexes — good or bad?](questions/37-too-many-indexes-good-or-bad.md)
- [38. Why adding a new index is hard](questions/38-why-adding-new-index-is-hard.md)
- [39. Tradeoffs of using indexes](questions/39-tradeoffs-of-using-indexes.md)
- [40. What happens when indexing a column](questions/40-what-happens-when-indexing-a-column.md)
- [41. EXPLAIN and ANALYZE in SQL](questions/41-explain-and-analyze-in-sql.md)
- [65. SQL JOIN types — INNER / LEFT / RIGHT / FULL / CROSS](questions/65-sql-join-types.md)
- [66. `GROUP BY`, `HAVING`, aggregates](questions/66-group-by-having-aggregates.md)
- [67. Subqueries vs CTEs](questions/67-subquery-vs-cte.md)
- [69. Normalization — 1NF / 2NF / 3NF](questions/69-normalization-1nf-2nf-3nf.md)

## Transactions / Concurrency
- [17. Redis booking concurrency](questions/17-redis-booking-concurrency.md)
- [18. Transaction isolation levels](questions/18-transaction-isolation-levels.md)
- [22. ACID deep dive](questions/22-acid-deep-dive.md)
- [24. JMM & happens-before](questions/24-jmm-happens-before.md)
- [27. Redisson distributed locks](questions/27-redisson-distributed-locks.md)
- [53. Redis — why fast & single-thread model](questions/53-redis-fast-single-thread.md)

## Java core & OOP
- [28. `equals()` vs `==`](questions/28-equals-vs-double-equals.md)
- [29. `interface` vs `abstract class`](questions/29-interface-vs-abstract-class.md)
- [30. 4 pillars of OOP](questions/30-oop-four-pillars.md)
- [31. SOLID principles](questions/31-solid-principles.md)
- [32. Java Collections framework](questions/32-java-collections-framework.md)
- [50. Streams & Lambda](questions/50-streams-lambda.md)
- [51. Threads — ExecutorService, synchronized, volatile](questions/51-threads.md)
- [54. `hashCode()` contract](questions/54-hashcode-contract.md)
- [55. Java Object Pool](questions/55-java-object-pool.md)
- [56. `String` immutability, String pool, `StringBuilder` vs `StringBuffer`](questions/56-string-immutability-pool.md)
- [57. Generics & wildcards (`? extends`, `? super`)](questions/57-generics-wildcards.md)
- [58. Exceptions — checked vs unchecked, try-with-resources](questions/58-exceptions-checked-unchecked.md)
- [59. JVM memory layout & garbage collection](questions/59-jvm-memory-gc.md)

## Security
- [43. JWT token](questions/43-jwt-token.md)
- [44. Symmetric & Asymmetric keys](questions/44-symmetric-asymmetric-keys.md)

## Spring advanced
- [45. Spring AOP](questions/45-spring-aop.md)
- [46. Circuit Breaker](questions/46-circuit-breaker.md)

## Distributed systems
- [47. Message queue — Kafka vs RabbitMQ](questions/47-message-queue.md)
- [48. Inbox/Outbox pattern](questions/48-inbox-outbox-pattern.md)
- [49. Domain-Driven Design (DDD)](questions/49-ddd.md)
