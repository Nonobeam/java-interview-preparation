# Cracking the Coding Interview (6th Edition) — Pages 201–300 Summary

Covers the end of Chapter 17 (Hard problems list) and the **Solutions** section for Chapters 1–4 (Arrays/Strings, Linked Lists, Stacks/Queues, and most of Trees/Graphs).

---

## Chapter 17 (continued): Remaining Hard Problems

Highlights from 17.17–17.26:

| # | Problem | Key Technique |
|---|---|---|
| 17.17 | Multi Search | Trie of T, scan b; or suffix trie of b |
| 17.18 | Shortest Supersequence | Sliding window with pointers per element in shorter array |
| 17.19 | Missing Two | Sum + sum-of-squares, or XOR with bit split |
| 17.20 | Continuous Median | Two heaps (max-heap lower half, min-heap upper half) |
| 17.21 | Volume of Histogram | For each bar: min(max_left, max_right) − height |
| 17.22 | Word Transformer | BFS over dictionary graph (edges = one-char-diff) |
| 17.23 | Max Black Square | Precompute row/col black runs; scan sizes largest-first |
| 17.24 | Max Submatrix | Kadane's on compressed row-range sums |
| 17.25 | Word Rectangle | Trie + DP over partial grids, largest area first |
| 17.26 | Sparse Similarity | Inverted index (element → docs), sum overlaps per pair |

---

## Solutions to Chapter 1 — Arrays and Strings

### 1.1 Is Unique — `O(N)` time, `O(1)` space

Boolean array of size 128 (ASCII). Return false on re-seeing any char, or if `length > 128`.

**Bit vector variant**: single int, toggle bit `c - 'a'` — 8× less space.

**Without data structures**: compare every pair `O(N²)`, or sort first `O(N log N)`.

> Clarify: ASCII vs Unicode? Case-sensitive? These questions score points.

### 1.2 Check Permutation

Two equally good solutions:
1. **Sort both, compare** — `O(N log N)`, simple and clean.
2. **Character count** — `O(N)`; increment for s, decrement for t, check all zero.

Early exit: unequal lengths → not permutations.

### 1.3 URLify — in-place replacement

**Two-scan reverse approach**:
1. First scan: count spaces → final length = `trueLength + 2 * spaceCount`.
2. Second scan, right-to-left: copy chars or insert `%20`.

Reverse iteration avoids overwriting unread chars.

### 1.4 Palindrome Permutation

**Key insight**: a palindrome can have at most one character with an odd count.

| Approach | Complexity |
|---|---|
| Count chars in hash table, verify ≤ 1 odd | O(N) time, O(1) extra pass |
| Single scan, track `countOdd` on the fly | O(N), slightly slower constants |
| **Bit vector toggle + `(v & (v-1)) == 0` check** | O(N), elegant — vector represents which chars have odd parity |

### 1.5 One Away

Define semantics: replacement differs in one position; insert/remove = identical with a single index shift.

Merge insert and replace into one pass: `oneEditAway` walks both strings with a `foundDifference` flag; advance one pointer on match, advance both on replace, advance only the longer pointer on insert.

**Runtime: `O(n)` where n is the shorter string** (different lengths by >1 → O(1) early exit).

### 1.6 String Compression

First-pass `String` concat is `O(p + k²)` due to string immutability — use `StringBuilder`.

