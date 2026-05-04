# 52. Transactional Outbox Pattern

Solves the **dual-write problem**: how to atomically update the database AND publish a message to a queue when you can't have a distributed transaction across both.

---

## The problem

```java
// WRONG — not atomic
@Transactional
public void placeOrder(Order order) {
    orderRepo.save(order);             // DB write
    kafkaTemplate.send("orders", ...); // Kafka publish
    // If Kafka fails here → DB has the order but no event published
    // If app crashes between the two → same problem
}
```

The DB and the message broker are two separate systems. You can't wrap both in one transaction.

---

## Outbox pattern solution

Add an `outbox` table in the **same database**. Write both the domain change and the outbox record in **one local transaction**. A separate relay process reads the outbox and publishes to the broker.

```sql
CREATE TABLE outbox_event (
    id          UUID PRIMARY KEY,
    event_type  VARCHAR(100) NOT NULL,
    payload     JSONB        NOT NULL,
    created_at  TIMESTAMP    NOT NULL DEFAULT NOW(),
    published   BOOLEAN      NOT NULL DEFAULT FALSE
);
```

```java
@Transactional
public void placeOrder(Order order) {
    orderRepo.save(order);                    // domain write

    outboxRepo.save(OutboxEvent.of(          // same transaction
        "ORDER_CREATED",
        new OrderCreatedPayload(order.getId(), order.getTotal())
    ));
    // If anything fails → both rolled back atomically
}
```

**Relay (publisher) process:**
```java
@Scheduled(fixedDelay = 1000)
@Transactional
public void publishPendingEvents() {
    List<OutboxEvent> events = outboxRepo.findByPublishedFalse();
    for (OutboxEvent e : events) {
        kafkaTemplate.send(e.getEventType(), e.getPayload());
        e.setPublished(true);
    }
    outboxRepo.saveAll(events);
}
```

---

## Inbox pattern (consumer side)

Prevents processing a message twice (at-least-once delivery from broker):

```sql
CREATE TABLE inbox_event (
    id          UUID PRIMARY KEY,   -- message ID from broker
    processed   BOOLEAN NOT NULL DEFAULT FALSE,
    processed_at TIMESTAMP
);
```

```java
@KafkaListener(topics = "orders.created")
@Transactional
public void handle(OrderCreatedEvent event, @Header(KafkaHeaders.MESSAGE_KEY) String messageId) {
    if (inboxRepo.existsById(messageId)) return;   // already processed — idempotency

    inventoryService.reserve(event.getOrderId());   // do work

    inboxRepo.save(new InboxEvent(messageId));       // mark as processed — same transaction
}
```

---

## Outbox vs Inbox — which solves what?

| Pattern | Solves | Where |
|---|---|---|
| **Outbox** | Reliable publishing (DB → broker) | Producer side |
| **Inbox** | Idempotent consumption (broker → DB) | Consumer side |

---

## Production options

Instead of a custom scheduler, use **CDC (Change Data Capture)**:
- **Debezium** watches the outbox table via DB transaction log (WAL for Postgres, redo log for Oracle)
- Zero polling delay, no DB lock contention
- Debezium publishes to Kafka automatically when a row is inserted

```yaml
# Debezium connector config
"transforms": "outbox",
"transforms.outbox.type": "io.debezium.transforms.outbox.EventRouter"
```

---

## Interview one-liner
> "The Outbox pattern solves the dual-write problem: instead of writing to DB and publishing to a broker in two separate operations (which can fail independently), you write both the domain change and an outbox record in one local transaction. A relay process then reads the outbox and publishes to the broker — guaranteeing at-least-once delivery without distributed transactions."
