# Cracking the Coding Interview (6th Edition) — Pages 1–100 Summary

---

## Chapter I: The Interview Process

### How You Are Evaluated

Interviewers assess five dimensions:

| Dimension | What They Look For |
|---|---|
| **Analytical Skills** | How optimally do you solve problems? Did you need hints? |
| **Coding Skills** | Clean, correct, organized code. Awareness of edge cases. |
| **Technical Knowledge / CS Fundamentals** | Strong CS foundation, not just knowing the syntax. |
| **Experience** | Past projects, leadership, initiative. |
| **Culture Fit / Communication** | Values, personality, work style alignment. |

### Why Algorithm Questions?

- Hard problems force differentiation — you can't just memorize the answer.
- They test how you *think*, not just what you know.
- Math/logic puzzles are **not** used at most top companies (Google, Amazon, Microsoft, Facebook).

### How Questions Are Selected

- Interviewers often choose questions they find interesting; there's no central bank enforced uniformly.
- Questions are calibrated relative to other candidates, not against a fixed pass/fail threshold.

### FAQ

- You don't need to get every question right to pass.
- Companies look for your best — one strong performance can carry an otherwise weak interview.
- Dress code: slightly better than the company's norm (business casual usually safe).
- A single bad interview doesn't always fail you — depends on the company's policy.

---

## Chapter II: Behind the Scenes

### Microsoft

- Emphasizes **smart, passionate people**.
- 4–5 interviews with engineers and possibly a hiring manager.
- Recruiter acts as your **advocate** — be kind to them.
- "Why do you want to work at Microsoft?" is a key question.
- Must impress every interviewer; one "No Hire" can block you.

### Amazon

- Starts with **phone screen(s)** then on-site with 4–5 interviews.
- Has a **Bar Raiser** — a specially trained interviewer empowered to veto.
- Heavy focus on **scalability** questions.
- Two written code submissions on the phone screen.

### Google

- **Hiring Committee (HC)** makes the final decision — your interviewers don't decide alone.
- Strong emphasis on **analytical (algorithm) skills**.
- Typically 4–6 on-site interviews.
- Feedback is written up and reviewed by the HC.

### Apple

- Requires genuine **passion for Apple products**.
- Technical + culture fit equally weighted.
- 6–8 on-site interviews; ends with a director/VP review.
- Consensus across interviewers is important.

### Facebook

- Three engineering tracks: **Jedi** (generalist), **Ninja** (technical excellence), **Pirate** (infrastructure/systems).
- Values **entrepreneurial spirit** — "move fast and break things" culture.
- Behavioral questions focus on building things and taking initiative.

### Palantir

- Known for the **hardest technical questions** in the industry.
- Uses a **HackerRank** coding challenge as initial filter.
- Typically two phone interviews before on-site.
- Focuses on systems design and analytical depth.

---

## Chapter III: Special Situations

### Experienced Candidates

- Expected to know more system design and architecture.
- Algorithms still matter but broader experience weighs more.

### SDETs (Software Dev Engineers in Test)

- Must prepare: **core coding + testing mindset**.
- Be ready for: test case design, test automation, edge cases.
- Also need general CS fundamentals.

### PM (Product Manager) Roles

- Key skills: **Handling ambiguity, customer focus, multi-level communication, teamwork**.
- Less focus on code, more on product thinking and prioritization.

### Dev Lead / Manager Roles

- Must still **code** — don't assume leadership exempts you.
- Additional focus: leadership style, prioritization, conflict resolution, communication.

### Startups

- **Personality fit** is critical in addition to skills.
- Be prepared to discuss why you want to work there specifically.
- Technical breadth valued over depth.

### For Interviewers

- Use **medium/hard problems** that require real thinking.
- Look for multiple solution approaches (not just one right answer).
- Use **positive reinforcement** — guide, don't intimidate.
- Ask follow-up questions to explore depth of knowledge.

---

## Chapter IV: Before the Interview

### Getting the Right Experience

- Contribute to **large, impactful projects** at your current job.
- Do **internships** if still in school — aim for brand-name ones.
- Build **side projects** (open source, apps, tools) to demonstrate initiative.
- Use platforms like HackerRank, LeetCode to practice.

### Resume Best Practices

- **1 page** for < 10 years of experience; 2 pages max for more.
- Use the formula: **"Accomplished X by implementing Y which led to Z"**
  - Bad: "Worked on optimizing the database."
  - Good: "Reduced database query time by 40% by implementing connection pooling, cutting average page load from 3s to 1.8s."
