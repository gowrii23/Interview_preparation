# Binary Search — Interview Q&A

## 1. What is the complexity, and where does the `log n` come from?

Each iteration throws away at least half of the remaining index range, so after `k` steps the width is at most `n / 2^k`. The loop stops around `k = log2(n)`. Time is `O(log n)`. Extra memory is `O(1)` if you use indices and `O(log n)` if each step is a recursive call. Slicing the half you keep copies `O(n)` elements over the whole recursion and destroys the logarithmic bound. The answer is a comparison count, not a statement about the values being random.

## 2. Why talk about a loop invariant instead of “move toward the target”?

“Move toward the target” does not say whether `mid` stays in the range. The invariant does. For an inclusive search, “if the target is present, its index is still between `lo` and `hi` inclusive.” A miss sets `lo` or `hi` just outside `mid`, so `mid` is discarded and the statement remains true. For a minimum in a rotated array, “the minimum index is still between `lo` and `hi`,” and when `nums[mid] > nums[hi]` the left half cannot hold the minimum, so `lo = mid + 1`. When you cannot explain the invariant, the off-by-one is a guess.

## 3. What bug does this loop have?

```java
while (lo < hi) {
    int mid = (lo + hi) / 2;
    if (nums[mid] < target) lo = mid;
    else hi = mid;
}
```

Three bugs. `(lo + hi)` can overflow a signed 32-bit `int` when both indices are large; use `lo + (hi - lo) / 2`. If `nums[mid] < target` and `mid == lo`, which happens at the end of a range, `lo = mid` does not move and the loop never ends. The branch should be `lo = mid + 1` when `mid` is strictly too small. The third issue is mixing predicates: `hi = mid` keeps `mid` as a possible answer, so it belongs to “first true boundary,” not to a search that already knows `nums[mid]` is not the target.

## 4. Follow-up: find the first and last position of a target in a sorted array with duplicates. What changes?

Run the boundary form twice. First position: if `nums[mid] >= target`, the answer is at `mid` or to the left (`hi = mid`); else `lo = mid + 1`. Last position: if `nums[mid] <= target`, the answer is at `mid` or to the right (`lo = mid`); else `hi = mid - 1`, and the `lo = mid` branch needs a mid biased upward (`lo + (hi - lo + 1) / 2`) or the loop stalls. Two `O(log n)` searches. Check the final index actually equals the target.

## 5. What if the constraints change and `n` is 10^9 logical positions you cannot allocate, or the predicate is “is the shipping capacity enough?”

You binary search the answer space, not an array index. `lo` and `hi` are capacities or speeds. The invariant is “the smallest feasible answer is still inside `[lo, hi]`.” Each mid is tested by an `O(n)` scan, so total time is `O(n log A)` where `A` is the size of the answer range. Indices must be 64-bit (`long`) if `n` exceeds `2^31 - 1`. You never build the array of 10^9 cells.

## 6. How do you choose between `while (lo <= hi)` and `while (lo < hi)`?

Use `lo <= hi` when a single index is checked for equality and both neighbors of `mid` are discarded with `mid + 1` and `mid - 1`. The loop can end with an empty range, which means “not found.” Use `lo < hi` when you are shrinking onto the first boundary and one branch is `hi = mid` (mid might still be the answer). The loop ends when `lo == hi`, and that index is the candidate you verify. Mixing `hi = mid` with `<=` is the infinite-loop pair. State the choice before writing the condition.
