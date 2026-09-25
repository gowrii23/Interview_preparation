# 53. Palindromic Substrings — Interview Q&A

## 1. Why this state?

A palindromic substring is uniquely recovered from its center and its radius. Counting radii that still match counts substrings, with no double-counting, because two different slices have different centers or different radii. The interval boolean `dp[L][R]` is the stored form of the same fact. Expansion keeps only the center you are walking and a counter, which is enough when the only query is “how many are true?”

## 2. How do you optimize space?

The DP triangle needs `O(n^2)` booleans if you materialize it. You can keep two diagonals (length and length-minus-2) and still count in `O(n)` extra memory, but the comparisons are identical to expansion, which is already `O(1)` extra memory. There is nothing further to roll: the answer is a scalar. Time remains quadratic because a string of identical characters has `Θ(n^2)` palindromic substrings and you must count each of them.

## 3. Variant: return the substrings, or count only odd lengths, or count palindromes that are also in a dictionary. What changes?

Returning the slices means each successful expand appends `s[left:right+1]` before the pointers move. Output size can be `Θ(n^2)`, so the time bound grows with the output. Odd lengths only: drop the even-center call. Dictionary filter: the counter becomes “expand, and if this window is in the set, add 1,” which is correct but you should cap the walk by the maximum word length. None of these change the center state. A subsequence count would.

## 4. What are the bounds on `"abc"` and on a string of `n` identical characters?

`"abc"` has no two matching neighbors, so the answer is `n` (here 3): only the trivial centers. A string of `n` identical characters has `n(n+1)/2` palindromic substrings, one per pair `L ≤ R`. The expansion counts exactly that many successful steps. Quote both so the interviewer hears that you know the quadratic worst case is tight.

## 5. What bug does this code have?

```text
count = 0
for i in range(n):
    if s[i] == s[i]:
        count += 1
    if i + 1 < n and s[i] == s[i + 1]:
        count += 1
```

It counts length 1 and length 2 only. `"aaa"` returns 3 + 2 = 5 and misses `"aaa"`. The walk has to continue past the first match, and each longer radius is another substring. A second bug is starting the even walk at `(i, i)` again and double-counting odds.

## 6. How does this differ from longest palindromic substring in the interview script?

Same centers, same while-loop, different scoreboard. Longest keeps a start index and a length, and updates when the new window is at least as long. Count adds one per successful comparison and never stores indices. Say that out loud so you do not accidentally return a string from the counting problem. If they then ask for both, compute them in one expansion: on each match, increment the count and maybe replace the best window.
