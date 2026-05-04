## Q22: Explain transaction isolation levels and the concurrency anomalies each one prevents. Which is Oracle's default and which does Spring use?

Cover: ACID, the 4 standard isolation levels (READ_UNCOMMITTED, READ_COMMITTED, REPEATABLE_READ, SERIALIZABLE), the 3 anomalies (dirty read, non-repeatable read, phantom read), Oracle's default (READ COMMITTED), Oracle's lack of true REPEATABLE_READ, Spring `@Transactional(isolation = ...)`.
