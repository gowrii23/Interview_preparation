# 33. Search in Rotated Sorted Array — Interview Q&A

## 1. What is the complexity, and how does it compare with finding the pivot first?

Both approaches are `O(log n)` time and `O(1)` extra space. Finding the minimum index is one binary search; searching the sorted piece to the left or right of that pivot is a second binary search. The single loop does the same halving without an explicit pivot: each step still discards half the index range because the sorted half’s endpoint test is decisive. A linear scan is `O(n)` and fails the usual follow-up that demands logarithmic reads.

## 2. Why does knowing which half is sorted tell you where the target must be?

A sorted half has a closed range of values. If the target is inside that range, it cannot hide in the other half, because values are distinct and the sorted half already covers every value between its endpoints. If the target is outside that range, it cannot be in the sorted half, so it is in the other half if it is anywhere in the current window. The rotation point is the reason one half might be unsorted; it is not a reason both halves are unsorted at once. The invariant stays “the index, if it exists, is inside `[lo, hi]`.”

## 3. What bug does this code have?

```java
if (nums[lo] < nums[mid]) {
    if (nums[lo] <= target && target <= nums[mid]) hi = mid - 1;
    else lo = mid + 1;
} else {
    if (nums[mid] <= target && target <= nums[hi]) lo = mid + 1;
    else hi = mid - 1;
}
```

Using `<` instead of `<=` mis-classifies `lo == mid`, where the left “half” is a single sorted element. The else branch then treats the right side as the sorted one and can discard the target sitting at `lo`. A second bug is `target <= nums[mid]` after you already returned on equality: it is harmless, but `target < nums[mid]` matches the fact that `mid` is not the answer. The harmful cousin is `hi = mid` in this inclusive loop, which can repeat the same `mid` forever. A third bug is `(lo + hi) / 2` overflowing `int`.

## 4. Follow-up: duplicates are allowed. What changes?

When `nums[lo] == nums[mid]`, you cannot tell which side is sorted. Shrink one end by one (`lo++`) and continue. Worst-case time becomes `O(n)`, for example an array of mostly equal values with the target absent or at the end. Average or best case can stay logarithmic when duplicates are rare. Say that worst case out loud. This is the search-in-rotated-sorted-array II problem.

## 5. What if the constraints change and `n` is 10^7, you need the value rather than the index, and the array might not be rotated at all?

The same search works when the rotation is zero: the left half test `nums[lo] <= nums[mid]` stays true and the algorithm becomes ordinary binary search. `n = 10^7` is a few dozen comparisons, which is the point. Returning the value is `nums[index]`, or the target itself if you only need to know it exists. If they drop random access and give you a stream, you cannot binary search; you scan in `O(n)`. Index math should use a 64-bit mid only if `n` can exceed the 32-bit index range.

## 6. What if the target is the first element, the last element, or the minimum at the rotation point?

All three are ordinary values in the same loop. The first element is found when a sorted half includes `lo` and the target equals `nums[lo]`, or when `mid` lands on it. The last element is included by `target <= nums[hi]` on a sorted right half. The minimum is just another distinct value; you do not search for the pivot unless the target happens to be there. An empty array returns `-1` because `lo > hi` immediately. A one-element array checks that element and returns `0` or `-1`.
