# Non-Overlapping Intervals

## Problem

Given an array `intervals` where `intervals[i] = [start, end]`, return the **minimum
number of intervals you must remove** so that the rest are non-overlapping.

Touching intervals are allowed: `[1, 2]` and `[2, 3]` don't overlap, so neither needs
removing.

### Example

```
Input:  intervals = [[1, 2], [2, 3], [3, 4], [1, 3]]
Output: 1
```

Remove `[1, 3]` and the remaining `[1, 2], [2, 3], [3, 4]` chain cleanly end-to-end.
That's the fewest possible — one removal.

```
Input:  intervals = [[1, 2], [1, 2], [1, 2]]
Output: 2
```

Three identical intervals all overlap; keep one, remove two.

## Mental model

"Remove the fewest to make them non-overlapping" is the same question as "**keep the
most** non-overlapping intervals" — the answer is `n - kept`. And *keep the most
non-overlapping intervals* is the textbook **activity-selection** greedy problem.

The greedy rule:

> Sort by **end** time. Walk left to right, and greedily **keep every interval whose
> start is ≥ the end of the last interval you kept.** Anything that overlaps the last
> kept interval gets dropped.

Why sort by *end* and not start? The interval that **finishes earliest** leaves the
most room for everything that comes after it. Each time you commit to the
earliest-ending compatible interval, you maximize the remaining free space — and a
standard exchange argument shows this is provably optimal. (Sorting by *start* doesn't
work: a very early-starting but very late-ending interval like `[1, 100]` would get
picked first and block everything, even though dropping it lets many others fit.)

This is the deliberate contrast to [Merge Intervals](merge-intervals.md): there you sort
by **start** to *combine* overlaps; here you sort by **end** to *survive* them.

## Optimized solution

```java
import java.util.Arrays;

public class Solution {
    public int eraseOverlapIntervals(int[][] intervals) {
        if (intervals.length == 0) return 0;

        // sort by END time: earliest-finishing interval first
        Arrays.sort(intervals, (a, b) -> Integer.compare(a[1], b[1]));

        int kept = 0;
        int prevEnd = Integer.MIN_VALUE; // end of the last interval we kept

        for (int[] curr : intervals) {
            if (curr[0] >= prevEnd) {
                // no overlap with the last kept interval → keep it
                kept++;
                prevEnd = curr[1];
            }
            // else: curr overlaps; drop it. Because we sorted by end, curr ends no
            // earlier than prevEnd, so keeping it could only hurt — safe to remove.
        }

        return intervals.length - kept; // removals = total minus those we kept
    }
}
```

**Complexity**: O(n log n) time (sort dominates), O(1) extra space beyond the sort.

## Walkthrough on the example

`intervals = [[1, 2], [2, 3], [3, 4], [1, 3]]`.

Sort by end → `[[1, 2], [2, 3], [1, 3], [3, 4]]` (ends `2, 3, 3, 4`).

| curr | test `curr.start >= prevEnd` | action | kept | prevEnd |
|---|---|---|---|---|
| `[1, 2]` | `1 >= -∞` → true | keep | 1 | 2 |
| `[2, 3]` | `2 >= 2` → true | keep | 2 | 3 |
| `[1, 3]` | `1 >= 3` → false | **drop** | 2 | 3 |
| `[3, 4]` | `3 >= 3` → true | keep | 3 | 4 |

`kept = 3`, total `= 4`, removals `= 4 - 3 = 1`. ✓

Notice `[1, 3]` and `[2, 3]` tie on end time. Whichever the sort places first is kept;
the other gets dropped against the *next* element. The greedy stays optimal regardless
of how ties are broken, because both finish at the same time.

## Pitfalls

!!! warning "Sorting by start instead of end"
    Sort-by-start breaks the greedy. `[[1, 100], [2, 3], [3, 4]]` sorted by start keeps
    `[1, 100]` first and forces two removals; sorted by end it keeps `[2, 3]` and
    `[3, 4]` and removes only `[1, 100]` — one removal. **End time is the key.**

!!! warning "Using `>` instead of `>=` for touching intervals"
    Here `[1, 2]` and `[2, 3]` don't overlap, so the keep test is `curr.start >=
    prevEnd`. If a variant treats touching as overlapping, switch to `>`. (This is the
    mirror of the `<` vs `<=` choice in the overlap test — same endpoint-semantics
    decision.)

!!! warning "Counting overlaps directly and getting tangled"
    It's easier and less error-prone to count what you *keep* and subtract, than to
    count removals on the fly. The "keep the earliest-ending compatible interval" frame
    has one clean invariant; ad-hoc removal counting invites double-counting.

!!! warning "Forgetting the empty-input guard"
    `prevEnd = Integer.MIN_VALUE` handles the first real interval, but an empty array
    should short-circuit to `0` — the loop produces that too, but the explicit guard
    documents intent.

## Takeaway

!!! tip "Min removals = total − max kept; keep by sorting on END"
    Reframe "remove the fewest" as "keep the most non-overlapping," which is
    activity-selection: sort by end time and greedily take the earliest-finishing
    interval that doesn't conflict. The sort key is the whole insight — **start for
    merging** ([Merge Intervals](merge-intervals.md)), **end for greedy scheduling**.
