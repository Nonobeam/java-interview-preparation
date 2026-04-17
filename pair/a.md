# Pair — Answers Index

Tech interview answers (Java, Spring, Oracle, JPA, Redis, concurrency).

## Spring fundamentals
- [1. Spring IoC container](answers/1-spring-ioc-container.md)
- [56. `@Component` vs `@Bean`](answers/56-component-vs-bean.md)
- [2. `@Transactional` deep dive](answers/2-transactional-deep-dive.md)
- [3. `@Transactional` propagation levels](answers/3-transactional-propagation-levels.md)
- [4. NESTED propagation & rollback](answers/4-nested-propagation-rollback.md)
- [5. Propagation — real-world usages](answers/5-propagation-example-usages.md)
- [25. `CrudRepository` vs `JpaRepository`](answers/25-crudrepo-vs-jparepo.md)
- [27. Spring bean scopes & thread-safety](answers/27-spring-bean-scopes-thread-safety.md)
- [29. `@Transactional` self-invocation](answers/29-transactional-self-invocation.md)
- [37. Spring bean lifecycle](answers/37-spring-bean-lifecycle.md)
- [38. Spring Boot auto-configuration](answers/38-spring-boot-autoconfiguration.md)
- [39. Constructor vs field injection](answers/39-constructor-vs-field-injection.md)
- [46. Spring MVC request lifecycle](answers/46-spring-mvc-request-lifecycle.md)

## JPA / Hibernate
- [6. N+1 query problem](answers/6-n-plus-1-query-problem.md)
- [7. `JOIN` vs `JOIN FETCH`](answers/7-join-vs-join-fetch.md)
- [19. Oracle ↔ Spring integration differences](answers/19-oracle-spring-integration-differences.md)
- [20. JPA `@Version` & Oracle locking](answers/20-jpa-version-oracle-locking.md)
- [30. JPA pessimistic locking](answers/30-jpa-pessimistic-locking.md)

## Oracle / SQL / PLSQL
- [8. Container/shipment schema](answers/8-container-shipment-schema.md)
- [9. Overdue containers query](answers/9-overdue-containers-query.md)
- [10. `SYSDATE` vs `CURRENT_DATE`](answers/10-oracle-sysdate-vs-current-date.md)
- [11. Oracle pagination](answers/11-oracle-pagination.md)
- [12. Oracle composite index](answers/12-oracle-composite-index.md)
- [13. Oracle `EXPLAIN PLAN`](answers/13-oracle-explain-plan.md)
- [14. `DELETE` vs `TRUNCATE` vs `DROP`](answers/14-delete-truncate-drop.md)
- [15. Container journey query](answers/15-container-journey-query.md)
- [16. Oracle PL/SQL procedures](answers/16-oracle-plsql-procedures.md)
- [17. Oracle deadlock in logistics](answers/17-oracle-deadlock-logistics.md)
- [18. `OFFSET` vs page pagination](answers/18-offset-vs-page-pagination.md)
- [23. Oracle `MERGE` / upsert](answers/23-oracle-merge-upsert.md)
- [24. Oracle `CONNECT BY` hierarchical](answers/24-oracle-connect-by-hierarchical.md)

## Transactions / Concurrency
- [21. Redis booking concurrency](answers/21-redis-booking-concurrency.md)
- [57. Redis — why fast & single-thread model](answers/57-redis-fast-single-thread.md)
- [22. Transaction isolation levels](answers/22-transaction-isolation-levels.md)
- [26. ACID deep dive](answers/26-acid-deep-dive.md)
- [28. JMM & happens-before](answers/28-jmm-happens-before.md)
- [31. Redisson distributed locks](answers/31-redisson-distributed-locks.md)

## Java core & OOP
- [32. `equals()` vs `==`](answers/32-equals-vs-double-equals.md)
- [33. `interface` vs `abstract class`](answers/33-interface-vs-abstract-class.md)
- [34. 4 pillars of OOP](answers/34-oop-four-pillars.md)
- [35. SOLID principles](answers/35-solid-principles.md)
- [36. Java Collections framework](answers/36-java-collections-framework.md)
- [54. Streams & Lambda](answers/54-streams-lambda.md)
- [55. Threads — ExecutorService, synchronized, volatile](answers/55-threads.md)
- [58. Java Object Pool](answers/58-java-object-pool.md)

## Security
- [47. JWT token](answers/47-jwt-token.md)
- [48. Symmetric & Asymmetric keys](answers/48-symmetric-asymmetric-keys.md)

## Spring advanced
- [49. Spring AOP](answers/49-spring-aop.md)
- [50. Circuit Breaker](answers/50-circuit-breaker.md)

## Distributed systems
- [51. Message queue — Kafka vs RabbitMQ](answers/51-message-queue.md)
- [52. Inbox/Outbox pattern](answers/52-inbox-outbox-pattern.md)
- [53. Domain-Driven Design (DDD)](answers/53-ddd.md)
