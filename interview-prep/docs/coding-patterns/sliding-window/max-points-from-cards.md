# Max Points You Can Obtain From Cards

## Problem

Given an array of integers `cards` and an integer `k`, pick exactly `k` cards to maximize their sum. Cards must be picked **in order from either end** — some prefix from the left, the rest as a suffix from the right. You cannot skip cards or pick from the middle.

Constraints: `1 <= k <= cards.length`.

### Examples

**Example 1**

```
Input:  cards = [2, 11, 4, 5, 3, 9, 2], k = 3
Output: 17
```

Valid splits:

| Left taken | Right taken | Sum |
|---|---|---|
| `2, 11, 4` | — | **17** |
| `2, 11` | `2` | 15 |
| `2` | `9, 2` | 13 |
| — | `3, 9, 2` | 14 |

**Example 2**

```
Input:  cards = [1, 100, 10, 0, 4, 5, 6], k = 3
Output: 111
```

Best split: first three cards `1 + 100 + 10 = 111`.

## Mental model

You're not really choosing "which cards" — you're choosing **how many to take from the left** (call it `i`). The right side gets the remaining `k - i`. So there are only `k + 1` possible splits:

| `i` (left) | `k - i` (right) |
|---|---|
| 0 | k |
| 1 | k − 1 |
| … | … |
| k | 0 |

The job is to find the split with the highest sum. Two clean ways to enumerate it without negative indices or wrap-around.

## Approach A — slide the complement (recommended)

### The reframe

Picking `k` cards from the ends is the same as **leaving `n - k` cards untouched in the middle** — and that untouched middle is always **contiguous**. So:

> **Your score = total of all cards − sum of the untouched middle.**
>
> Maximize the score ⇔ **minimize the size-`(n - k)` middle window**.

That's a textbook fixed-size sliding-window problem.

### Code

```java
public int maxScore(int[] cards, int k) {
    int n = cards.length;
    int total = 0;
    for (int c : cards) total += c;

    if (k == n) return total;            // window size 0 — nothing left behind

    int windowSize = n - k;
    int windowSum = 0;
    for (int i = 0; i < windowSize; i++) windowSum += cards[i];

    int minWindow = windowSum;
    for (int i = windowSize; i < n; i++) {
        windowSum += cards[i] - cards[i - windowSize];  // add new right, drop old left
        minWindow = Math.min(minWindow, windowSum);
    }
    return total - minWindow;
}
```

### Walkthrough on `[2, 11, 4, 5, 3, 9, 2]`, `k = 3`

`total = 36`, `windowSize = n - k = 4`.

**Step 0 — first window at indices `0..3`**

```
[ 2, 11,  4,  5 | 3,  9,  2 ]
    window           taken from right
```

```
windowSum = 2 + 11 + 4 + 5 = 22
minWindow = 22                     // score = 36 - 22 = 14
```

**Step 1 — slide right (add `cards[4]=3`, drop `cards[0]=2`)**

```
[ 2 | 11,  4,  5,  3 | 9,  2 ]
```

```
windowSum = 22 + 3 - 2 = 23
minWindow = 22                     // score = 36 - 23 = 13
```

**Step 2 — slide right (add `cards[5]=9`, drop `cards[1]=11`)**

```
[ 2, 11 |  4,  5,  3,  9 | 2 ]
```

```
windowSum = 23 + 9 - 11 = 21
minWindow = 21                     // score = 36 - 21 = 15
```

**Step 3 — slide right (add `cards[6]=2`, drop `cards[2]=4`)**

```
[ 2, 11,  4 |  5,  3,  9,  2 ]
```

```
windowSum = 21 + 2 - 4 = 19
minWindow = 19                     // score = 36 - 19 = 17 ✓
```

`answer = 36 − 19 = 17`.

### The one-line trick that makes it work

```java
windowSum += cards[i] - cards[i - windowSize];
```

Read it as: **"add the new card on the right, remove the old card on the left."** You're editing a running aggregate, not recomputing it.

