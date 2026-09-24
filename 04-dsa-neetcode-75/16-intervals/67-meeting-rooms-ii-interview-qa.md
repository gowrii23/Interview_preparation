# Meeting Rooms II — Interview Q&A

### 1. Why is the minimum number of rooms equal to the maximum overlap depth?

**Answer.** At any time `t`, every meeting that contains `t` needs a distinct room, so you need at least the maximum depth. That many rooms are also sufficient: along a sweep, the depth only grows when a meeting starts, and a free room exists whenever the depth is below the global maximum you allocated. You never need an extra room “just in case” beyond that peak.

### 2. What goes wrong if a sweep processes starts before ends at the same time? Give the smallest input.

**Answer.** `[[1,5],[5,10]]`. One room is enough, because the first meeting ends as the second starts. Events at time 5 are `end -1` and `start +1`. If `+1` is applied first, the running depth goes 1 → 2 → 1 and you report 2. If `-1` is applied first, the depth goes 1 → 0 → 1 and you report 1. The tie-break is part of the algorithm, not a cosmetic sort detail.

### 3. How does the heap know a room can be reused, and why is the final heap size the answer?

**Answer.** Meetings are processed in start order. The heap stores end times of meetings that still hold a room. If the smallest end is `<=` the meeting you are about to place, that room is free: pop it, which is the reuse, then push the new end. Every meeting is pushed once. A pop only happens together with a later push, so the size after all meetings equals the maximum number of rooms held at once. Using `<` instead of `<=` fails the `[1,5]`,`[5,10]` case by refusing to pop.

### 4. Two meetings start at the same time and a third ends then. What does each method do?

**Answer.** Suppose ends and starts at time `t`, with one end and two starts. The sweep applies the `-1` first, then each `+1`, so the end offsets one of the starts and the second start still increases the depth. The heap, because it sorts only by start, sees both new meetings after any meeting that ended at `t` has already been pushed. When each of those new meetings is processed, `peek() <= start` reuses the freed room once; the second meeting does not find another free room and the heap grows. Same number.

### 5. Can you solve this with a boolean “one room is enough” check, or by merging intervals?

**Answer.** The boolean check is Meeting Rooms I: it stops at the first overlap and cannot tell two rooms from five. Merging overlaps destroys how many originals piled up; `[1,10]`, `[2,3]`, `[4,5]` merge into one segment but need two rooms only while `[1,10]` overlaps each short meeting, and the merged shape looks like one room. Keep every start and end.

### 6. What complexity do you quote, and can it be linear?

**Answer.** Both the sweep and the heap are `O(n log n)` time and `O(n)` extra memory. Linear time needs a bound on the timestamp range so you can bucket-sort the events, or an input that is already sorted. In an interview, quote `O(n log n)` unless they hand you that extra promise. Empty input is 0 rooms.
