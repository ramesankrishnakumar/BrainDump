# Valid Parentheses

## Problem

Given a string `s` containing only the characters `'('`, `')'`, `'{'`, `'}'`, `'['`
and `']'`, determine whether the input string is valid.

A string is valid when:

1. Every closing bracket has a matching opening bracket of the same type.
2. Brackets close in the correct order — an open bracket must be closed by the same
   type of bracket, and open brackets must be closed in the right order.

### Examples

```
Input:  s = "()"
Output: true
```

```
Input:  s = "([)]"
Output: false
```

`[` opens, then `(` opens inside it — but `)` closes before `]`, so the nesting is wrong.

```
Input:  s = "{[]}"
Output: true
```

`{` wraps `[`, which wraps `]` then `}` — proper nesting throughout.

## Mental model

Scan left to right. Think of the stack as "brackets still waiting for their closer,"
with the most recent open on top.

- **Opening bracket** → push it. We'll need its partner later.
- **Closing bracket** → the top of the stack must be exactly that partner. Pop and
  verify; if the stack is empty or the partner doesn't match, return `false` immediately.
- **End of string** → valid only if nothing is left on the stack (every open closed).

The LIFO property is the whole trick: the last open you saw is the first one that must
close. That's why `([)]` fails — when `)` arrives, the top of the stack is `[`, not `(`.

## Optimized solution

```java
import java.util.ArrayDeque;
import java.util.Deque;
import java.util.Map;

public class Solution {
    public boolean isValid(String s) {
        if (s.length() % 2 == 1) {
            return false; // odd length → can't pair everything
        }

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
}
```

**Complexity**: O(n) time — each character is pushed or popped at most once. O(n) space
in the worst case (all opens, e.g. `"((((("`), for the stack.

## Walkthrough on the example

`s = "({[]})"`.

| char | action | stack after |
|---|---|---|
| `(` | push | `(` |
| `{` | push | `({` |
| `[` | push | `({[` |
| `]` | pop `[`, matches | `({` |
| `}` | pop `{`, matches | `(` |
| `)` | pop `(`, matches | *(empty)* |

End of scan: stack empty → **true** ✓.

Contrast `s = "([)]"` at the `)` step: stack top is `[`, but `)` needs `(` → mismatch
→ **false**.

## Pitfalls

!!! warning "Using `java.util.Stack`"
    The legacy `Stack` class extends `Vector` and is synchronized — unnecessary overhead.
    Prefer `ArrayDeque<Character>` with `push` / `pop`.

!!! warning "Popping from an empty stack"
    A closer with nothing on the stack (`")"`, `"]"`, `"}"` as the first char) is
    invalid. Always check `stack.isEmpty()` before `pop`.

!!! warning "Wrong partner mapping"
    Map **close → open**, not open → close. On a closer you compare `stack.pop()` against
    the expected open, not the other way around.

!!! warning "Forgetting the final empty check"
    `"((("` passes every pop test (no closers) but is invalid — opens remain. Return
    `stack.isEmpty()` after the loop.

!!! warning "Skipping the odd-length shortcut"
    Not required, but `s.length() % 2 == 1` catches impossible inputs in O(1) before
    any stack work.

## Takeaway

!!! tip "Open → push; close → pop must match"
    Bracket validity is pure LIFO. One pass, one stack, one map of closers to their
    opens. Every stack-matching problem in interviews is a variation on this skeleton —
    different symbols, different push rules, sometimes store indices instead of chars,
    but the shape stays the same.
