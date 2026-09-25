# 1D Dynamic Programming — Interview Q&A

## 1. Why is a single index enough state for this whole folder?

The future of the sequence does not care how you built the prefix, only what that prefix is worth. For climbing stairs the worth is a count of ways. For a robber it is the best loot so far under the adjacency rule, and that rule only reaches back one house, so it is already baked into `dp[i - 1]` versus `dp[i - 2]`. For coins the worth is the fewest coins that sum to a remaining amount. If two histories leave the same index or the same amount, they share a subproblem. You add a second dimension only when one number is not enough: a second string (edit distance), a capacity plus an item index you cannot reuse, or a product that can be largely negative, which you handle as a pair of running values rather than a second array index.

## 2. How do you optimize space, and when is the array already optimal?

If cell `i` reads only `i - 1` and `i - 2`, store those two numbers and overwrite them as you walk. Stairs, house robber, and decode ways all do this. The readable array is still the thing to derive on the board; the variables are a transcription. Coin change cannot drop below `O(amount)` because every smaller amount may be reused, but you can drop the coin-index dimension: fill one array from small amounts to large so `dp[a - c]` already allows another copy of `c`. A 0/1 knapsack walks the amount downward so each item is used once. Word break needs a boolean per prefix in the worst case. Maximum product and jump game are already a constant number of running values once the recurrence is understood.

## 3. What changes in a variant that adds a constraint?

Name the new constraint and see whether the old state still summarizes the past. “Also pay a cost to step” becomes min-cost climbing: same indices, `min` instead of `+` on ways. “Houses form a circle” does not need a circular state; solve two linear ranges, one that drops the first house and one that drops the last. “Return the number of coin combinations” is the same capacity row with `+=` instead of `min`, and the loop order decides whether order of coins matters. “Return one actual LIS” needs a predecessor index per cell, then a walk backward. If the variant makes two prefixes interact (two strings, or house `i` depends on a subset), stop calling it 1D.

## 4. What bug does this rolling sketch have?

```text
prev2 = 0
prev1 = 0
for x in nums:
    prev1 = max(prev1, prev2 + x)
    prev2 = prev1
```

`prev2` is overwritten with the new `prev1`, so both variables hold the same value and the skip/take choice collapses. The assignment has to move the old `prev1` into `prev2` before `prev1` changes, or it has to use a temporary. The same class of bug shows up as updating `curMax` and then using that new `curMax` to compute `curMin` in the product subarray, and as adding `1` to `Integer.MAX_VALUE` in coin change.

## 5. How should you talk through a 1D DP answer in under two minutes?

State, base case, recurrence, answer cell, complexity. Example shape: “`dp[i]` is the best loot from the first `i` houses. `dp[0] = 0`. `dp[i] = max(dp[i - 1], dp[i - 2] + nums[i - 1])`. The answer is `dp[n]`. Time is linear, and the lookback is constant so memory is two integers.” Then offer the twist you expect: circle, zeros, negatives, or “this boolean row is actually a reachable prefix.” Do not start from the optimized variables.

## 6. When is the DP row the wrong thing to code even though a recurrence exists?

When a cheaper argument proves the row has extra structure. Jump game’s reachable indices are always `0..farthest`, because from any index you can land on every offset up to `nums[i]`, so the max frontier is the whole answer. Palindrome substring checks share a center: expanding while the ends match is the recurrence `s[L] == s[R]` and the inside is already a palindrome, evaluated implicitly by the loop, in `O(1)` extra memory. Still write the recurrence if they ask. For LIS, the `O(n^2)` row is the required solution; patience sorting is the follow-up when they ask for `O(n log n)`.
