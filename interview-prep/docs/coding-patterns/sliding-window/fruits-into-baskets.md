# Fruits Into Baskets

## Problem

You're given an integer array `fruits`, where each element is the *type* of fruit at that position. Walking down a single row, you may start at any position, but you can only carry **two distinct fruit types** in your baskets. The moment you'd be forced to pick up a third type, you stop.

Return the maximum number of fruits you can collect — i.e., the length of the longest contiguous subarray containing **at most two distinct values**.

### Example

```
Input:  fruits = [3, 3, 2, 1, 2, 1, 0]
Output: 4
```

The best run is `[2, 1, 2, 1]` (indices 2–5) — two distinct types (`1` and `2`), length 4. Extending it left adds a `3` (third type); extending it right adds a `0` (third type). Either move breaks the invariant.

## Mental model

This is the prototypical **variable-size sliding window**. Unlike the fixed-size flavor (where `k` is given and the window size never changes), here the window **grows on the right and shrinks on the left** to maintain an invariant:

> The window contains **at most 2 distinct fruit types** at all times.

The shape of every variable-size window problem is the same three-step loop:

1. **Expand** — pull the next element in on the right.
2. **Restore the invariant** — while the window violates the constraint, evict from the left until it holds again.
3. **Record** — once the invariant holds, the current window is a valid candidate; update the answer.

The side-table here is a count map: `fruit type → how many of it are currently in the window`. The number of distinct types is just `basket.size()`, so the invariant check is O(1).

The key insight that makes the whole pass O(n): once the left pointer moves forward, it **never moves back**. Every index is visited at most twice — once by `end`, once by `start`. That's the difference between a sliding window and a brute-force "try every subarray" approach.

## Optimized solution

```java
import java.util.HashMap;
import java.util.Map;

public class Solution {
    public int totalFruit(int[] fruits) {
        Map<Integer, Integer> basket = new HashMap<>();
        int start = 0;
        int best = 0;

        for (int end = 0; end < fruits.length; end++) {
            // 1. expand: add the new right element
            basket.merge(fruits[end], 1, Integer::sum);

            // 2. shrink: while invariant broken (>2 distinct), evict from the left
            while (basket.size() > 2) {
                if (basket.merge(fruits[start], -1, Integer::sum) == 0) {
                    basket.remove(fruits[start]);
                }
                start++;
            }

            // 3. record: window [start..end] now satisfies the invariant
            best = Math.max(best, end - start + 1);
        }
        return best;
    }
}
```

**Notes on the polish vs. the original:**

- `basket.merge(key, delta, Integer::sum)` collapses the `getOrDefault(...) + 1` and `get(...) - 1` patterns into one call that also returns the new count — so the "drop zero-count keys" check is a single line.
- Dropping zero-count keys matters: `basket.size()` is the distinct-count sensor. If you leave `0` values in, the size lies and the loop never terminates.
- Renamed `maxFruit → best` to match the shape of the other sliding-window solutions on this site, but the original name is also fine.

**Complexity**: O(n) time (each index enters and leaves the window at most once), O(1) space (the basket holds at most 3 keys before the inner loop trims it back to 2).

## Walkthrough on the example

`fruits = [3, 3, 2, 1, 2, 1, 0]`.

| end | fruits[end] | basket (after expand) | inner-loop evictions | start | window | length | best |
|---|---|---|---|---|---|---|---|
| 0 | 3 | `{3:1}` | — | 0 | `[3]` | 1 | 1 |
| 1 | 3 | `{3:2}` | — | 0 | `[3,3]` | 2 | 2 |
| 2 | 2 | `{3:2, 2:1}` | — | 0 | `[3,3,2]` | 3 | 3 |
| 3 | 1 | `{3:2, 2:1, 1:1}` | evict `3` ×2 → `{2:1, 1:1}`, start=2 | 2 | `[2,1]` | 2 | 3 |
| 4 | 2 | `{2:2, 1:1}` | — | 2 | `[2,1,2]` | 3 | 3 |
| 5 | 1 | `{2:2, 1:2}` | — | 2 | `[2,1,2,1]` | **4** | **4** |
| 6 | 0 | `{2:2, 1:2, 0:1}` | evict `2`, `1`, `2`, `1` → `{0:1}`, start=6 | 6 | `[0]` | 1 | 4 |

Final answer: **4** ✓.

Watch the `end = 3` row: adding `1` makes the basket size jump to 3, the invariant breaks, and the inner `while` evicts from the left until only `{2, 1}` remain. The left pointer jumps from 0 to 2 in a single outer-loop iteration — that's fine, the *amortized* work per element is still O(1).

## Fixed-size vs. variable-size: how to tell them apart

The two flavors look similar, but the question being asked is different:

| | **Fixed-size** | **Variable-size** |
|---|---|---|
| Question | "Best aggregate over **every** window of size `k`" | "**Longest / shortest** window satisfying a predicate" |
| Window size | Constant `k` | Grows and shrinks |
| Loop shape | Expand on right, evict on left **every step** once size = `k` | Expand on right; **while** invariant broken, evict on left |
| Answer source | `max/min` over all length-`k` windows | `max/min` over `end - start + 1` whenever invariant holds |

If the problem statement names a length (`k`) and asks about *every* such window, it's fixed-size. If it asks for *the longest/shortest* window that has some property, it's variable-size.

## Pitfalls

!!! warning "Forgetting to remove zero-count keys"
    `basket.size() > 2` is the trigger for the inner loop. If you leave keys with count `0` in the map, the size never drops back to 2, the inner loop never exits, and you get an infinite loop. Always evict the key when its count hits zero.

!!! warning "Using `if` instead of `while` for the shrink"
    A single eviction may not be enough to restore the invariant — e.g., if the leftmost element is one of many copies of a type, you have to keep evicting until that type is fully gone. `while (basket.size() > 2)` handles this; `if (basket.size() > 2)` silently leaves the window in a broken state.

!!! warning "Recording the answer inside the shrink loop"
    The window is **invalid** during the shrink — that's the whole point of shrinking. Update `best` *after* the inner loop exits, not inside it. Otherwise you'll record windows that violate the invariant.

!!! warning "Resetting `start` to `end + 1` when you see a third type"
    Tempting shortcut: "third type appeared, restart the window from here." Wrong — the answer often straddles the change. In the walkthrough above, when `1` enters at `end = 3`, the new window starts at index 2 (keeping the `2`), not at index 3. The shrink loop figures this out for free; manual resets get it wrong.

## Takeaway

!!! tip "Variable-size window = expand → restore → record"
    Every variable-size sliding window problem has the same skeleton. Pick the invariant (here: "at most 2 distinct types"), pick the side-table that lets you check it in O(1) (here: a count map), and the loop writes itself. The expand step is always one line; the restore step is a `while` loop that evicts from the left until the invariant holds again; the record step compares `end - start + 1` to the running best.

!!! tip "Generalizes to *at most `k` distinct*"
    This exact template solves the broader "longest subarray with at most `k` distinct elements" problem — replace the `> 2` in the inner-loop guard with `> k`. The Fruits Into Baskets question is just the `k = 2` instance dressed up in a story.
