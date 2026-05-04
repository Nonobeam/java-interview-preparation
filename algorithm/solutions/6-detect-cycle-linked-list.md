# S6: Detect a cycle in a linked list

## Approach 1 — Floyd's tortoise & hare (O(1) space)

Two pointers, one moving 1 step (`slow`), one moving 2 steps (`fast`).

- If there's no cycle, `fast` reaches `null` and we return `false`.
- If there's a cycle, `fast` eventually laps `slow` from behind — they meet inside the cycle. Return `true`.

```java
public boolean hasCycle(ListNode head) {
    if (head == null || head.next == null) return false;

    ListNode slow = head;
    ListNode fast = head;

    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
```

**Time:** O(n). **Space:** O(1).

## Why fast catches slow

Inside the cycle, the gap between fast and slow shrinks by 1 each step (fast gains 1 net step on slow). If the cycle has length `c`, fast catches up within at most `c` steps after both are inside the cycle.

## Approach 2 — HashSet of visited nodes (O(n) space)

Easier to write; explain the trade-off:

```java
public boolean hasCycle(ListNode head) {
    Set<ListNode> seen = new HashSet<>();
    while (head != null) {
        if (!seen.add(head)) return true;   // already visited → cycle
        head = head.next;
    }
    return false;
}
```

**Time:** O(n). **Space:** O(n).

Use this only if the interviewer is fine with extra space; otherwise default to Floyd.

## Bonus — find the cycle's starting node

Once `slow` and `fast` meet inside the cycle, reset one pointer to the head and step both **one at a time**. They meet at the cycle's start.

```java
public ListNode detectCycleStart(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) {
            // phase 2: find the start
            ListNode p = head;
            while (p != slow) {
                p = p.next;
                slow = slow.next;
            }
            return p;
        }
    }
    return null;   // no cycle
}
```

### Why phase 2 works

Let:
- `L` = distance from head to cycle start
- `C` = cycle length
- `k` = distance from cycle start to the meeting point (going forward inside the cycle)

When they meet:
- slow has walked `L + k`
- fast has walked `2(L + k)` and is `k` past the cycle start (modulo `C`)
- fast - slow = `L + k`, which must be a multiple of `C` (fast is laps ahead)
- So `L + k = nC`, i.e. `L = nC - k`

Distance from head to the start = `L`.
Distance from the meeting point back to the start (going forward) = `C - k`.
For `n = 1`: `L = C - k`. They are equal — so two pointers stepping in lockstep meet exactly at the start.

(For `n > 1`, both walk an extra full cycle — still meet at the start.)

## Edge cases

- `head == null` → `false`.
- Single node, no self-loop → `false` (loop exits because `head.next == null`).
- Single node, self-loop (`head.next == head`) → `true` immediately on the first iteration.
- Very long acyclic list → fast hits null in n/2 steps. No special handling.

## Interview one-liner

> "Floyd's tortoise-and-hare: slow moves 1, fast moves 2; if they meet, there's a cycle. O(n) time, O(1) space. For the cycle-start follow-up, after they meet reset one pointer to head and step both by 1 — they meet at the start. The math: head-to-start distance equals meeting-point-to-start distance, modulo the cycle length."
