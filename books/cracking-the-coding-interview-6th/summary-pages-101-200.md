# Cracking the Coding Interview (6th Edition) — Pages 101–200 Summary

Covers the technical chapters from ArrayList/StringBuilder through the start of the Hard problems list: Chapters 1 (continued) → 17 (partial).

---

## Chapter 1 (continued): Arrays and Strings

### ArrayList & Resizable Arrays

- Fixed-length arrays (Java) vs. auto-resizing lists (Python, etc.).
- `ArrayList` doubles in size when full — each doubling is O(N) but amortizes to **O(1)** per insertion.

**Why amortized O(1)?** Total copies to grow to N = N/2 + N/4 + N/8 + ... + 1 ≈ N. So N inserts do ~2N work total.

### StringBuilder

Naive concatenation in a loop = **O(xn²)** (copies `x + 2x + 3x + ... = x·n(n+1)/2`).

`StringBuilder` uses a resizable array of characters, joining only at `toString()` → **O(xn)**.

```java
StringBuilder sb = new StringBuilder();
for (String w : words) sb.append(w);
return sb.toString();
```

### Interview Questions 1.1–1.9

| # | Problem | Core Idea |
|---|---|---|
| 1.1 | Is Unique | Hash set or bit vector; O(N) time |
| 1.2 | Check Permutation | Sort both, or count char frequencies |
| 1.3 | URLify | In-place, scan from end to front |
| 1.4 | Palindrome Permutation | At most one char with odd count |
| 1.5 | One Away | Single scan, compare chars by edit type |
| 1.6 | String Compression | Only compress if shorter; use StringBuilder |
| 1.7 | Rotate Matrix (90°) | Rotate layer by layer in-place |
| 1.8 | Zero Matrix | Record zero rows/cols first, then zero them |
| 1.9 | String Rotation | `s1s1` contains `s2` iff `s2` is a rotation |

---

## Chapter 2: Linked Lists

### Basics
- **Singly linked**: each node → next.
- **Doubly linked**: each node → next and prev.
- No O(1) random access; O(1) insert/delete at known position.

### Deleting a Node
Walk to find previous node; update `prev.next = n.next` (and `next.prev` if doubly linked). Handle head/tail specially.

