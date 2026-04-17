# 54. Java Streams & Lambda

Streams provide a declarative way to process collections. Lambdas are the anonymous functions that power them.

---

## Lambda expressions

A lambda is a concise way to implement a functional interface (an interface with exactly one abstract method).

```java
// Before Java 8
Comparator<String> comp = new Comparator<String>() {
    @Override
    public int compare(String a, String b) { return a.compareTo(b); }
};

// Lambda
Comparator<String> comp = (a, b) -> a.compareTo(b);

// Method reference (even shorter when calling an existing method)
Comparator<String> comp = String::compareTo;
```

Common functional interfaces:
```java
Predicate<T>       // T → boolean        (filter)
Function<T, R>     // T → R              (map)
Consumer<T>        // T → void           (forEach)
Supplier<T>        // () → T             (lazy value)
BiFunction<T,U,R>  // T, U → R
UnaryOperator<T>   // T → T
```

---

## Stream pipeline

```
Source → [Intermediate operations] → Terminal operation
```

**Lazy:** intermediate operations don't execute until a terminal operation is called.

```java
List<Order> orders = orderRepo.findAll();

// Find total revenue from CONFIRMED orders with amount > 1000
BigDecimal revenue = orders.stream()              // source
    .filter(o -> o.getStatus() == CONFIRMED)      // intermediate — lazy
    .filter(o -> o.getTotal().compareTo(BigDecimal.valueOf(1000)) > 0)
    .map(Order::getTotal)                         // intermediate — lazy
    .reduce(BigDecimal.ZERO, BigDecimal::add);    // terminal — triggers pipeline
```

---

## Common operations

### Intermediate (return Stream)

```java
// filter — keep elements matching predicate
stream.filter(o -> o.getStatus() == CONFIRMED)

// map — transform each element
stream.map(Order::getTotal)

// flatMap — flatten nested collections
stream.flatMap(o -> o.getItems().stream())

// sorted
stream.sorted(Comparator.comparing(Order::getCreatedAt).reversed())

// distinct
stream.distinct()

// limit / skip
stream.limit(10).skip(5)

// peek — for debugging (does not consume)
stream.peek(o -> log.debug("Processing: {}", o.getId()))
```

### Terminal (consume Stream)

```java
// collect
List<Long> ids = stream.map(Order::getId).collect(Collectors.toList());
// or Java 16+:
List<Long> ids = stream.map(Order::getId).toList();

// groupingBy
Map<OrderStatus, List<Order>> byStatus =
    orders.stream().collect(Collectors.groupingBy(Order::getStatus));

// counting
long count = orders.stream().filter(...).count();

// reduce
BigDecimal total = orders.stream()
    .map(Order::getTotal)
    .reduce(BigDecimal.ZERO, BigDecimal::add);

// findFirst / findAny
Optional<Order> first = orders.stream().filter(...).findFirst();

// anyMatch / allMatch / noneMatch
boolean hasOverdue = orders.stream().anyMatch(Order::isOverdue);

// forEach
orders.stream().forEach(o -> emailService.notify(o));
// but for side effects, prefer regular for-each loop over streams
```

---

## Optional

Avoids NullPointerException for absent values:

```java
Optional<Order> opt = orderRepo.findById(id);

// Bad — don't use get() without check
Order o = opt.get();  // throws NoSuchElementException if empty

// Good
Order o = opt.orElseThrow(() -> new OrderNotFoundException(id));
Order o = opt.orElse(Order.empty());
opt.ifPresent(order -> emailService.notify(order));

// Chaining
String city = opt
    .map(Order::getDeliveryAddress)
    .map(Address::getCity)
    .orElse("Unknown");
```

---

## Parallel streams

```java
long count = orders.parallelStream()
    .filter(o -> o.getTotal().compareTo(BigDecimal.valueOf(10000)) > 0)
    .count();
```

Uses the common ForkJoinPool. Only beneficial for CPU-intensive work on large datasets. Do not use for:
- IO-bound work (blocking threads in the common pool)
- Small collections (overhead exceeds benefit)
- Operations with side effects (thread-safety issues)

---

## Method references

```java
ClassName::staticMethod      // String::valueOf
instance::instanceMethod     // order::getTotal  (bound)
ClassName::instanceMethod    // Order::getTotal  (unbound — first arg is receiver)
ClassName::new               // Order::new       (constructor)
```

---

## Common collectors

```java
Collectors.toList()
Collectors.toSet()
Collectors.toMap(Order::getId, Order::getTotal)
Collectors.groupingBy(Order::getStatus)
Collectors.groupingBy(Order::getStatus, Collectors.counting())
Collectors.joining(", ", "[", "]")   // for String streams
Collectors.partitioningBy(Order::isOverdue)  // Map<Boolean, List<Order>>
```

---

## Interview one-liner
> "Streams provide a declarative, lazy pipeline over collections: filter → map → collect. Lambdas implement functional interfaces inline. Key points: streams are lazy (nothing runs until a terminal operation), parallel streams use ForkJoinPool (good for CPU work, not IO), and Optional wraps nullable results to force explicit null handling."
