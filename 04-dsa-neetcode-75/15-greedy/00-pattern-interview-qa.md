# Greedy Algorithms — Interview Q&A

### 1. What makes a greedy choice safe?

**Answer.** An exchange argument. Take any optimal solution that does not contain the greedy choice. Show you can swap the greedy choice in — replacing one conflicting piece, or discarding a negative prefix — without lowering the objective. After the swap, the same argument applies to what remains. If that rewrite can fail, the choice is a heuristic, not an algorithm.

### 2. Give a small case where sorting by start time loses, and sorting by end time wins.

**Answer.** Intervals `[1, 100]`, `[2, 3]`, `[4, 5]`. “Take the earliest start” keeps `[1, 100]` and then nothing else, for one meeting. “Take the earliest finish” keeps `[2, 3]` and `[4, 5]`, for two meetings, and drops `[1, 100]`. The objective is the number kept (or, equivalently, the number removed). Earliest finish leaves the most room for later meetings.

### 3. Why is Kadane described as both DP and greedy?

**Answer.** The DP state is “best sum of a subarray that ends at index i.” The transition is `cur = max(nums[i], cur + nums[i])`: either the best ending here is a brand new singleton, or it extends the best ending at `i-1`. The greedy reading of that same line is “if the sum so far is negative, throw it away.” Both descriptions compute the same value. The global answer is the maximum of those ending-here sums.

### 4. When is the coin-change “always take the largest coin” rule unsafe?

**Answer.** When the denominations are not a canonical coin system. With coins `{1, 5, 6}` and amount 10, always taking the largest coin picks 6 and then four 1s: five coins. Two 5s make 10 with two coins. US-style denominations hide this failure, so “it worked on the sample” is not an exchange proof. Arbitrary denominations are an unbounded knapsack and the safe algorithm is DP.

### 5. How do you talk about optimality without writing a formal proof on the whiteboard?

**Answer.** One concrete exchange is enough if it is the real one. “Suppose an optimal schedule’s first meeting ends later than mine. Swap mine in. It finishes earlier, so every later meeting that fit still fits, and the count does not fall.” For Kadane: “A prefix with negative sum cannot be a prefix of an optimal subarray, because deleting it raises the total.” Then code the choice that the sentence describes.

### 6. What complexity should you quote, and what do people forget?

**Answer.** Include the sort. “Sort by end, then one scan” is `O(n log n)` time, not `O(n)`. Extra memory is `O(1)` if you sort in place and only store the last kept endpoint, or `O(n)` if the language’s sort copies the list. Kadane is the exception that really is linear: one pass, a handful of integers.
