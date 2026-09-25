# 56. Maximum Product Subarray — Interview Q&A

## 1. Why this state?

The score of a contiguous block depends on every factor inside it, but the only decision at index `i` is whether the block ends at `i` starts there or continues. Continuation multiplies by one new number. The previous product that is worth continuing might be the largest or the smallest one ending at `i - 1`, because the new number can be negative. Those two scalars are the state. You do not need the start index in order to return the product. You would store it only if the follow-up asked for the slice itself.

## 2. How do you optimize space?

`maxEnd` and `minEnd` are arrays only if you want every index on the board. The loop reads solely index `i - 1`, so two running values and one `best` are enough: `O(1)` extra memory, `O(n)` time. Do not try to keep a single running product “and flip a sign bit.” A zero, then a negative, then another negative, needs the numeric min, not just a sign, because magnitudes differ. The prefix-product scan from both ends is also `O(1)` extra memory; mention it as an equivalent, and code the min/max form unless they ask for the other.

## 3. Variant: maximum sum subarray, or maximum product of a non-contiguous subsequence, or return the subarray. What changes?

Maximum sum is Kadane: `cur = max(x, cur + x)`, and there is no min companion, because a negative addend never makes a worse sum better. A non-contiguous product with optional skips is a different problem (and with negatives it becomes a matter of how many negatives you include). Returning the subarray: whenever `best` improves, record the start. The start resets when the chosen candidate is `x` itself rather than an extension. Practice that bookkeeping on `[2, 3, -2, 4]`, where the answer slice is `[2, 3]`, not the final `[4]`.

## 4. What happens at zero, and what if the array is all negative?

A zero forces both `nextMax` and `nextMin` to 0 when you multiply, and the `max`/`min` with `x` itself also yields 0 if `x` is 0. The next non-zero value then restarts because extending 0 is 0, and `max(x, 0)` is `x` when `x` is positive. The answer can be 0 if every other product is negative. If the whole array is negative, pairs of negatives are positive only when a window contains an even count, but a window of two negatives is a positive product; if you never get a positive, the least-negative single element wins because `best` was seeded with `nums[0]` and each `nextMax` considers `x` alone. `[-2]` returns `-2`. `[-3, -1, -2]` returns `3` from `[-3, -1]`, not `-1` and not the full product `-6`.

## 5. What bug does this code have?

```java
curMax = Math.max(x, Math.max(curMax * x, curMin * x));
curMin = Math.min(x, Math.min(curMax * x, curMin * x));
```

The second line uses the already updated `curMax`. On `[2, -5, -2]`, at `-2` the old max is `-5` and the old min is `-10`. After the first line `curMax` becomes 20. The second line then computes `min(-2, 20 · -2, -10 · -2) = min(-2, -40, 20) = -40`, and the next iteration is poisoned. Even this cell’s min is wrong, so a longer array fails. Compute both next values from the old pair, then assign.

## 6. Why can the answer end before the last index?

`[2, 3, -2, 4]` has `maxEnd = [2, 6, -2, 4]`. The last cell is 4, the product of `[4]` or of a restart. The best cell is 6, the prefix `[2, 3]`. Nothing after that prefix can be glued on without multiplying by `-2`. The state is “best ending here,” so the problem answer is a max over endings. If you returned only the last `curMax`, this input would fail.
