# 424. Longest Repeating Character Replacement — Interview Q&A

## 1. What is the complexity, and why does a stale `maxFreq` not make it wrong or slower?

Time is `O(n)` and extra space is `O(1)` for a 26-letter alphabet. `maxFreq` only increases. That can make `width - maxFreq` look smaller than the true number of replacements, so the shrink loop may stop early and leave a window that is not actually valid for the current counts. It still does not report a too-large answer, because any window longer than the best valid one would need a new record `maxFreq` to satisfy `width - maxFreq <= k`. Smaller windows are not interesting. The shrink loop still runs `O(n)` times total. If you want the invariant “the window is always truly valid,” recompute the max over 26 counts after each removal; the complexity stays linear.

## 2. Why is the replacement count `width - maxFreq`?

To make the window one character, you keep the copies of the most common character and change everything else. There is no benefit to aiming at a less common character: that would require at least as many changes. You do not need to know which letter wins until you look at the counts. The window structure is what localizes those counts; a global frequency would include letters outside the substring you are allowed to use.

## 3. What bug does this code have?

```python
while (right - left + 1 - max_freq) > k:
    left += 1
best = max(best, right - left + 1)
```

The left character’s count is never decremented, so `maxFreq` and the counts drift from the real window. A second bug is shrinking with `if` instead of `while`, which leaves a window that still needs more than `k` changes and then records its length. A third is using `width - k` as the answer directly without checking the actual max frequency, which assumes the window already has a character that appears `width - k` times.

## 4. Follow-up: you may replace at most `k` characters, but the final character must be `'A'`. What changes?

You no longer track a generic `maxFreq`. The number of changes is `width - count['A']`. Shrink while that exceeds `k`. This is simpler, and the stale-max trick is unnecessary because the target is fixed. If the allowed final characters are a small set, run the same window once per target, or keep counts and take the max among the allowed targets only.

## 5. What if the constraints change and the alphabet has size 10^5, `n` is 10^5, and `k` can be `n`?

A length-26 array does not apply. Use a hash map of counts. Recomputing the true max by scanning the map on every step can be `O(n)` per step if many distinct characters sit in the window. Keep the non-decreasing `maxFreq` argument, which stays `O(1)` per step, or a balanced tree of frequencies if you must maintain the exact max. If `k >= n`, the answer is `n`, and you can return immediately. Space becomes `O(alphabet in the string)`.

## 6. What if `k` is 0, or the string is one character?

`k = 0` forbids changes, so the window may contain only one distinct character, and the answer is the longest run. The same loop handles it: `width - maxFreq > 0` forces a shrink. One character returns 1, with zero replacements. An empty string returns 0. Do not special-case them unless you want a clearer early return.
