# Maximum Subarray — Interview Q&A

### 1. Why can you throw away a negative running sum?

**Answer.** Suppose the best subarray that ends at `i-1` has sum `cur < 0`, and some optimal subarray continues through index `i`. Deleting that negative prefix and starting at `i` increases the sum. So there is always an optimal subarray that does not begin with a negative ending-here prefix. The transition `max(nums[i], cur + nums[i])` is exactly that deletion.

### 2. What does the algorithm return on `[-2, -3, -1]`, and why do zero-initialized versions fail?

**Answer.** It returns `-1`, the largest element. A version that clamps the running sum at 0 never records a negative candidate, so it falls through to 0. The empty subarray is not allowed. Seeding `best` and `cur` from `nums[0]` fixes it with no extra branch.

### 3. Is this greedy or dynamic programming?

**Answer.** Both descriptions match the code. The DP state is the best sum ending at `i`, with a transition that looks only at `i-1`. The greedy description is the same transition read as a permanent choice: restart or extend. There is no table because the state is one integer. The global answer needs a second integer, the max over all ending positions.

### 4. How would you also return the start and end indexes?

**Answer.** Keep `curStart`. Whenever you choose `nums[i]` over `cur + nums[i]`, set `curStart = i`. Whenever `cur` becomes the new `best`, copy `curStart` into `bestStart` and `i` into `bestEnd`. The sum is still `best`. Do this in the same left-to-right pass so the indexes describe the window you actually summed.

### 5. What is the divide-and-conquer alternative, and when is it worse?

**Answer.** Split the array in half. The best subarray is entirely on the left, entirely on the right, or it crosses the midpoint. The crossing piece is the best suffix of the left half plus the best prefix of the right half, found with two linear scans. Recurrence `T(n) = 2T(n/2) + O(n)` is `O(n log n)`. Kadane is strictly faster and simpler. The recursive form is a teaching step toward similar midpoint problems, not the interview solution you want to ship.

### 6. Can Kadane handle a “product” version or a “circular” version unchanged?

**Answer.** No. Maximum product must track both the largest and the smallest product ending here, because a negative number flips them. Maximum circular sum is the max of ordinary Kadane and `(total sum - minimum subarray sum)`, with a special case when every element is negative so you do not wrap the entire array and leave an empty gap. Quote those as different states, not as the same five lines.