**Pre-compute final length** before building → decide early whether to return the original (if compression wouldn't shrink). Avoids wasted allocation.

### 1.7 Rotate Matrix (90° in place)

**Layer-by-layer swap**. For each layer and each offset `i`:
```
temp        = top[i]
top[i]      = left[i]
left[i]     = bottom[i]
bottom[i]   = right[i]
right[i]    = temp
```
Loop `layer < n/2`. O(N²) — optimal, since all N² cells move.

### 1.8 Zero Matrix

**O(1) space trick**: use first row and first column as marker arrays; remember separately whether the first row/column originally contained a zero.

1. Scan if first row or first col has a zero.
2. For each interior zero `matrix[i][j] = 0`, mark `matrix[i][0] = 0` and `matrix[0][j] = 0`.
3. Nullify rows/cols based on markers.
4. Nullify first row/col last, if needed.

### 1.9 String Rotation

`s2` is a rotation of `s1` iff `s2` is a substring of `s1+s1`.
- `s1 = xy`, so `s1s1 = xyxy` always contains `yx = s2`.
- One call to `isSubstring`, `O(A+B)` time.

---

## Solutions to Chapter 2 — Linked Lists

### 2.1 Remove Dups
- **With buffer**: HashSet — O(N) time.
- **No buffer**: two-pointer runner — O(N²) time, O(1) space.

### 2.2 Kth to Last

Three approaches:
1. **Recursive** (return-counter via wrapper class, since Java can't pass int by reference).
2. **Two-pointer iterative**: p2 moves K ahead first; then advance both until p2 hits null → p1 is at the answer. O(N) time, O(1) space.

### 2.3 Delete Middle Node

Given only the node: **copy next's data into current, then unlink next**.

Fails if the target is the last node — discuss: mark as dummy or surface the constraint.

### 2.4 Partition

Build two lists (`before`, `after`) while iterating, then stitch.

**Shorter alternative**: grow a single list at head (small) / tail (large). Not stable, but less bookkeeping.

### 2.5 Sum Lists (reversed)

Recursive digit-by-digit add with carry. Termination when both lists null and no carry remains.

**Follow-up (forward order)**: complication — must know lengths to align digits. Strategy: pad shorter with zeros, recurse to tail, carry-and-result packaged via `PartialSum` wrapper.

### 2.6 Palindrome — three approaches

| Approach | Details |
|---|---|
| **Reverse and compare** | Clone + reverse, compare node by node. O(N) time/space. |
| **Stack (fast/slow runner)** | Push first-half onto stack using runner; then compare popped values against remaining nodes. |
| **Recursive** | Recurse to middle using `length - 2` per call; each level returns the matching back-half node via a `Result` wrapper. |

### 2.7 Intersection

**Key insight**: two intersecting singly-linked lists share the same tail.

1. Get each list's length and tail.
2. If tails differ (by reference) → no intersection.
3. Advance the longer list's pointer by `|lenA - lenB|`.
4. Walk both pointers until they meet.

**O(A+B) time, O(1) space.**

### 2.8 Loop Detection (cycle start)

Classic Floyd's:
1. **Fast** moves 2, **Slow** moves 1. If they collide, there's a loop.
2. At collision, Slow is `LOOP_SIZE - k` behind Fast; both are `k` nodes before the loop start (where `k` is list's non-loop length, mod loop size).
3. Reset Slow to head. Move both one step. They meet at the loop's start.

---

## Solutions to Chapter 3 — Stacks and Queues

### 3.1 Three in One

Two approaches:
- **Fixed Division**: split array in thirds; one stack may overflow while others are empty.
- **Flexible + Circular**: grow a stack by shifting the neighbor over; use modular indexing. Complex but wastes no space.

### 3.2 Stack Min (O(1) push/pop/min)

Each node stores `value + min-so-far`. Push sets `newMin = min(value, oldMin)`. Min is always the top's stored min.

**Space optimization**: second stack of mins — only push to it when the new value is ≤ current min; pop only when the value leaving matches the current min.

### 3.3 Stack of Plates

`ArrayList<Stack>`. `push` to last stack; create new stack when full. `pop` from last; remove empty tail stacks.

**Follow-up `popAt(index)`**: "rollover" — pop bottom of the next stack and push onto the popped stack, cascading until the last.

### 3.4 Queue via Stacks

Two stacks: `stackNewest` (top = newest) and `stackOldest` (top = oldest). Push onto newest. Before peek/remove, if `stackOldest` is empty, pour everything over from newest — amortized O(1).

### 3.5 Sort Stack — with one auxiliary stack

**Insertion-sort style**: pop current `tmp` from s1; move elements from s2 back to s1 until the right slot for `tmp` is found; push `tmp` onto s2.

`O(N²)` time, `O(N)` space.

### 3.6 Animal Shelter (FIFO dogs + cats)

Two queues (dogs, cats). Each animal tagged with an order counter (`order++` on enqueue). `dequeueAny` compares the two heads' orders; returns the older.

Model: shared `Animal` abstract class with `isOlderThan` comparator.

---

## Solutions to Chapter 4 — Trees and Graphs (partial)

### 4.1 Route Between Nodes

Straightforward BFS from one node; mark states `Unvisited / Visiting / Visited` to avoid cycles. DFS works too; BFS is preferred for "shortest" and is only marginally more code.

### 4.2 Minimal BST from Sorted Array

Pick the middle → root. Recurse: `[start, mid-1]` → left, `[mid+1, end]` → right. No intermediate inserts needed.

```java
TreeNode createMinimalBST(int[] arr, int start, int end) {
    if (end < start) return null;
    int mid = (start + end) / 2;
    TreeNode n = new TreeNode(arr[mid]);
    n.left  = createMinimalBST(arr, start, mid - 1);
    n.right = createMinimalBST(arr, mid + 1, end);
    return n;
}
```

### 4.3 List of Depths

Two variants, both O(N):
- **Modified pre-order DFS**, pass `level` down; append to `lists[level]`.
- **BFS-style**: process each level by walking parents' children.

Note: recursion's O(log N) stack space is dwarfed by the O(N) output.

### 4.4 Check Balanced

**Naive**: recursive `getHeight`, called at every node → O(N log N) due to repeated work.

**Optimized**: single-pass `checkHeight` that returns either actual height OR `Integer.MIN_VALUE` to signal imbalance. Propagate the sentinel upward. **O(N) time, O(H) space**.

### 4.5 Validate BST — two approaches

1. **In-order traversal** with `last_printed` — each visited value must be strictly greater than the previous. Fails on duplicates if definition requires uniqueness.
2. **Min/Max bounds** passed down: `checkBST(node, min, max)` — the cleaner approach.
   ```
   check that min <= node.data <= max
   recurse left  with max = node.data
   recurse right with min = node.data
   ```

### 4.6 Successor in BST

Two cases:
- **Has right subtree**: leftmost node of right subtree.
- **No right subtree**: walk up parent chain until you climb from a left child (i.e., you were in the parent's left subtree — parent is next). If you walk off the root, return null (end of traversal).

### 4.7 Build Order (topological sort)

**Solution #1 — Kahn's style**:
1. Add all nodes with zero incoming edges to the build order.
2. "Remove" them: decrement dependency counts on their children.
3. Any child whose count drops to zero joins the order.
4. Repeat. If you run out without processing all, there's a cycle → return error.

**Solution #2 — DFS topological sort**:
- For each unvisited node, DFS. On return (all descendants processed), push onto stack.
- Final stack (top-to-bottom) is the build order.
- Use three states: `BLANK`, `PARTIAL` (in-progress → finding it again = cycle), `COMPLETE`.

Both are **O(P + D)** (projects + dependency pairs).

### 4.8 First Common Ancestor — four solutions

1. **With parent links**: walk each up to find depths, align, then climb together until they meet. O(d).
2. **With parent links (better worst case)**: walk up from p, at each step check if the newly exposed sibling subtree contains q.
3. **Without parent links**: recurse on sides. Both on same side → recurse there; split → current is the ancestor. O(N) balanced.
4. **Optimized single-pass**: `commonAncestor(root, p, q)` returns:
   - `p` if subtree has only p
   - `q` if only q
   - the ancestor if both (with a flag to distinguish)
   - `null` if neither
   
   Use a `Result { node, isAncestor }` wrapper to correctly handle the case where one of p/q isn't in the tree at all.

### 4.9 BST Sequences

Recursion + **weaving**.

- All sequences creating a subtree rooted at `node` = weave(sequences-of-left, sequences-of-right), with `node.data` prepended to each.
- **Weaving** two lists means merging while preserving relative order within each list — recurse: prepend head of one list to all weaves of (rest, other), and vice versa.
- Clone prefix when storing to avoid aliasing bugs across recursion branches.

### 4.10 Check Subtree — two approaches

**Simple approach (pre-order strings)**:
- Serialize both trees with a NULL marker (e.g., `X`).
- T2 is a subtree of T1 iff T2's serialization is a substring of T1's.
- O(n+m) time, O(n+m) space.

**Alternative (matchTree per occurrence)**:
- Walk T1; whenever T1.node.data == T2.root.data, call `matchTree(n, T2.root)`.
- `matchTree` recursively compares both subtrees structurally.
- Time: `O(n + km)` where k = occurrences of T2's root in T1. Typically far better since matching exits early on first mismatch.
- **Space: O(log n + log m)** — big win when trees are huge.

### 4.11 Random Node from BST

The question was phrased "building the tree class from scratch" — a hint that modifying internals is allowed.

Progression of solutions:
- **Slow**: dump to array; O(N) per call.
- **Slow**: maintain always-synced array; O(N) on deletes.
- **Slow**: label nodes 1..N in-order; random index + BST search by label.
- **Optimal**: each node tracks **size of its subtree**. `getRandomNode` picks a random `i ∈ [1, root.size]`; descends proportionally (left subtree if `i ≤ leftSize`, current if `i == leftSize + 1`, else right with adjusted index). O(log N) on a balanced tree.

Insert/delete must update `size` on every ancestor — O(H) overhead.

---

## Key Recurring Patterns From These Solutions

| Pattern | Seen In |
|---|---|
| **Two-scan reverse** writing | URLify (1.3) |
| **Char-count via array indexed by `c - 'a'`** | 1.2, 1.4 |
| **Bit-vector toggle + `v & (v-1)`** | 1.4, 1.8 (space reduction) |
| **Using first row/column as scratch space** | Zero Matrix (1.8) |
| **Double-buffer concat trick** (`s1 + s1`) | String Rotation (1.9) |
| **Runner (two-pointer)** | 2.1, 2.2, 2.6, 2.7, 2.8 |
| **Floyd's Tortoise and Hare** | 2.8 |
| **Wrapper class for pass-by-reference** | 2.2, 2.5, 2.6, 4.8 |
| **Auxiliary stack for state tracking** | 3.2, 3.4, 3.5 |
| **Lazy transfer between stacks** | 3.4 (queue via stacks) |
| **Min/max bounds propagation** | 4.5 (Validate BST) |
| **Single-pass with sentinel return value** | 4.4 (Check Balanced) |
| **Topological sort (Kahn's or DFS)** | 4.7 |
| **Serialize + substring search** | 4.10 (Check Subtree) |
| **Augment nodes with subtree size** | 4.11 (Random Node) |
| **Weave / merge preserving relative order** | 4.9 (BST Sequences) |

---

## Interview Habits Reinforced

1. **Ask clarifying questions first** (ASCII vs Unicode; case sensitivity; definition ambiguities like "balanced" or "BST w/ duplicates").
2. **State a brute force, then optimize** — don't skip to "clever" without acknowledging the simple approach.
3. **Analyze both time and space** — several problems hinge on the space distinction (e.g., 4.10 simple vs alternative).
4. **Modularize** helpers into named functions (`insertBefore`, `padList`, `leftShift`) — even in whiteboard code.
5. **Test on a real, non-degenerate example** — the book's walkthroughs consistently draw a meaningful tree/list of 5–8 nodes rather than a trivial 2-element case.
6. **Discuss tradeoffs with the interviewer** — many problems (3.3 popAt rollover, 4.10 simple-vs-alternative, 2.4 stable vs fast) explicitly have no single right answer.