## Approach B — two pointers from the ends

If the "complement" reframe doesn't click, the direct framing also works cleanly. Iterate over splits in order: start with all `k` from the left, then on each step **give one back from the left** and **take one new from the right**.

### Code

```java
public int maxScore(int[] cards, int k) {
    int n = cards.length;

    int leftSum = 0;
    for (int i = 0; i < k; i++) leftSum += cards[i];   // start: all k from the left

    int best = leftSum;
    int rightSum = 0;
    for (int i = 0; i < k; i++) {
        leftSum  -= cards[k - 1 - i];   // give back one from the left (innermost first)
        rightSum += cards[n - 1 - i];   // take one more from the right (rightmost first)
        best = Math.max(best, leftSum + rightSum);
    }
    return best;
}
```

The index expressions read naturally with no negative numbers:

- `cards[k - 1 - i]` walks the left side **inward**: `k-1, k-2, …, 0`.
- `cards[n - 1 - i]` walks the right side **inward**: `n-1, n-2, …, n-k`.

### Walkthrough on `[2, 11, 4, 5, 3, 9, 2]`, `k = 3`

**Setup — all 3 from the left**

```
leftSum  = 2 + 11 + 4 = 17
rightSum = 0
best     = 17                     // split: 3 left, 0 right
```

**i = 0** — give back `cards[2]=4`, take `cards[6]=2`

```
leftSum  = 17 - 4 = 13
rightSum =  0 + 2 =  2
best     = max(17, 15) = 17       // split: 2 left, 1 right
```

**i = 1** — give back `cards[1]=11`, take `cards[5]=9`

```
leftSum  = 13 - 11 =  2
rightSum =  2 +  9 = 11
best     = max(17, 13) = 17       // split: 1 left, 2 right
```

**i = 2** — give back `cards[0]=2`, take `cards[4]=3`

```
leftSum  =  2 - 2 =  0
rightSum = 11 + 3 = 14
best     = max(17, 14) = 17       // split: 0 left, 3 right
```

Final `best = 17` ✓.

## A vs B — comparison

| | Approach A (slide the complement) | Approach B (two pointers from ends) |
|---|---|---|
| Conceptual frame | "What am I leaving behind?" | "Some from left, some from right." |
| What's tracked | min sum of size-`(n − k)` window | running `leftSum` + `rightSum` |
| Final answer | `total − minWindow` | `max(leftSum + rightSum)` |
| Edge case | `k == n` → window size 0, return total | None — loop simply runs `k` times |
| When more natural | When you can see the complement trick | When the problem already speaks in "ends" |

Both are **O(n) time, O(1) space**. Same answer, different angle.

## Pitfalls

!!! warning "Negative indices and wrap-around"
    Don't simulate "the i-th card from the right" with negative indices and modular arithmetic (`n + start`). The array isn't circular. `cards[n - 1 - i]` reads more directly and never goes out of bounds.

!!! warning "One pointer doing two jobs"
    Avoid a single counter that you sometimes interpret as a left index and sometimes as a right index. Two clearly-named running sums (`leftSum`, `rightSum`) — or one `windowSum` plus an index — are far easier to reason about and review.

!!! warning "Validity checks inside the slide loop"
    If you find yourself writing something like `if (start - end + 1 == k)` inside the loop to decide whether the window is valid, that's a symptom of mixing "grow the window" with "slide the window." Build the first valid window in a separate setup loop, then run a slide loop that's always valid by construction.

!!! warning "Boxed `Integer` returns"
    Use `int` for return type and parameters here. Boxing buys nothing and just adds null-safety questions where there aren't any.

## Takeaway

!!! tip "Reach for the complement"
    When a problem says **"pick from either end,"** ask yourself: **"Is the thing I'm leaving behind simpler than the thing I'm picking?"** If the untouched region is contiguous and fixed-size, slide *that* instead. This reframe shows up in many array problems and is one of the cleanest moves you can make in an interview.
