# 20. Valid Parentheses — Interview Q&A

## 1. What is the complexity, and can you do it in constant memory?

Time is `O(n)`. Extra memory is `O(n)` because a string of `n` openers must all stay unmatched until a closer arrives. You cannot validate mixed bracket types in `O(1)` extra memory in one forward pass: the unmatched sequence is arbitrary and has to be stored. A single bracket type can use an integer counter and `O(1)` memory. Quoting `O(n)` time and `O(n)` space is the right pair for this problem.

## 2. Why push the expected closer instead of pushing the opener?

Both are correct. Pushing the closer makes the pop a single equality check against the current character, with no map lookup on the hot path. Pushing the opener requires a map from closer back to opener at each pop. The structure is the same stack either way. What matters is last-in first-out order, not which of the two characters you store. Pick one and stay with it so you do not compare an opener to a closer by accident.

## 3. What bug does this code have?

```python
def isValid(self, s: str) -> bool:
    stack = []
    for ch in s:
        if ch in "([{":
            stack.append(ch)
        elif stack.pop() != ch:
            return False
    return True
```

Three bugs. `stack.pop()` on an empty stack throws for a leading closer; check empty and return false. Openers are compared directly to closers, so they never match. The final `return True` ignores leftover openers, so `"("` is accepted. Also, `pop` before you know the stack is non-empty is the crash interviewers look for.

## 4. Follow-up: the string may contain letters that you should ignore, or you must return the index of the first error. What changes?

Letters: skip any character that is not a bracket, and keep the same stack. First error index: on a mismatch or an early closer, return the current index instead of false. If the stack is non-empty at the end, the first unmatched opener is at the bottom of the stack, so store indices rather than characters and return the bottom index. If they want every mismatched pair, one pass still works if you define whether you repair or only report.

## 5. What if the constraints change and `n` is 10^7, or there are many bracket types given as a list of pairs?

The same algorithm handles 10^7 characters if you use an explicit stack and not recursion. Build the opener-to-closer map from the given pairs. Time stays linear. Space stays linear in the depth. If the stream is infinite, you cannot wait for the end to check emptiness; define “valid so far” as “no mismatch yet” and report leftover openers only when the stream closes.

## 6. What if the string is empty, or it is `"(("` versus `"()()"`?

Empty: the loop does not run and the stack is empty, so it is valid. `"(("` pushes two closers and ends non-empty, so it is invalid. `"()()"` pushes and pops twice; the stack returns to empty and it is valid. `"(()"` leaves one closer and is invalid. None of these need a special case beyond the empty-stack check at the end and at each closer.
