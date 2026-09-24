# 51. House Robber II — Interview Q&A

## 1. Why this state?

The state stays the linear one: best loot on a contiguous range that is not circular. A true circular state would have to remember whether house 0 was taken while you scan the rest, which is a boolean flag on top of the prefix. That works, but it is the same search as two separate ranges: flag true means you already spent house `n - 1` and you start from house 1; flag false means you may still use the end but you have not taken house 0. The range split makes the flag implicit and reuses the linear code unchanged.

## 2. How do you optimize space?

Each range already rolls down to two integers, so the circular problem is `O(1)` extra memory and `O(n)` time. You do not need a length-`n` array unless you must reconstruct the houses. Do not try to share one pair of rolling variables across both ranges without resetting them to 0; the second range is a new subproblem.

## 3. Variant: three houses mutually adjacent, or a line of blocks where every k-th house wraps. What changes?

If only the first and last are adjacent, this solution is already the variant of the linear robber. If every house forbids the next two, go back to a longer lookback on a line first (`dp[i - 3]` on a take). A wraparound of that kind is no longer “drop one endpoint”: a take near the seam can forbid two houses on the other side, so casework on the seam (which of the boundary houses are taken) replaces the simple two-range max. State that the number of cases is the number of legal patterns on the boundary, and only then code.

## 4. Why is `max(linear(entire array), something)` incorrect?

The linear answer on the full circle-as-array is allowed to rob both `nums[0]` and `nums[n - 1]`. On `[2, 3, 2]` the linear answer is `2 + 2 = 4`, and that plan is exactly the one the circle forbids. There is no repair that subtracts a constant afterward, because you do not know whether the linear optimum used both ends. Running the two shorter ranges never generates that plan.

## 5. What bug does this range call have?

```text
if n == 0: return 0
return max(rob(nums[0:n-1]), rob(nums[1:n]))
```

For `n == 1`, `nums[0:0]` is empty and `nums[1:1]` is empty, so you return 0 instead of `nums[0]`. Also, in Java, copying each slice costs extra time and memory; passing `(lo, hi)` into one array keeps the passes truly linear. The math of the two ranges is right only after the one-house case is handled.

## 6. Walk through a case where the two ranges disagree.

On `[1, 2, 3, 1]` the range that drops the last house scores 4 (`1 + 3`). The range that drops the first house scores 3 (`2 + 1` or just `3`). The answer is 4. On `[2, 7, 9, 3, 1]` both endpoints are small; the best linear plan on the whole array may or may not use both ends, so you still trust the max of the two ranges rather than eyeballing it. Say the overlap: house `9` is available in both ranges, and that is fine because you do not add the range scores.
