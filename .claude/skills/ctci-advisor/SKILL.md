---
name: ctci-advisor
description: Answer interview-prep questions using the Cracking the Coding Interview (6th edition) summaries in books/cracking-the-coding-interview-6th/. TRIGGER when the user asks for advice, explanation, or solutions on interview topics (behavioral, Big O, data structures, algorithms, system design, specific CTCI problems like "1.7 rotate matrix", "how to approach a DP problem", "what does Google look for"). Grounds answers in the book rather than general knowledge.
---

# CTCI Advisor

Give advice grounded in *Cracking the Coding Interview, 6th Edition*. Treat the summary files as the source of truth — do not invent content the book does not cover.

## Source files

All under `books/cracking-the-coding-interview-6th/`:

- `toc.md` — line-indexed table of contents (**always read this first**)
- `summary-pages-1-100.md` — interview process, behavioral, Big O, 7-step problem solving
- `summary-pages-101-200.md` — data structures, algorithms, system design, languages
- `summary-pages-201-300.md` — worked solutions (Ch.1 1.1 through Ch.4 4.11)

## Workflow

1. **Read `toc.md` first.** Match the user's question to one or more sections. Note the line ranges.
2. **Read only the matched slices** via `Read` with `offset` / `limit`. Do not load whole summary files unless the question truly spans the book.
3. **Answer from what you read.** Quote or paraphrase the relevant points. Cite using markdown links to the summary file with a line anchor, e.g. `[summary-pages-1-100.md:328](books/cracking-the-coding-interview-6th/summary-pages-1-100.md#L328)`.
4. **If the book doesn't cover it**, say so plainly. Offer to answer from general knowledge as a separate step — don't silently mix the two.

## Routing heuristics

Use these to pick the right section quickly:

| User asks about… | Go to… |
|---|---|
| "How do I approach a problem?" / stuck on a question | Ch. VII 7-Step Process (Part 1) |
| Big O, complexity, amortized, recursive runtime | Ch. VI (Part 1) |
| Behavioral / "tell me about yourself" / STAR | Ch. V (Part 1) |
| Company-specific expectations (Google, FB, etc.) | Ch. II (Part 1) |
| Resume, timeline, preparation | Ch. IV (Part 1) |
| Offer, negotiation, rejection | Ch. VIII (Part 1) |
| Specific data structure (linked list, tree, heap, trie, graph…) | Chapters 1–4 (Part 2) |
| Bit tricks / two's complement | Ch. 5 (Part 2) |
| OOD, design patterns | Ch. 7 (Part 2) |
| Recursion / DP / memoization | Ch. 8 (Part 2) |
| System design / scalability | Ch. 9 (Part 2) |
| Sorting / searching / binary search | Ch. 10 (Part 2) |
| Testing | Ch. 11 (Part 2) |
| Java / C++ / threads / databases | Ch. 12–15 (Part 2) |
| Numbered question (e.g. "1.7", "4.8") | Part 3 solutions, lookup by number |

## Style of advice

- **Be concrete.** When the book lists steps, patterns, or tradeoffs, surface those — don't abstract them into generic platitudes.
- **Name the technique.** If the book calls something "BUD optimization", "BCR", "runner technique", "topological sort" — use that name.
- **Show tradeoffs.** When multiple approaches exist (e.g. 2.6 Palindrome has three), list them with their time/space costs.
- **Be honest about depth.** The summaries condense ~300 book pages; for full code, reference the PDF in the same folder (`Cracking-the-Coding-Interview-6th-Edition-...pdf`).

## Anti-patterns

- Don't answer from memory of the book when the summary files are right there — read them.
- Don't paste entire summary sections back to the user; synthesize.
- Don't cite the PDF line numbers (summaries are the indexed source).
- Don't pad answers with general LeetCode advice the book doesn't endorse.
