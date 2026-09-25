# Interview Q&A: Find Median from Data Stream (LeetCode 295)

## Q1. Why do two heaps work when one sorted structure feels more obvious?

**Answer:** The median only depends on the middle one or two values, not on the full order. A max-heap can surface the largest of the small half, and a min-heap can surface the smallest of the large half. Those two tops are exactly the middle. Insertions disturb only one path in each heap, so they cost O(log n) instead of the O(n) shift a sorted array pays to open a hole. A balanced binary search tree also does this in O(log n) and can track the size of subtrees to find the middle, which is a valid answer. The two-heap version is smaller: you do not implement rotations or parent pointers, and the rebalance rule is a size check of at most one.

## Q2. State the invariant, and show how `addNum(3)` restores it after the stream `1, 2`.

**Answer:** After `1, 2` the low max-heap is `[1]` and the high min-heap is `[2]`. Sizes are equal, and `1 <= 2`. `addNum` pushes `3` onto the low heap, whose top is now `3`. It then pops `3` into the high heap. High is `[2, 3]` and low is `[1]`, so high is taller. The size repair pops high's minimum, `2`, back into low. Final state: low tops at `2` and also contains `1`, high contains `3`. Every low value is still `<=` every high value, and low has one extra element. The median is the low top, `2`. If I had stopped before the size repair, the two middle numbers would be on the wrong sides and `findMedian` would average `1` and `2`, which is the previous median.

## Q3. Why negate values in Python? What goes wrong if you negate only on insert?

**Answer:** `heapq` pops the smallest stored number. I want the low half to pop its largest number, so I store the negation: the largest original becomes the smallest negated key. `heapq.heappop(self.lo)` therefore returns `-max`, and pushing that onto the high heap requires a second negation, `-(-max)`, to store the original max. Reading the median negates `lo[0]` once more. If I negate on insert and forget the pop into `hi`, the high heap receives a negative key and the order invariant flips. If I forget the negation only when reading, I return the wrong sign. Java does not need this trick when the low heap is built with `Collections.reverseOrder()`.

## Q4. How do you compute an even median without overflow or integer division?

**Answer:** The mathematical value is `(lowTop + highTop) / 2` in real division. In Python 3 the `/` operator already returns a float, and ints are unbounded, so `(-self.lo[0] + self.hi[0]) / 2.0` is safe. In Java, `lo.peek()` and `hi.peek()` are `Integer` values. The expression `lo.peek() + hi.peek()` is a 32-bit add. Tops near `2_000_000_000` overflow to a negative int, and a later cast cannot recover the lost bits. `(lo.peek() + hi.peek()) / 2` is also integer division, so `1` and `2` become `1` instead of `1.5`. I write `((double) lo.peek() + hi.peek()) / 2.0`. The return type of `findMedian` is `double` even when the count is odd, where the low top is already the exact median.

## Q5. What are the complexities, and why can't you discard numbers that are far from the current median?

**Answer:** Each insertion does a constant number of heap operations, so it is O(log n). Each median read looks at one or two roots, so it is O(1). Space is O(n). A number that is currently deep in a heap can become a top later. After a long run of small values, one large value slides the boundary, and yesterday's deep low value moves up. If I had thrown it away to "save space," the median would be wrong. You can discard values only in a different problem with a sliding window, and then you need a way to delete arbitrary old keys, not only the roots. That is a harder structure (policy heaps, or a balanced tree of the window).

## Q6. The heaps can contain duplicates and negatives. Does any comparison need to be strict?

**Answer:** No. The order invariant is `<=`, not `<`. Two equal middle values are a valid even median: both tops are that value and the average equals it. A new value equal to both tops may land in either heap depending on the push-then-move sequence; the median is unchanged. Negatives are ordered correctly by the same comparisons. I do not special-case the sign bit. The one empty-structure edge I would mention is that `findMedian` is only specified after the first `addNum`. Before that, `lo` has no top. I would not invent a sentinel median for the empty stream unless the interviewer adds that requirement.
