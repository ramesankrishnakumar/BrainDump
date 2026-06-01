# Max Sum of Distinct Subarrays of Length k

## Problem

Given an integer array `nums` and an integer `k`, find the **maximum sum of any length-`k` contiguous subarray whose elements are all distinct**. If no such subarray exists, return `0`.

### Examples

**Example 1**

```
Input:  nums = [3, 2, 2, 3, 4, 6, 7, 7, -1], k = 4
Output: 20
```

Walking through every length-4 window:

| Window | All distinct? | Sum |
|---|---|---|
| `[3, 2, 2, 3]` | no (3, 2 repeat) | — |
| `[2, 2, 3, 4]` | no (2 repeats) | — |
| `[2, 3, 4, 6]` | yes | 15 |
| `[3, 4, 6, 7]` | yes | **20** |
| `[4, 6, 7, 7]` | no (7 repeats) | — |
| `[6, 7, 7, -1]` | no (7 repeats) | — |

**Example 2**

```
Input:  nums = [5, 5, 5, 5, 5], k = 3
Output: 0
```

Every length-3 window has duplicates, so return `0`.

## Mental model

A textbook **fixed-size sliding window** with a side table. The window has size `k`, slides one step at a time, and you keep two running pieces of state:

1. **`windowSum`** — the sum of elements currently inside the window. Updated in O(1) per slide: add the new element on the right, subtract the old element on the left.
2. **A count map** — `value → how many times it appears in the current window`. Lets you answer "are all `k` elements distinct?" in O(1).

The key equivalence:

> A length-`k` window has all distinct elements ⇔ the count map has exactly `k` keys.

So the validity check collapses to `counts.size() == k`. No need for a separate "found anything yet?" flag — initialize `best = 0` and the spec is already satisfied (return 0 when nothing qualifies).

## Optimized solution

```java
import java.util.HashMap;
import java.util.Map;

public class Solution {
    public long maxSum(int[] nums, int k) {
        Map<Integer, Integer> counts = new HashMap<>();
        long windowSum = 0;
        long best = 0;                            // spec: return 0 if no valid window — best already satisfies it
        int start = 0;

        for (int end = 0; end < nums.length; end++) {
            // 1. expand: pull the new right element in
            windowSum += nums[end];
            counts.merge(nums[end], 1, Integer::sum);

            // 2. shrink: keep the window at size k (fixed-size: at most one eviction per step)
            if (end - start + 1 > k) {
                int leaving = nums[start];
                windowSum -= leaving;
                if (counts.merge(leaving, -1, Integer::sum) == 0) {
                    counts.remove(leaving);       // drop zero-count keys so size() == distinct count
                }
                start++;
            }

            // 3. check: once the window is exactly size k, see if it's all-distinct
            if (end - start + 1 == k && counts.size() == k) {
                best = Math.max(best, windowSum);
            }
        }
        return best;
    }
}
```

**Why this is faster to read and write than the original:**

