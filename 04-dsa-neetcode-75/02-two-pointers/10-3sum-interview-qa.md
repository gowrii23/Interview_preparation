# 15. 3Sum — Interview Q&A

## 1. What is the complexity, and why is it not `O(n^3)` or `O(n^2 log n)`?

Sorting is `O(n log n)`. The outer loop runs `n` times and each two-pointer scan is `O(n)`, so the searches are `O(n^2)`. That dominates. A binary search for the third value, for every pair, is `O(n^2 log n)`, which is slower and still needs duplicate handling. Extra memory is `O(1)` aside from the output and sort scratch. Name the output separately: there can be `O(n^2)` unique triplets.

## 2. Why sort and use two pointers instead of a hash set for the third value?

Both are quadratic. The hash approach, for each pair, asks whether the complement is in a set. Duplicates and “same index used twice” are easy to get wrong, and you often add a sort of each triplet anyway to dedupe. Two pointers on a sorted array make uniqueness local: equal values are neighbors, so skipping them is a few `while` loops. The sort also enables the `nums[i] > 0` cutoff.

## 3. What bug does this code have?

```java
if (sum == 0) {
    answer.add(Arrays.asList(nums[i], nums[left], nums[right]));
    left++;
    right--;
}
```

There is no duplicate skip. Input `[-2, 0, 0, 2, 2]` records `[ -2, 0, 2 ]` more than once. Another bug is skipping duplicates of `nums[i]` with `while (nums[i] == nums[i + 1]) i++` inside the `for` loop without care, which can skip the only start or walk off the array. Skip with `if (i > 0 && nums[i] == nums[i - 1]) continue` so the first copy still runs. A third bug is `int sum = nums[i] + nums[left] + nums[right]` when values can be `10^9`: the sum overflows before the comparison. Use `long`.

## 4. Follow-up: 3Sum closest, or count triplets with sum less than `target`. What changes?

Closest: keep a running best difference. On each `left, right`, update the best, then move `left` if the sum is below the target and `right` if it is above. You no longer stop at the first hit, and duplicates do not need skipping if you only return one sum. Count of sums `< target`: when `nums[i] + nums[left] + nums[right] < target`, every right index from `left + 1` to `right` works, so add `right - left` and move `left`. That count is `O(1)` per step and the total time stays `O(n^2)`.

## 5. What if the constraints change and `n` is 5000, values are `±10^9`, and you must return unique index triples rather than unique values?

`n = 5000` still allows `O(n^2)` if the constant is small, about 25 million inner steps. Values at `10^9` force a 64-bit sum in Java. Unique indices mean you do not skip equal values; you emit every `(i, j, k)` with `i < j < k` and the sum equal to zero. Output size can jump to `O(n^3)` in an all-zero array if the target is zero, so confirm they really want every index triple. If they do, the duplicate skips come out.

## 6. What if the array is all zeros, or it has fewer than three elements?

Fewer than three elements: return an empty list. The loops simply do not run a valid `left < right`. All zeros with target zero: one triplet `[0, 0, 0]`. The duplicate skips are what keep you from emitting it once per starting index. If your skip logic is wrong you either emit nothing (skipped the first zero) or emit many copies.
