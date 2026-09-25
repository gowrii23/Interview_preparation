# Sliding Window — Interview Q&A

## 1. Why is a window that shrinks in an inner loop still `O(n)`?

`right` advances once per element. `left` advances at most once per element across the whole algorithm, because it only increases and it stops at `n`. The inner loop’s total iterations are `O(n)`, not `O(n)` per outer step. That bound dies if `add` or `remove` scans the window, or if you slice the window to rebuild state. Say “amortized `O(1)` per index” and then give the total `O(n)`.

## 2. Why a window instead of a prefix array or dynamic programming?

The score of the current range can be updated from the previous range by one arrival and one departure: a character count, a running sum, a min price. A prefix array recomputes a sum in `O(1)` but does not know where the segment should start when the constraint is “no repeats” or “covers this multiset.” Those constraints need the left edge to react to the right edge. Use the smallest state that makes “still valid?” `O(1)`.

## 3. What bug does this sketch have?

```text
for right in range(n):
    add(nums[right])
    if invalid():
        remove(nums[left])
        left += 1
    best = max(best, right - left)
```

Three bugs. `if` instead of `while` shrinks by only one element, so a window can stay invalid. The width is `right - left + 1` for an inclusive range. Updating `best` after a partial shrink can record an illegal window if you forget to recheck. For a minimum cover, recording the length only when you expand misses the shorter windows found while you shrink.

## 4. Follow-up: the window must have exact length `k`, or at most `k` distinct values. What changes?

Exact length `k`: move `right`, and when `right - left + 1` exceeds `k`, remove `nums[left]` and advance `left` once. There is no “while invalid” beyond that size check. At most `k` distinct: keep a frequency map and a distinct counter; while distinct exceeds `k`, remove from the left. Both stay linear if map updates are `O(1)`.

## 5. What if the constraints change and `n` is 10^7, the alphabet is Unicode, and you need the substring itself?

Time can stay linear, but a hash map of code points replaces a length-26 array. Copying the best substring is `O(n)` in the worst case and is required only at the end: store `bestLeft` and `bestRight`, and slice once. Slicing on every improvement copies overlapping pieces and can become quadratic. At `n = 10^7`, avoid allocating a new string per step.

## 6. How do you recognize that a problem is not a sliding window?

If the chosen elements need not be contiguous, it is a subset, a two-pointer pair on a sorted array, or a knapsack, not a window. If the left edge must move backward, the monotonic-queue or stack pattern may fit better. If adding an element on the right does not have an inverse remove on the left, you cannot maintain the state. Ask “does the feasible region stay one interval?” before you commit.
