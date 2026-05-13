# Q73 — Distributed Transactions in Microservices (Saga Pattern)

Scenario: An e-commerce order flow spans three services: `OrderService`, `PaymentService`, `InventoryService`. A single user action must span all three atomically — but they are separate deployments with separate databases.

## Questions

1. Why can't you use a single database transaction (or 2PC) across microservices? What are the practical problems?
2. Explain the Saga pattern. What are the two flavors (choreography vs orchestration)? What are the trade-offs of each?
3. In the order flow above, what does a compensating transaction look like for each step when payment succeeds but inventory reservation fails?
4. How do you handle retries safely in a saga? What makes a step idempotent, and why does idempotency matter for retries?
5. What is the "dual write" problem in a saga step (e.g., writing to DB and publishing an event)? How does the Outbox pattern solve it?
6. If `InventoryService` is down when the saga reaches it, what are your options? (retry with backoff, dead-letter queue, manual compensation)
