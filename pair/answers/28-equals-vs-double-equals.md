# 32. `equals()` vs `==` in Java

## Short answer
- `==` compares **references** (for objects) or **primitive values** (for primitives).
- `equals()` compares **logical/content equality**, as defined by the class's implementation.

## `==` operator
- For **primitives** (`int`, `long`, `double`, `boolean`, ...): compares the actual values.
  ```java
  int a = 5, b = 5;
  a == b; // true
  ```
- For **objects** (reference types): compares memory addresses (are they the **same** object in the heap?).
  ```java
  String s1 = new String("hello");
  String s2 = new String("hello");
  s1 == s2;      // false — two different objects
  s1.equals(s2); // true  — same content
  ```

## `equals()` method
- Defined on `java.lang.Object`. Default implementation is `return this == obj;` (reference equality).
- Classes override it to give **meaningful equality** based on fields.
  - `String.equals()` → compares characters.
  - `Integer.equals()` → compares int values.
  - `List.equals()` → compares elements in order.
- Must be overridden together with `hashCode()` to keep the contract: equal objects must have equal hash codes (needed by `HashMap`, `HashSet`).

## Gotchas

### 1. String literal pool
```java
String s1 = "hello";
String s2 = "hello";
s1 == s2;      // true  — both point to the same interned literal
s1.equals(s2); // true
```
vs
```java
String s3 = new String("hello");
s1 == s3;      // false — new object on the heap
s1.equals(s3); // true
```

### 2. Integer autoboxing cache (`-128..127`)
```java
Integer a = 100, b = 100;
a == b; // true  — cached
Integer c = 200, d = 200;
c == d; // false — outside cache, new objects
c.equals(d); // true
```

### 3. `null` safety
- `a == null` is safe.
- `a.equals(null)` throws `NullPointerException` if `a` is null. Use `Objects.equals(a, b)` to null-safely compare.

## Rule of thumb
- Comparing primitives → `==`.
- Comparing object **content/value** → `equals()` (or `Objects.equals()` for null safety).
- Comparing object **identity** (same instance) → `==`.

## Interview bonus: the `equals()` contract
For any non-null references `x`, `y`, `z`:
- **Reflexive:** `x.equals(x)` is true.
- **Symmetric:** `x.equals(y) == y.equals(x)`.
- **Transitive:** if `x.equals(y)` and `y.equals(z)`, then `x.equals(z)`.
- **Consistent:** repeated calls return the same result (no mutated fields).
- **Null:** `x.equals(null)` is false.
- Always override `hashCode()` alongside.
