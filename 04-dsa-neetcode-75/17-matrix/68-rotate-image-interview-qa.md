# Rotate Image — Interview Q&A

### 1. Where does cell `(r, c)` go, and why can’t you write that formula in a single assignment?

**Answer.** Clockwise, `(r, c)` goes to `(c, n - 1 - r)`. The source of the value that must end up at `(r, c)` is therefore `(n - 1 - c, r)`. Writing the formula cell by cell destroys the source before every member of the cycle has been read. Four cells feed each other, so you save one and assign the other three in an order that always reads a not-yet-overwritten cell, then drop the saved value into the last hole.

### 2. How many iterations does the inner loop run on a layer, and why?

**Answer.** If the layer spans indexes `first..last` inclusive, the top edge has `last - first + 1` cells, but the rightmost of those is the top of the right edge, which is moved as part of the corner cycle that starts further left. The number of cycles is `last - first`, which is the edge length minus one. `for i in first .. last-1` is that count. On a 2×2, `last - first = 1`, one cycle, four cells, done.

### 3. Show the transpose-then-reverse method and the order bug.

**Answer.** Swap `matrix[r][c]` with `matrix[c][r]` for all `c > r`, then reverse each row. On `[[1,2],[3,4]]`, transpose produces `[[1,3],[2,4]]`, and reversing rows produces `[[3,1],[4,2]]`, which is clockwise. Reverse-then-transpose on the same input produces `[[2,4],[1,3]]` after the row reverse and `[[2,1],[4,3]]` after the transpose, which is counterclockwise. The order is the difference between the two directions.

### 4. What is the extra memory, and do language-level row reverses violate it?

**Answer.** A correct solution uses `O(1)` extra memory. An in-place row reverse uses a few swaps, so it stays `O(1)`. Building a new row with slicing allocates `O(n)` per row and `O(n^2)` overall if you are not careful to write back into the same list; in Python, `matrix[r].reverse()` is in place and is fine. A second matrix is the thing the problem forbids.

### 5. How do you test a 1×1, a 2×2, a 3×3, and the inner ring of a 4×4 quickly?

**Answer.** 1×1 is unchanged because `n/2 = 0` layers. 2×2 is the single cycle in the walkthrough, `[[1,2],[3,4]] → [[3,1],[4,2]]`. 3×3 moves the ring and leaves the center. 4×4 must move both the outer ring and the inner 2×2; if the inner four cells are untouched, the layer loop ran only once. One assertion per size is enough before you talk about complexity.

### 6. Can the same cycle rotate by 180 or 270?

**Answer.** 180 swaps each cell with the opposite cell: `(r, c)` with `(n-1-r, n-1-c)`, and you do each pair once so you do not swap back. 270 clockwise is one counterclockwise cycle, which is the reverse assignment order of the four-cycle above, or three clockwise cycles. Do not run the 90-degree function three times in an interview unless you say you are trading simplicity for three passes; it is still `O(n^2)` and still in place, but the direct cycle is cleaner.
