# Binary Search

## What it is

**Binary search** halves a sorted search space on every comparison. Instead of scanning
every element (O(n)), you maintain a range `[lo, hi]` of indices that could still hold
the answer, pick `mid`, and discard half the range based on one compare.

It only works when the structure gives you a **monotone** decision: "if `nums[mid]` is
too small, everything to the left is also too small (or not the answer we want), so
shrink the range in one direction." Sorted arrays are the textbook case.

## When to reach for it

Strong signals you're in binary-search territory:

- The input is **sorted** (or you can sort once and search many times).
- You're asked for a **specific index**, the **first/last** occurrence, or whether a
  value exists — not "enumerate all matches."
- The problem says O(log n) time, or the array is huge and linear scan is ruled out.
- **Answer-space search** (advanced): "minimum speed so all packages arrive on time" —
  binary search on the answer, not on the array. The classic sorted-array template below
  is the foundation; answer-space problems reuse the same `lo` / `hi` / `mid` loop with
  a different predicate.

If the data is unsorted and order doesn't help you eliminate half the candidates, binary
search is the wrong tool.

## Core technique: closed interval `[lo, hi]`

Use indices `lo` and `hi` inclusive — both ends are still candidates. Loop while
`lo <= hi`. Compute `mid` without overflow: `lo + (hi - lo) / 2`.

```java
public int search(int[] nums, int target) {
    int lo = 0;
    int hi = nums.length - 1;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;

        if (nums[mid] == target) {
            return mid;
        }
        if (nums[mid] < target) {
            lo = mid + 1;  // target is strictly to the right
        } else {
            hi = mid - 1;  // target is strictly to the left
        }
    }

    return -1; // lo > hi → empty range → not found
}
```

**Invariant**: if `target` exists in `nums`, its index is always in `[lo, hi]`. Each
iteration shrinks the range. When `lo > hi`, the range is empty and the target isn't
there.

**Complexity**: O(log n) time — halving each step. O(1) extra space.

## `lo <= hi` vs `lo < hi`

This guide uses the **closed-interval** form (`lo <= hi`, both ends inclusive). An
alternative is half-open `[lo, hi)` with `while (lo < hi)` — equally valid if you're
consistent about whether `hi` is inclusive. Pick one template and stick to it; mixing
forms mid-interview is where off-by-one bugs come from.

## Problems

- [Binary Search on Sorted Array](binary-search.md) *(Easy)* — baseline template: find
  `target` index or return `-1`. Everything else in this family extends the same loop.
