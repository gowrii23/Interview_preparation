# Stack — Interview Q&A

## 1. What are the time and space bounds of a single left-to-right stack scan?

Time is `O(n)` because each element is pushed at most once and popped at most once, and each of those steps is `O(1)`. Space is `O(n)` in the worst case, when many openers are still unmatched at the middle of the input. A well-nested string of one pair uses `O(1)` stack space at any moment only if you interleave open and close; the bound you quote is the worst case. An array used as a stack has the same bounds. A recursive call per opener is also a stack, with the same `O(n)` depth risk and a higher chance of blowing the language’s call limit.

## 2. Why a stack instead of a counter or a queue?

A counter tracks a single bracket type: it knows how many are open, not which kind. Mixed types need the identity of the most recent unmatched opener, and they need it in last-in first-out order. A queue would close the oldest opener first, which is the opposite of nesting. The stack’s top is exactly “the innermost thing still waiting.”

## 3. What bug does this code have?

```python
stack = []
for ch in s:
    if ch in "([{":
        stack.append(ch)
    elif stack and stack[-1] == ch:
        stack.pop()
return not stack
```

Closers are compared to openers, so `')'` never equals `'('`, and every closer fails the match. Either push the expected closer, or compare through a map `{')': '(', ...}` and pop when `stack[-1]` is the opener for this closer. A second bug is `elif stack and ...` falling through when the closer does not match: the character is ignored and the function can return true for `"([)]"` if you forget the `return False` branch. A third is forgetting the final empty check, so `"((("` returns true.

## 4. Follow-up: return the length of the longest valid parentheses substring. What changes?

A boolean scan is not enough. Push indices. Push `-1` as a base, or push the index of each unmatched opener. On a closer, pop, and if the stack becomes empty the closer is unmatched (push its index); otherwise the valid length ends at the current index and starts just after the new top. Track the maximum `currentIndex - stackTop`. Time stays `O(n)`. This is the longest-valid-parentheses problem, and the stack stores indices rather than characters.

## 5. What if the constraints change and `n` is 10^7, or the brackets arrive as a stream you cannot rewind?

`O(n)` time and `O(n)` worst-case memory still hold, and 10^7 stack frames of characters fit easily. For a stream, you still need the stack of unmatched openers; you cannot answer “is the whole stream valid?” in `O(1)` memory when types are mixed, because an arbitrary prefix of openers must be remembered. If there is only one bracket type, a counter and a “went negative” flag are `O(1)` memory. If they ask only whether some prefix is valid, you can answer online as you go.

## 6. How do you recognize the next stack problem after valid parentheses?

Look for “nearest greater element,” “span until a warmer temperature,” or “previous smaller bar” . You keep indices in monotonic order and pop the ones the current value resolves. State the monotonic direction before coding: increasing stack or decreasing stack. The valid-parentheses stack is not monotonic in value; it is unmatched-opener order. Say which of the two you are using so the interviewer knows you are not mixing them.