- List **2–4 projects** with technologies used and impact.
- **Language stigmas to be aware of**: VB6, early PHP, and similar can hurt perception — list them carefully.
- Include GPA only if strong (3.5+).

### Preparation Timeline

| Timeline | Focus |
|---|---|
| 1+ year out | Get experience, contribute to big projects |
| 3–12 months | Practice algorithms, build projects |
| 1–3 months | Intensive coding practice (LeetCode-style) |
| 4 weeks | Mock interviews, focus weak areas |
| 1 week | Review data structures/algorithms, do timed problems |
| Day before | Light review, rest, no cramming |
| Day of | Eat well, arrive early, be calm |

---

## Chapter V: Behavioral Questions

### Interview Prep Grid

Before your interview, fill out a grid with stories from your experience for each category:

| Category | What to Prepare |
|---|---|
| **Challenges** | A hard technical or interpersonal problem you overcame |
| **Mistakes** | Something that went wrong and what you learned |
| **Enjoyed** | A project/task that genuinely excited you |
| **Leadership** | A time you led without formal authority |
| **Conflicts** | Disagreement with a coworker/manager and how you resolved it |
| **What You'd Do Differently** | Reflective story showing growth |

### Know Your Technical Projects

For each project you list:
- The **challenges** you faced
- The **mistakes** you made
- What you **enjoyed**
- Your specific **technical contributions**
- What you'd do **differently**

### How to Answer Behavioral Questions

- **Be Specific, Not Arrogant**: Say what *you* did, but stay factual.
- **Limit Details**: Give the key story, not the full novel — let them ask for more.
- **Focus on Yourself**: Even in team stories, emphasize your role.
- **Give Structured Answers**:
  - **Nugget First**: Start with a one-sentence summary, then elaborate.
  - **SAR**: Situation → Action → Result (keep it tight and concrete).

### "Tell Me About Yourself" Structure

1. **Current Role**: One line about your current job and impact.
2. **College**: Relevant education/internships.
3. **Post College**: Career progression since graduation.
4. **Current Role Details**: What you're doing day-to-day.
5. **Outside of Work**: Side projects, open source, interests.
6. **Wrap Up**: Why you're here and what you're looking for.

---

## Chapter VI: Big O Notation

### The Core Idea

Big O describes how **runtime or space scales** as input size grows.

**File transfer analogy**:
- Email a 1 GB file: O(s) — grows linearly with file size.
- Drive it on a plane: O(1) — constant regardless of file size.
- For small files email wins; at some large size the plane wins.

### Academic vs. Industry Terminology

| Term | Meaning |
|---|---|
| **Big O** (O) | Upper bound — worst case |
| **Big Omega** (Ω) | Lower bound — best case |
| **Big Theta** (Θ) | Tight bound — both upper and lower |
| **Industry "Big O"** | Usually means Theta (tight bound) in practice |

### Best / Worst / Expected Case

- **Best case**: Rarely useful; trivial inputs.
- **Worst case**: Most commonly what interviewers mean by "Big O".
- **Expected case**: Average performance across inputs (e.g., QuickSort).

### Space Complexity

- Count the memory your algorithm *allocates*, not the input itself.
- **Stack space counts**: recursive calls consume O(depth) stack frames.
  ```
  sum(n) = n == 0 ? 0 : n + sum(n-1)
  ```
  → O(n) space for n recursive calls on the call stack.

### Drop the Constants

- O(2N) → **O(N)** — constants don't matter at scale.
- O(N + N) → O(N) — same.

### Drop Non-Dominant Terms

- O(N² + N) → **O(N²)**
- O(N + log N) → **O(N)**
- Keep only the term that dominates as N → ∞.

### Multi-Part Algorithms

- **Sequential** (do A, then do B): **Add** — O(A + B)
- **Nested** (for each A, do B): **Multiply** — O(A × B)

### Amortized Time — ArrayList

- Inserting N elements triggers occasional O(N) copy (when doubling).
- Total work: N + N/2 + N/4 + ... ≈ 2N → amortized **O(1)** per insert.

### Log N Runtimes

Any algorithm that **halves the problem** at each step is O(log N).

Example — Binary Search:
```
search(arr, x, lo, hi):
  if lo > hi: return -1
  mid = (lo + hi) / 2
  if arr[mid] == x: return mid
  if arr[mid] < x: return search(arr, x, mid+1, hi)
  else: return search(arr, x, lo, mid-1)
```
→ O(log N) — halving each time.

### Recursive Runtimes

**O(branches^depth)** for balanced recursion trees.

