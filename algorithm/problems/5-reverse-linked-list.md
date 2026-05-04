# P5: Reverse a singly linked list

Given the head of a singly linked list, reverse it and return the new head.

```java
class ListNode {
    int val;
    ListNode next;
    ListNode(int val) { this.val = val; }
}

ListNode reverse(ListNode head)
```

## Examples

```
1 → 2 → 3 → 4 → null
becomes
4 → 3 → 2 → 1 → null

null               → null
1 → null           → 1 → null
```

## Constraints

- The list may be empty (`head == null`) or have a single node.
- The list is mutated **in place** — no new nodes allocated.
- Both **iterative** and **recursive** solutions are expected; know the trade-offs.

## What's being tested

- Pointer manipulation without losing the rest of the list (the classic "save next, then rewire" pattern).
- Recursion: writing the base case correctly + the rewire on unwind.
- Stack vs. iterative trade-offs (recursion is O(n) stack space).
