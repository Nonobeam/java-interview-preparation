# 53. Domain-Driven Design (DDD)

DDD is a software design approach that models code around the **business domain** rather than technical concerns. The domain model is the center of the application; infrastructure (DB, HTTP, queues) is a detail.

---

## Strategic design (the big picture)

### Bounded Context
A boundary within which a specific domain model applies. The same word can mean different things in different contexts.

```
Order Management Context    ─── "Order" = items, total, payment status
Shipping Context            ─── "Order" = shipment, tracking, address
Inventory Context           ─── "Order" = reservation, stock reduction
```

Each bounded context has its own model, its own database, and communicates via events or APIs — never by sharing tables.

### Ubiquitous Language
The business and developers use the **same terms**. If the business says "invoice", the code says `Invoice`, not `BillingDocument`. This eliminates translation errors.

---

## Tactical design (the building blocks)

### Entity
Has a unique identity that persists over time. Identity matters more than attribute values.

```java
@Entity
public class Order {
    @Id
    private Long id;           // identity — what makes it "this Order"
    private OrderStatus status;
    private List<OrderItem> items;
}
```

Two orders with the same items are still two different orders.

### Value Object
Defined entirely by its attributes — no identity. Immutable. Interchangeable if values are equal.

```java
@Embeddable
public record Money(BigDecimal amount, String currency) {
    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) throw new IllegalArgumentException();
        return new Money(this.amount.add(other.amount), this.currency);
    }
}

@Embeddable
public record Address(String street, String city, String zipCode) {}
```

### Aggregate & Aggregate Root
A cluster of entities and value objects treated as a single unit. The **Aggregate Root** is the only entry point — external objects can only reference the root, not internal entities.

```java
@Entity
public class Order {   // Aggregate Root
    @Id private Long id;

    @OneToMany(cascade = ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();  // internal entity

    public void addItem(Product product, int quantity) {
        // All business rules enforced here
        if (status != DRAFT) throw new IllegalStateException("Cannot modify confirmed order");
        items.add(new OrderItem(product.getId(), quantity, product.getPrice()));
    }

    public void confirm() {
        if (items.isEmpty()) throw new IllegalStateException("Cannot confirm empty order");
        this.status = CONFIRMED;
        registerEvent(new OrderConfirmedEvent(this.id));
    }
}
```

- Never modify `OrderItem` directly from outside — always go through `Order`.
- One repository per aggregate root.

### Repository
Abstracts persistence. Returns fully-formed aggregates. The domain doesn't know about SQL.

```java
public interface OrderRepository {
    Order findById(Long id);
    void save(Order order);
}
```

### Domain Service
Business logic that doesn't naturally belong to one entity or value object.

```java
@Service
public class TransferService {
    public void transfer(Account from, Account to, Money amount) {
        from.debit(amount);    // debit belongs on Account
        to.credit(amount);     // credit belongs on Account
        // but the transfer operation spans two aggregates → domain service
    }
}
```

### Domain Event
Something that happened in the domain — past tense, immutable.

```java
public record OrderConfirmedEvent(Long orderId, Instant occurredAt) {}
```

Events decouple bounded contexts: Order context publishes `OrderConfirmedEvent`; Inventory and Shipping contexts subscribe independently.

---

## DDD layers

```
┌─────────────────────────────────┐
│         Interfaces              │  REST controllers, consumers, schedulers
├─────────────────────────────────┤
│         Application             │  Use cases, orchestration (@Service)
├─────────────────────────────────┤
│         Domain                  │  Entities, Value Objects, Aggregates,
│                                 │  Domain Services, Domain Events
├─────────────────────────────────┤
│         Infrastructure          │  JPA repos, Kafka, external APIs
└─────────────────────────────────┘
```

Domain layer has **zero** infrastructure dependencies — no `@Repository`, no `JpaRepository`.

---

## When DDD is worth it

- Complex business rules with many domain concepts
- Large team where shared language matters
- Long-lived system expected to grow

DDD adds overhead. For simple CRUD apps, it's over-engineering.

---

## Interview one-liner
> "DDD organizes code around the business domain. The key building blocks are: Entities (identity-based), Value Objects (attribute-based, immutable), Aggregates (consistency boundary with a single Root), Repositories (persistence abstraction), Domain Services (cross-aggregate logic), and Domain Events (decouple bounded contexts). The goal is a model that matches how the business actually thinks."