```java
int f(int n) {
    if (n <= 1) return 1;
    return f(n-1) + f(n-1);
}
```
→ 2 branches, depth N → **O(2^N)**

### Key Worked Examples

| Problem | Time Complexity | Notes |
|---|---|---|
| Print pairs in array | O(N²) | Two nested loops |
| Print unordered pairs across two arrays | O(A × B) | Different variables |
| Fibonacci (naive) | O(2^N) | Binary tree of calls |
| Fibonacci (memoized) | O(N) | Each value computed once |
| All permutations of string | O(N × N!) | N! permutations, N chars each |
| Powers of 2 up to N | O(log N) | Halves each time |
| Binary search | O(log N) | Halves search space |
| Merge sort | O(N log N) | log N levels, N work per level |

---

## Chapter VII: Technical Questions

### How to Prepare

1. Solve problems **on your own** first — no hints, no looking up.
2. Write code **on paper** (whiteboard simulation).
3. **Test on paper** with actual test cases.
4. Type into a computer and run it to validate.

### Must-Know Data Structures

| Category | Structures |
|---|---|
| **Data Structures** | Linked Lists, Trees, Tries, Graphs, Stacks, Queues, Heaps, Vectors/ArrayLists, Hash Tables |
| **Algorithms** | BFS, DFS, Binary Search, Merge Sort, Quick Sort |
| **Concepts** | Bit Manipulation, Memory (Stack vs Heap), Recursion, Dynamic Programming, Big O (Time & Space) |

### Powers of 2 — Quick Reference

| Power | Value |
|---|---|
| 2^7 | 128 |
| 2^8 | 256 |
| 2^10 | 1,024 (~1K) |
| 2^16 | 65,536 |
| 2^20 | 1,048,576 (~1M) |
| 2^30 | ~1 billion (~1GB) |
| 2^32 | ~4 billion |
| 2^40 | ~1 trillion (~1TB) |

### The 7-Step Problem-Solving Process

```
1. LISTEN Carefully
2. Draw an EXAMPLE (large, not trivial)
3. State a BRUTE FORCE solution
4. OPTIMIZE the brute force
5. WALK THROUGH your approach (before coding)
6. IMPLEMENT the code
7. TEST your solution
```

#### Step 1: Listen Carefully
- Absorb every constraint in the problem statement.
- Ask clarifying questions.
- Unique/unusual details are usually there for a reason.

#### Step 2: Draw an Example
- Use a **large, real example** — not degenerate cases like a 2-element array.
- Avoid special-case examples that don't reflect the general structure.

#### Step 3: State the Brute Force
- Even if it's O(N³) — say it out loud, then optimize.
- Shows you can reason systematically.

#### Step 4: Optimize — BUD Technique

**B — Bottlenecks**: Find the step that dominates runtime and attack it.

**U — Unnecessary Work**: Remove redundant operations.
```
Example: Finding a³ + b³ = c³ + d³
Brute force: 4 nested loops = O(N⁴)
Fix: precompute a³ + b³ for all pairs → O(N²) lookup
```

**D — Duplicated Work**: Cache results that are recomputed.
```
Example: Same as above — after BU optimization, also remove 
redundant inner loops by storing in a hash map.
```

#### Step 5: Walk Through
- Before typing code, verify your algorithm mentally.
- Understand how each piece fits together.
- Know what variables you need.

#### Step 6: Implement
- Write **modular** code — use helper functions.
- Use **good variable names** (not `i`, `j` unless conventional — use `startChild`, etc.).
- Don't panic — the interviewer can see you think out loud.

#### Step 7: Test
Test in this order:
1. **Conceptual test**: Read code like a code review.
2. **Unusual/weird cases**: null inputs, 0, single elements.
3. **Edge cases**: Off-by-one, empty array, large values.
4. **Small test cases**: Trace through a 4–5 element example step by step.
5. **Big test cases**: Verify performance characteristics.

> Don't just say "looks correct" — actually trace through with test data.

---

### Optimization Techniques Summary

#### DIY (Do It Yourself)
- When stuck, ask: "How would *I* do this by hand with a large example?"
- Your brain naturally finds optimizations — then code that algorithm.
- Example: Find permutation of string s2 within s1 → naturally compare character frequencies in sliding windows → sliding window approach.

#### Simplify & Generalize
1. Simplify the problem (change constraints, simplify types).
2. Solve the simplified version.
3. Generalize the solution back to the original problem.

#### Base Case and Build
- Solve for n=1, then n=2, then n=3, building on prior solutions.
- Often leads naturally to **recursive** solutions.
- Example: Print all permutations → base: 1 char; build: insert new char at each position of shorter permutations.

