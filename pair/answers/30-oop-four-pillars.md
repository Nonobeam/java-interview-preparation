# 34. The 4 Pillars of OOP

**Encapsulation, Inheritance, Polymorphism, Abstraction.**

---

## 1. Encapsulation
**Bundle state + behavior together; hide internals behind a controlled API.**

- Fields `private`, expose via getters/setters or behavior methods.
- Protects invariants, lets you change internals without breaking callers.

```java
public class BankAccount {
    private double balance;   // hidden

    public void deposit(double amount) {
        if (amount <= 0) throw new IllegalArgumentException();
        balance += amount;
    }
    public double getBalance() { return balance; }
}
```
Caller can't set `balance = -100` directly — the invariant is enforced.

---

## 2. Inheritance
**A subclass reuses and extends a superclass (`IS-A`).**

- Promotes code reuse and a type hierarchy.
- Overuse creates tight coupling — **favor composition over inheritance**.

```java
class Animal {
    void breathe() { System.out.println("breathing"); }
}
class Dog extends Animal {
    void bark() { System.out.println("woof"); }
}

Dog d = new Dog();
d.breathe();  // inherited
d.bark();     // own
```

---

## 3. Polymorphism
**"Many forms" — same interface, different implementations.**

Two kinds:

### Runtime (dynamic dispatch, method overriding)
```java
class Shape { double area() { return 0; } }
class Circle extends Shape { double area() { return Math.PI * r * r; } }
class Square extends Shape { double area() { return s * s; } }

Shape s = new Circle();   // reference type Shape, actual type Circle
s.area();                 // runs Circle.area() — resolved at runtime
```

### Compile-time (method overloading)
```java
int add(int a, int b)       { return a + b; }
double add(double a, double b) { return a + b; }
```

Power: write code against the base type; plug in new subclasses later without changing callers.

---

## 4. Abstraction
**Expose *what* something does; hide *how*.**

- Achieved via `abstract` classes and `interface`s.
- Callers depend on the contract, not the implementation.

```java
interface PaymentGateway {
    void charge(Money amount);
}

class StripeGateway  implements PaymentGateway { /* HTTP calls */ }
class PayPalGateway  implements PaymentGateway { /* HTTP calls */ }

class CheckoutService {
    private final PaymentGateway gateway;   // depends on abstraction
    CheckoutService(PaymentGateway g) { this.gateway = g; }
}
```
Swap Stripe → PayPal without touching `CheckoutService`.

---

## Encapsulation vs Abstraction — the subtle difference
- **Encapsulation** = *how* to hide (access modifiers, bundling data with behavior). Implementation-level.
- **Abstraction** = *what* to expose (designing a clean contract). Design-level.

## Quick recap table

| Pillar | Keyword | Purpose |
|---|---|---|
| Encapsulation | `private`, getters/setters | Protect invariants |
| Inheritance | `extends` | Reuse, type hierarchy |
| Polymorphism | overriding, overloading | Same API, many behaviors |
| Abstraction | `abstract`, `interface` | Hide complexity, code to contracts |