- `counts.merge(key, delta, Integer::sum)` replaces the `getOrDefault(...) + 1` / `get(...) - 1` dance and returns the new value, so the "remove on zero" check is a single line.
- No `found` flag — `best = 0` is correct from the start because the spec demands `0` when nothing qualifies.
- No `Long.MIN_VALUE` — `best` is `long` and seeded with the answer for the "no valid window" case.
- Two pointers, `start` and `end`, matching the [variable-size template](index.md#when-to-use-variable-size). The only structural difference is `if (end - start + 1 > k)` (single eviction, fixed-size) vs `while (invariant broken)` (multi-eviction, variable-size). Keeping the scaffolding identical across both flavors makes the contrast easy to see.
- Three clearly-named phases per iteration (expand, shrink, check) — easy to point at in an interview.

**Complexity**: O(n) time, O(k) space (the count map holds at most `k` keys).

## Walkthrough on Example 1

`nums = [3, 2, 2, 3, 4, 6, 7, 7, -1]`, `k = 4`.

I'll show `end`, `start`, the window contents, `counts.size()`, `windowSum`, and `best` after each iteration. **Bold** rows mark iterations where the window first becomes size `k`.

| end | nums[end] | start | window | counts.size() | windowSum | best |
|---|---|---|---|---|---|---|
| 0 | 3 | 0 | `[3]` | 1 | 3 | 0 |
| 1 | 2 | 0 | `[3, 2]` | 2 | 5 | 0 |
| 2 | 2 | 0 | `[3, 2, 2]` | 2 | 7 | 0 |
| **3** | **3** | **0** | `[3, 2, 2, 3]` | **2** | **10** | 0 (counts.size() != 4 → skip) |
| **4** | **4** | **1** | `[2, 2, 3, 4]` | **3** | **11** | 0 (counts.size() != 4 → skip) |
| **5** | **6** | **2** | `[2, 3, 4, 6]` | **4** | **15** | **15** ✓ |
| **6** | **7** | **3** | `[3, 4, 6, 7]` | **4** | **20** | **20** ✓ |
| **7** | **7** | **4** | `[4, 6, 7, 7]` | **3** | **24** | 20 (counts.size() != 4 → skip) |
| **8** | **−1** | **5** | `[6, 7, 7, -1]` | **3** | **19** | 20 (counts.size() != 4 → skip) |

Final answer: **20** ✓.

Notice how `counts.size()` acts as a "distinctness sensor" — it equals `k` exactly when the window is valid, and drops below `k` the instant a duplicate enters.

## What was specifically off in the original

A close read of your code:

1. **`Long.MIN_VALUE` + `found` flag** — both unnecessary. The spec says "return 0 if no valid window exists," so initialize `best = 0` and you can return it directly. Two pieces of state collapse to one.
2. **`Integer` / `Long` boxing** — `Integer k`, `Long` return: no reason to box. Use `int` and `long`. Avoids null-safety questions and removes auto-unboxing overhead.
3. **`memory.get(nums[start]) == null || memory.get(nums[start]) == 1`** — two map lookups for one decision, and the `== null` branch can't actually fire here (the key is always present when you're evicting it). `merge(key, -1, Integer::sum)` returns the new count in one call.
4. **Naming** — `memory` is generic; `counts` (or `freq`) names what it actually holds. Small thing, but interviewers read variable names as a signal of how clearly you think.

None of these is wrong — your solution is correct and O(n). They're the polish that separates "works" from "clearly reads as production code" in an interview.

## Pitfalls

!!! warning "Forgetting to remove zero-count keys"
    `counts.size() == k` only works as a distinctness check if you **remove** keys whose count drops to zero. If you leave them in the map with value `0`, the size will lie and you'll over-count distinct elements.

!!! warning "`Long.MIN_VALUE` sentinels"
    When the spec already names the "nothing valid" return value (here, `0`), use it as your initializer instead of `MIN_VALUE` + a `found` flag. One fewer piece of state to keep in sync.

!!! warning "Putting the validity check in the wrong place"
    Check `counts.size() == k` **after** evicting the leftmost element of the next window? You'll be checking the wrong window. The order each iteration is: **expand → shrink → check** (or expand → check → shrink — but pick one and stick with it). The version above shrinks first, then checks, so the check always sees a window of size exactly `k`.

!!! warning "Off-by-one on the first check"
    Don't check `counts.size() == k` until the window has actually grown to size `k`. Guard with `end - start + 1 == k` — true the first time the window reaches size `k` (at `end = k - 1`) and stays true forever after (since the shrink step caps the size at `k`).

## Takeaway

!!! tip "Map-as-sensor for sliding windows"
    When a fixed-size window has a *property* you need to check at each step (all distinct, exactly `k` distinct, at most `k` distinct, all even, sum ≥ target, …), maintain a small piece of state that **mirrors the window** — usually a count map or a running sum/counter. Update it incrementally on every slide, and reduce the property check to a single O(1) read of that state. The window slides; the sensor tells you when it's interesting.
