# 36. The Java Collections Framework

## What it is
A unified architecture (in `java.util`) for storing and manipulating groups of objects. It provides:
- **Interfaces** — abstract data types (`List`, `Set`, `Queue`, `Map`, ...).
- **Implementations** — concrete classes (`ArrayList`, `HashMap`, ...).
- **Algorithms** — utility methods (`Collections.sort`, `Collections.binarySearch`, ...).

> Note: `Map` is part of the framework but does **not** extend `Collection` (it's a key→value pair, not a group of elements).

---

## The hierarchy

```
             Iterable
                │
            Collection
      ┌────────┼────────┐
     List     Set     Queue
                      │
                    Deque

Map  (separate root)
 ├─ SortedMap → NavigableMap
```

---

## Core interfaces & when to use

### `List` — ordered, allows duplicates, index-access
| Impl | Backed by | Strength | Watch out |
|---|---|---|---|
| `ArrayList` | resizable array | O(1) random access, fast iteration | O(n) insert/remove in middle |
| `LinkedList` | doubly-linked list | O(1) add/remove at ends | O(n) random access, high memory overhead |
| `Vector` | array (synchronized) | thread-safe | legacy, slow — use `Collections.synchronizedList` or `CopyOnWriteArrayList` instead |
| `CopyOnWriteArrayList` | array (copy on write) | safe for read-heavy concurrent use | expensive writes |

### `Set` — no duplicates
| Impl | Ordering | Notes |
|---|---|---|
| `HashSet` | none | O(1) avg add/contains, needs correct `hashCode/equals` |
| `LinkedHashSet` | insertion order | slightly slower than HashSet |
| `TreeSet` | sorted (natural / `Comparator`) | O(log n), backed by Red-Black tree |

### `Queue` / `Deque` — FIFO / double-ended
- `ArrayDeque` — fast general-purpose deque/stack (prefer over legacy `Stack`).
- `LinkedList` — also implements `Deque`.
- `PriorityQueue` — min-heap (by natural order or comparator).
- Concurrent: `ArrayBlockingQueue`, `LinkedBlockingQueue`, `ConcurrentLinkedQueue`.

### `Map` — key → value
| Impl | Ordering | Notes |
|---|---|---|
| `HashMap` | none | O(1) avg; allows one `null` key |
| `LinkedHashMap` | insertion or access order | good for LRU caches (override `removeEldestEntry`) |
| `TreeMap` | sorted by key | O(log n), `NavigableMap` operations |
| `Hashtable` | none, synchronized | legacy — use `ConcurrentHashMap` |
| `ConcurrentHashMap` | none | lock-striped; thread-safe without global lock |

---

## Common utility methods

```java
Collections.sort(list);              // natural order
Collections.sort(list, comparator);  // custom order
Collections.reverse(list);
Collections.binarySearch(sortedList, key);
Collections.unmodifiableList(list);
Collections.synchronizedMap(map);
Collections.emptyList();  Collections.emptyMap();
List.of(1,2,3);                      // immutable factory (Java 9+)
Map.of("k","v");                     // immutable factory
```

---

## `equals` / `hashCode` contract
Any object used as a **HashSet** element or **HashMap** key must:
1. Override `equals()` **and** `hashCode()` consistently — equal objects must return equal hash codes.
2. Be effectively immutable (or never mutated while stored) — mutating a key after insert corrupts the map.

---

## Iteration & fail-fast
- Most collections return **fail-fast** iterators — modifying the collection during iteration (except via `iterator.remove()`) throws `ConcurrentModificationException`.
- Concurrent collections (`ConcurrentHashMap`, `CopyOnWriteArrayList`) are **fail-safe** / **weakly consistent** — no CME, but you may not see every update.

---

## Big-O cheat sheet

| Op | ArrayList | LinkedList | HashMap | TreeMap |
|---|---|---|---|---|
| get(i) / get(key) | O(1) | O(n) | O(1) avg | O(log n) |
| add at end | O(1) amort. | O(1) | O(1) avg | O(log n) |
| insert middle / by key | O(n) | O(n) | O(1) avg | O(log n) |
| remove | O(n) | O(1) at node | O(1) avg | O(log n) |
| sorted iteration | no | no | no | yes |

---

## Picking one — quick decision tree
- Need order + duplicates + index access → `ArrayList`.
- Need unique, fast lookup, order doesn't matter → `HashSet`.
- Need sorted unique → `TreeSet`.
- Need key → value, order doesn't matter → `HashMap`.
- Need sorted map / range queries → `TreeMap`.
- Concurrent high-throughput map → `ConcurrentHashMap`.
- FIFO with blocking (producer-consumer) → `LinkedBlockingQueue`.
- Stack → `ArrayDeque` (not `Stack`).
