# Employee Free Time

## Problem

You're given a list `schedule` where `schedule[i]` is a list of **non-overlapping,
sorted** intervals representing employee `i`'s **busy** times. Return the list of finite
intervals during which **all** employees are simultaneously free — the **common free
time**, sorted.

Free time before everyone's first meeting and after everyone's last is infinite and not
reported; only the *gaps between* busy periods count.

### Example

```
Input:  schedule = [[[1, 3], [6, 7]], [[2, 4]], [[2, 5], [9, 12]]]
Output: [[5, 6], [7, 9]]
```

Pooling every employee's busy intervals and merging them gives the collective busy
blocks `[1, 5]`, `[6, 7]`, `[9, 12]`. The gaps between consecutive blocks — `[5, 6]` and
`[7, 9]` — are the times when nobody is busy.

## Mental model

"When is *everyone* free?" sounds like an intersection problem, but it's cleaner to flip
it: **everyone is free exactly when nobody is busy.** So:

1. **Flatten** — dump every employee's intervals into one big list. Whose interval it is
   no longer matters; only coverage of the timeline does.
2. **Merge** — run [Merge Intervals](merge-intervals.md) on the flattened list to get
   the collective **busy** blocks (sort by start, fold overlaps into the frontier).
3. **Read the gaps** — sweep the merged busy blocks in order; the span between the end
   of one block and the start of the next is a common free interval.

That's the **complement trick**: rather than compute free time directly, compute the
busy coverage and report the holes. Step 2 is exactly the merge you already know; step 3
is a one-line subtraction between neighbors.

## Optimized solution

This uses a plain `int[][]` flattening so the merge is identical to the
[Merge Intervals](merge-intervals.md) solution. (LeetCode's official version wraps
intervals in an `Interval` class; the logic is the same.)

```java
import java.util.Arrays;
import java.util.ArrayList;
import java.util.List;

public class Solution {
    public int[][] employeeFreeTime(int[][][] schedule) {
        // 1. flatten every employee's busy intervals into one list
        List<int[]> all = new ArrayList<>();
        for (int[][] employee : schedule) {
            for (int[] interval : employee) {
                all.add(interval);
            }
        }

        // 2. sort by start, then merge into collective busy blocks
        all.sort((a, b) -> Integer.compare(a[0], b[0]));
        List<int[]> busy = new ArrayList<>();
        for (int[] curr : all) {
            int[] last = busy.isEmpty() ? null : busy.get(busy.size() - 1);
            if (last != null && curr[0] <= last[1]) {
                last[1] = Math.max(last[1], curr[1]); // extend the frontier
            } else {
                busy.add(new int[] {curr[0], curr[1]});
            }
        }

        // 3. the gaps between consecutive busy blocks are the common free time
        List<int[]> free = new ArrayList<>();
        for (int i = 1; i < busy.size(); i++) {
            int gapStart = busy.get(i - 1)[1]; // end of previous busy block
            int gapEnd = busy.get(i)[0];       // start of next busy block
            if (gapStart < gapEnd) {           // non-empty gap
                free.add(new int[] {gapStart, gapEnd});
            }
        }

        return free.toArray(new int[0][]);
    }
}
```

**Complexity**: with `N` total intervals across all employees, O(N log N) time (the sort
dominates) and O(N) space.

!!! note "A faster merge with a heap"
    Each employee's intervals are *already sorted*, so instead of pooling and re-sorting
    you can k-way merge them with a min-heap keyed on start time — O(N log k) for `k`
    employees. Worth mentioning in an interview, but the sort-everything version above is
    simpler to write correctly and usually fast enough.

## Walkthrough on the example

`schedule = [[[1, 3], [6, 7]], [[2, 4]], [[2, 5], [9, 12]]]`.

**Flatten** → `[[1, 3], [6, 7], [2, 4], [2, 5], [9, 12]]`.

**Sort by start** → `[[1, 3], [2, 4], [2, 5], [6, 7], [9, 12]]`.

**Merge** into busy blocks:

| curr | frontier | test `curr.start <= frontier.end` | busy after |
|---|---|---|---|
| `[1, 3]` | — | — | `[[1,3]]` |
| `[2, 4]` | `[1, 3]` | `2 <= 3` → true, max(3,4)=4 | `[[1,4]]` |
| `[2, 5]` | `[1, 4]` | `2 <= 4` → true, max(4,5)=5 | `[[1,5]]` |
| `[6, 7]` | `[1, 5]` | `6 <= 5` → false | `[[1,5],[6,7]]` |
| `[9, 12]` | `[6, 7]` | `9 <= 7` → false | `[[1,5],[6,7],[9,12]]` |

**Gaps** between busy blocks `[1, 5]`, `[6, 7]`, `[9, 12]`:

| i | prev busy end | next busy start | gap |
|---|---|---|---|
| 1 | 5 | 6 | `[5, 6]` |
| 2 | 7 | 9 | `[7, 9]` |

Final: `[[5, 6], [7, 9]]`. ✓

## Pitfalls

!!! warning "Reporting the infinite ends"
    The time before the first busy block and after the last is free forever — but it's
    unbounded and not part of the answer. Only emit gaps that sit *between* two merged
    busy blocks, which the `i` starting at `1` loop does naturally.

!!! warning "Skipping the merge and diffing raw intervals"
    You must merge the busy times into collective blocks first. Computing gaps off the
    unmerged, interleaved list produces phantom "free" windows that are actually covered
    by another employee. The merge is what makes the gap read-off correct.

!!! warning "Emitting zero-width gaps when busy blocks touch"
    If one busy block ends exactly where the next begins (`busy[i-1].end ==
    busy[i].start`), there's no free time there. The `gapStart < gapEnd` guard drops
    these empty intervals.

!!! warning "Storing input references then mutating them during merge"
    As in [Merge Intervals](merge-intervals.md), copy with `new int[] {curr[0],
    curr[1]}` when starting a new frontier so the later `last[1] = ...` doesn't rewrite a
    caller's interval.

## Takeaway

!!! tip "Common free time = the complement of merged busy time"
    Don't compute the intersection of everyone's free time directly. Flatten all busy
    intervals, [merge](merge-intervals.md) them into collective busy blocks, then report
    the holes between blocks. This *merge-then-take-the-gaps* combination — the
    complement trick — is the capstone that ties the whole intervals pattern together.
