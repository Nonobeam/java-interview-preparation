# Cracking the Coding Interview (6th Edition) — Pages 301–350 Summary

Covers the rest of **Chapter 4 Solutions** (Trees/Graphs 4.11–4.12), **Chapter 5 Solutions** (Bit Manipulation 5.1–5.8), **Chapter 6 Solutions** (Math & Logic 6.1–6.10), and the start of **Chapter 7 Solutions** (Object-Oriented Design 7.1–7.9).

---

## Solutions to Chapter 4 (continued)

### 4.11 Random Node — Options #4–#7

The book walks through several failed attempts before arriving at the optimal solution — an important interview lesson.

| Option | Issue |
|---|---|
| #4 (by random depth) | Levels have unequal node counts → not uniform |
| #5 (random 1/3 at each node) | Root has 1/3 probability — violates uniform |
| #6 (weight by subtree size) | **Works**. Each node tracks its subtree size; at each level pick left/current/right with probabilities proportional to subtree sizes |
| #7 (single random number, in-order index) | **Also works, fewer RNG calls**. Pick `i ∈ [0, size)`, then `getIthNode(i)` navigates using `leftSize`: `i < leftSize` → left; `i == leftSize` → current; else subtract and go right |

Both optimal solutions are `O(log N)` on a balanced tree (more precisely `O(D)` where D = depth). Every insert/delete must update `size` on the path from the inserted node up to the root.

### 4.12 Paths with Sum

**Brute force**: at every node, try all downward paths and count those summing to target.
- Worst case at each depth d, `countPathsWithSumFromNode` is called by d ancestors.
- Balanced tree: `O(N log N)`. Unbalanced (linear): `O(N²)`.

**Optimized — running sum hash map** (same trick as "subarray sum equals K"):

> `runningSum_y − runningSum_x = targetSum` ⇔ the path from node after x to y sums to targetSum.

Algorithm (DFS):
1. Track `runningSum` down the path; increment by `node.value`.
2. Look up `runningSum − targetSum` in the hash map — that's the number of paths ending at this node.
3. If `runningSum == targetSum`, add 1 (path from root).
4. Increment `pathCount[runningSum]` before recursing left/right.
5. **Decrement on return** — essential, or other branches would double-count.

**`O(N)` time, `O(log N)` space** (balanced) — hash map only grows with path depth.

---

## Solutions to Chapter 5 — Bit Manipulation

### 5.1 Insertion — place M into N at bits j..i

Three steps: (1) clear bits j..i in N via a mask, (2) shift M left by i, (3) OR together.

```java
int mask = ~((allOnes << (j + 1)) | ((1 << i) - 1));
// Actually: left half = all-ones << (j+1); right half = (1<<i) - 1;
// mask      = ~(left | right)   — zeros in bits j..i
return (n & mask) | (m << i);
```

Off-by-one errors are the killer here — test carefully.

### 5.2 Binary to String — fractional double → binary

Repeatedly multiply by 2. If `num ≥ 1`, append `1` and subtract 1; else append `0`.

```java
while (num > 0) {
    if (binary.length() >= 32) return "ERROR";
    num *= 2;
    if (num >= 1) { binary.append('1'); num -= 1; }
    else { binary.append('0'); }
}
```

Alternative: compare against `.5`, `.25`, `.125`, ... (same computation, different framing).

### 5.3 Flip Bit to Win — longest run of 1s with one flip

**Bit's-length analysis**: Think of the integer as alternating 0-runs and 1-runs.

**Optimal O(b) time, O(1) space**: scan bit by bit, track:
- `currentLength`: length of current 1-run.
- `previousLength`: length of the previous 1-run (0 if the gap between previous and current was more than a single 0).

On each `0` bit:
- `previousLength = (next bit is 0) ? 0 : currentLength`
- `currentLength = 0`

Max answer so far: `previousLength + currentLength + 1`.

Note on terminology: don't say `O(n)` when n is ambiguous (value? bits?). Use a fresh variable like `b`.

### 5.4 Next Number — same number of 1s, next larger and next smaller

**Bit manipulation for `getNext`** (next larger):
- Let `c0` = count of trailing zeros, `c1` = size of the 1-block immediately after. `p = c0 + c1` = position of the rightmost non-trailing zero.
- (1) Flip bit `p` from 0 to 1.
- (2) Clear bits 0..p−1.
- (3) Insert `c1 − 1` ones at positions 0..c1−2.

**Arithmetic shortcut**:
```
next = n + (1 << c0) + (1 << (c1 - 1)) - 1
```

**`getPrev`** (next smaller) is symmetric — flip the rightmost non-trailing **one**, then rearrange 1s/0s.

