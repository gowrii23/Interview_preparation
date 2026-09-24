# 76. Minimum Window Substring — Interview Q&A

## 1. What is the complexity, and why is the shrink loop not quadratic?

`O(n + m)` time. `right` and `left` each move at most `n` times, and each move updates a count in `O(1)`. Building the need table is `O(m)`. Space is `O(alphabet)` or `O(distinct characters)`. It becomes quadratic if you slice `s[left:right+1]` and count characters from scratch on every `right`, because each slice copies the window.

## 2. Why a `formed` counter instead of comparing the whole need array every time?

You only care whether each required character has reached its needed count, not whether extra unrelated characters match. `formed` increments when a character hits its target and decrements when it falls below. The test `formed == required` is `O(1)`. Comparing 128 slots every step is still linear overall, so it is acceptable; the counter is the cleaner invariant. A set of “still missing” characters works the same way.

## 3. What bug does this code have?

```python
if have[ch] == need[ch]:
    formed += 1
# ...
have[drop] -= 1
if have[drop] == need[drop]:
    formed -= 1
```

The decrement check is wrong. After you subtract, equality means you are still exactly at the requirement, so coverage did not break. You should decrement `formed` when `have[drop]` becomes strictly less than `need[drop]`. Another bug is incrementing `formed` for a character that is not in `t` whenever its have-count equals a default need of 0, or incrementing again when `have` goes from 2 to 3 while need is 1. Only the transition into “exactly need” counts, and only for characters `t` actually asks for.

## 4. Follow-up: return the minimum window that covers `t` as a subsequence, not as a multiset. What changes?

Contiguous multiset cover is this problem. A subsequence cover can skip characters and is a different algorithm (often dynamic programming or a greedy walk with next-position links). If they still want a contiguous window but the characters of `t` must appear in order, shrink and expand will not do; you need the smallest window that contains `t` as a subsequence, which you can find with a two-pointer scan that advances a match index into `t`. Restate which one they want.

## 5. What if the constraints change and `s` has length 10^6, `t` has length 10^5, and both are Unicode?

Hash maps replace fixed arrays. Expected time stays linear if hashing a character is `O(1)`. Do not allocate the best window until the end; store indices. If `t` is longer than `s`, return empty immediately. Memory for maps is proportional to distinct code points seen, which is fine at this size, but copying a million-character slice on every improvement is not. If they ask for every minimum window, collect starts while the length equals the best, after you know that best length.

## 6. What if `t` is empty, or `t` has repeated letters and `s` has those letters far apart?

Confirm the empty-`t` contract. Many statements say the answer is empty; others say it is the empty string at index 0. Ask. For two `'A'`s in `t`, a window with one `'A'` must not count as covered: `have['A'] == need['A']` only when the second copy arrives. Letters of `s` that are not in `t` stay inside the window until the shrink drops them; they increase the length, so the shrink loop removes them when they sit on the left edge.
