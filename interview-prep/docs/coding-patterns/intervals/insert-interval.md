# Insert Interval

## Problem

You're given a list `intervals` that is **already sorted by start** and contains **no
overlaps**, plus a single `newInterval = [start, end]`. Insert the new interval into the
list so the result is still sorted and non-overlapping, **merging where necessary**.
Return the new list.

### Example

```
Input:  intervals = [[1, 3], [6, 9]], newInterval = [2, 5]
Output: [[1, 5], [6, 9]]
```

`[2, 5]` overlaps `[1, 3]` (starts at 2, before 3), so they merge into `[1, 5]`.
`[6, 9]` starts after 5, so it's untouched.

```
Input:  intervals = [[1, 2], [3, 5], [6, 7], [8, 10], [12, 16]], newInterval = [4, 8]
Output: [[1, 2], [3, 10], [12, 16]]
```

`[4, 8]` swallows `[3, 5]`, `[6, 7]`, and `[8, 10]` into one block `[3, 10]`.

## Mental model

The input is **already sorted and clean**, so you don't need to re-sort and run a full
[Merge Intervals](merge-intervals.md) pass — that would throw away the structure you
were handed. Instead, walk the list once in **three phases**:

1. **Before** — copy every interval that ends *strictly before* `newInterval` starts
   (`interval.end < newInterval.start`). These sit entirely to the left and can't merge.
2. **Merge** — for every interval that *overlaps* `newInterval`
   (`interval.start <= newInterval.end`), absorb it by widening `newInterval`:
   `newInterval.start = min(...)`, `newInterval.end = max(...)`. Don't emit anything yet
   — keep growing the single combined interval. When the overlapping run ends, push the
   final merged `newInterval` once.
3. **After** — copy every remaining interval (they all start after the merged block).

Because the list is pre-sorted, these three groups appear **contiguously** in order, so
a single left-to-right pointer cleanly transitions phase 1 → 2 → 3. No sorting required,
which is what makes this O(n) instead of O(n log n).

## Optimized solution

```java
import java.util.ArrayList;
import java.util.List;

public class Solution {
    public int[][] insert(int[][] intervals, int[] newInterval) {
        List<int[]> out = new ArrayList<>();
        int i = 0;
        int n = intervals.length;

        // Phase 1 — intervals entirely before newInterval: copy as-is
        while (i < n && intervals[i][1] < newInterval[0]) {
            out.add(intervals[i]);
            i++;
        }

        // Phase 2 — intervals overlapping newInterval: absorb into newInterval
        while (i < n && intervals[i][0] <= newInterval[1]) {
            newInterval[0] = Math.min(newInterval[0], intervals[i][0]);
            newInterval[1] = Math.max(newInterval[1], intervals[i][1]);
            i++;
        }
        out.add(newInterval); // emit the single merged interval once

        // Phase 3 — intervals entirely after: copy the rest
        while (i < n) {
            out.add(intervals[i]);
            i++;
        }

        return out.toArray(new int[0][]);
    }
}
```

**Complexity**: O(n) time — a single pass, no sort, since the input is already ordered.
O(n) space for the output.

## Walkthrough on the example

`intervals = [[1, 2], [3, 5], [6, 7], [8, 10], [12, 16]]`, `newInterval = [4, 8]`.

| phase | i | intervals[i] | test | action | `newInterval` | `out` |
|---|---|---|---|---|---|---|
| 1 | 0 | `[1, 2]` | `2 < 4` → true | copy | `[4, 8]` | `[[1,2]]` |
| 1 | 1 | `[3, 5]` | `5 < 4` → false | exit phase 1 | `[4, 8]` | `[[1,2]]` |
| 2 | 1 | `[3, 5]` | `3 <= 8` → true | merge: min(4,3)=3, max(8,5)=8 | `[3, 8]` | `[[1,2]]` |
| 2 | 2 | `[6, 7]` | `6 <= 8` → true | merge: max(8,7)=8 | `[3, 8]` | `[[1,2]]` |
| 2 | 3 | `[8, 10]` | `8 <= 8` → true | merge: max(8,10)=10 | `[3, 10]` | `[[1,2]]` |
| 2 | 4 | `[12, 16]` | `12 <= 10` → false | exit phase 2, emit | `[3, 10]` | `[[1,2],[3,10]]` |
| 3 | 4 | `[12, 16]` | — | copy | `[3, 10]` | `[[1,2],[3,10],[12,16]]` |

Final: `[[1, 2], [3, 10], [12, 16]]`. ✓

## Pitfalls

!!! warning "Emitting merged pieces inside the loop instead of once at the end"
    Phase 2 keeps *growing* a single `newInterval`; it must be added to the output
    exactly once, **after** the overlap run finishes. Adding it on every iteration
    produces duplicate, half-merged intervals.

!!! warning "Re-sorting the input"
    The list arrives sorted and non-overlapping — that's the gift that makes this O(n).
    Sorting again is wasted work and signals you've reverted to plain
    [Merge Intervals](merge-intervals.md).

!!! warning "Boundary confusion: `<` for phase 1, `<=` for phase 2"
    Phase 1 keeps intervals strictly to the left (`end < newStart`). Phase 2 absorbs
    anything that touches or overlaps (`start <= newEnd`). Mixing these up either
    swallows a non-overlapping neighbor or skips a touching one. Touching intervals
    *do* merge here.

!!! warning "Mutating the caller's newInterval without expecting side effects"
    The solution widens `newInterval` in place. That's fine for the return value, but if
    the caller reuses the array afterward they'll see the merged bounds. Copy first if
    that matters.

## Takeaway

!!! tip "Pre-sorted input → a three-phase single pass, no sort"
    When intervals are already sorted and clean, don't reach for a full merge. Split the
    work into *before / merge / after* and let one pointer flow through the phases. The
    overlap math is identical to [Merge Intervals](merge-intervals.md) — only the
    surrounding structure changes because you've been handed order for free.
