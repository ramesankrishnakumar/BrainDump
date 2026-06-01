# Can Attend Meetings

## Problem

You're given an array `intervals` where `intervals[i] = [start, end]` is the time span
of meeting `i`. Determine whether a single person could attend **all** of them — i.e.,
whether **no two meetings overlap**.

A meeting `[start, end]` is end-exclusive: a meeting `[0, 30]` and one `[30, 60]` do
**not** conflict, because the first ends exactly as the second begins.

### Example

```
Input:  intervals = [[0, 30], [5, 10], [15, 20]]
Output: false
```

Sorted by start, `[0, 30]` overlaps `[5, 10]` (the second meeting starts at 5, while the
first is still running until 30). One person can't be in two places at once, so they
can't attend all three.

```
Input:  intervals = [[7, 10], [2, 4]]
Output: true
```

`[2, 4]` finishes before `[7, 10]` starts — no conflict.

## Mental model

This is the gentlest interval problem and the one that establishes the core primitive:
**sort by start time, then it's enough to compare each meeting with the one immediately
before it.**

Why only the neighbor? After sorting by start, the meetings march left to right in
time. If meeting `i` doesn't overlap meeting `i-1` (the one with the closest earlier
start), it can't overlap anything before `i-1` either — those started even earlier and,
since the previous one already cleared, the whole prefix is clear. So a single backward
glance at the previous meeting decides everything.

The overlap test for adjacent sorted meetings `prev` and `curr`:

> `curr.start < prev.end` → they overlap → answer is `false`.

Because meetings are end-exclusive, we use strict `<`: touching endpoints
(`curr.start == prev.end`) are fine.

## Optimized solution

```java
import java.util.Arrays;

public class Solution {
    public boolean canAttendMeetings(int[][] intervals) {
        // sort by start time so conflicts are always between neighbors
        Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));

        for (int i = 1; i < intervals.length; i++) {
            // current meeting begins before the previous one ends → overlap
            if (intervals[i][0] < intervals[i - 1][1]) {
                return false;
            }
        }
        return true; // no neighbor conflicted → all meetings are attendable
    }
}
```

**Complexity**: O(n log n) time, dominated by the sort; the scan itself is O(n). O(1)
extra space if the sort is in place (Java's `Arrays.sort` on objects is a stable
mergesort, so O(n) auxiliary in practice — but no extra structures beyond that).

## Walkthrough on the example

`intervals = [[0, 30], [5, 10], [15, 20]]`.

After sorting by start: `[[0, 30], [5, 10], [15, 20]]` (already in order).

| i | prev = intervals[i-1] | curr = intervals[i] | test `curr.start < prev.end` | verdict |
|---|---|---|---|---|
| 1 | `[0, 30]` | `[5, 10]` | `5 < 30` → true | **overlap → return false** |

We never reach `i = 2`; the first conflict short-circuits the answer to `false`. ✓

Contrast with `[[2, 4], [7, 10]]` (already sorted): at `i = 1`, `7 < 4` is false, the
loop finishes, and we return `true`.

## Pitfalls

!!! warning "Forgetting to sort"
    The neighbor-only comparison is **only valid after sorting by start**. On unsorted
    input, two conflicting meetings can sit far apart in the array and the adjacent scan
    misses them. Sort first, always.

!!! warning "Using `<=` instead of `<` for end-exclusive meetings"
    With end-exclusive times, `[0, 30]` and `[30, 60]` do **not** conflict. The test
    must be `curr.start < prev.end`. If the problem says intervals are *inclusive* on
    both ends, flip to `<=`. This single character is the most common bug in the whole
    intervals family — re-read the problem's endpoint semantics before committing.

!!! warning "Comparing against the wrong frontier"
    Compare `curr.start` against `prev.end`, not `prev.start`. The conflict is about
    when the previous meeting *finishes*, not when it began.

## Takeaway

!!! tip "Sort by start, then a single backward glance decides overlap"
    Once intervals are sorted by start time, any overlap shows up between *adjacent*
    elements. This "sort then scan neighbors" move is the seed of every interval
    problem — [Merge Intervals](merge-intervals.md) is the exact same scan, except
    instead of returning `false` on a conflict you *combine* the two intervals and keep
    going.
