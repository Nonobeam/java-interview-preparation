# 74. How to Prevent Duplicate Requests

## Real scenarios where duplicates happen

1. **Network retry**: client sends a POST, the network drops the response. Client retries. Server received both.
2. **User double-click**: user clicks "Pay" twice before the first response renders.
3. **Message queue redelivery**: Kafka / RabbitMQ delivers the same message twice if the consumer crashes before acking.
4. **Load balancer retry**: some LBs retry on 5xx, causing the upstream service to receive the request twice.
5. **Mobile app background sync**: offline app queues operations and replays them on reconnect.

## Idempotency key

An **idempotency key** is a client-generated unique ID attached to every request. The server uses it to detect and deduplicate retries.

```
POST /payments
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
Content-Type: application/json

{ "amount": 100, "currency": "USD" }
```

### Server-side schema

```sql
CREATE TABLE idempotency_keys (
    key         VARCHAR(64) PRIMARY KEY,
    status      VARCHAR(16) NOT NULL,   -- PROCESSING, DONE
    response    TEXT,                   -- cached JSON response
    created_at  TIMESTAMP NOT NULL,
    expires_at  TIMESTAMP NOT NULL      -- clean up old keys after TTL
);
```

### Server logic (step by step)

```
1. Extract Idempotency-Key from header.
2. Look up the key in idempotency_keys table.

   Case A — not found:
     INSERT key with status='PROCESSING' (use unique constraint to prevent race)
     Execute the actual business logic (charge the card, etc.)
     UPDATE key: status='DONE', response=<result>
     Return result to client.

   Case B — status='DONE':
     Return the cached response immediately (same HTTP status + body).
     No business logic runs.

   Case C — status='PROCESSING':
     Another thread is mid-flight with the same key.
     Return 409 Conflict or 202 Accepted (client should poll or retry later).
```

The INSERT in Case A uses a **unique constraint** on `key` — if two threads race, one gets a unique-violation exception and waits or returns 409. This prevents double processing even under concurrent retries.

## Deduplication at the message queue layer

### Kafka
Kafka does **not** guarantee exactly-once by default at the consumer level. At-least-once is the standard guarantee. Options:
- **Idempotent producer** (`enable.idempotence=true`): Kafka deduplicates *within a producer session* using sequence numbers. Prevents producer-side duplicates.
- **Transactional producer** (`transactional.id`): exactly-once *write to Kafka* (produce + offset commit in one atomic operation).
- **Consumer-side idempotency**: the most practical approach — make your consumer logic idempotent (see below). Store the last-processed offset in your own DB transactionally with your business write.

### RabbitMQ
Delivers at-least-once. No built-in deduplication. You must implement it in the consumer:
- Store a `message_id` in a "processed messages" table.
- Before processing: `INSERT INTO processed_messages (id) VALUES (?)` — if unique constraint fires, the message was already handled.

## At-least-once vs exactly-once

| | At-least-once | Exactly-once |
|---|---|---|
| Guarantee | Will deliver, may duplicate | Delivered exactly one time |
| Complexity | Simple | Complex; requires coordination |
| Common approach | Kafka default consumer | Kafka transactions / special protocol |

**"At-least-once + idempotent consumer"** is usually the right call. True exactly-once across distributed systems is expensive and rarely worth it. If your handler is idempotent, duplicates are harmless.

## Database-level deduplication (unique constraint)

Use a **unique constraint** instead of an idempotency key table when:
- The operation is a data write that has a natural business key (e.g., `(order_id, payment_attempt_number)`).
- You want the database to enforce uniqueness as the single source of truth.

```sql
CREATE TABLE payments (
    id              BIGSERIAL PRIMARY KEY,
    order_id        BIGINT NOT NULL,
    attempt_number  INT NOT NULL,
    amount          NUMERIC NOT NULL,
    UNIQUE (order_id, attempt_number)   -- database-level deduplication
);
```

An idempotency key table is more general (works for reads, complex flows, cross-service). A unique constraint is simpler and more robust for pure write operations.

## Interview one-liner

> "Duplicate requests happen from network retries, user double-clicks, and message redelivery. The standard API-layer solution is an idempotency key: the client sends a unique ID per logical operation, and the server stores it with the cached response — retries return the cached result without re-executing business logic. At the queue layer, you make consumers idempotent (store processed message IDs) rather than relying on exactly-once delivery. For simple write operations, a database unique constraint is the cleanest deduplication mechanism."
