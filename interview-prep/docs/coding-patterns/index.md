# Coding Pattern Study Guides

!!! tip "In a hurry? Start with the [Coding Interview Cheat Sheet](cheatsheet.md)"
    A single, Ctrl-F-able page covering ~18 techniques and the most-asked questions — each with
    *how to recognize it*, *what to say to the interviewer*, and a *template skeleton* (not the
    full solution). Use it to match a problem to a pattern fast, then dive into the deep-dive
    pages below for full walkthroughs.

Pattern-first interview prep. Each pattern has its own section with a short overview ("what is it, when do you reach for it") and one or more representative problems. Every problem write-up follows the same arc:

1. **Problem** — clean statement plus examples.
2. **Mental model** — the single insight that makes the problem easy.
3. **Approaches** — annotated code, step-by-step walkthroughs on a concrete input.
4. **Pitfalls** — the traps you hit the first time you tried it.
5. **Takeaway** — the one-line rule to carry into the next problem.

## Patterns

- [Sliding Window](sliding-window/index.md) — fixed/variable contiguous windows over arrays and strings.
- [Intervals](intervals/index.md) — sort by start (to merge) or by end (to schedule), then sweep once comparing each interval to the last one kept.
- [Stack](stack/index.md) — LIFO matching and nested structure.
- [Linked List](linked-list/index.md) — pointer rewiring on singly linked nodes.
- [Binary Search](binary-search/index.md) — halve a sorted search space each step.

## Quick reference

- [Coding Interview Cheat Sheet](cheatsheet.md) — all ~18 techniques on one page: recognition index, talk track, and template per problem. The primary in-interview reference.
- [Java Quick Hacks](java-quick-hacks.md) — Java-API idioms (not algorithms) mapped to LeetCode families: both-ends indexing, `Map.merge`/`compute`, `Map.of`, `PriorityQueue`, `Deque`, `Arrays.sort`, grids/intervals, `StringBuilder`, `record` BFS state, and more. Skim before an interview.
- [Solve by Negation](solve-by-negation.md) — a thinking habit, not an algorithm: when a condition fans out into many positive cases (overlap, collision, "at least one"), detect its small *opposite* and flip it. Fewer branches, fewer off-by-one bugs.