### The "Runner" Technique
Two pointers, one faster. Classic uses:
- Find middle (fast moves 2x).
- Detect cycle (Floyd's — meeting point → cycle start).
- Find Kth from end (lead pointer K ahead).
- Rearrange `a1→a2→...→an→b1→...→bn` into `a1→b1→a2→b2→...`.

### Recursive Solutions
Many linked list problems (reverse, palindrome, sum) have natural recursive forms. Remember: recursion uses **O(depth)** stack space.

### Interview Questions 2.1–2.8

| # | Problem | Core Idea |
|---|---|---|
| 2.1 | Remove Dups | HashSet; follow-up: O(1) space → runner pointer O(N²) |
| 2.2 | Kth to Last | Two pointers, one K ahead |
| 2.3 | Delete Middle Node | Copy next node's data, delete next |
| 2.4 | Partition | Build two lists, stitch together |
| 2.5 | Sum Lists | Simulate addition with carry; reversed is easier |
| 2.6 | Palindrome | Reverse half + compare, or recursive with runner |
| 2.7 | Intersection | Align tails (length diff), walk until same node |
| 2.8 | Loop Detection | Floyd's tortoise-and-hare; second phase finds loop start |

---

## Chapter 3: Stacks and Queues

### Stack (LIFO)
Ops: `push`, `pop`, `peek`, `isEmpty`. Easy to implement with linked list (head = top).

Key insight: stacks naturally model recursion iteratively — and many recursive algorithms can be rewritten with an explicit stack.

### Queue (FIFO)
Ops: `add`, `remove`, `peek`, `isEmpty`. Linked list with `first` and `last` pointers.

Used in BFS and cache implementations. Easy to mess up the `first`/`last` pointer updates — double check edge cases (empty, one-element).

### Interview Questions 3.1–3.6

| # | Problem | Core Idea |
|---|---|---|
| 3.1 | Three in One | One array, fixed or flexible partitioning |
| 3.2 | Stack Min | Auxiliary min-stack (each node carries min-so-far) |
| 3.3 | Stack of Plates | List of stacks; new one when threshold hit |
| 3.4 | Queue via Stacks | Two stacks: push onto stack1, pop from stack2 (transfer when empty) |
| 3.5 | Sort Stack | Use second temp stack, insertion-sort-like |
| 3.6 | Animal Shelter | Two queues (dogs, cats); compare timestamps for "any" |

---

## Chapter 4: Trees and Graphs

### Tree Terminology — Always Clarify

| Type | Property |
|---|---|
| **Binary Tree** | Each node ≤ 2 children |
| **Binary Search Tree** | `left ≤ n < right` (for all descendants, not just children) |
| **Balanced** | Not necessarily same size — just O(log N) operations |
| **Complete** | Every level full except possibly last (filled left-to-right) |
| **Full** | Every node has 0 or 2 children |
| **Perfect** | Both full AND complete — has exactly `2^k − 1` nodes |

### Binary Tree Traversal
- **In-order**: left, node, right → gives sorted order on a BST.
- **Pre-order**: node, left, right → root first.
- **Post-order**: left, right, node → root last.

### Binary Heaps (Min-Heap)
Complete binary tree where every node ≤ its children.
- **Insert**: add at bottom rightmost, bubble up — O(log N).
- **Extract-min**: swap root with last node, remove last, bubble down — O(log N).

### Tries (Prefix Trees)
- n-ary tree where paths spell words.
- Terminating nodes (`*`) mark complete words.
- Check if string is a valid prefix in **O(K)** where K = string length.
- Better than hash table when you need prefix queries.

### Graphs

**Representation**:
- **Adjacency list**: per node, list of neighbors. Easy neighbor iteration.
- **Adjacency matrix**: NxN boolean/int matrix. Simpler but O(N) neighbor scan.

**Search**:
- **DFS**: recursion, go deep first. Simpler, good for "visit all."
- **BFS**: queue, go wide first. Best for shortest path.
- **Bidirectional Search**: two BFS from both ends — collision finds shortest path. Reduces O(k^d) to O(k^(d/2)), supporting paths twice as long.

### Interview Questions 4.1–4.12

| # | Problem | Core Idea |
|---|---|---|
| 4.1 | Route Between Nodes | BFS from source |
| 4.2 | Minimal Tree | Recurse on middle element of sorted array |
| 4.3 | List of Depths | BFS level-by-level, or DFS with depth arg |
| 4.4 | Check Balanced | Recursive height check; propagate "unbalanced" |
| 4.5 | Validate BST | Pass min/max bounds down; or in-order traversal |
| 4.6 | Successor | Right subtree's leftmost; else walk up until we're left child |
| 4.7 | Build Order | Topological sort (DFS or Kahn's) |
| 4.8 | First Common Ancestor | Recursive; track which side each target is on |
| 4.9 | BST Sequences | Weave all valid orderings from left+right subtrees |
| 4.10 | Check Subtree | Match at each candidate root; or pre-order string compare |
| 4.11 | Random Node | Store subtree sizes; traverse proportionally |
| 4.12 | Paths with Sum | Running-sum hash map (like subarray sum problem) |

---

## Chapter 5: Bit Manipulation

### Core Identities
```
x ^ 0s = x        x & 0s = 0        x | 0s = x
x ^ 1s = ~x       x & 1s = x        x | 1s = 1s
x ^ x  = 0        x & x  = x        x | x  = x
```

### Two's Complement
Negative K as N-bit = `concat(1, 2^(N-1) - K)` — equivalently, flip bits and add 1.

### Arithmetic vs. Logical Shift
- `>>` (arithmetic): fills with sign bit. `-75 >> 1 = -38`.
- `>>>` (logical): fills with 0. Repeatedly `>>>` → 0; repeatedly `>>` on negative → -1.

### Common Bit Operations

| Task | Code |
|---|---|
| Get bit i | `(num & (1 << i)) != 0` |
| Set bit i | `num | (1 << i)` |
| Clear bit i | `num & ~(1 << i)` |
| Clear MSB through i | `num & ((1 << i) - 1)` |
| Clear i through 0 | `num & (-1 << (i+1))` |
| Update bit i to v | `(num & ~(1<<i)) | (v << i)` |

### Useful Tricks
- `(n & (n-1)) == 0` → n is a power of 2 (or zero).
- `n & (n-1)` → clears the lowest set bit.

### Interview Questions 5.1–5.8
Insertion, Binary to String, Flip Bit to Win, Next Number (same 1-count), Debugger (power-of-2 check), Conversion (bit-flip count), Pairwise Swap, Draw Line.

---

## Chapter 6: Math and Logic Puzzles

### Prime Numbers
- **Check primality**: iterate only up to `√n` — every composite has a factor ≤ √n.
- **Sieve of Eratosthenes**: cross off multiples of each prime starting from `prime*prime` (smaller multiples already crossed off). Generates all primes up to N.

### Probability Basics

```
P(A and B) = P(B | A) · P(A)
P(A or B)  = P(A) + P(B) − P(A and B)
Bayes:      P(A | B) = P(B | A) · P(A) / P(B)
```

- **Independent**: P(A and B) = P(A) · P(B).
- **Mutually Exclusive**: P(A and B) = 0.
- Two events can never be *both* independent AND mutually exclusive (unless one has probability 0).

### Problem-Solving Approaches
- **Develop Rules**: write down invariants as you find them (e.g., two ropes — rule: "burn a rope = x+y minutes when chained").
- **Worst Case Shifting (balancing)**: in "nine balls" problem, divide into 3 groups of 3, not 4+4+1 — balances worst case to 2 weighings.
- **Try Base Case and Build**, **Do It Yourself**, or apply the regular algorithm toolkit.

### Interview Questions 6.1–6.10
Heavy Pill, Basketball (1-shot vs. 2-of-3), Dominos (8x8 with 2 opposite corners removed), Ants on Triangle, Jugs of Water, Blue-Eyed Island, Apocalypse (gender ratio), Egg Drop, 100 Lockers, Poison (1000 bottles, 10 strips).

---

## Chapter 7: Object-Oriented Design

### 4-Step Approach
1. **Handle Ambiguity** — ask who uses it, how, why (the "six Ws").
2. **Define Core Objects** — typically one per real-world concept.
3. **Analyze Relationships** — many-to-many? Inheritance? Ownership?
4. **Investigate Actions** — trace use cases through the design.

### Design Patterns (interview-relevant)

**Singleton**: one global instance.
```java
public static Restaurant getInstance() {
    if (_instance == null) _instance = new Restaurant();
    return _instance;
}
```
Criticized as an anti-pattern — hurts unit testing.

**Factory Method**: creator decides which concrete class to instantiate.
```java
public static CardGame createCardGame(GameType type) {
    if (type == GameType.Poker) return new PokerGame();
    else if (type == GameType.Blackjack) return new BlackjackGame();
    return null;
}
```

### Interview Questions 7.1–7.12
Deck of Cards, Call Center, Jukebox, Parking Lot, Online Book Reader, Jigsaw, Chat Server, Othello, Circular Array, Minesweeper, File System, Hash Table.

---

## Chapter 8: Recursion and Dynamic Programming

### Three Approaches
- **Bottom-Up**: build from base cases upward.
- **Top-Down**: break N into subproblems (watch for overlap).
- **Half-and-Half**: divide data in half (merge sort, binary search).

### Recursive vs. Iterative
Every recursive algorithm can be iterative; recursion uses O(depth) stack. Discuss the tradeoff when relevant.

### Dynamic Programming = Recursion + Caching

**Fibonacci example**:

| Approach | Runtime |
|---|---|
| Naive recursion | O(2^N) — full binary tree of calls |
| Memoization (top-down) | O(N) — each value computed once |
| Bottom-up table | O(N) time, O(N) space |
| Rolling variables | O(N) time, **O(1) space** |

Draw the recursion tree to find overlapping subproblems — that's where caching wins.

### Interview Questions 8.1–8.14
Triple Step, Robot in a Grid, Magic Index, Power Set, Recursive Multiply, Towers of Hanoi, Permutations (w/ and w/o dups), Parens, Paint Fill, Coins, Eight Queens, Stack of Boxes, Boolean Evaluation.

---

## Chapter 9: System Design and Scalability

### Handling the Questions
Communicate, go broad first, use the whiteboard, acknowledge interviewer concerns, state assumptions explicitly, estimate when necessary, drive the conversation.

### Design Step-By-Step
1. **Scope** — list major features/use cases.
2. **Make Reasonable Assumptions** — volumes, staleness tolerance.
3. **Draw Major Components** — end-to-end flow.
4. **Identify Key Issues** — bottlenecks, peaks.
5. **Redesign for Key Issues** — caches, partitioning.

### Algorithms That Scale
1. Ask Questions. 2. Make Believe (all data on one machine). 3. Get Real (what breaks?). 4. Solve Problems.

### Key Concepts

| Concept | Summary |
|---|---|
| **Vertical vs. Horizontal Scaling** | Bigger machine vs. more machines |
| **Load Balancer** | Distributes traffic across cloned servers |
| **Denormalization** | Store redundant data to avoid joins |
| **NoSQL** | No joins; scale-friendly |
| **Sharding** | Vertical (by feature), key-based (hash), directory-based (lookup table) |
| **Caching** | In-memory key-value layer in front of the DB |
| **Async Processing / Queues** | Slow ops off the request path |
| **MapReduce** | Map emits (k,v); Reduce aggregates by key |

### Networking Metrics
- **Bandwidth**: max data/sec.
- **Throughput**: actual data/sec.
- **Latency**: time for one packet to traverse.
- Conveyor belt analogy: fatter belt = more throughput/bandwidth; shorter belt = lower latency; faster belt = all three.

### Considerations
Failures, availability vs. reliability, read-heavy vs. write-heavy, security. **There is no "perfect" system** — articulate tradeoffs.

### Example: Find Documents Containing Words
Pre-process into `word → [docIds]` index. Shard by keyword alphabetically; intersect per-shard results.

### Interview Questions 9.1–9.8
Stock Data, Social Network, Web Crawler, Duplicate URLs (10 billion), Cache, Sales Rank, Personal Financial Manager, Pastebin.

---

## Chapter 10: Sorting and Searching

### Common Sorts

| Algorithm | Avg | Worst | Memory | Notes |
|---|---|---|---|---|
| Bubble | O(N²) | O(N²) | O(1) | Rarely used |
| Selection | O(N²) | O(N²) | O(1) | Simple but inefficient |
| **Merge Sort** | O(N log N) | O(N log N) | O(N) | Stable; good for linked lists |
| **Quick Sort** | O(N log N) | O(N²) | O(log N) | Fast in practice; bad pivot = worst case |
| **Radix Sort** | O(kN) | — | — | For integers; beats comparison-based bound |
| **Bucket Sort** | O(N) | — | — | When values fit into a small range |

### Binary Search

```java
while (low <= high) {
    int mid = (low + high) / 2;
    if (a[mid] < x) low = mid + 1;
    else if (a[mid] > x) high = mid - 1;
    else return mid;
}
```

Watch the `+1`/`-1` on the bounds.

### Interview Questions 10.1–10.11
Sorted Merge, Group Anagrams, Search in Rotated Array, Sorted Search No-Size, Sparse Search, Sort Big File (external sort), Missing Int (bit vector), Find Duplicates (bit vector w/ 4KB), Sorted Matrix Search, Rank from Stream (augmented BST), Peaks and Valleys.

---

## Chapter 11: Testing

### Four Types
1. **Real-world object** (a pen).
2. **Piece of software** (a browser).
3. **A specific function** (sort).
4. **Troubleshooting** (app crashes).

Across all types: don't assume the input is nice. Expect abuse.

### What Interviewers Look For
Big picture understanding, knowing how pieces fit together, organization, and practicality.

### Testing a Real-World Object / Software
Steps: (1) Who uses it & why? (2) Use cases. (3) Bounds of use. (4) Stress/failure conditions. (5) How to perform the testing (manual vs. automated; black-box vs. white-box).

### Testing a Function
Normal case, extremes (empty, one element, huge), nulls/illegal input, strange input (already sorted, reversed).

### Troubleshooting
Understand the scenario → break down the problem into testable units → create specific, manageable tests.

### Interview Questions 11.1–11.6
Mistake (unsigned int loop bug), Random Crashes, Chess Test, No Test Tools (load test a webpage manually), Test a Pen, Test an ATM.

---

## Chapter 12: C and C++

Covered mainly so Java/other candidates can prep if C++ is on the resume.

| Topic | Key Point |
|---|---|
| **Classes/Inheritance** | Members private by default; `public` to expose |
| **Constructors** | Use init list for const/reference members |
| **Destructors** | Called on delete; take no args |
| **Virtual Functions** | Enable dynamic dispatch; `= 0` = pure virtual (abstract class) |
| **Virtual Destructor** | Required in any base class meant for polymorphic delete |
| **Default Values** | Must be trailing args |
| **Operator Overloading** | e.g., `Bookshelf operator+(Bookshelf other)` |
| **Pointers vs. References** | References can't be null or reassigned |
| **Pointer Arithmetic** | `p++` advances by `sizeof(*p)` bytes |
| **Templates** | Compile-time code reuse across types |

### Interview Questions 12.1–12.11
Last K Lines, Reverse String, Hash Table vs. STL Map, Virtual Functions, Shallow vs. Deep Copy, Volatile, Virtual Base Class, Copy Node, Smart Pointer, Aligned Malloc, 2D Alloc.

---

## Chapter 13: Java

### Overloading vs. Overriding
- **Overloading**: same method name, different signatures (in the same class).
- **Overriding**: subclass provides its own implementation of a superclass method.

### Collection Framework
`ArrayList` (resizable array), `Vector` (synchronized ArrayList), `LinkedList` (built-in doubly linked), `HashMap` (key-value).

### Interview Questions 13.1–13.8
Private Constructor, Return from Finally (yes — finally still runs), Final vs. Finally vs. Finalize, Generics vs. Templates, TreeMap/HashMap/LinkedHashMap differences, Object Reflection, Lambda Expressions, Lambda Random (random subset).

---

## Chapter 14: Databases

### Normalized vs. Denormalized
- **Normalized**: minimize redundancy; requires joins.
- **Denormalized**: redundant copies for faster reads; used at scale.

### Key Query Gotchas
- Use **LEFT JOIN** when you want all rows from one side (even with no match).
- `count(*)` counts groups, including rows with no matches — use `count(Specific.Column)` to exclude them.
- When `SELECT`ing a column not in `GROUP BY`, wrap it in an aggregate (`MAX`, `first`, etc.) or add to GROUP BY.

### Small Database Design
Same 4-step OOD-style approach: ambiguity → core objects → relationships (many-to-many often needs a bridge table) → actions.

### Large Database Design
**Denormalize**. Joins are slow at scale.

### Interview Questions 14.1–14.7
Multiple Apartments (tenants with >1 apartment), Open Requests, Close All Requests, Joins (INNER/LEFT/RIGHT/FULL OUTER), Denormalization pros/cons, ER Diagram, Design Grade Database.

---

## Chapter 15: Threads and Locks

### Threads in Java
Two ways to create:
1. Implement `Runnable` interface → pass to `Thread` constructor. Preferred when you also need to extend another class.
2. Extend `Thread` class → override `run()`. Call `start()` on the instance.

### Synchronization
- `synchronized` method: one thread per *instance* can execute it. Static synchronized methods lock on the class.
- `synchronized(obj) { ... }` block: finer control.
- `Lock` / `ReentrantLock`: explicit `lock()`/`unlock()` for granular control.

### Deadlock — Four Conditions (all must hold)
1. **Mutual Exclusion** — only one thread holds the resource.
2. **Hold and Wait** — threads keep what they have while requesting more.
3. **No Preemption** — resources can't be forcibly taken.
4. **Circular Wait** — cyclic chain of waits.

Most prevention targets #4 (e.g., global lock ordering).

### Interview Questions 15.1–15.7
Thread vs. Process, Context Switch timing, Dining Philosophers, Deadlock-Free Class, Call In Order (ensure first→second→third), Synchronized Methods semantics, FizzBuzz (multithreaded).

---

## Chapter 16: Moderate Problems (Start)

A large problem set — highlights from 16.1–16.26:

| # | Problem |
|---|---|
| 16.1 | Number Swapper (XOR trick) |
| 16.2 | Word Frequencies (pre-process on repeats) |
| 16.3 | Line Intersection |
| 16.4 | Tic-Tac Win |
| 16.5 | Factorial Zeros (count factors of 5) |
| 16.6 | Smallest Difference (sort + two pointers) |
| 16.7 | Number Max (no comparisons — use sign bit) |
| 16.8 | English Int (format a number) |
| 16.9 | Operations (only using `+`) |
| 16.10 | Living People (sort events / delta array) |
| 16.11 | Diving Board |
| 16.12 | XML Encoding |
| 16.13 | Bisect Squares (line through centers) |
| 16.14 | Best Line (hash map of slope→points) |
| 16.15 | Master Mind |
| 16.16 | Sub Sort (find bounds of unsorted middle) |
| 16.17 | Contiguous Sequence (Kadane's algorithm) |
| 16.18 | Pattern Matching |
| 16.19 | Pond Sizes (DFS on grid) |
| 16.20 | T9 (phone keypad decoder) |
| 16.21 | Sum Swap |
| 16.22 | Langton's Ant |
| 16.23 | Rand7 from Rand5 (reject-and-retry) |
| 16.24 | Pairs with Sum |
| 16.25 | LRU Cache (hash map + doubly linked list) |
| 16.26 | Calculator |

---

## Chapter 17: Hard Problems (Start)

Highlights from 17.1–17.16 (full chapter continues past page 200):

| # | Problem |
|---|---|
| 17.1 | Add Without Plus (bitwise carry) |
| 17.2 | Shuffle (Fisher–Yates) |
| 17.3 | Random Set (reservoir sampling) |
| 17.4 | Missing Number (O(N) with bit-access constraint) |
| 17.5 | Letters and Numbers (prefix-sum hash) |
| 17.6 | Count of 2s in 0..N |
| 17.7 | Baby Names (union-find over synonyms) |
| 17.8 | Circus Tower (longest increasing subsequence on sorted heights) |
| 17.9 | Kth Multiple (of 3, 5, 7 — heap or 3-queue merge) |
| 17.10 | Majority Element (Boyer–Moore voting) |
| 17.11 | Word Distance (with repeated queries → preprocess) |
| 17.12 | BiNode (BST → doubly linked list in place) |
| 17.13 | Re-Space (DP over dictionary splits) |
| 17.14 | Smallest K (heap or quickselect) |
| 17.15 | Longest Word (trie + DFS on suffixes) |
| 17.16 | The Masseuse (DP: take-or-skip) |

---

## Recurring Technique Cheat Sheet

| Technique | When to reach for it |
|---|---|
| **Hash table** | O(1) lookup; pair/complement problems |
| **Two pointers / runner** | Sorted array or linked list traversal |
| **Sliding window** | "Find subarray with property X" |
| **BFS** | Shortest path, level-by-level |
| **DFS** | Visit all, path reconstruction, graph connectivity |
| **Bidirectional BFS** | Shortest path in huge graphs |
| **Topological sort** | Dependency / ordering problems |
| **DP** | Overlapping subproblems + optimal substructure |
| **Heap** | "Top K", streaming median, scheduling |
| **Trie** | Prefix queries over word lists |
| **Union-Find** | Connected components, equivalence classes |
| **Bit manipulation** | Tight memory (bit vectors), pairing/XOR tricks |
| **Reservoir sampling** | Random pick from unknown-size stream |