### 5.5 Debugger — what is `(n & (n-1)) == 0`?

Subtracting 1 flips the lowest set bit plus all trailing zeros. If `n & (n-1) == 0`, n and n−1 share no set bits → n has exactly one bit set → **n is a power of 2** (or zero).

### 5.6 Conversion — bits differing between A and B

`A ^ B` has 1s exactly where they differ. Count the bits:

```java
int count = 0;
for (int c = a ^ b; c != 0; c = c & (c - 1)) count++;
```

The `c & (c - 1)` trick clears the lowest set bit each iteration.

### 5.7 PairwiseSwap — swap odd and even bits

Mask odd and even bits, shift, OR:

```java
return ((x & 0xaaaaaaaa) >>> 1) | ((x & 0x55555555) << 1);
```

Use logical shift `>>>` so the sign bit doesn't propagate.

### 5.8 Draw Line — horizontal line in a packed bitmap

Byte-aligned screen (8 pixels per byte, width divisible by 8).

**Algorithm**:
1. Identify `first_full_byte` and `last_full_byte` — the fully-covered middle region.
2. Set middle bytes to `0xFF`.
3. For the partial start byte: mask `0xFF >> start_offset`.
4. For the partial end byte: mask `~(0xFF >> (end_offset + 1))`.
5. **Special case**: x1 and x2 in the same byte — AND the two masks and write once.

Lots of edge cases — only careful candidates get it bug-free.

---

## Solutions to Chapter 6 — Math and Logic Puzzles

### 6.1 Heavy Pill (20 bottles, 1 weighing)

**Trick**: take a *different* number of pills from each bottle.

Take 1 from bottle #1, 2 from #2, …, 20 from #20. Expected weight if all 1g: `1+2+…+20 = 210`. Overage = weight − 210; since heavy pill adds 0.1g per pill, bottle = `(weight − 210) / 0.1`.

### 6.2 Basketball (1 shot vs. 2-of-3)

- `P(win Game 1) = p`
- `P(win Game 2) = p³ + 3p²(1−p) = 3p² − 2p³`

Game 1 better iff `p > 3p² − 2p³` → `(2p−1)(p−1) > 0` → `p < 0.5`.

**Play Game 1 if p < 0.5, Game 2 if p > 0.5.** Equal at p = 0, 0.5, 1.

### 6.3 Dominos on a Mutilated Chessboard

**Proof by coloring**: The two opposite corners are the same color. Removing them leaves 30 of one color, 32 of the other. Each domino covers one black + one white. 31 dominoes = 31 of each color. Contradiction → **impossible**.

### 6.4 Ants on a Triangle — probability of collision

No collision only if all ants move the same direction (all CW or all CCW).
- `P(same direction) = 2 / 2³ = 1/4`.
- **P(collision) = 3/4**.

General n-vertex polygon: `P(collision) = 1 − 2/2ⁿ = 1 − 2^(1−n)`.

### 6.5 Jugs of Water (5-quart, 3-quart → 4 quarts)

Classic pour sequence: Fill 5, pour into 3 → (2, 3). Empty 3 → (2, 0). Pour 5→3 → (0, 2). Fill 5 → (5, 2). Top off 3 from 5 → (4, 3). **Done: 4 quarts**.

Generalization: any value between 1 and the sum is reachable iff the two jug sizes are relatively prime.

### 6.6 Blue-Eyed Island — induction / common knowledge

