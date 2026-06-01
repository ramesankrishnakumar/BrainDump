# Stack

## What it is

A **stack** is a last-in, first-out (LIFO) structure: the most recent item you put on
top is the first one you take off. In interview problems that usually means "whatever
opened last must close first" — nested parentheses, undo operations, parsing nested
structure.

In Java, use `ArrayDeque` as a stack (`push` / `pop` on one end). Avoid the legacy
`Stack` class — it's synchronized and slower for no benefit here.

## When to reach for it

Strong signals you're in stack territory:

- **Matching or nesting** — brackets, tags, open/close pairs where order matters.
- **"Most recent unmatched …"** — the answer depends on what happened *last*, not
  first.
- **Parse or evaluate** — RPN calculators, nested expressions, directory paths.
- **Monotonic stack** (a variant) — "next greater element," daily temperatures; keep
  indices or values in decreasing/increasing order and pop when the invariant breaks.
  Valid Parentheses below is the foundational LIFO-matching case; monotonic stacks build
  on the same `Deque` primitive.

If you only need a running count or a sliding aggregate over a contiguous range, a stack
is usually the wrong tool — reach for a sliding window or a hash map instead.

## Core technique: push opens, pop closes

Scan left to right. Opening symbols go on the stack. Closing symbols must pop the top
and match the partner of what you just removed. If the stack is empty when you need to
pop, or the popped symbol doesn't pair with the closer, the string is invalid. After the
scan, the stack must be empty — any leftover opens mean something never closed.

```java
import java.util.ArrayDeque;
import java.util.Deque;
import java.util.Map;

public boolean stackMatching(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    Map<Character, Character> closeToOpen = Map.of(
        ')', '(',
        ']', '[',
        '}', '{'
    );

    for (char c : s.toCharArray()) {
        if (closeToOpen.containsKey(c)) {
            if (stack.isEmpty() || stack.pop() != closeToOpen.get(c)) {
                return false;
            }
        } else {
            stack.push(c);
        }
    }
    return stack.isEmpty();
}
```

The only state is "what's still open, in order from bottom to top." Every decision is
local: push, or pop-and-verify.

## Problems

- [Valid Parentheses](valid-parentheses.md) *(Easy)* — the prototypical bracket-matching
  stack. Establishes push-on-open, pop-on-close, empty-stack check.
