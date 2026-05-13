# 72. `HashMap` vs `HashSet` in Java

## The key relationship

`HashSet` is backed by a `HashMap`. When you add an element to a `HashSet`, it calls `hashMap.put(element, PRESENT)` where `PRESENT` is a dummy static `Object`. The `HashSet` is literally a thin wrapper.

```java
// Inside HashSet source
private transient HashMap<E,Object> map;
private static final Object PRESENT = new Object();

public boolean add(E e) {
    return map.put(e, PRESENT) == null;
}
```

## Internal data structure — the hash table

Both rely on the same structure: an **array of buckets** (also called a table).

```
index:  0    1    2    3    4    5    6    7
       [ ]  [ ]  [A]  [ ]  [B→C] [ ]  [ ]  [D]
                  ↑           ↑              ↑
               single    collision:      single
               entry     linked list
```

Each bucket holds a linked list (Java 8+: converts to a **red-black tree** when a bucket has ≥ 8 entries, for O(log n) worst case instead of O(n)).

## How `hashMap.put(key, value)` works step by step

1. Call `key.hashCode()` — get an integer hash.
2. Apply a spread function: `(n-1) & hash` → bucket index in the array.
3. If the bucket is empty, store the entry there directly.
4. If the bucket has entries (collision):
   - Walk the linked list, comparing with `key.equals(existing_key)`.
   - If equal → **replace** the value.
   - If no match → **append** to the list.
5. If load factor exceeded (`size / capacity > 0.75` by default) → **resize**: double the array, rehash all entries.

## How `hashSet.add(obj)` works

Exactly the same — it just calls `map.put(obj, PRESENT)`. Whether the add is a "new entry" or "already present" is determined by `hashCode()` + `equals()` on `obj`.

## The `hashCode()` + `equals()` contract

Two rules you must never break:

1. If `a.equals(b)` then `a.hashCode() == b.hashCode()`
2. If `a.hashCode() == b.hashCode()` then `a.equals(b)` may or may not be true (collision is allowed)

**Why override both together:** If you override `equals()` without `hashCode()`, two objects that are logically equal may hash to different buckets — the map/set will treat them as different keys and you'll get duplicate entries or missed lookups.

```java
// BAD — overrides equals but not hashCode
class Point {
    int x, y;
    @Override public boolean equals(Object o) { ... }
    // missing hashCode → broken in HashMap/HashSet
}

// GOOD
@Override public int hashCode() {
    return Objects.hash(x, y);
}
```

## What happens if you mutate a key after inserting it

The bucket index was computed from the *old* hash. After mutation, the new hash points to a different bucket. The entry is now effectively lost — `get()` and `contains()` will return null/false even though the object is in the map.

**Rule:** Never use mutable objects as `HashMap` keys or `HashSet` elements. Prefer immutable types (`String`, `Integer`, value records).

## Comparison table

| | `HashMap` | `HashSet` |
|---|---|---|
| Stores | key-value pairs | unique elements |
| Internal table | `Entry<K,V>[]` array | `Entry<E,Object>[]` (backed by `HashMap`) |
| Duplicate handling | Same key → replaces value | Same element → ignored (add returns false) |
| Allows null | One null key, many null values | One null element |
| Use case | Lookup by key | Membership test / deduplication |
| Thread-safe? | No (use `ConcurrentHashMap`) | No (use `Collections.synchronizedSet` or `ConcurrentHashMap.newKeySet()`) |

## When to use each

- **`HashMap`**: you need to associate a value with a key — caches, frequency counts, grouping.
- **`HashSet`**: you only care whether something exists — deduplication, visited-node sets in graph traversal, fast membership checks.

## Interview one-liner

> "`HashSet` is backed by a `HashMap` — adding an element calls `map.put(element, DUMMY)`. Both use a hash table internally: an array of buckets, with linked lists (or red-black trees on collision). The bucket index is derived from `hashCode()`; within the bucket, `equals()` decides identity. You must always override both together: same logical value must hash to the same bucket, or `HashMap`/`HashSet` breaks silently."
