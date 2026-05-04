## Q21: How do you use Redis to handle booking concurrency at scale? Cover atomic counters, Lua scripts, distributed locks, TTL-based holds, and how to keep Redis consistent with the database.

Focus on: why Redis beats DB locking for hot counters, specific Redis commands (INCR/DECR/SETNX), Lua scripts for atomic compound logic, Redlock for distributed locks, reservation expiry with TTL, and the outbox pattern for DB reconciliation. Include Spring Boot code.
