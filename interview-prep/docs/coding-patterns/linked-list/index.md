# Linked List

## What it is

A **singly linked list** is a chain of nodes. Each node holds a value (`val`) and a
pointer to the next node (`next`). The list starts at `head`; the last node's `next` is
`null`.

Unlike an array, you cannot jump to index `i` in O(1). To reach the i-th node you walk
from `head`, which makes linked-list problems fundamentally about **pointer
manipulation** — rewiring `next` links while keeping track of what you've already
visited and what comes after.

## When to reach for it

Strong signals you're in linked-list territory:

- The input is a `ListNode` (or a custom node type), not an array.
- The problem asks to **reverse, merge, reorder, or remove** nodes in place.
- You need **cycle detection** or "find the k-th from the end" (often two pointers on
  the same list).
- Memory constraints favor O(1) extra space — no copying into a new array.

If the problem gives you an array and never mentions nodes, you probably don't need a
linked-list technique (though the *ideas* — fast/slow pointers, reversal — sometimes
transfer).

## Core technique: three-pointer reversal

The workhorse for "reverse a singly linked list" is an iterative loop with three
pointers:

- `prev` — the reversed prefix (starts `null`)
- `curr` — the node you're currently flipping
- `next` — saved successor so you don't lose the rest of the list

Each step: save `next`, point `curr.next` to `prev`, then march all three forward. When
`curr` is `null`, `prev` is the new head.

```java
public ListNode reverseList(ListNode head) {
    ListNode prev = null;
    ListNode curr = head;

    while (curr != null) {
        ListNode next = curr.next; // save rest of list
        curr.next = prev;          // reverse link
        prev = curr;               // advance reversed prefix
        curr = next;               // advance to next original node
    }

    return prev; // prev is the new head when curr runs off the end
}
```

**Complexity**: O(n) time, O(1) extra space. Every node is visited once; only three
pointer variables beyond the list itself.

## Second core technique: fast & slow pointers

The other workhorse moves two pointers at **different speeds** (slow ×1, fast ×2) to find the
middle or detect a cycle, or holds a **fixed gap** between same-speed pointers to locate a node
relative to the end — all in one pass with O(1) space. See
[Fast & Slow Pointers](fast-slow-pointers.md) for the full pattern and its four "rungs".

## Problems

- [Reverse Linked List](reverse-linked-list.md) *(Easy)* — the foundational pointer
  rewiring problem. Master this loop before merge, reorder, or k-group reverse.
- [Fast & Slow Pointers](fast-slow-pointers.md) — the pattern overview (two speeds vs. fixed gap).
- [Middle of the Linked List](middle-of-linked-list.md) *(Easy)* — slow ×1, fast ×2.
- [Linked List Cycle](linked-list-cycle.md) *(Easy)* — collision ⇒ cycle (Floyd's).
- [Linked List Cycle II](linked-list-cycle-ii.md) *(Medium)* — reset-to-head to find the cycle entry.
- [Remove Nth Node From End](remove-nth-node-from-end.md) *(Medium)* — fixed-gap head start + dummy node.
- [Find the Duplicate Number](find-the-duplicate-number.md) *(Medium)* — an array read as a linked list with a cycle.
