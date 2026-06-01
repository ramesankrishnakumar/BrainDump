# Solve by Negation, Not Enumeration

## What it is

A problem-solving habit, not an algorithm. When a condition has **many positive cases**
but **few negative ones**, don't try to detect the thing directly — detect its
**opposite** (which is small) and flip it.

The trap it saves you from: facing a question like "do these two things overlap?" and
trying to enumerate *every* shape the "yes" can take — fully inside, straddling the left
edge, straddling the right, one containing the other, exact-touch... five fiddly `if`s,
each a chance to get a boundary wrong. The "no" is almost always simpler.

!!! tip "The core move"
    Ask **"when is it clearly NOT true?"** That answer is usually one or two clean
    comparisons. Everything else is the "yes". `overlap = !(separated)`.

## When to reach for it

Strong signals:

- The positive case **fans out into many sub-cases**, but the negative case is one or two
  conditions (overlap, collision, containment).
- You're **counting arrangements** and "satisfies X" is hard, but "total" and "violates
  X" are both easy → answer = `total − bad`.
- A prior step (sorting, an earlier loop, a filter) has already **eliminated a whole
  category**, so only one way-to-fail remains.

## The canonical example: interval overlap

**Goal:** does interval `[a, b]` overlap `[c, d]`?

**Enumeration (what beginners reach for):** list every overlapping configuration. Messy
and bug-prone.

**Negation:** two intervals fail to overlap *only* when one is entirely left **or**
entirely right of the other — two comparisons:

```
not overlapping  ⟺  b < c   (a..b entirely left)
                 OR  d < a   (a..b entirely right)
```

Negate it (De Morgan) and you get the whole overlap test in one line:

```java
boolean overlap = a <= d && c <= b;   // start of each ≤ end of the other
```

That `a <= d && c <= b` is the **general two-interval overlap test** — keep it in your
back pocket.

### How an earlier step shrinks the check further

In [Insert Interval](intervals/insert-interval.md) the merge loop gets away with checking
*half* of that test — just `intervals[i].start <= newInterval.end`. Why? Because the
**first loop already removed every interval entirely to the left**. Once "entirely left"
is impossible, the only remaining way to *not* overlap is "entirely right", so its
negation alone (`start <= end`) means overlap.

!!! warning "Negate the right boundary, not the one you started from"
    The "before" loop tests `interval.end < newStart`. It's tempting to think overlap is
    just its negation, `interval.end >= newStart`. **It isn't** — that's still true for
    intervals far off to the right (e.g. `new = [5,6]`, candidate `[8,10]`:
    `10 >= 5` is true but they don't overlap). The negation only kills the *left* case;
    you must negate the *right* case (`start > newEnd`) to get the overlap condition.
    Lesson: be precise about **which** non-overlap case the earlier step removed.

## Other places the same habit pays off

| Problem shape | Don't enumerate... | Negate to... |
|---|---|---|
| Two intervals/segments overlap | every overlap layout | `!(one fully left or right)` → `a <= d && c <= b` |
| Count subarrays/subsets satisfying X | the valid ones | `total − (those that violate X)` (complement counting) |
| Is a string a palindrome / valid | every "valid" structure | find the **first** spot it breaks; no break ⇒ valid |
| Rectangles intersect | overlap geometry | `!(separated on x-axis or y-axis)` |
| "At least one" of N conditions holds | each combination | `!(none hold)` |

**Complement counting in one line:** "subarrays with *at most* K distinct" is easy with a
sliding window; "subarrays with *exactly* K" is hard — so compute
`atMost(K) − atMost(K−1)`. Same negation instinct: solve the easy bounding version, then
subtract.

## Takeaway

!!! tip "When the 'yes' explodes, describe the 'no'"
    Before writing five `if` branches to detect a condition, pause and ask what makes it
    *fail* — that's usually one or two comparisons. Then negate. And check whether an
    **earlier step (sort, filter, prior loop) already eliminated a case**, so your final
    check can be even smaller. Fewer branches means fewer off-by-one bugs and a cleaner
    explanation in the interview.