Base: 1 blue-eyed person leaves night 1 (sees no other blue eyes, but knows ≥1 exists → it's him).

Induction: c blue-eyed people leave on **night c** — each has been waiting to see if the other c−1 would leave; when they don't, everyone deduces c = total → all leave together.

### 6.7 Apocalypse — gender ratio under "stop after a girl"

**Intuition trap**: the policy favors girls, right?

**Answer**: **50/50**. Each individual birth is still a 50/50 coin flip regardless of any stopping rule. Concatenate all family sequences (`BGBBGBGBBBG...`) and you just have a stream of fair coin flips. No stopping policy can change per-flip probabilities.

Mathematical proof: compute E[boys per family] = Σ n·(1/2)^(n+1) = 1. Each family contributes exactly 1 girl and on average 1 boy.

### 6.8 Egg Drop — 100 floors, 2 eggs

**Key insight: worst-case balancing**. If egg 1 breaks on drop number k, egg 2 must do a linear search of the `k−1` floors since the last drop. Every time egg 1 doesn't break, we should reduce the remaining span by one.

Find X such that `X + (X−1) + (X−2) + … + 1 = 100`, i.e., `X(X+1)/2 = 100`, X ≈ 13.65. Round up to **14**.

Drop egg 1 at floors 14, 27, 39, 50, 60, 69, 77, 84, 90, 95, 99, 100. When it breaks, binary-linear search with egg 2. **Worst case: 14 drops.**

### 6.9 100 Lockers — which are open at the end?

A locker is toggled once per factor of its number. Open = odd number of factors. Odd factor count = **perfect square** (factors pair except for √n). Perfect squares ≤ 100: 1, 4, 9, …, 100 → **10 lockers open**.

### 6.10 Poison — 1000 bottles, 10 test strips, 7-day results

**Naive (28 days)**: divide into groups, eliminate, repeat. Each round takes 7 days.

**Optimal (7 days) — binary encoding**: each bottle's ID is a 10-bit number (since `2¹⁰ = 1024 ≥ 1000`). For bottle i, add a drop to strip j iff bit j of i is set. After 7 days, read which strips turned positive — those bits form the bottle ID.

```java
for (Bottle bottle : bottles) {
    int id = bottle.getId();
    for (int bit = 0; id > 0; bit++, id >>= 1) {
        if ((id & 1) == 1) testStrips.get(bit).addDrop(bottle);
    }
}
// After 7 days:
int poisonedId = 0;
for (TestStrip strip : testStrips) {
    if (strip.isPositive()) poisonedId |= (1 << strip.getId());
}
```

**Intermediate (10 days)** decomposes bottle ID by decimal digits across three days, handling edge cases around duplicate digits with a 4th "shift" day — instructive but less elegant than the binary trick.

---

## Solutions to Chapter 7 — Object-Oriented Design (7.1–7.9)

### 7.1 Deck of Cards (Blackjack)

Design hierarchy:
- `Suit` enum.
- `Deck<T extends Card>` — ArrayList of T, `dealtIndex` pointer instead of removing. `shuffle`, `dealHand`, `dealCard`.
- `Card` abstract — `faceValue`, `suit`, `available` flag, abstract `value()`.
- `Hand<T extends Card>` — `score`, `addCard`.
- For Blackjack: `BlackJackHand extends Hand<BlackJackCard>`, `BlackJackCard extends Card`.

Blackjack subtlety: **Aces have two values** (1 or 11). Approach: `possibleScores()` returns all combinations; `score()` returns highest under 21, else lowest over 21.

### 7.2 Call Center

**Core classes**: `CallHandler` (central dispatcher, singleton-ish), `Call`, abstract `Employee`, concrete `Respondent`/`Manager`/`Director`.

**`CallHandler.dispatchCall(Call)`**: try to route to lowest-rank free employee; otherwise queue by rank.

Call carries a `Rank` (minimum rank that can handle it) — allows escalation by `incrementRank()`.

Tradeoff: lots of code for an interview — most candidates would be asked for structure/skeleton, not full implementation.

### 7.3 Jukebox

Clarify first: CDs? MP3s? physical? money? currency?

Core classes: `Jukebox`, `CDPlayer`, `Playlist` (wraps a queue of Songs), `CD`, `Song`, `Artist`, `User`, `Display`.

`Playlist.getNextSToPlay()` and `queueUpSong(Song)` wrap a queue with convenience methods.

### 7.4 Parking Lot

Assumptions: multiple levels, rows; three vehicle types (motorcycle, car, bus); spots sized motorcycle/compact/large; a bus needs 5 consecutive large spots in one row.

- `Vehicle` abstract → `Motorcycle`/`Car`/`Bus`; each sets `spotsNeeded` and `size`.
- `ParkingSpot` — stores its size; `canFitVehicle(v)` checks size only.
- `Level` — array of spots; `parkVehicle` finds first range of `spotsNeeded` consecutive spots that fit.
- `ParkingLot` — array of `Level`s.

Cleaner to have a single `ParkingSpot` class with a size enum than `LargeSpot/CompactSpot/MotorcycleSpot` subclasses — no behavioral difference.

### 7.5 Online Book Reader

Decompose `OnlineReaderSystem` into:
- `Library` (HashMap<id, Book> — add/remove/find)
- `UserManager` (HashMap<id, User>)
- `Display` (knows current `Book`, `User`, `pageNumber`; `displayBook`, `turnPageForward/Backward`, `refreshX`).

`Book`, `User` are thin data classes.

Justifies the decomposition: small systems can be one class; as systems grow, split responsibilities.

### 7.6 Jigsaw (NxN puzzle)

**Core classes**:
- `Puzzle` — `LinkedList<Piece>` remaining, `Piece[][] solution`, `size`.
- `Piece` — `HashMap<Orientation, Edge> edges` (rotatable).
- `Edge` — `Shape` (INNER/OUTER/FLAT), back-pointer to parent `Piece`, `fitsWith(otherEdge)`.
- `Orientation` / `Shape` enums with `getOpposite()` methods.

**Solve algorithm**:
1. Group pieces into corners, borders, and insides.
2. Place a corner at (0,0).
3. Walk row-by-row, column-by-column. At each spot, select from the right group, find a matching edge, rotate the piece to match.

### 7.7 Chat Server

Focus: user management + conversations (skip networking).

Key objects:
- `UserManager` (singleton) — `usersById`, `usersByAccountName`, `onlineUsers` maps; mediates add/approve/reject requests.
- `User` — id, status, `privateChats` (map by other user), `groupChats` list, `contacts`, `sentAddRequests`, `receivedAddRequests`.
- `Conversation` abstract → `GroupChat` (add/remove participants) and `PrivateChat` (always two users).
- `Message` (content, date).
- `AddRequest`, `UserStatus`, enums `UserStatusType`, `RequestStatus`.

Flow for an add request:
1. UserA.requestAddUser(UserB)
2. → UserManager.addUser(fromA, toB)
3. → populates UserA.sentAddRequest and UserB.receivedAddRequest

Interesting discussion questions:
- How do we really know someone is online? (ping)
- Conflict between cache and DB?
- Scaling & cross-machine sync.
- DoS prevention.

### 7.8 Othello

Design decisions worth discussing:
- **Don't** subclass `BlackPiece`/`WhitePiece` — pieces flip constantly; destroying/creating is wasteful. Single `Piece` class with a `Color` flag.
- **Do** keep `Board` and `Game` separate: Board handles placement/flipping, Game handles flow/state.
- Score — maintained by Board (where it naturally sits).
- `Game` as singleton — convenient but debatable.

`Game.playPiece(r, c)` → `Board.placeColor(r, c, color)` → if valid, flip captured runs in all 4 directions → update score.

### 7.9 Circular Array (with iterator)

**Don't shift elements** on rotate — instead move a conceptual `head` pointer. `convert(i) = (head + i) % length`. `rotate(shiftRight)` = `head = convert(shiftRight)`.

**Iterator pattern** (Java):
1. `class CircularArray<T> implements Iterable<T>`.
2. Inner `CircularArrayIterator<TI> implements Iterator<TI>`.
3. Must implement `hasNext()`, `next()`, `remove()` (can throw UnsupportedOperationException).

Watch out for:
- Java doesn't allow `new T[size]`; cast from `Object[]`.
- Java's `%` can return negative: `((index % max) + max) % max`.
- First `for-each` iteration calls `hasNext` then `next` — `_current = -1` initially so `++_current` gives 0.

---

## Recurring OOD Patterns From These Solutions

| Pattern | Where Used |
|---|---|
| **Abstract base + concrete subclasses** | Employee / Vehicle / Card / Animal / Conversation |
| **Avoid over-subclassing when state flips** | Othello Piece (color flag vs. BlackPiece/WhitePiece) |
| **Don't merge when separation of concerns matters** | Board vs. Game (Othello, Minesweeper) |
| **Central manager / dispatcher** | UserManager (chat), CallHandler, Jukebox |
| **Data-only leaf classes** | CD, Song, Book, Message |
| **Enum for fixed categories** | Suit, Color, VehicleSize, Shape, Orientation, Rank |
| **Thin wrapper over a container for domain concepts** | Playlist (Queue), Deck (ArrayList + dealtIndex) |
| **Iterator implementation** | CircularArray |
| **Deferred allocation via dealtIndex pointer** | Deck.shuffle (don't remove cards, mark them unavailable) |

---

## Interview Takeaways

1. **Explicit assumptions beat guessed ones** — every OOD problem in this chapter starts with "ask the interviewer X, Y, Z; we'll assume...". Write the assumptions down.
2. **The walk through wrong solutions matters** — Random Node (4.11) and Poison (6.10) explicitly show failed options before the winner; this is the shape of real interview conversations.
3. **Worst-case balancing** is a recurring technique in puzzle problems (Egg Drop, Nine Balls).
4. **"Why 7 days to return a result?"** Wording clues reveal hidden constraints (Poison) — the delay is why parallel testing matters.
5. **Don't rattle off design patterns by name** — use them only when they fit the specific problem. Interviewers want judgment, not pattern vocabulary.
6. **Base Case and Build** reappears constantly: Blue-Eyed Island, Power Set, Recursion problems.
7. **Simplify representations**: single `Piece` with a color flag (Othello), single `ParkingSpot` with a size enum (Parking Lot). Subclassing for trivial differences is a code smell.
