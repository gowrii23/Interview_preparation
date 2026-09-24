# Meeting Rooms — Interview Q&A

### 1. Why does checking only consecutive meetings, after sorting by start, catch every overlap?

**Answer.** Assume a start-time sort, and suppose meeting `i` overlaps meeting `k` with `k >= i+2`, while every consecutive pair in between is disjoint. Let meeting `i` end at `e`. Meeting `i+1` starts at or after `e`, and meeting `k` starts even later, so meeting `k` also starts at or after `e` and cannot intersect meeting `i`. That contradiction means some consecutive pair must already overlap. One neighbor check is enough.

### 2. What is the exact overlap test, and how does it differ from merge intervals?

**Answer.** Here, `next.start < prev.end` means one person cannot attend both. `next.start == prev.end` is allowed: the first meeting is over. Merge intervals uses the opposite boundary: `next.start <= prev.end` fuses the ranges, because a shared point belongs to both closed intervals and the union is one segment. Say which problem you are in before you pick `<` or `<=`.

### 3. What do you return for no meetings and for one meeting?

**Answer.** True in both cases. There is no pair that conflicts. The loop starts at index 1, so those inputs never look at a neighbor. Do not special-case them unless you want the code to read that way.

### 4. A candidate sorts by end time and still compares neighbors. What do you tell them?

**Answer.** The neighbor argument was proved for start order. End order can place a short early meeting between two overlapping ones and the consecutive end-ordered pairs are not the right witnesses. Example shape: a short meeting that ends first, then a long meeting that started earlier, then another that overlaps the long one. Ask them to re-sort by start or to switch to the “keep earliest finish” scan, which answers a different question (how many can you keep), not this boolean.

### 5. How would you find one conflicting pair to show the interviewer, not just the boolean?

**Answer.** The same loop. When `intervals[i][0] < intervals[i-1][1]`, return those two original intervals (copy them if the sort is allowed to reorder the array and the caller needs original positions, and record indexes before sorting if identity matters). The boolean version returns immediately, so you do not need to scan the rest.

### 6. What is the complexity, and what is the follow-up if the answer is false?

**Answer.** `O(n log n)` time, `O(1)` extra memory with an in-place sort. If they ask “then what is the minimum number of rooms so that every meeting still happens?”, that is Meeting Rooms II: you cannot stop at the first overlap. You sweep starts and ends, or you keep a min-heap of finish times, and you report the maximum depth.
