# 73. Distributed Transactions in Microservices — Saga Pattern

## Why you can't use a single DB transaction across microservices

- Each service owns its own database. A cross-service transaction would require the transaction coordinator to hold locks across multiple databases simultaneously.
- **2PC (Two-Phase Commit)** technically works but is brittle in practice: if the coordinator crashes between prepare and commit, all participants are stuck holding locks forever. It also creates tight coupling and kills horizontal scaling.
- Most microservice databases are different technologies (PostgreSQL, MySQL, Mongo) — 2PC support varies.

The Saga pattern is the standard alternative.

## What a Saga is

A saga is a sequence of **local transactions**, where each step publishes an event or message that triggers the next step. If any step fails, **compensating transactions** undo the work done by previous steps.

There is **no global lock**. Each service commits immediately after its local step. Consistency is **eventual**.

## Two flavors

### Choreography
Each service listens for events and reacts. No central coordinator.

```
OrderService                PaymentService             InventoryService
    |                             |                           |
CREATE order                      |                           |
    │─── OrderCreated ──────────→ │                           |
    |                        CHARGE card                      |
    |                             │─── PaymentCompleted ────→ |
    |                             |                      RESERVE stock
    |                             |                           │─── StockReserved ──→ (done)
```

**Pros:** simple, no single point of failure, services are fully decoupled.  
**Cons:** hard to trace the overall flow; risk of cyclic event chains; difficult to add new steps.

### Orchestration
A central **orchestrator** (often a dedicated service or workflow engine) tells each service what to do and waits for responses.

```
OrderOrchestrator
    │──→ PaymentService:  charge()
    │        ← PaymentDone
    │──→ InventoryService: reserve()
    │        ← StockReserved
    │──→ ShippingService: schedule()
    │        ← ShipmentScheduled
    └──→ Mark order CONFIRMED
```

**Pros:** the entire flow is visible in one place; easy to add steps; straightforward error handling.  
**Cons:** orchestrator can become a bottleneck/single point of failure; more infrastructure.

**Rule of thumb:** use choreography for simple 2–3 step flows; orchestration for complex, long-running sagas.

## Compensating transactions

Each step that can be undone needs a compensating action:

| Step | Compensation |
|------|-------------|
| `PaymentService.charge(orderId)` | `PaymentService.refund(orderId)` |
| `InventoryService.reserve(orderId)` | `InventoryService.release(orderId)` |
| `OrderService.create(orderId)` | `OrderService.cancel(orderId)` |

Compensations run in **reverse order** when a step fails.

Important: some steps are **not compensatable** (e.g., "email sent"). Those are called *pivot transactions* — design the saga so they happen as late as possible, after all risky steps.

## Making steps idempotent (safe to retry)

Idempotent = calling the same operation multiple times produces the same result.

Implementation options:
- **Idempotency key**: include a unique ID in every request. Before processing, check if the ID was already handled (stored in DB). If yes, return the cached response.
- **Conditional update**: `UPDATE payments SET status='CHARGED' WHERE id=? AND status='PENDING'` — only succeeds once.
- **Unique constraint**: DB-level enforcement that the operation can't run twice.

Why it matters: message queues deliver **at least once**. Without idempotency, a retry doubles the charge.

## The dual write problem and the Outbox pattern

In each saga step you need to:
1. Write to your own database (e.g., `status = PAID`)
2. Publish an event to the message broker

These are two separate systems. If you write to the DB but crash before publishing the event — the saga stalls. If you publish the event but the DB write fails — you have a phantom event.

**Outbox pattern solution:**

```
Step 1: In ONE local transaction, write:
    - business row:   payments.status = 'PAID'
    - outbox row:     outbox.event = 'PaymentCompleted', status = 'PENDING'

Step 2: A separate poller/CDC (Debezium) reads the outbox table
        and publishes to Kafka, then marks status = 'SENT'
```

Now the event is only published after the DB transaction commits. No dual write risk.

## Handling a downstream service being down

Options in priority order:
1. **Retry with exponential backoff** — transient failures (network blip, restart).
2. **Dead-letter queue (DLQ)** — after N retries, move the message to a DLQ for manual inspection or reprocessing.
3. **Circuit breaker** — if a service is consistently down, stop retrying and fail fast. Trigger compensation immediately.
4. **Manual compensation** — rare, but some sagas (e.g., "email already sent") require human intervention.

## Interview one-liner

> "A saga breaks a distributed transaction into a chain of local transactions, each immediately committed, with compensating transactions for rollback. Choreography uses events between services; orchestration uses a central coordinator. The key challenges are making every step idempotent (safe to retry), avoiding dual writes (solved by the Outbox pattern), and implementing compensations for every step that can fail."
