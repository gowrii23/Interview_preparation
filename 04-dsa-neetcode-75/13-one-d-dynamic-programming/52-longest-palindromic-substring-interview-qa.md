# 52. Longest Palindromic Substring — Interview Q&A

## 1. Why this state?

The property “this slice is a palindrome” depends only on the two ends and the inner slice, so an interval boolean is a correct state. You do not need that whole triangle of booleans to recover only the longest slice. Every palindrome has a center, and the longest one is the maximum radius over `2n - 1` centers. The center index plus the radius you just walked is enough state to update a global best. The inner characters are checked on the way out, which is why the overlapping inner subproblems do not need to be stored.

## 2. How do you optimize space?

The interval DP table is `O(n^2)` memory. Expansion throws each boolean away after the comparison, so extra memory is `O(1)` plus the output. You can also roll the DP diagonal down to two lengths if you insist on the table, because length `L` only reads length `L - 2`, but that is still more fiddly than expansion and the same `O(n^2)` time. Do not claim linear time unless you are prepared to explain Manacher’s odd/even transformed string and its mirror radii.

## 3. Variant: count every palindromic substring, or return the longest palindromic subsequence. What changes?

Counting slices is the next problem: the same expansion, but every successful step adds 1 instead of keeping a max window. Longest palindromic subsequence is not solved by centers. Characters may be skipped, so the state goes back to two indices moving along one string (`dp[L][R]` = best subsequence inside `L..R`), with a match taking `s[L]` plus the inside, and a mismatch taking the better of dropping `L` or dropping `R`. That table is `O(n^2)` time and is a different recurrence. Say “contiguous” before you choose.

## 4. Why can ties return either slice, and which one does this code return?

The problem statement accepts any longest palindrome. `"babad"` has both `"bab"` and `"aba"`. This code replaces the saved window whenever `len > end - start`. Because `end - start` equals `bestLength - 1`, an equal length satisfies the inequality and the later center wins. `"ac"` therefore returns `"c"`, which is a valid length-1 answer. If a grader demanded the leftmost slice, you would change the comparison to strict length improvement only.

## 5. What bug does this expansion have?

```text
while left >= 0 and right < n and s[left] == s[right]:
    left -= 1
    right += 1
return right - left + 1
```

The `+ 1` counts the failed step. On a single character the loop matches once, then both pointers move out, and `right - left + 1` is 3 instead of 1. The length of the last successful window is `right - left - 1`. The same off-by-two shows up as a substring slice that includes a mismatched end.

## 6. What are the bounds, and when would you switch to the boolean table?

Expansion is `O(n^2)` time and `O(1)` extra space. Use the boolean table when the follow-up needs every interval answer anyway (count of palindromes is still easier by expansion; “is every substring a palindrome?” or a later query over many ranges might want the table). Fill by increasing length so `dp[L+1][R-1]` is already known. Base length 1 is true. Length 2 is `s[L] == s[R]`. Do not fill in raw row-major order or you will read an uncomputed inside.
