# 51. Message Queue

A message queue decouples producers from consumers: the producer sends a message without knowing who or when it will be processed. Enables async processing, load leveling, and resilience.

---

## Core concepts

```
Producer → [Queue / Topic] → Consumer
```

| Term | Meaning |
|---|---|
| **Producer** | Publishes messages |
| **Consumer** | Reads and processes messages |
| **Queue** | Point-to-point — one message, one consumer |
| **Topic** | Pub/Sub — one message, many consumers |
| **Broker** | The server that stores and routes messages (Kafka, RabbitMQ) |
| **Acknowledgment** | Consumer signals it processed the message; broker then deletes it |

---

## Why use a message queue?

| Problem | MQ solution |
|---|---|
| Service B is slow | Producer returns immediately; B processes at its own pace |
| Traffic spikes | Queue absorbs burst; consumers work through it |
| Service B is down | Messages wait in queue; no data lost |
| Multiple consumers need same event | Topic fan-out |
| Audit trail needed | Messages are persisted (Kafka: replayed anytime) |

---

## Kafka vs RabbitMQ

| | **Kafka** | **RabbitMQ** |
|---|---|---|
| Model | Distributed log (pull) | Message broker (push) |
| Message retention | Retained by time/size (replayable) | Deleted after ack |
| Throughput | Very high (millions/sec) | High (thousands/sec) |
| Ordering | Per partition | Per queue (with caution) |
| Consumer groups | Multiple independent groups, each reads full log | Competing consumers share queue |
| Routing | By topic/partition | Flexible (exchanges: direct, fanout, topic) |
| Use case | Event streaming, audit logs, analytics | Task queues, RPC, complex routing |
| Message replay | Yes | No (once acked, gone) |

**Rule of thumb:**
- Use **Kafka** when you need event streaming, replay, or high throughput (order events, audit logs, analytics pipeline).
- Use **RabbitMQ** when you need flexible routing, priority queues, or simple task distribution (send email, process image).

---

## Spring Boot + Kafka

```java
// Producer
@Service
public class OrderProducer {
    private final KafkaTemplate<String, OrderEvent> kafkaTemplate;

    public void publishOrderCreated(Order order) {
        kafkaTemplate.send("orders.created", order.getId().toString(),
                new OrderEvent(order.getId(), "CREATED"));
    }
}

// Consumer
@Service
public class OrderConsumer {
    @KafkaListener(topics = "orders.created", groupId = "inventory-service")
    public void handleOrderCreated(OrderEvent event) {
        inventoryService.reserve(event.getOrderId());
    }
}
```

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    consumer:
      group-id: inventory-service
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
```

---

## Delivery guarantees

| Guarantee | Meaning | How |
|---|---|---|
| **At-most-once** | May lose messages | Ack before processing |
| **At-least-once** | May duplicate | Ack after processing (most common) |
| **Exactly-once** | No loss, no duplicate | Kafka transactions + idempotent consumers |

For at-least-once: make your consumer **idempotent** — processing the same message twice has the same effect as once (use a deduplication key in DB).

---

## Interview one-liner
> "A message queue decouples producers and consumers, enabling async processing and resilience. Use Kafka for high-throughput event streaming with replay capability; use RabbitMQ for flexible routing and simpler task queues. Most systems use at-least-once delivery, so consumers must be idempotent."
