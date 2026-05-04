# 59. The `hashCode()` contract

## Short answer
- `hashCode()` returns an `int` used by hash-based collections (`HashMap`, `HashSet`, `HashTable`) to find a bucket fast.
- The contract: **equal objects MUST have equal hash codes.** The reverse is not required (different objects may share a hash).
- If you override `equals()` and forget `hashCode()`, your objects break in hash-based collections — you can `put` then fail to `get`.

## The full contract

For any non-null references `x`, `y`:

1. **Consistency** — repeated calls to `x.hashCode()` in the same execution return the same int, as long as no field used in `equals()` has changed.
2. **Equals → equal hash** — if `x.equals(y)` is true, then `x.hashCode() == y.hashCode()` MUST be true.
3. **Unequal → not required** — if `x.equals(y)` is false, hashes MAY still be equal (a collision). Good hashes minimize collisions for performance, but correctness doesn't require it.

## What breaks if you violate it

```java
class User {
    private final String email;
    User(String email) { this.email = email; }

    @Override
    public boolean equals(Object o) {
        return o instanceof User u && email.equals(u.email);
    }
    // hashCode() NOT overridden — uses Object's identity-based default
}

Set<User> set = new HashSet<>();
set.add(new User("a@x.com"));
set.contains(new User("a@x.com")); // false  — different bucket!
```

`HashSet` first computes `hashCode()` to pick a bucket, then `equals()` only against entries in that bucket. Two "equal" users land in different buckets → invisible to each other.

## Idiomatic implementation

```java
import java.util.Objects;

class User {
    private final String email;
    private final String name;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof User u)) return false;
        return Objects.equals(email, u.email)
            && Objects.equals(name, u.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(email, name);
    }
}
```

Rules of thumb:
- Use the **same fields** in `equals()` and `hashCode()`.
- Use **immutable fields** when possible — if a field used in the hash mutates while the object lives in a `HashSet`/`HashMap` key, the entry becomes lost (still in old bucket, hash now points elsewhere).
- Lombok `@EqualsAndHashCode` or Java records (`record User(String email, String name) {}`) generate both correctly for free.

## Common interview gotchas
- "Can two unequal objects share a hashCode?" → Yes, that's a collision. Legal.
- "Why do mutable keys break HashMap?" → If you mutate a field used in the hash after `put`, the key's bucket changes, so `get` looks in the wrong bucket.
- "What does Object's default hashCode return?" → Typically derived from the object's memory address (identity hash). That's why two distinct-but-equal objects produce different hashes if you forget to override.

## Interview one-liner

> "Equal objects must produce equal hash codes — that's the only hard rule. Collisions are allowed but should be rare for performance. Always override `hashCode()` and `equals()` together using the same immutable fields, or use `record` / Lombok to get it for free."
