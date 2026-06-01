# Binary Search on Sorted Array

## Problem

Given an array of integers `nums` sorted in ascending order (no duplicates in the classic
formulation) and an integer `target`, return the index of `target` if it is in `nums`,
or `-1` if it is not.

You must write an algorithm with O(log n) runtime complexity.

### Example

```
Input:  nums = [-1, 0, 3, 5, 9, 12], target = 9
Output: 4
```

`9` is at index `4`.

```
Input:  nums = [-1, 0, 3, 5, 9, 12], target = 2
Output: -1
```

`2` is not in the array.

## Mental model

Maintain the set of indices that could still be the answer. Initially that's the whole
array: `lo = 0`, `hi = n - 1`.

Each step:

1. Look at the middle index `mid`.
2. If `nums[mid] == target`, done.
3. If `nums[mid] < target`, the target (if it exists) must be **strictly to the right** —
   discard `mid` and everything left of it: `lo = mid + 1`.
4. If `nums[mid] > target`, discard `mid` and everything right: `hi = mid - 1`.

Because the array is sorted, one comparison really does eliminate half the remaining
indices. After at most ⌈log₂(n)⌉ iterations, either you found `target` or `lo > hi`
(empty range → not found).

Use `mid = lo + (hi - lo) / 2` instead of `(lo + hi) / 2` — in languages where
integers overflow, the sum form can wrap; the difference form is safe everywhere.

## Optimized solution

```java
public class Solution {
    public int search(int[] nums, int target) {
        int lo = 0;
        int hi = nums.length - 1;

        while (lo <= hi) {
            int mid = lo + (hi - lo) / 2;

            if (nums[mid] == target) {
                return mid;
            }
            if (nums[mid] < target) {
                lo = mid + 1;
            } else {
                hi = mid - 1;
            }
        }

        return -1;
    }
}
```

**Complexity**: O(log n) time, O(1) space.

## Walkthrough on the example

`nums = [-1, 0, 3, 5, 9, 12]`, `target = 9`.

| iteration | lo | hi | mid | nums[mid] | branch | new range |
|---|---|---|---|---|---|---|
| 1 | 0 | 5 | 2 | 3 | `3 < 9` → too small | `lo = 3`, hi = 5 → `[3..5]` |
| 2 | 3 | 5 | 4 | 9 | `9 == 9` | **return 4** ✓ |

Second example: `target = 2`.

| iteration | lo | hi | mid | nums[mid] | branch | new range |
|---|---|---|---|---|---|---|
| 1 | 0 | 5 | 2 | 3 | `3 > 2` → too large | `[0..1]` |
| 2 | 0 | 1 | 0 | -1 | `-1 < 2` | `[1..1]` |
| 3 | 1 | 1 | 1 | 0 | `0 < 2` | `lo = 2`, hi = 1 |

`lo > hi` → return **-1** ✓.

## Pitfalls

!!! warning "Using `while (lo < hi)` with the closed-interval updates"
    With `lo = mid + 1` and `hi = mid - 1`, the loop guard must be `lo <= hi`. If you
    use `lo < hi` but keep those updates, you can skip the only remaining index or exit
    one step early. Match the guard to your interval definition.

!!! warning "Not moving `lo` or `hi` past `mid`"
    Writing `lo = mid` or `hi = mid` when `nums[mid]` isn't the answer risks an infinite
    loop on two-element ranges. When `nums[mid] < target`, you already know `mid` isn't
    the answer — use `lo = mid + 1`. Symmetrically `hi = mid - 1`.

!!! warning "`mid = (lo + hi) / 2` overflow"
    Rare in Java for interview-sized inputs, but interviewers notice: prefer
    `lo + (hi - lo) / 2`.

!!! warning "Binary search on an unsorted array"
    Order is what lets you discard half. If the array isn't sorted, sort first (O(n log
    n)) or use a hash map for O(1) lookup — don't binary search blindly.

## Takeaway

!!! tip "Sorted + index answer → halve the range each step"
    The closed-interval template (`lo`, `hi`, `while (lo <= hi)`, safe `mid`, move
    `lo`/`hi` past `mid`) is the root of the whole family: first/last position, insert
    position, rotated array, minimum in rotated array, and binary search on the answer.
    Learn this loop until it's muscle memory; harder problems only change *what* you
    compare at `mid`.
