# 35. SOLID Principles

SOLID is **5 design principles** (Robert C. Martin) for writing maintainable OO code. Not a pattern — a set of guidelines that lead to cleaner designs.

---

## S — Single Responsibility Principle (SRP)
**A class should have one and only one reason to change.**

Bad — does too much:
```java
class Invoice {
    void calculateTotal() { ... }
    void saveToDatabase()  { ... }   // persistence concern
    void sendEmail()       { ... }   // notification concern
}
```

Good — split responsibilities:
```java
class Invoice          { void calculateTotal() { ... } }
class InvoiceRepository{ void save(Invoice i)  { ... } }
class InvoiceMailer    { void send(Invoice i)  { ... } }
```

---

## O — Open/Closed Principle (OCP)
**Open for extension, closed for modification.** Add new behavior without editing existing code.

Bad — every new shape modifies this class:
```java
class AreaCalculator {
    double area(Object s) {
        if (s instanceof Circle c) return Math.PI * c.r * c.r;
        if (s instanceof Square q) return q.side * q.side;
        // add Triangle? edit this file again...
    }
}
```

Good — polymorphism lets you add `Triangle` without touching the calculator:
```java
interface Shape { double area(); }
class Circle implements Shape { public double area() { return Math.PI * r * r; } }
class Square implements Shape { public double area() { return side * side; } }

class AreaCalculator {
    double total(List<Shape> shapes) {
        return shapes.stream().mapToDouble(Shape::area).sum();
    }
}
```

---

## L — Liskov Substitution Principle (LSP)
**Subtypes must be substitutable for their base types without breaking behavior.**

Classic violation — `Square extends Rectangle` then `setWidth` also sets height → breaks callers that assume Rectangle semantics.

```java
class Rectangle { void setWidth(int w){...} void setHeight(int h){...} }
class Square extends Rectangle { // BAD — not substitutable
    void setWidth(int w)  { super.setWidth(w); super.setHeight(w); }
    void setHeight(int h) { super.setWidth(h); super.setHeight(h); }
}
```
Rule of thumb: **a subclass must not strengthen preconditions nor weaken postconditions.** If overriding makes the parent's contract a lie, it's an LSP violation.

---

## I — Interface Segregation Principle (ISP)
**Clients shouldn't depend on methods they don't use.** Prefer many small interfaces over one fat one.

Bad:
```java
interface Worker {
    void work();
    void eat();   // a RobotWorker doesn't eat — forced to implement
}
```

Good:
```java
interface Workable { void work(); }
interface Eatable  { void eat(); }

class Human implements Workable, Eatable { ... }
class Robot implements Workable          { ... }
```

---

## D — Dependency Inversion Principle (DIP)
**Depend on abstractions, not concretions.** High-level modules shouldn't depend on low-level modules; both depend on interfaces.

Bad — tightly coupled to MySQL:
```java
class OrderService {
    private MySqlOrderRepository repo = new MySqlOrderRepository();
}
```

Good — depends on abstraction, injected:
```java
interface OrderRepository { void save(Order o); }

class OrderService {
    private final OrderRepository repo;
    OrderService(OrderRepository repo) { this.repo = repo; }   // inject
}
```
Spring's IoC container is **DIP in practice** — the framework wires the concrete bean behind the interface.

---

## Quick recap

| Letter | Principle | One-line |
|---|---|---|
| S | Single Responsibility | One reason to change |
| O | Open/Closed | Extend, don't modify |
| L | Liskov Substitution | Subtype must honor base contract |
| I | Interface Segregation | Many small interfaces |
| D | Dependency Inversion | Depend on abstractions |

**Interview tip:** Tie SOLID back to real pain — e.g., "SRP is why we split `OrderService` from `OrderRepository`; otherwise a schema change and a business rule change both touch the same class."
