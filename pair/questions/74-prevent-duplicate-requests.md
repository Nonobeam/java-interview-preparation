# Q74 — How to Prevent Duplicate Requests in a System

Open question — no single correct answer. Evaluate depth and trade-off awareness.

## Questions

1. Give three real scenarios where duplicate requests happen in production. (network retry, user double-click, message queue redelivery)
2. What is an idempotency key? How does it work at the API layer? Show a rough schema for storing idempotency keys server-side.
3. A payment API receives the same `POST /payments` twice with the same idempotency key. Walk through the server logic step by step.
4. How do you implement deduplication at the message queue layer (Kafka / RabbitMQ)? What guarantees does each give you out of the box?
5. What is the difference between "at-least-once" and "exactly-once" delivery? When is "at-least-once + idempotent consumer" good enough?
6. Database-level deduplication: when would you use a unique constraint instead of an idempotency key table?
