# 128. Longest Consecutive Sequence — Interview Q&A

## 1. Why is the set solution linear, and why does sorting not count as the target?

Each element is inserted once, tested once as a potential start, and stepped on at most once inside a forward walk. That is a constant amount of expected hash work per element, so `O(n)` time and `O(n)` space. Sorting is `O(n log n)` and then a linear scan of neighbors. It is correct and easier, and it misses the bound this problem is known for. If the interviewer accepts `O(n log n)`, say so and sort.

## 2. Why skip a number that has a predecessor instead of walking from every number?

Walking from every number repeats the same run. From `1, 2, 3, 4` you would recount the tail starting at 2, at 3, and at 4, which is `O(n^2)` on a single long run. The predecessor test keeps one start per run. The set is the right structure because “is `n - 1` here?” and “is `n + 1` here?” are membership questions, not order questions.

## 3. What bug does this code have?

```java
for (int value : nums) {
    if (!values.contains(value - 1)) {
        int length = 1;
        while (values.contains(value + length)) {
            length++;
        }
    }
}
```

Two bugs. First, `value - 1` wraps when `value` is `Integer.MIN_VALUE`, so a real start can be skipped or a fake predecessor can be detected. Second, `value + length` wraps near `Integer.MAX_VALUE`, and the loop can run forever or count the wrong neighbor. Use a `long` cursor. A third logical bug is iterating the array instead of the set without deduping: duplicates are fine only because the set lookup collapses them, but if you also remove elements incorrectly you can break later runs.

## 4. Follow-up: return one such sequence, not only its length. What changes?

When you finish a walk whose length beats the best, remember `start` and `length`. At the end emit `start, start+1, …`. Still `O(n)` expected time. If they want every longest sequence, collect each walk that ties the maximum; you may need two passes, one to learn the max length and one to record the starts, unless you store candidates and filter.

## 5. What if the constraints change and the array does not fit in memory, or values are 64-bit and `n` is 10^8?

An in-memory hash set of 10^8 longs is tens of bytes each and may not fit. External sort, then a linear scan of the sorted run, finds the longest consecutive range in `O(n log n)` I/O. If the set does fit, a 64-bit key is still expected `O(n)`; watch the start test `value - 1` in languages where `long` is 64-bit and `Long.MIN_VALUE - 1` wraps. Python’s int is safe at that width.

## 6. What if the sequence must be consecutive in index as well as in value?

That is a different problem (a contiguous subarray). The set solution ignores positions, so it would accept values that are far apart in the array. For index-consecutive, use Kadane-style or a sliding window, depending on whether you need increasing by one each step or only a set of distinct values inside a window. Ask which one they mean before you code.
