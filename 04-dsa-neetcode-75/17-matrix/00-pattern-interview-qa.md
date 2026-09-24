# Matrix Patterns — Interview Q&A

### 1. How many layers does an `n × n` rotation have, and what happens to the center?

**Answer.** `n // 2` layers. Layer `k` is the ring whose top-left is `(k, k)` and whose bottom-right is `(n-1-k, n-1-k)`. Each ring is rotated onto itself. When `n` is odd the center cell is not on any ring, and a 90-degree turn leaves it in place. You do not write a special case; the loop bound `layer < n/2` simply never visits it.

### 2. Why does the spiral’s bottom pass and left pass need extra bound checks?

**Answer.** The top pass and the right pass always shrink `top` and `right`. On a one-row matrix the top pass consumes that row and `top` moves past `bottom`. A bottom pass would walk the same row backward. On a one-column matrix the right pass consumes the column and `right` moves past `left`; a left pass would walk it again. Guarding the bottom walk with `top <= bottom` and the left walk with `left <= right` drops those empty or already-consumed sides.

### 3. Why can’t `matrix[0][0]` mark both the first row and the first column?

**Answer.** Those two facts are independent. A zero somewhere in row 0 means the whole first row must be cleared, including cells whose columns are otherwise fine. A zero somewhere in column 0 means the whole first column must be cleared. One shared cell cannot store both bits once you start writing zeros into it. Read both facts into booleans before the marker pass, use `matrix[i][0]` for row `i` and `matrix[0][j]` for column `j` on the interior, and apply the two booleans last.

### 4. Transpose then reverse each row versus a four-cycle. When do they differ?

**Answer.** They do not differ in the result. Transpose swaps `(r, c)` with `(c, r)`, and reversing each row sends that cell to column `n-1-c`, which is the clockwise image. The four-cycle writes the same permutation with one temporary scalar and one pass over the rings. Transpose is easier to remember; the cycle is easier to generalize to “rotate a ring of a rectangle” and matches the layer story. Both are `O(n^2)` time and `O(1)` extra memory. Reversing rows before the transpose is the counterclockwise rotation; do not mix the order.

### 5. What extra memory do you admit to, and what do you refuse?

**Answer.** A few integers and booleans: loop bounds, one saved cell, two zero flags. The output list of a spiral is required by the signature and is not “extra” workspace. You refuse a second `n × n` board for rotation and a second `m × n` board of flags for zeroes, unless you are deliberately showing the `O(m+n)` marker-array version first and then compressing it.

### 6. What is the bug if the rotation inner loop uses `i <= last`?

**Answer.** The four cells of a cycle are already in their final places when you have stepped `last - first` times, not `last - first + 1`. One more step rotates the same cycle again and then again, and four extra quarter-turns bring the ring back to the start, so a full extra cycle is a no-op only if you do it four times. A single extra index rotates every group once too often and the board is wrong. The legal offsets are `0, 1, ..., width-2`, i.e. `i` from `first` inclusive to `last` exclusive.
