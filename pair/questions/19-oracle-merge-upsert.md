## Q23: What is Oracle's MERGE statement? How do you do an UPSERT in Oracle, and how does it compare to Postgres `INSERT ... ON CONFLICT`?

Cover: MERGE syntax (INTO, USING, ON, WHEN MATCHED, WHEN NOT MATCHED), Oracle's lack of `INSERT ... ON CONFLICT`, use cases (sync from staging, idempotent inserts), interaction with JPA, common pitfalls (NULL handling, deadlocks).
