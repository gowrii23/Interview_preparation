# Interval Patterns — Interview Q&A

### 1. How do you choose between sorting by start and sorting by end?

**Answer.** Sort by start when the next decision is “does this interval collide with the one I am currently building?” That is merge, insert, and the single-room attendance check. Sort by end when the decision is “which interval should I keep so the timeline frees up as early as possible?” That is minimum removals to eliminate overlaps. Using the other key changes the problem you are solving.

### 2. Do two intervals that only share an endpoint overlap?

**Answer.** It depends on the problem’s definition, and on this set the definitions disagree in a precise way. Merge treats `[1,4]` and `[4,5]` as overlapping and produces `[1,5]`: the test is `start <= currentEnd`. One attendee can still sit in both `[1,2]` and `[2,3]`, and non-overlapping removal treats them as compatible: the test is `start >= keptEnd` to keep, and `start < keptEnd` to call it a conflict. Say the comparison out loud before you code it.

### 3. Why must a meeting-room sweep process the end event before the start event at the same time?

**Answer.** A meeting that ends at time `t` releases its room at `t`, and a meeting that starts at `t` may use that room. If you apply `+1` first, the running depth is one too high at that instant and the recorded maximum can be wrong. Sorting the event `(time, delta)` with `delta = -1` for an end and `delta = +1` for a start puts the end first automatically, because `-1 < +1`.

### 4. What is the exchange argument for erasing the minimum number of intervals?

**Answer.** Sort by finishing time. Let `G` be the interval that ends first. Some optimal set of kept intervals includes a first interval `X`. `X` ends no earlier than `G`. Replacing `X` with `G` still leaves a non-overlapping set, because anything that started at or after `X`’s end also starts at or after `G`’s end, and the size does not change. Repeat on the suffix. Therefore always keeping the compatible interval that ends earliest is optimal, and the number erased is `n - kept`.

### 5. Why is “merge everything, then look at the merged list” the wrong preprocessing step for minimum deletions?

**Answer.** Merging collapses several original intervals into one object, so you lose how many inputs you removed. `[1,3]` and `[2,4]` become one interval whether you deleted zero or one of them. Deletion counts original pieces. Keep the input intervals intact, sort copies by end, and count skips.

### 6. What complexities do you quote for merge, erase, and minimum rooms?

**Answer.** All three are `O(n log n)` time because of the sort. Merge writes `O(n)` output. Erase needs `O(1)` extra integers if the sort is in place. Minimum rooms via sweep stores `O(n)` events; via a heap it stores `O(n)` end times in the worst case (every meeting overlaps). A linear algorithm would require the intervals to arrive already sorted.
