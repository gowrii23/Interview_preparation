# Two Pointers — Interview Q&A

## 1. What is the time and space of a two-pointer sweep, including the sort?

The sweep itself is `O(n)` time and `O(1)` extra memory because each iteration advances at least one pointer and does a constant amount of work. If you sorted first, add `O(n log n)` time. Say both pieces. Space stays `O(1)` extra when the sort is in place, aside from the sort’s own stack. Building an answer list of triplets can take `O(n^2)` space in the worst case; that is output size, and it does not change the sweep’s extra memory.

## 2. Why is moving one pointer safe? Why not move both whenever the sum is wrong?

The safety comes from monotonicity. On a sorted array, if the sum is too small, every pair that keeps the same left index and a smaller right index is even smaller, so those pairs are dead. Moving both pointers would skip the left index paired with a slightly smaller right index, and you can miss the target. You move both only after you have recorded a match and you are hunting the next distinct pair.

## 3. What bug does this loop have?

```java
while (left < right) {
    int sum = nums[left] + nums[right];
    if (sum == target) return new int[] { left, right };
    else if (sum < target) right--;
    else left++;
}
```

The moves are reversed. A sum that is too small gets even smaller when `right` decreases. The search walks away from the target and can miss a pair that was still inside the range. The other classic bug is `while (left <= right)` when the same index cannot be used twice.

## 4. Follow-up: count the number of pairs with sum at most `target`, array unsorted. What changes?

Sort a copy, then for each `left`, binary-search the farthest `right` with `nums[left] + nums[right] <= target`, or advance `right` only forward across the outer loop so the total stays `O(n log n)`. Two pointers still work: move `right` left while the sum is too big, and when it fits, every index from `left + 1` through `right` pairs with `left`. That inner count is `O(1)` per left move if `right` only decreases.

## 5. What if the constraints change and the array has 10^7 elements, values near `±10^9`, and you cannot sort because order is frozen?

If you need indices in the original array for a single pair, go back to a hash map: expected `O(n)` time, `O(n)` memory, and no sort. Two pointers without order are not correct. If you only need values and may use extra memory, sort a copy of the values; 10^7 is comfortable for `O(n log n)`. In Java, add the two values in a `long` so `10^9 + 10^9` does not overflow `int`.

## 6. How do you explain duplicate skipping without sounding unsure?

Sort first so equals are adjacent. After you accept a triplet or pair, advance the pointer while the next value equals the one you just used. Also skip a fixed outer value when it equals the previous outer value, otherwise the same multiset is rebuilt from a later identical start. Do not skip before you record, or you drop the only copy of a valid pair. State that this produces unique values, not unique index tuples.
