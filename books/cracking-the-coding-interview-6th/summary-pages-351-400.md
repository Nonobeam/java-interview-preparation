# Cracking the Coding Interview (6th Edition) — Pages 351–400 Summary

Covers the **end of Chapter 7 Solutions** (7.12 Hash Table) and the bulk of **Chapter 8 Solutions** — Recursion and Dynamic Programming (8.1–8.13 plus the start of 8.14).

> Pages 344–350 (7.10 Minesweeper, 7.11 File System) were skipped by the prior file; they are summarized briefly in the appendix at the end of this file so the OOD chapter is complete.

---

## Solutions to Chapter 7 (final)

### 7.12 Hash Table with Chaining

Naive sketch: `LinkedList<V>[] items` keyed by `hash(key) % length`. **Bug**: collisions need to disambiguate the original key, but only the value is stored. Fix: store a `Cell` (key + value) in each linked-list node.

Final design:

| Member | Purpose |
|---|---|
| `LinkedListNode<K,V>` (doubly linked) | `key`, `value`, `next`, `prev` |
| `ArrayList<LinkedListNode<K,V>> arr` | bucket array sized to `capacity` |
| `getNodeForKey(key)` | walk bucket comparing `current.key == key` |
| `getIndexForKey(key)` | `Math.abs(key.hashCode() % arr.size())` |

`put`: if node already exists, overwrite value; else create node, prepend to bucket head (`node.next = arr.get(index); node.next.prev = node`).

`remove`: if `node.prev == null`, the node is the bucket head — update `arr.set(hashKey, node.next)`. Otherwise relink `prev`/`next`.

**Alternative** mentioned: use a BST as the underlying structure instead of an array of linked lists. Lookup becomes `O(log N)` but avoids over-sizing the array.

---

## Solutions to Chapter 8 — Recursion and Dynamic Programming

Recurring recipe across the entire chapter:

1. **Brute-force recursion** based on a "what is the last decision?" framing.
2. Detect overlapping subproblems → **memoize** (top-down DP).
3. Sometimes a third pass simplifies further (e.g., 8.5) by removing redundant branches entirely.

### 8.1 Triple Step (1, 2, or 3 steps)

`countWays(n) = countWays(n-1) + countWays(n-2) + countWays(n-3)`. Base: `countWays(0) = 1` (chosen for convenience — `0` would force extra base cases).

- Brute force: `O(3^n)` (each call branches 3-way).
- Memoized: `O(n)` time/space using `int[n+1]` initialized to `-1`.
- **Communicate the overflow**: `int` overflows at `n = 37`; `long` only delays it. `BigInteger` is the principled fix.

### 8.2 Robot in a Grid (right + down only, with blocked cells)

Reverse direction: build path from destination back to origin. To reach `(r,c)`, must reach `(r-1,c)` or `(r,c-1)`.

- Brute force: `O(2^(r+c))` — every cell visited many times.
- Memoize using a `HashSet<Point> failedPoints` of cells already proven unreachable. Now `O(rc)` since each cell is visited once.

> Sidebar: prefer `(row, col)` over `(x, y)` — `matrix[x][y]` is a common bug because the first index is the row (vertical, conceptually `y`).

### 8.3 Magic Index — `A[i] == i` in a sorted array

**Distinct values**: binary search works. If `A[mid] > mid`, the magic index can't be on the right (values grow at least as fast as indices, so the gap only widens). Recurse left. Symmetric argument for `A[mid] < mid`.

**Follow-up — duplicates allowed**: a single comparison no longer rules out either side. Example: `[-10,-5,2,2,2,3,4,7,9,12,13]` — `A[5]=3` so the magic index is on the *left*, not right. But: `A[4]` can't be magic because `A[4] ≤ A[5] = 3 < 4`. General rule:

- Left side: search `[start, min(midIndex - 1, midValue)]`.
- Right side: search `[max(midIndex + 1, midValue), end]`.

Reduces to ordinary binary search when all values are distinct.

### 8.4 Power Set

Lower bound: `O(n · 2^n)` for time and space — there are `2^n` subsets and total elements across them is `n · 2^(n-1)`.

**Approach 1 — Base Case and Build (recursive)**: `P(n) = P(n-1) ∪ { s ∪ {a_n} : s ∈ P(n-1) }`. Clone every subset of `P(n-1)` and add `a_n`.

