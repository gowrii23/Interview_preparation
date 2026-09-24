# 3. Longest Substring Without Repeating Characters — Interview Q&A

## 1. What is the complexity of the last-index jump versus a set with a shrink loop?

Both are `O(n)` time. The jump version moves `left` directly and does `O(1)` work per index with an array, so the constant is smaller. The set version’s `while` loop still totals `O(n)` removals. Space is `O(alphabet)` for the array or the set. If you rebuild a set from a slice at every index, time becomes `O(n^2)` because each slice copies up to `n` characters.

## 2. Why store the last index instead of only a count inside the window?

A count tells you that a repeat exists, and then you still have to walk `left` forward until that count drops. The last index tells you exactly where the current window must start, so one assignment replaces the walk. You still need the `>= left` test: the stored index might belong to a copy you already evicted. A boolean “seen anywhere in the string” is wrong because a character may appear again after its first copy has left the window.

## 3. What bug does this code have?

```python
if ch in last:
    left = last[ch] + 1
last[ch] = right
```

It moves `left` even when `last[ch]` is already behind `left`. On `"abba"`, when the final `a` is read, `last['a']` is 0, and `left` jumps backward from 2 to 1. The window grows backward and can count a repeated `b`. The fix is `if last[ch] >= left`. Another bug is setting `left = last[ch]` without the `+ 1`, which leaves the previous copy inside the window.

## 4. Follow-up: return the substring, or allow at most `k` repeated characters. What changes?

For the substring, remember `bestLeft` whenever the length improves, and slice once at the end (`s[bestLeft:bestLeft + best]`). Slicing inside the loop copies on every improvement. For at most `k` repeats, a single last index is not enough; keep counts and shrink while some count exceeds the allowed repeat. If “at most one repeat in the whole substring” means something else, restate the constraint before coding.

## 5. What if the constraints change and the string is 10^6 Unicode code points, and you need every longest substring?

Use a hash map, still expected linear time. At `10^6`, avoid per-step string allocation. Collect start indices of windows whose length equals the best; you may need a second pass or store candidates and drop shorter ones after you know the maximum. Space can grow to the number of such windows. An ASCII array of 128 is incorrect for Unicode.

## 6. What if all characters are unique, or all characters are the same?

All unique: `left` stays 0, and the answer is `n`. The map never pulls `left` forward. All the same: every new character finds a previous index at `right - 1`, so `left` becomes `right`, and the length stays 1. A string of length 0 returns 0 because the loop does not run.
