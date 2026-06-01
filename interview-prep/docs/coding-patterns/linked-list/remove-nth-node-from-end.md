# Remove Nth Node From End

LeetCode 19 *(Medium)* — a [fast & slow pointer](fast-slow-pointers.md) problem (rung 4).

## Problem

Given the `head` of a linked list, remove the **nth node from the end** and return the head.

### Example

```
Input:  1 → 2 → 3 → 4 → 5,  n = 2
Output: 1 → 2 → 3 → 5            (the 2nd-from-end, node 4, is removed)
```

```
Input:  1,  n = 1
Output: null
```

## Mental model

This rung uses the **fixed-gap** flavor: same speed, but one pointer starts ahead.

> Two people walk a hallway at the same speed, but the leader starts a fixed number of steps
> ahead. That gap never changes. The moment the leader reaches the far wall, the follower is
> exactly that many steps from the wall.

To *remove* the nth node we need to stop on the node **just before** it, so we give `fast` an
`n + 1` step head start. A **dummy node** in front of `head` removes the special case where the
node to delete is the head itself.

## Optimized solution

```java
public class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode dummy = new ListNode(0, head);
        ListNode slow = dummy, fast = dummy;

        // head start of n+1 so slow lands just BEFORE the target
        for (int i = 0; i <= n; i++) {
            fast = fast.next;
        }

        // move both at the same speed until fast runs off the end
        while (fast != null) {
            slow = slow.next;
            fast = fast.next;
        }

        slow.next = slow.next.next;   // splice out the nth-from-end node
        return dummy.next;
    }
}
```

**Complexity**: O(n) time — one pass (the head start plus the walk total one traversal).
O(1) extra space.

## Walkthrough on the example

`1 → 2 → 3 → 4 → 5`, `n = 2`. With `dummy` in front, head start = `n + 1 = 3`:
`fast` ends at node 3, `slow` at `dummy`.

| step | slow | fast | note |
|---|---|---|---|
| start | dummy | 3 | gap of 3 (n+1) holds |
| 1 | 1 | 4 | |
| 2 | 2 | 5 | |
| 3 | 3 | null | `fast` off the end → stop |

`slow` = node 3, which sits just before the target node 4. `slow.next = slow.next.next` links
3 → 5, removing 4. Result `1 → 2 → 3 → 5` ✓.

## Variations

| Goal | Head start | Dummy needed? |
|---|---|---|
| **Remove** nth from end | `n + 1` (stop one before target) | Yes — target might be the head |
| **Return** nth-from-end node (don't remove) | `n` | No |

## Pitfalls

!!! warning "Off-by-one in the head start"
    To *delete* you must stop on the predecessor, so advance `fast` by `n + 1`, not `n`. To
    just *return* the node, advance by `n`. Mixing these up removes the wrong node.

!!! warning "Removing the head without a dummy"
    If the target is the head (`n == length`), there's no predecessor to rewire. The dummy node
    gives you one uniformly, so no separate branch is needed.

!!! warning "n larger than the list"
    A robust solution guards against `n` exceeding the length (the head start would walk `fast`
    past `null` and crash on `fast.next`). LeetCode guarantees valid `n`, but mention the guard.

## Takeaway

!!! tip "Fixed gap = position from the end in one pass"
    Hold a constant gap between two same-speed pointers and the trailing one is always pinned to
    a fixed distance from the end — no length count required. The `n + 1` head start plus a dummy
    head is the clean template for end-relative deletions.
