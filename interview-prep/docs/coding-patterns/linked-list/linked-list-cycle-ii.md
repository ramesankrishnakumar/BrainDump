# Linked List Cycle II

LeetCode 142 *(Medium)* — a [fast & slow pointer](fast-slow-pointers.md) problem (rung 3).

## Problem

Given the `head` of a linked list, return the node where the **cycle begins**. If there is no
cycle, return `null`.

### Example

```
1 → 2 → 3 → 4 → 5
        ↑       ↓
        7   ←   6        entry = node 3
```

## Mental model

[Cycle detection](linked-list-cycle.md) tells you a loop *exists*; here you must find the exact
node where the tail re-enters the loop. Picture the list as a **lollipop** 🍭:

```
   stick            loop (candy)
1 → 2 → 3 → 4 → 5
        ↑           ↘
        7  ←  ←  ←   6
```

- The **stick** is the part before the loop (nodes 1, 2).
- The **candy** is the loop (3 → 4 → 5 → 6 → 7 → back to 3).
- The **entry** is node 3, where the stick joins the candy — the node we want.

The recipe: run fast/slow until they collide, **reset one pointer to `head`**, then advance
both **one step at a time**; they meet at the entry.

The reason (without algebra): because `fast` moves exactly twice as fast as `slow`, the
collision point is never random — its distance to the entry (going forward around the loop)
**equals** the distance from `head` to the entry. So a runner from `head` and a runner from
the collision point, both at the same pace, reach the entry gate together.

??? note "The distance proof, for the curious"
    Let `a` = head→entry, `b` = entry→collision, `c` = collision→entry (so the loop length is
    `b + c`). At collision, `slow` walked `a + b` and `fast` walked twice that, plus some whole
    number `k` of loops: `2(a + b) = a + b + k(b + c)`. Simplify to `a + b = k(b + c)`, then
    `a = (k − 1)(b + c) + c`. Reading it: `a` equals `c` plus some whole loops — which land on
    the same node. Hence head→entry ≡ collision→entry.

## Optimized solution

```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        ListNode slow = head, fast = head;

        // Phase 1: find a collision point inside the loop
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            if (slow == fast) {                  // collided
                // Phase 2: reset one pointer to head, advance both by 1
                ListNode p = head;
                while (p != slow) {
                    p = p.next;
                    slow = slow.next;
                }
                return p;                        // == cycle entry
            }
        }

        return null;                             // fast hit null → no cycle
    }
}
```

**Complexity**: O(n) time — both phases are linear. O(1) extra space.

## Walkthrough on the example

`1 → 2 → 3 → 4 → 5 → 6 → (back to 3)`, entry = node 3.

**Phase 1** runs fast/slow until they collide somewhere in the loop — say at node 5.

**Phase 2**: reset `p` to head (node 1), advance both one step:

| step | p (from head) | slow (from collision) |
|---|---|---|
| start | 1 | 5 |
| 1 | 2 | 6 |
| 2 | **3** | **3** ← meet at the entry ✓ |

## Pitfalls

!!! warning "Resetting the wrong pointer / moving at the wrong speed"
    In phase 2 **both** pointers move one step at a time. A common bug is keeping `fast` at
    double speed — it must drop to speed 1 here.

!!! warning "Returning the collision point as the answer"
    The collision happens somewhere *inside* the loop, not at its entry. Phase 2 is what
    converts the collision into the entry node.

!!! warning "Forgetting the no-cycle exit"
    If `fast` reaches `null`, there is no cycle — return `null` and never enter phase 2.

## Takeaway

!!! tip "Collide, reset one to head, walk both at equal speed → meet at the entry"
    The equality "head→entry ≡ collision→entry (mod loop length)" is a known property of the
    2× speed; you can state it in an interview without re-deriving the algebra. The same idea
    powers [Find the Duplicate Number](find-the-duplicate-number.md), where an array is read as
    a linked list with a cycle.
