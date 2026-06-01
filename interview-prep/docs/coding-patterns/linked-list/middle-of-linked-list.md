# Middle of the Linked List

LeetCode 876 *(Easy)* — a [fast & slow pointer](fast-slow-pointers.md) problem (rung 1).

## Problem

Given the `head` of a singly linked list, return the **middle** node. If there are two middle
nodes (even length), return the **second** one.

### Example

```
Input:  1 → 2 → 3 → 4 → 5
Output: 3            (one middle)
```

```
Input:  1 → 2 → 3 → 4
Output: 3            (two candidates 2 and 3; return the second)
```

## Mental model

You don't know the length, and you can only walk forward. The naive route is two passes:
count, then walk to `length / 2`. The fast/slow trick does it in **one pass**.

> Two runners on a straight track; the sprinter goes twice as fast. When the sprinter reaches
> the end, the jogger is exactly halfway.

`slow` moves one node per step, `fast` moves two. When `fast` runs off the end, `slow` is
sitting on the middle. The speed ratio measures the halfway point for you — no length needed.

## Optimized solution

```java
public class Solution {
    public ListNode middleNode(ListNode head) {
        ListNode slow = head;
        ListNode fast = head;

        while (fast != null && fast.next != null) {
            slow = slow.next;        // 1 step
            fast = fast.next.next;   // 2 steps
        }

        return slow; // second of the two middles on an even-length list
    }
}
```

**Complexity**: O(n) time — one pass. O(1) extra space — two pointers.

## Walkthrough on the example

`head = 1 → 2 → 3 → 4 → 5` (odd, true middle = 3):

| step | slow | fast | loop test |
|---|---|---|---|
| start | 1 | 1 | enter |
| 1 | 2 | 3 | `fast.next`=4 → continue |
| 2 | 3 | 5 | `fast.next`=null → **stop** |

Returns `slow` = **3** ✓.

`head = 1 → 2 → 3 → 4` (even, candidates 2 and 3):

| step | slow | fast | loop test |
|---|---|---|---|
| start | 1 | 1 | enter |
| 1 | 2 | 3 | `fast.next`=4 → continue |
| 2 | 3 | null | `fast`=null → **stop** |

Returns `slow` = **3** — the *second* middle.

## Choosing which middle

The exact loop condition decides which middle you land on for even-length lists:

| You want | Change | Why |
|---|---|---|
| **Second** middle (default) | `slow = head; fast = head;` | `fast` finishes one step later, `slow` lands one further |
| **First** middle | `slow = head; fast = head.next;` | `fast`'s head start makes it finish one step sooner |

On **odd**-length lists there is exactly one middle and both variants return it — the offset
only breaks the tie that exists on even lists.

## Pitfalls

!!! warning "Loop condition order: `fast != null && fast.next != null`"
    Both checks are required and the order matters. `fast != null` guards even-length lists
    (where `fast` lands exactly on the end); `fast.next != null` guards odd-length lists
    (where `fast` lands one before the end). Drop either and you risk a `NullPointerException`
    on `fast.next.next`.

!!! warning "Returning the wrong middle"
    The default code returns the **second** middle on even lists. If the problem wants the
    first, give `fast` a one-node head start — don't try to special-case the length.

!!! warning "Empty list"
    `head == null`: the loop never runs and you return `null`. No special case needed if the
    condition is written correctly.

## Takeaway

!!! tip "slow ×1, fast ×2 → slow stops at the middle"
    This is the foundational fast/slow move and a *subroutine* inside bigger problems:
    [Reorder List](reverse-linked-list.md), Palindrome Linked List, and Sort List all start
    by finding the middle this way, then reverse or split from there.
