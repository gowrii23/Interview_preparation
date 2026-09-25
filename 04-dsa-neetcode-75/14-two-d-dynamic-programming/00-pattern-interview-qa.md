# Two-Dimensional Dynamic Programming — Interview Q&A

### 1. How do you decide a problem is 2D DP instead of plain recursion or BFS?

**Answer.** The input is a grid or a pair of sequences, and the answer for a larger piece is a constant-time combination of answers for smaller pieces that overlap heavily. Recursion still describes the same recurrence, but without a table you recompute the same `(i, j)` many times. BFS fits when you need a shortest path in an unweighted graph of states, not when every state is a numeric combination of two neighbors (path counts, LCS length, min edit cost).

### 2. What is the state, and why does the fill order matter?

**Answer.** For a grid, `dp[i][j]` is the answer using rows `0..i` and columns `0..j`. For two strings, `dp[i][j]` is the answer using `a[0..i)` and `b[0..j)`. The recurrence only reads strictly smaller `i` or `j`, so nested loops that increase both indexes are a valid topological order. If you walk the wrong way you read an uninitialized or already-overwritten neighbor and the corner value is meaningless.

### 3. How do base cases differ between grid counting and LCS?

**Answer.** A grid path count seeds the first row and first column with 1, because there is exactly one way to walk along a single edge of the grid. LCS seeds a zero row and a zero column: the LCS of any string with the empty prefix is 0. Mixing those up is the usual bug — a path grid cannot start at 0, and an LCS table cannot start at 1.

### 4. When can you shrink the table to one dimension, and when is that unsafe?

**Answer.** You can keep one row (or one column) when each cell needs only the previous row and earlier cells in the current row, as in unique paths. You must update that row from the side that still holds the “above” value you are about to use. LCS can also roll to two rows, or one row if you stash the diagonal before you overwrite it. If the recurrence needs an arbitrary earlier row, the full table stays.

### 5. What do you say when the interviewer asks for the actual subsequence, not just the length?

**Answer.** The length table is enough to rebuild one subsequence. Start at `(m, n)`. If the characters match, prepend that character and step to `(i-1, j-1)`. If they differ, step to whichever neighbor stored the same max (`up` or `left`). That walk is `O(m + n)` after the `O(mn)` fill. Keep the full table if reconstruction is required; a single rolling row throws the parent pointers away.

### 6. What complexity do you quote, and what overflow trap do you mention?

**Answer.** Time and extra memory are both proportional to the number of states, `O(mn)`. Each state is `O(1)` work. For path counts, say that Java’s `int` matches the problem’s contract that the answer fits in 32 bits; if the interviewer removes that promise, switch the cell type to a 64-bit integer. For LCS the length is at most `min(m, n)`, so overflow is not the issue — index off-by-one is.
