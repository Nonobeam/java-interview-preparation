# P6: Detect a cycle in a linked list

Given the head of a singly linked list, return `true` if it contains a cycle, otherwise `false`. A cycle exists when some node's `next` points back to a previously visited node.

```java
boolean hasCycle(ListNode head)
```

Bonus follow-up: return the **node where the cycle begins**.

## Examples

```
1 → 2 → 3 → 4 → null              → false
1 → 2 → 3 → 4 ──┐                 → true   (cycle starts at node 2)
        ↑       │
        └───────┘
1 → 1 (single node, points to itself) → true
null                              → false
```

## Constraints

- Cannot modify the list.
- Solve it in **O(1) extra space** (the classic ask). The HashSet approach is acceptable but call out its O(n) space cost.

## What's being tested

- Knowing **Floyd's tortoise-and-hare** algorithm (two-pointer fast/slow).
- The math intuition for **why** the cycle-start follow-up works.
- Trade-off vs the easier HashSet-of-visited-nodes approach.
