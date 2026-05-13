# Q72 — `HashMap` vs `HashSet` in Java

Note: This question is sometimes asked as "hashCode vs HashSet" — clarify the distinction: `hashCode()` is a method on every object; `HashMap` and `HashSet` are data structures that *use* it internally.

## Questions

1. What internal data structure does `HashMap` use? What internal data structure does `HashSet` use? (Hint: one backs the other — which direction?)
2. Walk through what happens when you call `hashSet.add(obj)` at the hash-table level: which method is called on `obj`, how is the bucket chosen, and how is collision handled?
3. Same question for `hashMap.put(key, value)`.
4. What is the `hashCode()` contract with `equals()`? Why must you override both together?
5. What happens to a `HashSet` or `HashMap` if you mutate a key object after inserting it?
6. `HashMap` vs `HashSet` — when do you reach for each one?
