# Pair — Questions Index

Tech interview questions (Java, Spring, Oracle, JPA, Redis, concurrency).

## Spring fundamentals
- [1. Spring IoC container](questions/1-spring-ioc-container.md)
- [56. `@Component` vs `@Bean`](questions/56-component-vs-bean.md)
- [2. `@Transactional` deep dive](questions/2-transactional-deep-dive.md)
- [3. `@Transactional` propagation levels](questions/3-transactional-propagation-levels.md)
- [4. NESTED propagation & rollback](questions/4-nested-propagation-rollback.md)
- [5. Propagation — real-world usages](questions/5-propagation-example-usages.md)
- [25. `CrudRepository` vs `JpaRepository`](questions/25-crudrepo-vs-jparepo.md)
- [27. Spring bean scopes & thread-safety](questions/27-spring-bean-scopes-thread-safety.md)
- [29. `@Transactional` self-invocation](questions/29-transactional-self-invocation.md)
- [37. Spring bean lifecycle](questions/37-spring-bean-lifecycle.md)
- [38. Spring Boot auto-configuration](questions/38-spring-boot-autoconfiguration.md)
- [39. Constructor vs field injection](questions/39-constructor-vs-field-injection.md)
- [46. Spring MVC request lifecycle](questions/46-spring-mvc-request-lifecycle.md)

## JPA / Hibernate
- [6. N+1 query problem](questions/6-n-plus-1-query-problem.md)
- [7. `JOIN` vs `JOIN FETCH`](questions/7-join-vs-join-fetch.md)
- [19. Oracle ↔ Spring integration differences](questions/19-oracle-spring-integration-differences.md)
- [20. JPA `@Version` & Oracle locking](questions/20-jpa-version-oracle-locking.md)
- [30. JPA pessimistic locking](questions/30-jpa-pessimistic-locking.md)

## Oracle / SQL / PLSQL
- [8. Container/shipment schema](questions/8-container-shipment-schema.md)
- [9. Overdue containers query](questions/9-overdue-containers-query.md)
- [10. `SYSDATE` vs `CURRENT_DATE`](questions/10-oracle-sysdate-vs-current-date.md)
- [11. Oracle pagination](questions/11-oracle-pagination.md)
- [12. Oracle composite index](questions/12-oracle-composite-index.md)
- [13. Oracle `EXPLAIN PLAN`](questions/13-oracle-explain-plan.md)
- [14. `DELETE` vs `TRUNCATE` vs `DROP`](questions/14-delete-truncate-drop.md)
- [15. Container journey query](questions/15-container-journey-query.md)
- [16. Oracle PL/SQL procedures](questions/16-oracle-plsql-procedures.md)
- [17. Oracle deadlock in logistics](questions/17-oracle-deadlock-logistics.md)
- [18. `OFFSET` vs page pagination](questions/18-offset-vs-page-pagination.md)
- [23. Oracle `MERGE` / upsert](questions/23-oracle-merge-upsert.md)
- [24. Oracle `CONNECT BY` hierarchical](questions/24-oracle-connect-by-hierarchical.md)

## Transactions / Concurrency
- [21. Redis booking concurrency](questions/21-redis-booking-concurrency.md)
- [57. Redis — why fast & single-thread model](questions/57-redis-fast-single-thread.md)
- [22. Transaction isolation levels](questions/22-transaction-isolation-levels.md)
- [26. ACID deep dive](questions/26-acid-deep-dive.md)
- [28. JMM & happens-before](questions/28-jmm-happens-before.md)
- [31. Redisson distributed locks](questions/31-redisson-distributed-locks.md)

## Java core & OOP
- [32. `equals()` vs `==`](questions/32-equals-vs-double-equals.md)
- [33. `interface` vs `abstract class`](questions/33-interface-vs-abstract-class.md)
- [34. 4 pillars of OOP](questions/34-oop-four-pillars.md)
- [35. SOLID principles](questions/35-solid-principles.md)
- [36. Java Collections framework](questions/36-java-collections-framework.md)
- [54. Streams & Lambda](questions/54-streams-lambda.md)
- [55. Threads — ExecutorService, synchronized, volatile](questions/55-threads.md)
- [58. Java Object Pool](questions/58-java-object-pool.md)

## Security
- [47. JWT token](questions/47-jwt-token.md)
- [48. Symmetric & Asymmetric keys](questions/48-symmetric-asymmetric-keys.md)

## Spring advanced
- [49. Spring AOP](questions/49-spring-aop.md)
- [50. Circuit Breaker](questions/50-circuit-breaker.md)

## Distributed systems
- [51. Message queue — Kafka vs RabbitMQ](questions/51-message-queue.md)
- [52. Inbox/Outbox pattern](questions/52-inbox-outbox-pattern.md)
- [53. Domain-Driven Design (DDD)](questions/53-ddd.md)
