# S5: Reverse a singly linked list

## Iterative — three pointers

Walk forward, flipping each `next` pointer to point backward.

```java
public ListNode reverse(ListNode head) {
    ListNode prev = null;
    ListNode curr = head;
    while (curr != null) {
        ListNode next = curr.next;   // save the rest
        curr.next = prev;            // flip
        prev = curr;                 // advance prev
        curr = next;                 // advance curr
    }
    return prev;                     // new head
}
```

**Time:** O(n). **Space:** O(1).

### Walkthrough on `1 → 2 → 3 → null`

| step | prev | curr | next |
|------|------|------|------|
| init | null | 1    | —    |
| 1    | 1    | 2    | 2    |
| 2    | 2    | 3    | 3    |
| 3    | 3    | null | null |

After the loop `prev = 3`. List is now `3 → 2 → 1 → null`. ✅

## Recursive — recurse to the tail, then rewire on unwind

```java
public ListNode reverse(ListNode head) {
    if (head == null || head.next == null) return head;   // base case

    ListNode newHead = reverse(head.next);   // reverse the rest
    head.next.next = head;                   // rewire: tail of reversed sublist points back to us
    head.next = null;                        // we are now the tail

    return newHead;                          // unchanged through all levels
}
```

**Time:** O(n). **Space:** O(n) on the call stack — risk of `StackOverflowError` for very long lists (10⁵+).

### Why `head.next.next = head` works

After `reverse(head.next)`, the sublist starting at `head.next` is reversed. `head.next` is now the **tail** of that reversed sublist (its old role was the second node, now it's last). So `head.next.next = head` appends `head` after it, and `head.next = null` cuts the old forward link.

## Compare

| | Iterative | Recursive |
|---|-----------|-----------|
| Time      | O(n)      | O(n)       |
| Space     | O(1)      | O(n) stack |
| Risk      | none      | StackOverflowError on long lists |
| Reads as  | "shift three pointers" | "trust recursion, fix the boundary" |

For interviews, **always prefer iterative** unless asked specifically; mention the recursive version exists and what its trade-off is.

## Common bugs to avoid

- Forgetting to save `curr.next` before reassigning → you lose the rest of the list.
- Returning `head` instead of `prev` at the end → returns the original head, which is now the last node.
- In recursion, forgetting `head.next = null` → the new tail still points back into the list, creating a cycle.

## Variant: reverse a sublist between positions `m..n`

Same three-pointer trick applied between two anchors. Common LeetCode follow-up — worth knowing the structure but not required for this problem.

## Interview one-liner

> "Iterative: keep three pointers `prev/curr/next`, save next, flip curr's next to prev, advance — O(1) space. Recursive: recurse to the tail, then `head.next.next = head; head.next = null` on unwind — clean code but O(n) stack. Default to iterative; mention recursion as a stylistic alternative with a stack-overflow caveat."
