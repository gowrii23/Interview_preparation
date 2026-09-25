# 153. Find Minimum in Rotated Sorted Array — Interview Q&A

## 1. What is the complexity, and why is a linear scan the wrong target here?

With unique elements, each comparison discards half the remaining indices, so the time is `O(log n)` and the extra space is `O(1)`. A linear scan looks at every element in the worst case, which is correct but misses the structure the rotation preserves: one of the two halves is strictly increasing and cannot hide the minimum once you compare `nums[mid]` with `nums[hi]`. The `log n` bound is the reason to binary search.

## 2. Why compare `nums[mid]` to `nums[hi]` instead of looking for the single descent with neighbors?

Neighbor checks (`nums[i] < nums[i - 1]`) describe the answer, and you can still binary search that predicate, but you have to be careful at index 0. Comparing with `hi` uses the invariant directly: a mid value greater than the right end means the drop has not happened yet, so it is to the right. A mid value less than or equal to the right end means you are in the same increasing run as the right end, so the drop is at `mid` or to the left. One comparison, no neighbor read, no special case for `mid == 0`.

## 3. What bug does this code have?

```java
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;
    if (nums[mid] > nums[hi]) lo = mid + 1;
    else hi = mid;
}
```

When `lo == hi`, `mid == lo` and `nums[mid] <= nums[hi]`, so the else branch sets `hi = mid` and the condition `lo <= hi` stays true forever. The loop condition must be `lo < hi` if you assign `hi = mid`. Another bug is `hi = mid - 1` in the else branch, which drops the minimum when it is exactly at `mid`. A third is `(lo + hi) / 2` overflowing `int`.

## 4. Follow-up: the array may contain duplicates. What changes?

When `nums[mid] == nums[hi]`, you cannot tell which side has the minimum. The safe move is `hi--`, which only discards one index, so the worst case is `O(n)` (an array of all equal values, or a long plateau). When they are not equal, keep the same half-discard as the unique case. Do not claim `O(log n)` worst case once duplicates are allowed. This is LeetCode 154.

## 5. What if the constraints change and `n` is 10^8, values are 64-bit, and the array is rotated but you may not read it randomly, only as a stream?

Binary search needs random access. On a pure stream you must scan, `O(n)`, and 10^8 is a single pass if you only keep the minimum. If random access exists, `O(log n)` reads still hold and values being 64-bit does not change the index math; compare with 64-bit loads. If the rotation amount `k` is given, the minimum is at index `k % n` for a right rotation of a sorted array, which is `O(1)`. Ask whether `k` is known before you search.

## 6. What if the array has one element, or it was rotated zero times?

One element: `lo == hi` immediately, the loop does not run, and you return that element. Zero rotations: `nums[mid] <= nums[hi]` on every step because the whole array is increasing, so `hi` walks down to 0 and you return `nums[0]`, the true minimum. You do not need a separate “is it rotated?” test. The largest element sits just left of the minimum; this problem does not ask for it, but it is at `lo - 1` modulo `n` after you find `lo`.
