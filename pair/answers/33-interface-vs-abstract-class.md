# 33. `interface` vs `abstract class`

## Short answer
- **Abstract class** → partial implementation + state. A class can extend only **one**.
- **Interface** → a contract (capability). A class can implement **many**.

## Side-by-side

| Feature | Abstract class | Interface |
|---|---|---|
| Instantiation | Cannot instantiate | Cannot instantiate |
| Inheritance | Single (`extends ONE`) | Multiple (`implements MANY`) |
| Fields | Instance fields, any modifier | Only `public static final` (constants) |
| Methods | Abstract + concrete, any modifier | Abstract, `default`, `static`, `private` (Java 8/9+); implicitly `public` |
| Constructors | Yes (called by subclass) | No |
| State | Yes (fields with values) | No instance state |
| Use for | "IS-A" with shared code/state | "CAN-DO" capability/contract |

## Examples

### Abstract class — shared code + state
```java
abstract class Shape {
    private final String name;            // state
    Shape(String name) { this.name = name; }
    public String getName() { return name; }   // concrete
    public abstract double area();             // must be implemented
}

class Circle extends Shape {
    private final double r;
    Circle(double r) { super("Circle"); this.r = r; }
    public double area() { return Math.PI * r * r; }
}
```

### Interface — capability/contract
```java
interface Flyable { void fly(); }
interface Swimmable { void swim(); }

class Duck implements Flyable, Swimmable {   // multiple
    public void fly()  { System.out.println("flying"); }
    public void swim() { System.out.println("swimming"); }
}
```

### Interface `default` and `static` methods (Java 8+)
```java
interface Vehicle {
    void start();
    default void honk() { System.out.println("beep"); }  // default impl
    static Vehicle nullObject() { return () -> {}; }     // static factory
}
```

### Interface `private` method (Java 9+)
Used to share code between `default` methods without exposing it.

## When to pick which
- Need **fields / constructors / partial implementation** → abstract class.
- Need **multiple inheritance of type** → interface.
- Unrelated classes sharing a capability (e.g., `Comparable`, `Serializable`) → interface.
- Designing a framework base with template-method pattern → abstract class.
- **Modern trend:** prefer interfaces (with `default` methods) for flexibility; use abstract classes only when you truly need state or constructors.

## Gotchas
- Interface fields are implicitly `public static final` — they are **constants**, not instance state.
- Interface methods are implicitly `public`; you can't weaken to `protected` in the implementing class.
- Diamond problem with default methods: if two interfaces provide the same default, the implementing class **must** override it.
- An abstract class can have **zero** abstract methods (still can't be instantiated).
