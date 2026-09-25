# Spiral Matrix — Interview Q&A

### 1. Why are the bottom and left walks conditional, while the top and right walks are not?

**Answer.** The loop condition already guarantees a non-empty rectangle, so the top row from `left` to `right` exists, and after `top` advances there is still a well-defined right column to attempt — it may be empty if that advance crossed `bottom`, and the `for` loop simply does not run. The bottom row is a second visit to the same row when only one row was left, and the left column is a second visit to the same column when only one column was left. Those two need an explicit “still in bounds after the shrink” test.

### 2. Walk a 1×4 and a 4×1 out loud. Where do the guards fire?

**Answer.** 1×4: the top walk emits all four cells and `top` becomes 1, which is past `bottom`. The right walk’s row range is empty. `top <= bottom` is false, so the bottom walk is skipped and does not replay the row. The left walk may still be entered, but its downward range starts at `bottom` and stops before `top`, so it adds nothing. 4×1: the top walk emits the first cell, the right walk emits the rest of the column, and `right` becomes `-1`. The bottom walk is entered because a row range remains, but its column range is empty because `right < left`, so it writes nothing. The left guard `left <= right` then fails, so the column is not climbed again. Both shapes emit each cell once, top-to-bottom or left-to-right.

### 3. What does Python’s `range(right, left, -1)` get wrong?

**Answer.** `range` excludes the stop value. `range(right, left, -1)` stops before `left`, so the bottom-left corner of this layer is missing and the later left-column walk may pick it up in the wrong order or not at all. The inclusive walk is `range(right, left - 1, -1)`. The same off-by-one exists on the way up: `range(bottom, top - 1, -1)`.

### 4. How does a visited-cell simulation differ, and when is it acceptable?

**Answer.** Keep a direction index and a visited matrix, or mark visited in place if the cell type allows a sentinel. Move forward; when the next step would leave the board or hit a visited cell, turn clockwise. The dotted “already visited” check is the turn condition. It is easier to invent under pressure and easier to get an off-by-one on the turn. It costs `O(mn)` extra memory. Use it as a fallback, then offer the four-bound peel if they ask for constant extra memory.

### 5. What is the length of the output, and how do you know you did not stop early or loop forever?

**Answer.** The output length must equal `m * n`. Every iteration of the loop increments `top` before the next check, so the top bound moves even when the other three walks add nothing. A skipped bottom or left walk does not have to move its bound; the while-condition `top <= bottom and left <= right` still fails once a strip has been consumed. The guards are what stop a consumed row or column from being appended twice. A useful test is `len(order) == m * n`, with the first row equal to the first `n` outputs.

### 6. Can you generate an `m × n` matrix filled in spiral order with the same idea?

**Answer.** Yes, that is the sibling problem. Start from the same four bounds and write `1..m*n` along the sides instead of reading. The same two guards stop you from overwriting a one-row or one-column core. Time stays `O(mn)`, extra memory is the board you were asked to build.
