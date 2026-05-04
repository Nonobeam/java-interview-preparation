# 61. Generics & wildcards (`? extends`, `? super`)

## Why generics exist

Compile-time type safety + no casts.

```java
List<String> xs = new ArrayList<>();
xs.add("hi");
String s = xs.get(0);   // no cast
// xs.add(42);          // compile error
```

At runtime the JVM doesn't know `T` — generics are **erased** to their bound (`Object` by default). That's why you can't do `new T()`, `T.class`, or `instanceof T`.

## Invariance: `List<Dog>` is NOT a `List<Animal>`

```java
List<Dog> dogs = new ArrayList<>();
List<Animal> animals = dogs;   // compile error
```

If this were allowed, you could `animals.add(new Cat())` and break the dog list. So Java makes generics **invariant** by default. Wildcards are how you get back the flexibility safely.

## `? extends T` — upper-bounded (covariant), "producer"

```java
List<? extends Animal> animals = ...; // could be List<Animal>, List<Dog>, List<Cat>...

Animal a = animals.get(0);  // OK — whatever it is, it's an Animal
// animals.add(new Dog());  // compile error — could be a List<Cat>!
```

You can **read** as `T`, but you can't **write** (except `null`).

Use when the collection is a **source you read from**.

## `? super T` — lower-bounded (contravariant), "consumer"

```java
List<? super Dog> sink = ...; // could be List<Dog>, List<Animal>, List<Object>

sink.add(new Dog());          // OK — anything is at least an Object
sink.add(new Puppy());        // OK if Puppy extends Dog
// Dog d = sink.get(0);       // compile error — could be List<Object>
Object o = sink.get(0);       // only Object guaranteed
```

You can **write** `T` (and subtypes), but you can only **read** as `Object`.

Use when the collection is a **sink you write to**.

## PECS — "Producer Extends, Consumer Super"

The mnemonic for choosing.

```java
public static <T> void copy(List<? extends T> src, List<? super T> dst) {
    for (T x : src) dst.add(x);
}

copy(List.of(new Dog(), new Cat()), new ArrayList<Animal>()); // works
```

`src` produces `T`s → `extends`. `dst` consumes `T`s → `super`.

This is exactly the signature of `Collections.copy` in the JDK.

## Unbounded wildcard `?`

```java
void printAll(List<?> list) {
    for (Object o : list) System.out.println(o);
}
```

Use when you don't care about the element type — only `Object` operations and reading `null`.

## Type erasure gotchas

- You can't overload only by generic type: `void f(List<String>)` and `void f(List<Integer>)` clash — both erase to `f(List)`.
- You can't create generic arrays: `new T[10]` and `new List<String>[10]` are illegal.
- `instanceof List<String>` is illegal; `instanceof List<?>` is fine.
- Bridge methods are inserted by the compiler to preserve polymorphism after erasure.

## Bounded type parameter (different from wildcard)

```java
public static <T extends Comparable<T>> T max(List<T> list) {
    T best = list.get(0);
    for (T x : list) if (x.compareTo(best) > 0) best = x;
    return best;
}
```

Use a bounded type parameter (`<T extends ...>`) when the method needs to **refer to the type more than once** or **return** it. Wildcards are for single-use parameter slots.

## Interview one-liner

> "Generics give compile-time safety with type erasure at runtime. `? extends T` is for read-only producers, `? super T` is for write-only consumers — PECS, Producer Extends Consumer Super. Use a named type parameter `<T extends Bound>` when the method signature mentions the type more than once."
