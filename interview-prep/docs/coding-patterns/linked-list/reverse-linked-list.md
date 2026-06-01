# Reverse Linked List

## Problem

Given the `head` of a singly linked list, reverse the list and return the new head.

### Example

```
Input:  head = 1 → 2 → 3 → null
Output: 3 → 2 → 1 → null
```

```
Input:  head = null
Output: null
```

```
Input:  head = 1 → null
Output: 1 → null
```

## Mental model

You are not building a new list — you are **rewiring existing nodes in place**. At each
step, peel off the current node and attach it to the front of the reversed prefix.

Three pointers keep the invariants straight:

| Pointer | Role |
|---|---|
| `prev` | Head of the reversed prefix built so far (starts `null`) |
| `curr` | Node you're about to flip |
| `next` | Saved `curr.next` so the tail of the unreversed suffix isn't lost |

The critical order inside the loop: **save `next` before you overwrite `curr.next`**. Once
you point `curr.next` at `prev`, the old forward link is gone unless you saved it.

When `curr` becomes `null`, every node has been flipped and `prev` points at the new
head.

## Optimized solution

```java
public class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode prev = null;
        ListNode curr = head;

        while (curr != null) {
            ListNode next = curr.next;
            curr.next = prev;
            prev = curr;
            curr = next;
        }

        return prev;
    }
}
```

**Complexity**: O(n) time — one pass. O(1) extra space — only the three pointers.

A recursive solution also works (`reverse rest, then attach head at tail`), but the
iterative version is what you want under time pressure: no call-stack depth, same logic
spelled out explicitly.

## Walkthrough on the example

`head = 1 → 2 → 3 → null`.

| step | prev | curr | next (saved) | after `curr.next = prev` | list state (conceptual) |
|---|---|---|---|---|---|
| start | null | 1 | — | — | `1 → 2 → 3` |
| 1 | 1 | 2 | 2 | `1 → null` | `1 ← null`, rest `2 → 3` |
| 2 | 2 | 3 | 3 | `2 → 1` | `2 → 1`, rest `3` |
| 3 | 3 | null | — | `3 → 2` | `3 → 2 → 1` |

Loop exits (`curr == null`). Return `prev` → node `3` → **new head** ✓.

## Pitfalls

!!! warning "Losing `next` before rewiring"
    If you set `curr.next = prev` before saving `curr.next` into `next`, you can no
    longer reach the rest of the list. Always `ListNode next = curr.next` first.

!!! warning "Returning `curr` instead of `prev`"
    When the loop ends, `curr` is `null` — that's how you know you're done. The new head
    is `prev`, the last node you processed. Returning `curr` is always wrong here.

!!! warning "Not handling empty or single-node lists"
    Empty (`head == null`): loop never runs, return `null`. Single node: one iteration,
    `prev` becomes that node, `curr` becomes `null`, return the node. No special cases
    required if the loop structure is correct.

!!! warning "Creating new nodes"
    Reversal in interviews almost always means rewire `next` pointers on existing nodes.
    Allocating new `ListNode` objects works but wastes space and misses the skill being
    tested.

## Takeaway

!!! tip "Save next, reverse link, march forward"
    The three-pointer reversal loop is the template behind merge-two-lists, reorder-list,
    and reverse-k-group. Learn this loop cold — the only changes in harder problems are
    *where* you stop, *how* you reconnect segments, and whether you need a dummy head
    before the real head moves.