**Approach 2 — Combinatorial / bit mask**: each subset ↔ a binary number in `[0, 2^n)`. Iterate `k = 0..2^n-1`; bit `i` of `k` decides whether `set[i]` is included. Slick, no recursion, same asymptotic cost.

### 8.5 Recursive Multiply (no `*` or `/`)

Three progressively better solutions on `minProduct(a, b)` where `smaller = min(a,b)`:

1. **Halve and double** (`O(log s)` calls but with **duplicate work** for odd numbers): split smaller into `s/2` + `(s - s/2)`; if even, double one side; if odd, recurse twice with different parameters.
2. **Memoize** the duplicate calls.
3. **Direct recurrence — no memo needed**: `minProduct(s, b) = 2 * minProduct(s/2, b) + (s odd ? b : 0)`. Strictly downward recursion, never repeats. `O(log s)`.

Lesson: removing the duplicate branch entirely beats caching it.

### 8.6 Towers of Hanoi

Pure Base Case and Build. To move `n` from origin → destination using buffer:

```
move(n-1, origin → buffer, using destination as buffer)
move 1 disk:  origin → destination
move(n-1, buffer → destination, using origin as buffer)
```

Implementation hint: model `Tower` as an object with a `Stack<Integer>`. The recursion lives on the tower itself — `origin.moveDisks(n, dest, buffer)`.

### 8.7 Permutations without Dups

**Approach 1 — strip first char**: `P(s) = insert s[0] into every position of every permutation of s[1..]`. Recurse on the suffix.

**Approach 2 — choose first char**: `P(s) = ⋃_i { s[i] + p : p ∈ P(s without index i) }`.

**Variant on Approach 2 — push prefix down** instead of pulling permutations up. When `remainder` is empty, `prefix` contains a complete permutation. Cleaner stack semantics; same asymptotics.

Runtime analysis lives in Big-O Example 12 (page 51).

### 8.8 Permutations with Duplicates

Naive: generate all `n!` then dedupe with a hash set. Worst case still `n!`.

**Better — frequency-counted recursion**: build a `HashMap<Character, Integer>` of counts. At each level, iterate keys with `count > 0`, decrement, recurse with `prefix + c, remaining - 1`, then increment back (classic backtracking restore). Generates *only* unique permutations, so `aaaa…a` runs in one call instead of 6 billion.

### 8.9 Parens — all valid combinations of `n` pairs

**Naive**: build `f(n)` from `f(n-1)` by inserting `()` after each existing `(` and at the start. Generates duplicates (e.g., `()(())` twice) → must dedupe with a `HashSet`.

**Better — left/right counters**: at each character slot, choose `(` if any left remain; choose `)` only if `rightRem > leftRem` (i.e., there are more open `(` than `)` placed so far). Each call writes a unique character into a fixed `char[2n]` buffer, so every produced string is automatically distinct.

```
addParen(list, leftRem, rightRem, str, index):
  if leftRem < 0 or rightRem < leftRem: return  // invalid
  if leftRem == 0 and rightRem == 0: emit str
  else: try '(' (leftRem-1) and try ')' (rightRem-1)
```

### 8.10 Paint Fill

DFS outwards from the click point. Stop when out of bounds or color ≠ original. Two-phase entry: outer call captures `screen[r][c]` as `oColor`, then inner recursion replaces with `nColor` and recurses U/D/L/R.

> Same `(r, c)` vs `(x, y)` warning: `screen[y][x]` because `x` is the column.

This is essentially DFS on an implicit grid graph; BFS is an equally valid choice.

### 8.11 Coins — number of ways to make `n` cents from {25, 10, 5, 1}

`makeChange(amount, denoms, index)` = sum over `i = 0..amount/denomAmount` of `makeChange(amount - i*denom, denoms, index+1)`. Base case: `index == denoms.length - 1` → 1 way (only pennies left).

**Memoize on `(amount, index)`** — `int[n+1][denoms.length]`. Prevents recomputing the many shared subproblems (e.g., 75 cents with 0 quarters appears under multiple parent paths).

### 8.12 Eight Queens

Place one queen per row. State is `Integer[] columns` where `columns[r] = c`. For each row `r`, try every column `c`; `checkValid` walks rows `0..r-1` and rejects if same column or same diagonal (`|c - columns[r2]| == r - r2`).

