# Find the Duplicate Number

LeetCode 287 *(Medium)* — [fast & slow pointers](fast-slow-pointers.md) (rungs 2 + 3) hiding
inside an array problem.

## Problem

An array `nums` of `n + 1` integers, each in the range `[1, n]`. By the pigeonhole principle
at least one value repeats. Return the duplicate **without modifying the array** and using
**O(1) extra space**.

### Example

```
Input:  [1, 3, 4, 2, 2]
Output: 2
```

```
Input:  [3, 1, 3, 4, 2]
Output: 3
```

## Mental model

The constraints are the whole puzzle: *don't modify the array* rules out sorting; *O(1) space*
rules out a hash set. What structure gives O(1) space on a "find the repeat" problem? A
**cycle** in a linked list.

Read each value as a **pointer**: from index `i`, the "next" index is `nums[i]`. Because every
value is in `[1, n]`, every arrow lands on a valid index — you've built a linked list over the
indices, starting at index 0.

```
i:      0  1  2  3  4
nums:  [1, 3, 4, 2, 2]

0 → nums[0]=1 → nums[1]=3 → nums[3]=2 → nums[2]=4 → nums[4]=2 → nums[2]=4 → ...
                                              ↑__________________|
```

A duplicate value means **two different indices point to the same index** — that's two arrows
into one node, which forces a **cycle**. And the node where the cycle *starts* is the duplicate
value. So this is exactly [Linked List Cycle II](linked-list-cycle-ii.md): Floyd's collision
(rung 2) followed by the reset-to-start trick (rung 3).

## Optimized solution

```java
public class Solution {
    public int findDuplicate(int[] nums) {
        // Phase 1: detect the cycle (rung 2). "next of x" is nums[x].
        int slow = nums[0], fast = nums[0];
        do {
            slow = nums[slow];
            fast = nums[nums[fast]];
        } while (slow != fast);

        // Phase 2: find the cycle entry = the duplicate value (rung 3).
        slow = nums[0];
        while (slow != fast) {
            slow = nums[slow];
            fast = nums[fast];
        }
        return slow;
    }
}
```

**Complexity**: O(n) time, O(1) extra space. The array is never modified.

## Walkthrough on the example

`nums = [1, 3, 4, 2, 2]`, sequence starting at index 0: `0 → 1 → 3 → 2 → 4 → 2 → 4 → …`
(cycle is `2 → 4 → 2`, entry = value **2**).

**Phase 1** (slow ×1, fast ×2 over the `nums[...]` arrows) runs until `slow == fast` inside the
`2 → 4` cycle.

**Phase 2**: reset `slow` to `nums[0] = 1`, advance both by one arrow:

| step | slow | fast |
|---|---|---|
| start | 1 | (collision node, in {2,4}) |
| … | converge | converge |
| meet | **2** | **2** |

They meet at value **2** — the duplicate ✓.

## Pitfalls

!!! warning "Initialize from nums[0], use a do-while"
    Phase 1 must take at least one step before comparing (start `slow` and `fast` equal, then
    move). A `do { … } while (slow != fast)` is the cleanest way; a plain `while` that checks
    first would exit immediately.

!!! warning "fast advances two arrows: nums[nums[fast]]"
    The double hop is `nums[nums[fast]]`, not `nums[fast] + 1`. You're moving along the value-
    pointer chain, not along array indices.

!!! warning "Don't sort or use a set"
    Both solve it but violate the constraints (sorting mutates / O(n log n); a set is O(n)
    space). The constraints are a hint pointing straight at Floyd's.

!!! warning "Index 0 is the chosen start"
    The sequence starts at index 0. Because values are in `[1, n]`, no arrow ever returns to
    index 0, so index 0 is on the "stick", never inside the cycle — which is what makes the
    entry-finding step valid.

## Takeaway

!!! tip "Array + 'find the repeat' + O(1) space + no mutation ⇒ Floyd's cycle"
    Reading `nums[i]` as a pointer turns the array into a linked list whose cycle entry is the
    duplicate. Recognizing this disguise is the entire problem — the code is just
    [Linked List Cycle II](linked-list-cycle-ii.md).
