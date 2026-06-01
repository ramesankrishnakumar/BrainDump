# Merge Intervals

## Problem

Given an array `intervals` where `intervals[i] = [start, end]`, **merge all overlapping
intervals** and return an array of the non-overlapping intervals that cover exactly the
same set of points.

Touching intervals count as overlapping here: `[1, 4]` and `[4, 5]` merge into `[1, 5]`.

### Example

```
Input:  intervals = [[1, 3], [2, 6], [8, 10], [15, 18]]
Output: [[1, 6], [8, 10], [15, 18]]
```

`[1, 3]` and `[2, 6]` overlap (the second starts at 2, before the first ends at 3), so
they collapse into `[1, 6]`. `[8, 10]` and `[15, 18]` touch nothing and pass through
unchanged.

## Mental model

This is the **foundational interval technique** — the one the rest of the family is
built on. The recipe:

1. **Sort by start time.** Now any interval can only overlap with the run of intervals
   immediately before it, never with something far behind.
2. **Sweep once, folding into the last output interval.** Keep a growing output list.
   For each interval, look at the **last interval already in the output** (the
   *frontier*):
   - If the current interval starts at or before the frontier ends, they overlap —
     **extend the frontier** by pushing its end out to `max(frontier.end, curr.end)`.
   - Otherwise there's a gap — **append** the current interval as a new frontier.

The one subtlety worth internalizing: when merging, the new end is
`max(frontier.end, curr.end)`, **not** just `curr.end`. A later interval can be fully
*contained* inside the frontier (e.g. frontier `[1, 9]`, current `[2, 5]`); naively
taking `curr.end` would shrink the frontier and lose coverage. The `max` guards against
that.

Because sorting put the smallest start first, the frontier's *start* never needs
updating — only its end can grow.

## Optimized solution

```java
import java.util.Arrays;
import java.util.ArrayList;
import java.util.List;

public class Solution {
    public int[][] merge(int[][] intervals) {
        // 1. sort by start so overlapping intervals are adjacent
        Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));

        List<int[]> merged = new ArrayList<>();
        for (int[] curr : intervals) {
            int[] last = merged.isEmpty() ? null : merged.get(merged.size() - 1);

            if (last != null && curr[0] <= last[1]) {
                // overlap (or touch): push the frontier's end outward
                last[1] = Math.max(last[1], curr[1]);
            } else {
                // disjoint: curr starts a new merged block
                // copy into a fresh array so we mutate the output, not the input
                merged.add(new int[] {curr[0], curr[1]});
            }
        }
        return merged.toArray(new int[0][]);
    }
}
```

**Complexity**: O(n log n) time (the sort dominates; the sweep is O(n)), O(n) space for
the output list.

## Walkthrough on the example

`intervals = [[1, 3], [2, 6], [8, 10], [15, 18]]` — already sorted by start.

| curr | frontier (last in `merged`) | test `curr.start <= frontier.end` | action | `merged` after |
|---|---|---|---|---|
| `[1, 3]` | — (empty) | — | append | `[[1,3]]` |
| `[2, 6]` | `[1, 3]` | `2 <= 3` → true | extend end to `max(3,6)=6` | `[[1,6]]` |
| `[8, 10]` | `[1, 6]` | `8 <= 6` → false | append | `[[1,6],[8,10]]` |
| `[15, 18]` | `[8, 10]` | `15 <= 10` → false | append | `[[1,6],[8,10],[15,18]]` |

Final: `[[1, 6], [8, 10], [15, 18]]`. ✓

To see the `max` matter, picture `[[1, 9], [2, 5]]`: at `[2, 5]` the test `2 <= 9` is
true, and `max(9, 5) = 9` correctly keeps the frontier at `[1, 9]` instead of shrinking
it to `[1, 5]`.

## Pitfalls

!!! warning "Taking `curr.end` instead of `max(frontier.end, curr.end)`"
    A fully-contained interval (`[2, 5]` inside `[1, 9]`) would shrink the frontier and
    drop coverage. Always extend with `max`.

!!! warning "Mutating the input array instead of copying"
    `merged.add(curr)` stores a reference to the *input* row. A later
    `last[1] = ...` then silently rewrites the caller's data. Copy with
    `new int[] {curr[0], curr[1]}` when appending a new frontier.

!!! warning "Forgetting that touching intervals merge"
    Here `[1, 4]` and `[4, 5]` merge, so the test is `<=`. If a variant treats touching
    as *non*-overlapping, switch to `<`. As always, the endpoint semantics come from the
    problem statement — see the note in [Can Attend Meetings](can-attend-meetings.md).

!!! warning "Sorting by end instead of start"
    Merging requires sort by **start** so the frontier's start is fixed and only its end
    grows. Sorting by end is for the *greedy keep-the-most* problems like
    [Non-Overlapping Intervals](non-overlapping-intervals.md) — different goal, different
    key.

## Takeaway

!!! tip "Sort by start, then fold each interval into the running frontier"
    This is the interval pattern's workhorse. The entire decision is local: compare the
    current interval to the *last one in the output*; extend it with `max` or start a
    new block. [Insert Interval](insert-interval.md) is this same fold restricted to a
    single new interval on already-sorted data, and [Employee Free Time](employee-free-time.md)
    runs this merge and then reports the gaps between blocks.
