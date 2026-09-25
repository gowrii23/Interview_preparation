# 49. Climbing Stairs — Interview Q&A

## 1. Why this state?

`dp[i]` is the number of ways to reach step `i`. Any full sequence ends in a single hop of size 1 or size 2, and those two families are disjoint, so they add. The earlier hops are already counted inside `dp[i - 1]` and `dp[i - 2]`. Remembering the path would turn the state into the path itself and you would be back to enumerating sequences. The index is both the subproblem and the position on the stairs.

## 2. How do you optimize space?

The full row is `n + 1` integers, but each cell reads only the previous two. Keep `prev2` and `prev1`, replace them as `i` moves, and return `prev1`. Time stays `O(n)`. You cannot do better than a constant amount of memory unless you jump to a matrix power of the Fibonacci companion matrix, which is `O(log n)` arithmetic operations and only worth mentioning if `n` is huge. For the usual constraint the rolling loop is the answer.

## 3. Variant: you may climb 1, 2, or 3 steps, or each step has a cost. What changes?

For step sizes `{1, 2, 3}`, the state is unchanged and the recurrence becomes `dp[i] = dp[i - 1] + dp[i - 2] + dp[i - 3]`, with three seeds and three rolling variables. For min-cost climbing stairs, `dp[i]` becomes the minimum cost to reach `i` (or to leave `i`), and `+` of counts becomes `min` of `dp[i - 1] + cost[i - 1]` and `dp[i - 2] + cost[i - 2]`. Same table shape, different combine. If order of climbs did not matter, you would be counting partitions, not sequences, and this recurrence would overcount.

## 4. What is the time and space, and what does the recursion tree waste?

The DP loop is `O(n)` time and `O(1)` extra memory. Recursion that branches “take 1” and “take 2” without a memo recomputes each `climb(k)` exponentially often. The number of leaves is the answer itself, which is `Θ(φ^n)`. Memoizing on `k` brings it back to `O(n)` time and `O(n)` stack or map memory. The bottom-up loop is that memo written in order, with the stack removed.

## 5. What bug does a base-case slip cause?

Seeding `dp[0] = 0` and `dp[1] = 1` and then applying `dp[i] = dp[i - 1] + dp[i - 2]` makes `dp[2] = 1`, but two stairs have two climbs (`1+1` and `2`). Either seed `dp[0] = 1` so the empty start counts as one way, or special-case `n ≤ 2` and start the loop at 3 with `(1, 2)`. Mixing those conventions double-counts or drops the all-ones path.

## 6. How do you explain this in under a minute without saying “Fibonacci” first?

“Last step is one or two. Add the ways to reach the two possible previous steps. Fill from 1 to `n`. Answer is the last cell. It happens to be a Fibonacci shift, which is a check, not the derivation.” If they ask for the list of climbs rather than the count, the state has to store parent choices and the output can be exponential, so you say that before coding a generator.
