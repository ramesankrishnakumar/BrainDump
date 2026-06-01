# Sliding Window

## What it is

A **contiguous** range over an array or string (a "window") that you advance one step at a time. Instead of recomputing the window's aggregate (sum, count, max, distinct-count, …) from scratch at every position, you **edit it incrementally**: add the new element on one side, remove the old element on the other. That makes each slide O(1) and the whole pass O(n).

Two flavors:

| Flavor | Window size | Typical question |
|---|---|---|
| **Fixed-size** | `k`, never changes | "Best/worst aggregate over every contiguous run of size `k`." |
| **Variable-size** | grows and shrinks | "Longest/shortest contiguous range satisfying some predicate." |

## When to reach for it

You're looking at a problem about a **contiguous** subarray or substring, and the answer can be computed from a *running aggregate* that's cheap to update as the window moves:

- Max / min / sum / average over every window of size `k`.
- Longest substring with at most `k` distinct characters.
- Shortest subarray with sum ≥ `target`.
- Number of subarrays satisfying some condition.

If the candidate subarrays are **not** required to be contiguous, sliding window is the wrong tool.

## When to use fixed-size

Reach for the fixed-size flavor when the problem **names a length `k`** and asks about *every* contiguous window of that size — typically "best aggregate over any length-`k` window."

Signals:

- The word "length `k`" or "size `k`" appears in the spec.
- You're asked for the **best/worst aggregate** (max sum, min average, max distinct count, …) over all such windows.
- The window size never depends on the data — it's fixed by the input parameter.

Shape: every iteration does the same work — add the new right element, evict the old left element once the window is full, check the aggregate.

```java
public int fixedLengthSlidingWindow(int[] nums, int k) {
    Map<Integer, Integer> state = new HashMap<>(); // or running sum, max-deque, etc.
    int start = 0;
    int best = 0;

    for (int end = 0; end < nums.length; end++) {
        // extend window
        // add nums[end] to state in O(1) time

        if (end - start + 1 > k) {
            // contract window exactly once to keep size == k
            // remove nums[start] from state in O(1) time
            start++;
        }

        if (end - start + 1 == k) {
            // INVARIANT: window is exactly size k here — read the aggregate and update best
        }
    }

    return best;
}
```

Two pointers, `start` and `end`, same as the variable-size template — that's deliberate. The **only** structural difference between the two flavors is the shrink step: fixed-size uses `if` and contracts at most once (to maintain `size == k`); variable-size uses `while` and contracts as many times as needed (to restore the invariant).

## When to use variable-size

Reach for the variable-size flavor when the problem asks for the **longest or shortest** contiguous range satisfying some **predicate** — the window size is the *answer*, not an input.

Signals:

- "Longest/shortest subarray such that …"
- "Maximum/minimum length window where …"
- The constraint is a **monotone invariant**: once a window violates it, *every* larger window containing it also violates it. (E.g., "at most `k` distinct" — adding more elements can only add distinct types.) That's what lets the left pointer move forward and never back.

Shape: expand on the right every step; when the invariant breaks, shrink from the left in a `while` loop until it holds again; record the candidate.

```java
public int variableLengthSlidingWindow(int[] nums) {
    Map<Integer, Integer> state = new HashMap<>(); // choose appropriate data structure
    int start = 0;
    int max_ = 0;

    for (int end = 0; end < nums.length; end++) {
        // extend window
        // add nums[end] to state in O(1) in time

        while (/* state is not valid */) {
            // repeatedly contract window until it is valid again
            // remove nums[start] from state in O(1) in time
            start++;
        }

        // INVARIANT: state of current window is valid here.
        max_ = Math.max(max_, end - start + 1);
    }

    return max_;
}
```

The `while` is the key difference from the fixed-size version: shrinking continues until the invariant is restored, which may take multiple steps in one iteration. Amortized cost is still O(1) per element because `start` only moves forward.

## The mental trick

Sometimes the cleanest window is **not** the thing you're picking — it's the thing you're **leaving behind**. If you're asked to pick elements from the ends of an array, the untouched middle is a contiguous window of a fixed size. Sliding the *complement* is often simpler than tracking the picked region directly. The Max Points From Cards problem below is a clean example of this.

## Problems

- [Max Points You Can Obtain From Cards](max-points-from-cards.md) — pick `k` cards from either end; slide the complement to make it a clean fixed-size window problem.
- [Max Sum of Distinct Subarrays of Length k](max-sum-distinct-subarrays.md) — fixed-size window plus a count map acting as a "distinctness sensor."
- [Fruits Into Baskets](fruits-into-baskets.md) — variable-size window; expand → restore invariant (≤ 2 distinct) → record. Generalizes to "longest subarray with at most `k` distinct."