#### Data Structure Brainstorm
- Rapidly cycle through: hash table, tree, heap, stack, queue, etc.
- Ask: "Would using X help here?"
- Example (tracking median):
  - Array? Insertion O(N).
  - Sorted array? Insert O(N), find median O(1).
  - Min-heap + Max-heap? Insert O(log N), median O(1). ✓

### Best Conceivable Runtime (BCR)

The theoretical minimum time any correct algorithm *must* take, given what the problem requires.

> "You cannot do better than BCR — it's a lower bound."

Example — Sorted array intersection:
- BCR: O(N) — must at least read all elements.
- Brute force: O(N²) — two nested loops.
- Sort + two pointers: O(N log N) then O(N) → **O(N log N)**.
- Actually O(N) achievable because array is already sorted → use two pointers → **O(N)**.
- Since O(N) = BCR, no further optimization possible — stop.

### Handling Incorrect Answers

- It's not about getting it right immediately — it's about **how you reason**.
- Incorrect approaches that are analyzed and corrected **demonstrate good thinking**.
- Never give up or freeze — talk through your reasoning even when unsure.

### What Good Code Looks Like

| Quality | Description |
|---|---|
| **Correct** | Handles all cases, including edge cases |
| **Efficient** | Optimal time and space complexity |
| **Simple** | No unnecessary complexity |
| **Readable** | Other engineers can understand it |
| **Maintainable** | Easy to extend and modify |

> Don't blindly optimize for performance at the cost of readability — balance matters.

---

## Chapter VIII: The Offer and Beyond

### Offer Deadlines

- Most companies give 1–4 weeks to decide.
- **Ask for more time** if you need it — they often grant it.
- Don't accept and renege — it damages relationships and your reputation.

### Declining Offers

- Be gracious and prompt.
- Keep the relationship warm — you may want to apply again later.
- Send a personal note to your interviewers.

### Handling Rejection

- Not every rejection is permanent — you can re-apply after 6–12 months.
- Ask for feedback when possible (many companies won't give it, but some will).
- Use the experience to identify weak areas and improve.

### Evaluating an Offer

Evaluate on four dimensions — don't just optimize salary:

| Factor | What to Consider |
|---|---|
| **Financial Package** | Salary, bonus, equity vesting, signing bonus |
| **Career Development** | Mentorship, learning opportunities, promotion path |
| **Company Stability** | Funding runway, growth trajectory, leadership quality |
| **Happiness Factor** | Team culture, work-life balance, project type |

### Salary Negotiation

- **Just do it** — negotiating never costs you the offer.
- **Have a viable alternative** — competing offers give you real leverage.
- **Make a specific ask** — "I'm looking for $X" beats "more money."
- **Overshoot** slightly — you can only go down in negotiation, not up.
- **Think beyond salary**: sign-on bonus, equity acceleration, extra PTO, remote flexibility.

### On the Job

- **Set a timeline**: Decide how long you'll give a role before reassessing.
- **Build relationships**: Strong networks open doors more than technical skills alone.
- **Ask for what you want**: Promotions, projects, raises — advocate for yourself.
- **Keep interviewing**: Stay sharp; interview every 1–2 years even if happy.

---

## Section IX: Interview Questions

### Chapter 1: Arrays and Strings

#### Hash Tables

A **hash table** maps keys to values for O(1) average-case lookup.

**Implementation**:
- An array of **linked lists** (to handle collisions via chaining).
- A **hash function** maps keys → array indices.

**Process for lookup**:
1. Compute `hash(key)`.
2. Map hash to array index: `hash % array_size`.
3. Walk the linked list at that index to find the key.

**Time Complexity**:
- **Average**: O(1) lookup, insert, delete.
- **Worst case**: O(N) if many collisions (all keys hash to same bucket).

**Alternative implementation**: Balanced BST instead of linked lists.
- Guarantees O(log N) worst case.
- Uses less memory (no pre-allocated large array).

---

## Quick Reference: Must-Know Complexity

| Operation | Data Structure | Time |
|---|---|---|
| Access by index | Array | O(1) |
| Search unsorted | Array/Linked List | O(N) |
| Search sorted | Array (binary search) | O(log N) |
| Insert/Delete at end | Array | O(1) amortized |
| Insert/Delete at middle | Array | O(N) |
| Insert/Delete | Linked List | O(1) with pointer |
| Lookup | Hash Table | O(1) average |
| Insert/Delete/Find | Balanced BST | O(log N) |
| Insert/Delete/Find | Heap | O(log N) |
| BFS/DFS | Graph | O(V + E) |
