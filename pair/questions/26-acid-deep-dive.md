## Q26: Explain ACID properties in depth. What does each guarantee actually mean, how is it implemented internally, and what can break each one?

Cover: Atomicity (undo logs, rollback), Consistency (what it really means — often misunderstood), Isolation (anomalies prevented at each level), Durability (WAL, fsync, commit protocols). Include Oracle-specific mechanisms (UNDO tablespace, redo log, LGWR), Spring `@Transactional` behavior, and common ways each property is accidentally violated.
