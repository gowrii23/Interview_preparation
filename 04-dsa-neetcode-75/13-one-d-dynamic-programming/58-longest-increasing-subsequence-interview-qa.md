# 58. Longest Increasing Subsequence — Interview Q&A

## 1. Why this state?

`dp[i]` forces `nums[i]` to be the last element. That makes the transition local: you may append `nums[i]` to a chain ending at `j` exactly when `j < i` and `nums[j] < nums[i]`. Without “ends at `i`,” the subproblem “best subsequence inside the prefix `i`” does not tell you the tail value, so you would not know whether `nums[i]` can extend it. The end index carries the tail value for free because the value sits in the array. The global answer is the max over ends.

## 2. How do you optimize space?

The quadratic DP stores `n` lengths. Each cell can depend on any earlier cell, not a fixed window, so you cannot roll the row down to a constant. You can drop the stored `dp[i]` only if you switch algorithms. Patience sorting keeps a `tails` array of the current answer length, which is still `O(n)` worst-case memory, with `O(log n)` work per element. If they ask for one subsequence, both methods need parent links, and the `O(n)` memory comes back anyway. Say that before you “optimize” the DP array away.

## 3. Variant: non-decreasing subsequence, number of longest subsequences, or the subsequence itself. What changes?

Non-decreasing: change `<` to `≤` in the DP. In patience sorting, binary-search the first tail strictly greater than `x` (upper bound) so an equal value extends the chain instead of replacing a same-valued tail. Number of LIS: keep a second array `ways[i]`, and when `dp[j] + 1` beats `dp[i]` set `ways[i] = ways[j]`; when it ties, add `ways[j]`. The answer is the sum of `ways[i]` over indices whose `dp[i]` equals the best length. The subsequence itself: store `prev[j]` as the predecessor, then walk from the argmax of `dp` backward and reverse. Patience sorting’s `tails` is not that list.

## 4. What does the `O(n log n)` tails array mean, in one example?

Process `[10, 9, 2, 5, 3, 7]`. `tails` evolves as `[10]`, `[9]`, `[2]`, `[2, 5]`, `[2, 3]`, `[2, 3, 7]`. Each replace keeps the smallest possible tail for that length, which leaves more room for a future extension. Length 3 is the answer. `[2, 3, 7]` happens to be increasing here; it is still safer to say “the length of tails is the LIS length” than to return tails as the sequence. `[3, 1, 4]` is a quick second check: tails become `[3]`, `[1]`, `[1, 4]`, length 2, and `[1, 4]` is a real subsequence only by coincidence of the values that remain.

## 5. What bug does this code have?

```java
dp[i] = 1;
for (int j = 0; j < i; j++) {
    if (nums[j] <= nums[i]) {
        dp[i] = Math.max(dp[i], dp[j] + 1);
    }
}
return dp[nums.length - 1];
```

Two bugs. `<=` counts a plateau of equal numbers as increasing, so `[7, 7, 7]` returns 3 instead of 1. Returning the last cell drops a longer chain that ended earlier: on `[1, 3, 0]` the last cell is 1 (nothing smaller than 0 is a useful extension that beats `[1, 3]`), and the answer is 2. Track a `best` over every `i`, and compare with `<`.

## 6. How do you choose which algorithm to code first?

Code the `O(n^2)` DP unless they have asked for a faster bound or `n` is large enough that `n^2` will not pass. It is shorter to get right, and the predecessor array falls out of the same loops. After it works, say: “The length alone can be computed in `O(n log n)` by keeping the smallest tail of each length and binary-searching it.” Wait for them to ask before you rewrite the solution. If you start with patience sorting, be ready for “print the subsequence,” and do not read `tails` backward as if it were one.
