# 50. House Robber — Interview Q&A

## 1. Why this state?

`dp[i]` is the best loot from the first `i` houses. The adjacency rule is local: taking house `i - 1` only forbids house `i - 2`, and the best way to rob a prefix that already respects the rule is enough. You do not store “was the previous house robbed?” as a second index, because the recurrence tries both decisions explicitly: `dp[i - 1]` already represents plans that may have robbed the previous house, and you simply refuse to extend those plans with another take. A second boolean would also work and would use more memory for the same numbers.

## 2. How do you optimize space?

The row is length `n + 1`, and each cell reads two earlier cells. Keep those two numbers (`prev2`, `prev1`) and a candidate `cur`. After the loop, `prev1` is `dp[n]`. Time is still `O(n)`, extra memory is `O(1)`. Do not drop to one variable: skip and take need two different prefixes. If the interviewer wants the houses, not the sum, store a predecessor choice per index and walk back, which forces the `O(n)` array to stay.

## 3. Variant: the first and last houses are neighbors, or you must skip `k` houses after a robbery. What changes?

Neighbors at the ends is house robber II: run this linear routine on `nums[0:n-1]` and on `nums[1:n]`, then take the max. A single house is a special case because those two ranges would both be empty. If a robbery forbids the next `k` houses, the take branch reads `dp[i - k - 1] + nums[i - 1]` and you roll `k + 1` values, or you keep the array. The state sentence is still “best on a prefix.”

## 4. What are the bounds, and why is sorting the houses wrong?

Time `O(n)`, extra space `O(1)`. Sorting by money destroys the neighbor relation, which is about indices, not about rank. Greedy “rob every strict local peak” fails on `[2, 7, 9, 3, 1]`: the only strict peak is `9` (sum 9), while the optimum is `2 + 9 + 1 = 12`. The DP keeps that plan because the prefix that skipped `7` is still available when it considers `9`. A local choice that commits to `7` and then `3` scores 10 and is also worse; the recurrence compares both and keeps 12.

## 5. What bug does this code have?

```java
int prev1 = 0, prev2 = 0;
for (int money : nums) {
    prev1 = Math.max(prev1, prev2 + money);
    prev2 = prev1;
}
```

`prev2` copies the updated `prev1`. On `[2, 7]`, after 2 both variables are 2; then `prev2 + 7` uses 2 instead of 0 and returns 9, which robs both houses. Swap into a temporary, or assign `prev2, prev1 = prev1, max(...)` in one step.

## 6. How do you derive it on a whiteboard in under two minutes?

Draw five houses and a row `dp[0..5]`. Say `dp[i]` is the best prefix. Write `max(skip, take previous-previous + this)`. Fill `[2, 7, 9, 3, 1]` until 12. Circle the rejected take of `3` (10 loses to 11). Say you only kept the last two cells in code. Stop. If they ask about a circle, do not twist this recurrence; say you will call it twice.
