# Insert Interval — Interview Q&A

### 1. Why is a full sort unnecessary?

**Answer.** The contract is that `intervals` is already sorted by start and pairwise non-overlapping. The new interval can intersect only a contiguous segment of that list: everything before it is completely to the left, everything after it is completely to the right. One pointer finds that segment. Sorting would be `O(n log n)` and would also be the right tool only if the input lost that guarantee.

### 2. What is the difference between the two loop conditions `end < newStart` and `start <= newEnd`?

**Answer.** The first loop skips intervals that are strictly finished before the new one begins. Equality means they touch, and touching must merge, so those intervals fall through to the second loop. The second loop consumes an interval when it starts at or before the current new end. After each consume, the new end may move right, which can pull in later intervals that did not touch the original new interval.

### 3. Walk through `[[1,5]]` inserted with `[2,3]`, and through an insert that swallows nothing.

**Answer.** `[1,5]` does not end before 2, and it starts at 1 which is `<= 3`, so the new interval expands to `[1,5]`. Output is `[[1,5]]`. Inserting `[6,7]` into `[[1,5]]`: the first loop copies `[1,5]` because `5 < 6`, the merge loop does not run, and the result is `[[1,5],[6,7]]`. Inserting `[0,0]` in front copies nothing first, does not overlap `[1,5]`, and the tail copies `[1,5]`.

### 4. What should happen on an empty list, and on a new interval that covers every existing one?

**Answer.** Empty input: both scan loops do nothing and the result is a one-element list holding the new interval. A new interval that starts before everything and ends after everything fails the first test for every element, then the second loop expands it to the min start and max end of the whole array, and the tail is empty. One interval comes out.

### 5. Is it safe to mutate `newInterval` and to append the input subarrays by reference?

**Answer.** Mutating `newInterval` is fine if that array is yours to edit; copy `[start, end]` first if the caller still needs the original pair. Appending `intervals[i]` reuses the input arrays for the unmerged pieces. That matches the usual in-place-friendly signatures. If a later step will mutate those rows, append a copy instead so you do not surprise the caller.

### 6. How is this related to merge intervals, and what is the complexity?

**Answer.** Insert-interval is merge-intervals with a special input: `n` disjoint sorted intervals plus one outsider. You could append and run a full merge, which is correct and `O(n log n)` only because of the sort you no longer need. The three-phase scan is `O(n)` time and `O(n)` output space. The overlap rule is the same rule merge uses: share an endpoint and they fuse.
