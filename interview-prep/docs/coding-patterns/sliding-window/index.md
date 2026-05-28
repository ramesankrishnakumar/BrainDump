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

## The mental trick

Sometimes the cleanest window is **not** the thing you're picking — it's the thing you're **leaving behind**. If you're asked to pick elements from the ends of an array, the untouched middle is a contiguous window of a fixed size. Sliding the *complement* is often simpler than tracking the picked region directly. The Max Points From Cards problem below is a clean example of this.

## Problems

- [Max Points You Can Obtain From Cards](max-points-from-cards.md) — pick `k` cards from either end; slide the complement to make it a clean fixed-size window problem.
- [Max Sum of Distinct Subarrays of Length k](max-sum-distinct-subarrays.md) — fixed-size window plus a count map acting as a "distinctness sensor."
