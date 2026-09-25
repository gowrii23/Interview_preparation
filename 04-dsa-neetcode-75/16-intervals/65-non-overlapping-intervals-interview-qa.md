# Non-overlapping Intervals — Interview Q&A

### 1. Why sort by end time instead of by start time or by length?

**Answer.** The safe greedy choice is the interval that finishes first among those still available. It leaves the largest possible suffix of the timeline for everyone else. An exchange proves it: in any optimal keep-set, replace the first kept interval with this earlier-finishing one and the rest still fit. Shortest length is a different rule and fails when a short interval sits in the middle of two otherwise compatible ones and blocks both. Earliest start fails when a long interval starts first and covers several short ones.

### 2. What is the smallest counterexample for “sort by start and drop overlaps with the first interval”?

**Answer.** `[[1,100],[2,3],[3,4]]`. That rule keeps `[1,100]` and deletes the other two, so it reports 2. Sorting by end yields `[2,3]`, `[3,4]`, `[1,100]`. Keep the first two (`3 >= 3`), delete `[1,100]`, report 1. One deletion is optimal.

### 3. Are `[1,2]` and `[2,3]` a conflict? What comparison encodes that?

**Answer.** They are not a conflict on this problem. A meeting that ends at 2 and a meeting that starts at 2 can both be kept. After you keep an interval, the next one is kept when `start >= end`. Using `>` deletes a legal pair and inflates the answer by one.

### 4. Why does the algorithm still work when several intervals share the same end?

**Answer.** If two intervals end together, keeping either one frees the timeline at the same moment, so the suffix of the problem is identical. The sort may place them in either order. You keep the first, and the second starts strictly before that shared end (otherwise it would be a point-touch or a duplicate span). A duplicate `[1,2],[1,2]` starts before the kept end, so it is removed, which is right: they overlap on a whole segment, not just an endpoint.

### 5. Could you return the intervals to delete instead of the count? What would you store?

**Answer.** The same scan. When `start < end`, record that interval as deleted; when you keep one, record it only if the caller wants the schedule. The count is `n - kept` either way. Store copies of the pairs if the caller must not see your sort mutate the original order. The greedy set of kept intervals is one optimal set, not the only one, when several choices end at the same time.

### 6. What complexity do you quote, and why isn’t this a DP?

**Answer.** `O(n log n)` time, `O(1)` extra memory past the sorting workspace. A DP “longest chain of compatible intervals” after the same sort is also correct and also `O(n log n)` or `O(n^2)` depending on how you search the predecessor, but it is unnecessary: the exchange argument says the earliest-finish choice is always part of some optimum, so there is nothing to memoize. DP becomes the right tool if each interval has a weight and you want maximum weight instead of maximum count.
