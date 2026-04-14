## Q20: How does JPA `@Version` work with Oracle? Does Oracle have its own row versioning mechanism, and what locking features does Oracle provide beyond Postgres?

Focus on: `@Version` being DB-agnostic (Hibernate-level, not DB-level), Oracle's `ORA_ROWSCN` with `ROWDEPENDENCIES`, and Oracle's `SELECT FOR UPDATE` variants (`NOWAIT`, `WAIT n`, `SKIP LOCKED`). When to use each and why most teams stick with `@Version` for portability.
