# Merge Intervals — Interview Q&A

### 1. Why is sorting by start required for a single left-to-right pass?

**Answer.** After the sort, any interval that overlaps the one you are building starts at or before that interval’s current end, and it appears before any interval that is completely to the right. So you only compare against the last merged interval, not against the whole output. Without the sort, a late-starting interval can sit between two others in the input and a single “last” pointer misses a gap.

### 2. Why `max` when you extend the end? Give a counterexample if you forget it.

**Answer.** The next interval can start inside the current one and finish earlier. `[[1,10],[2,3]]`: after the sort, `2 <= 10`, and the union still ends at 10. Assigning `last.end = 3` chops the union down to `[1,3]` and then a later `[4,5]` looks like a gap when it is still inside the original `[1,10]`. `max` keeps 10.

### 3. Should `[1,4]` and `[4,5]` merge? What about `[1,4]` and `[5,6]`?

**Answer.** `[1,4]` and `[4,5]` share the point 4, so they merge into `[1,5]`. The gap test is strict: a new interval starts only when `last.end < next.start`. `[1,4]` and `[5,6]` have `4 < 5`, so they stay separate. If a problem redefined intervals as half-open `[start, end)`, the equality case would flip; this problem uses closed intervals.

### 4. What does the algorithm do with an empty list, a single interval, and duplicates?

**Answer.** Empty input returns an empty list before the sort matters. A single interval is the seed and the loop never extends it. Exact duplicates have equal starts, so the sort keeps them adjacent and `start <= end` merges them into one copy. `[[1,4],[1,4]]` becomes `[[1,4]]`.

### 5. Can you merge in place, and what is the complexity either way?

**Answer.** You can overwrite a prefix of the sorted array and return the used length, which avoids a second array of intervals. You still need the `O(n log n)` sort. The version that builds a new list uses `O(n)` extra memory and is easier to get right because it does not alias input rows. Quote `O(n log n)` time either way. Linear time only if the input is presorted.

### 6. How do you explain this to someone who wants to check every pair?

**Answer.** Every pair is `O(n^2)` and still needs a union-find or a second merge pass to collapse chains such as `[1,2]`, `[2,3]`, `[3,4]`. Sorting gives you the chain for free: each interval is considered once against the current union. The invariant is “the output is sorted, disjoint, and covers every input interval seen so far.”
