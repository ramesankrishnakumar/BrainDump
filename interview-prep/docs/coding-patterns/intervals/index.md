# Intervals

## What it is

An **interval** is a pair `[start, end]` describing a span — a meeting from 9 to 10, a
booking, a range of indices. Interval problems hand you a *list* of these spans and ask
you to reason about how they relate: do they **overlap**? Can you **merge** them? How
many must you **remove** to untangle them? Where are the **gaps** between them?

Almost every one of these problems collapses to the same two-move recipe:

1. **Sort** the intervals — by `start` for merging/combining problems, by `end` for
   greedy "keep as many as possible" problems.
2. **Sweep** through them once, left to right, comparing each interval only to the
   **last one you kept** (the running frontier). You never need to look further back.

That second move is the whole trick. Once the list is sorted, an interval can only
overlap with the run of intervals immediately before it, so a single pointer to "the
last kept interval" is enough state to make a decision in O(1).

## When to reach for it

Strong signals you're in interval territory:

- The input is a list of `[start, end]` pairs (times, ranges, segments).
- The question is about **overlap, merging, gaps, or scheduling** — "do any conflict",
  "combine the overlapping ones", "fit the most meetings", "find the free time".
- The answer depends on the *relative ordering* of the spans, not their original
  positions — which is the hint that you're allowed to **sort first**.

If the intervals are already sorted and non-overlapping (as in *Insert Interval*), you
may not even need to re-sort — you can sweep directly.

## The overlap test

Two intervals `a` and `b` with `a.start <= b.start` (i.e., `a` comes first after
sorting) **overlap** when:

```
b.start <= a.end
```

`b` starts before (or exactly when) `a` ends. Whether you use `<` or `<=` is a
per-problem decision about whether **touching** endpoints count as overlapping. For a
meeting `[1, 2]` and `[2, 3]`: with `<=` they conflict (one ends as the other begins),
with `<` they don't. Read the problem's wording on "end-exclusive" vs "end-inclusive"
carefully — it's the single most common source of off-by-one bugs here.

## Core technique: sort by start, then sweep

The merge-style problems (Merge Intervals, Insert Interval, Employee Free Time) all
share this skeleton: sort by start, then fold each interval into the last one in the
output if they overlap, otherwise start a new block.

```java
import java.util.Arrays;
import java.util.ArrayList;
import java.util.List;

public int[][] sweepBySorting(int[][] intervals) {
    // 1. sort by start so overlaps are always adjacent
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));

    List<int[]> out = new ArrayList<>();
    for (int[] curr : intervals) {
        int[] last = out.isEmpty() ? null : out.get(out.size() - 1);

        if (last != null && curr[0] <= last[1]) {
            // overlap: extend the frontier in place
            last[1] = Math.max(last[1], curr[1]);
        } else {
            // disjoint: curr opens a new block
            out.add(new int[] {curr[0], curr[1]});
        }
    }
    return out.toArray(new int[0][]);
}
```

The only state is `out.get(out.size() - 1)` — "the last interval I committed to." Every
decision is local: extend it, or start fresh.

## Core technique: sort by end, then grab greedily

The scheduling problems (Can Attend Meetings is the trivial case, Non-Overlapping
Intervals the real one) flip the sort key. When the goal is "**keep the maximum number
of non-overlapping intervals**" (equivalently, "**remove the fewest**"), the classic
greedy result is: **always keep the interval that ends earliest.** An interval that
finishes soonest leaves the most room for everything after it.

```java
Arrays.sort(intervals, (a, b) -> Integer.compare(a[1], b[1])); // sort by END

int kept = 0;
int prevEnd = Integer.MIN_VALUE;
for (int[] curr : intervals) {
    if (curr[0] >= prevEnd) {   // no overlap with the last kept interval
        kept++;
        prevEnd = curr[1];      // advance the frontier
    }
    // else: curr conflicts — drop it (it ends no earlier than what we kept)
}
```

Sort-by-start asks "what overlaps?"; sort-by-end asks "what survives?". Knowing which
key to sort on is most of the battle.

## Problems

Work these in order — each builds on the technique before it:

- [Can Attend Meetings](can-attend-meetings.md) *(Easy)* — sort by start, scan adjacent
  pairs for any overlap. The simplest possible sweep; establishes the overlap test.
- [Merge Intervals](merge-intervals.md) *(Medium)* — sort by start, fold overlaps into
  the last output interval. The foundational "build the merged output" technique.
- [Insert Interval](insert-interval.md) *(Medium)* — list is already sorted; splice one
  new interval in with a three-phase scan (before / merge / after). No re-sort needed.
- [Non-Overlapping Intervals](non-overlapping-intervals.md) *(Medium)* — min removals to
  untangle. Greedy: sort by **end**, keep the earliest-finishing interval. The contrast
  to sort-by-start merging.
- [Employee Free Time](employee-free-time.md) *(Hard)* — flatten everyone's busy
  intervals, merge them, then read off the **gaps**. Capstone that reuses Merge
  Intervals and takes the complement.