Don't store the board as 8×8 — one queen per row means a single int per row suffices.

### 8.13 Stack of Boxes (strictly larger in W/H/D)

**Pre-step**: sort boxes descending by one dimension (e.g., height). Then the bottom of any valid sub-stack must come *earlier* in the sorted list — no need to look backward.

**Solution 1 — bottom-anchored**: `createStack(boxes, bottomIndex)` = `bottom.height + max( createStack(boxes, i) for i > bottomIndex if boxes[i].canBeAbove(bottom) )`. Memoize on `bottomIndex`.

**Solution 2 — include/exclude**: at each offset, recurse twice — one path puts box `offset` into the stack as the new bottom, the other skips it. Take the max. Also memoized on `offset`.

> Memoization hygiene: the line that **reads** the cache and the line that **writes** it should be symmetric (here both keyed on `bottomIndex` / `offset`). Mismatched read/write keys are a classic DP bug.

### 8.14 Boolean Evaluation (start)

Setup only on these pages — full algorithm continues past page 400.

`countEval(expr, result)` = sum over each operator position `i` of `combinations(left, right, op, result)` where `left = countEval(expr[0..i])` for each truth value and similarly for the right. Each operator (`&`, `|`, `^`) determines which `(leftValue, rightValue)` pairs produce the desired `result`. Memoize on `(expr, result)`.

---

## Recurring DP Patterns From These Solutions

| Pattern | Where Used |
|---|---|
| Last-decision recurrence | Triple Step, Coins, Boolean Evaluation |
| Memoize a `HashSet` of failures | Robot in a Grid (`failedPoints`) |
| Bit-mask enumeration of subsets | Power Set #2 |
| Backtracking with state restore | Permutations w/ Dups, Eight Queens |
| Constraint-driven generation (avoid duplicates by construction) | Parens (left/right counters), Permutations w/ Dups |
| Sort + only-look-forward | Stack of Boxes |
| Symmetry of cache read & write | Stack of Boxes (warning called out explicitly) |
| Eliminate duplicate branches *instead of* caching | Recursive Multiply #3 |

---

## Interview Takeaways

1. **Talk about overflow even when not asked to fix it** (Triple Step). It signals seniority.
2. **Wrong solutions teach the framing** — Power Set, Magic Index follow-up, Parens — the book repeatedly walks through the naive approach before pivoting. Mirror this in interviews.
3. **Right vs. wrong axis labels**: `matrix[row][col]` over `matrix[x][y]`. Easy to miscount under pressure.
4. **Choose the convenient base case**: `countWays(0) = 1` not `0` — your future self will thank you.
5. **Construct unique outputs rather than dedupe** when possible (Parens left/right counters; frequency-map permutations). Dedupe is `O(work to make duplicates)` wasted.
6. **Memoization is mechanical once the recurrence is right** — most of the chapter is "now add a `memo` array". Spend time on the recurrence, not the cache.
7. **Sometimes the third pass beats memoization** (Recursive Multiply): if you can structure recursion to never repeat itself, you don't need a cache at all.

---

## Appendix — 7.10 / 7.11 (pages 344–350, not in prior file)

### 7.10 Minesweeper (text-based)

Two-phase board setup:
- **Place bombs**: shuffle then assign first `B` cells as bombs.
- **Set numbers**: rather than scanning every cell and counting nearby bombs, walk *each bomb* and increment its 8 neighbors. Cleaner and fewer ops.

**Expanding a blank region** — iterative BFS with a `Queue<Cell>`. For each dequeued blank, flip neighbors; if a neighbor is itself blank, enqueue it. Recursive variant works too but risks deep stacks on large boards.

### 7.11 File System (in-memory)

Class hierarchy:
- `Entry` (abstract) — `name`, `parent`, timestamps, `delete()`, `getFullPath()`, abstract `size()`.
- `File extends Entry` — `content`, `size`.
- `Directory extends Entry` — `ArrayList<Entry> contents`, `size()` recurses, `numberOfFiles()` uses `instanceof` to count.

**Trade-off discussed**: storing files and subdirectories in *separate* lists removes the `instanceof` check in `numberOfFiles` but prevents uniformly sorting children by date or name.
