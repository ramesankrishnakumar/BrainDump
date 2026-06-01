# Linked List Cycle

LeetCode 141 *(Easy)* — a [fast & slow pointer](fast-slow-pointers.md) problem (rung 2).

## Problem

Given the `head` of a linked list, determine whether the list has a **cycle** — i.e. some node
whose `next` points back to an earlier node, so the list never terminates.

### Example

```
1 → 2 → 3 → 4
        ↑    ↓
        7 ← 6 ← 5        (6.next points back to 3)  →  true
```

```
1 → 2 → 3 → null                                     →  false
```

## Mental model

Every method that "walks to the end" assumes the list *has* an end (`fast` eventually hits
`null`). A cycle breaks that assumption. So first we answer: does a loop exist at all?

> Two runners on a **circular** track; the sprinter goes twice as fast. The sprinter keeps
> lapping and will eventually collide with the jogger from behind. On a straight track they
> never meet — the sprinter just exits at the finish.

So: **if `slow` and `fast` ever land on the same node, there's a cycle. If `fast` reaches
`null`, there isn't.**

## Optimized solution

```java
public class Solution {
    public boolean hasCycle(ListNode head) {
        ListNode slow = head;
        ListNode fast = head;

        while (fast != null && fast.next != null) {
            slow = slow.next;        // 1 step
            fast = fast.next.next;   // 2 steps
            if (slow == fast) {      // same node → collision
                return true;
            }
        }

        return false;                // fast escaped to null → no cycle
    }
}
```

**Complexity**: O(n) time — `slow` traverses at most the full list before a collision or
`null`. O(1) extra space — the alternative (a `HashSet` of visited nodes) costs O(n) space.

`slow == fast` compares **node identity** (same object), not values.

## Why they're guaranteed to meet

Once both pointers are inside the loop, the gap between them (measured in steps around the
loop) shrinks by exactly **1** each iteration: `fast` moves 2, `slow` moves 1, net +1 closure.
A gap that decreases by exactly one can never skip over `slow` — it must hit 0, which is a
collision.

And on a **straight** list a false collision is impossible: after the first step `fast` is
ahead of `slow` and moving away faster, so the gap only grows until `fast` falls off the end.

| Situation | What happens | Result |
|---|---|---|
| Straight list | gap grows, `fast` hits `null` | `false` |
| Looped list | gap shrinks by 1 each step → 0 | `true` |

## Pitfalls

!!! warning "Checking `slow == fast` before the first move"
    Both pointers start at `head`, so they're equal before any movement. Move first, *then*
    compare — otherwise you falsely report a cycle immediately.

!!! warning "Loop guard `fast != null && fast.next != null`"
    You dereference `fast.next.next`, so both `fast` and `fast.next` must be non-null. Short-
    circuit order matters: test `fast != null` first.

!!! warning "Reaching for a HashSet"
    Storing visited nodes works and is O(n) time, but it's O(n) space. Floyd's gets the same
    answer in O(1) space — mention the set as a baseline, then optimize.

## Takeaway

!!! tip "Collision ⇔ cycle"
    Two speeds, net gap closure of one per step: they collide if and only if the track loops.
    Knowing a cycle *exists* sets up the harder follow-up — finding **where** it starts, in
    [Linked List Cycle II](linked-list-cycle-ii.md).
