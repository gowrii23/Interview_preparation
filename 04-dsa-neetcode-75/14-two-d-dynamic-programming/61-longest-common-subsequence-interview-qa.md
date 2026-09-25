# Longest Common Subsequence — Interview Q&A

### 1. Why does a matching character use only the diagonal, not `max` with the two neighbors?

**Answer.** If the characters at the ends of the prefixes are equal, there exists an optimal LCS that uses that pair. Its remaining part is an LCS of the two shorter prefixes, so the value is exactly `dp[i-1][j-1] + 1`. That quantity is already at least `dp[i-1][j]` and `dp[i][j-1]`, because those neighbors are at most the diagonal plus one. Taking a max is redundant, and it hides the reason the recurrence is correct.

### 2. Why is the table `(m+1)` by `(n+1)` instead of `m` by `n`?

**Answer.** Row 0 and column 0 represent empty prefixes. Every real character then sits at index `i-1` while `dp[i]` means “length i considered.” Without the border you have to special-case the first characters inside the loops, which is where off-by-one bugs show up. The extra row and column are all zeros and cost nothing asymptotically.

### 3. How do you recover one actual subsequence from the length table?

**Answer.** Start at `(m, n)`. While both indexes are positive: if `text1[i-1] == text2[j-1]`, append that character and move to `(i-1, j-1)`. Otherwise move to the neighbor with the larger value (prefer either one on a tie). Reverse the collected characters at the end. The walk is `O(m+n)` and needs the full table, so do not roll the rows away if reconstruction was requested.

### 4. What is the difference between this and longest common substring?

**Answer.** A subsequence may skip. A substring may not. Substring DP writes `dp[i][j] = dp[i-1][j-1] + 1` on a match and `0` on a mismatch, and the answer is the maximum cell anywhere in the table, not necessarily the corner. Using the substring reset on this problem drops characters that were allowed to be non-contiguous.

### 5. How do you cut memory to `O(min(m, n))` and still get the length?

**Answer.** Iterate the longer string on the outside and keep one array of size `shorter + 1`. Before overwriting `dp[j]`, copy it into `prevDiag`, because that cell is the diagonal you need if the next comparison matches. Then set `dp[j]` to `prevDiag + 1` on a match, or `max(dp[j], dp[j-1])` on a mismatch (`dp[j]` is still the old “up” value until you store the new one). The last cell of the array is the answer.

### 6. A recursive solution with memoization is suggested. Is it the same algorithm?

**Answer.** Yes. `f(i, j)` with memo on the pair `(i, j)` evaluates the same recurrence and the same `O(mn)` distinct states. The iterative table makes the order obvious and avoids recursion depth on strings of length 1000. Either one is acceptable if you can state the state, the base case `i == 0 or j == 0 → 0`, and the `O(mn)` bound.
