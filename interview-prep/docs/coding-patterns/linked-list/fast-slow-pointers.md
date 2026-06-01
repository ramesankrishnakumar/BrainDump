# Fast & Slow Pointers (Floyd's Tortoise and Hare)

## What it is

Two pointers walk the *same* list at **different speeds** — classically `slow` moves one node
per step and `fast` moves two. With no way to index into a linked list or measure its length
up front, the speed difference *does the measuring for you* in a single pass.

A close cousin uses **the same speed but a fixed gap** (one pointer starts `n` nodes ahead).
That gap, held constant, locates a position relative to the end without counting.

## The mental model

> Two runners on a track. The sprinter moves twice as fast as the jogger.
>
> - On a **straight** track, when the sprinter reaches the finish line the jogger is exactly
>   **halfway** → find the middle.
> - On a **circular** track, the sprinter eventually **laps and collides** with the jogger →
>   detect a cycle. On a straight track they never meet.

The single idea to carry: **moving at different speeds turns "length" and "loops" into
something you can detect, without ever measuring the list.**

## The four rungs

| # | Problem | The move | Signal that ends it |
|---|---------|----------|---------------------|
| 1 | [Middle of the list](middle-of-linked-list.md) | slow ×1, fast ×2 | `fast` hits the end → `slow` is the middle |
| 2 | [Cycle detection](linked-list-cycle.md) | slow ×1, fast ×2 | they collide → cycle; `fast` hits null → none |
| 3 | [Cycle entry](linked-list-cycle-ii.md) | collide, reset one to head, both ×1 | they meet → that's the entry |
| 4 | [Nth node from the end](remove-nth-node-from-end.md) | same speed, `fast` gets an N-step head start | `fast` hits the end → `slow` is the answer |

A fifth problem, [Find the Duplicate Number](find-the-duplicate-number.md), is rungs 2+3 in
disguise on an array.

## Why fast moves exactly 2

Once both pointers are inside a loop, picture the **gap** between them measured in steps along
the loop. Each iteration `fast` gains exactly **1** on `slow` (moves 2, slow moves 1, net +1),
so the gap shrinks `3 → 2 → 1 → 0`. Because it closes by exactly one each step, `fast` can
never *leap over* `slow` without landing on it — collision is guaranteed. Speeds faster than 2
can jump the gap and miss; speed 2 is the clean choice.

## When to reach for it

- The input is a `ListNode` and the question mentions the **middle**, a **cycle/loop**, or a
  position **relative to the end**.
- The problem demands **one pass** and/or **O(1) extra space** — that constraint is the tell
  that rules out "measure then walk again" or a hash set of visited nodes.

## The deciding question

When you spot a linked-list problem, ask: do I need **different speeds** (to measure a
length/loop) or a **fixed gap** (to position relative to the end)? That single question routes
you to the right variant.

## Problems

- [Middle of the Linked List](middle-of-linked-list.md) *(Easy)* — rung 1.
- [Linked List Cycle](linked-list-cycle.md) *(Easy)* — rung 2.
- [Linked List Cycle II](linked-list-cycle-ii.md) *(Medium)* — rung 3.
- [Remove Nth Node From End](remove-nth-node-from-end.md) *(Medium)* — rung 4.
- [Find the Duplicate Number](find-the-duplicate-number.md) *(Medium)* — rungs 2+3 on an array.
