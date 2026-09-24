# Unique Paths — Interview Q&A

### 1. Why is the recurrence `above + left` and not something that includes the diagonal?

**Answer.** A move enters `(i, j)` from directly above or directly left. There is no single move from `(i-1, j-1)`. Routes that passed through the diagonal are already inside the above-count or the left-count, so adding the diagonal again would double-count them.

### 2. What are the answers for a single row, a single column, and a 1×1 grid?

**Answer.** All of them are 1. A 1×1 grid needs zero moves. A single row is only rights, in one fixed order. A single column is only downs. The base-case loops cover this, and the nested loops start at 1 so they do not run.

### 3. How do you get the memory down to one row without changing the answer?

**Answer.** Keep an array `row` of length `n`, initially all ones (the first row). For each later grid row, scan `j` from 1 to `n-1` and set `row[j] = row[j] + row[j-1]`. Before the assignment, `row[j]` is still the old value from the previous grid row, which is “above,” and `row[j-1]` is already this row’s left neighbor. Scanning right to left would replace “above” before you used it.

### 4. Can you solve it with combinatorics, and why might an interviewer still prefer DP?

**Answer.** Yes. Any path is a sequence of exactly `m-1` downs and `n-1` rights, so the count is `C(m+n-2, m-1)`. You compute that with a multiplicative loop that reduces fractions as you go. DP is the default in an interview because the same table extends immediately to obstacles, costs, or blocked cells, while the closed form does not.

### 5. How would the DP change if some cells were blocked?

**Answer.** A blocked cell stores 0 and contributes nothing to its neighbors. You can no longer seed an entire first row with ones: a blocked cell on the border, and everything beyond it in that border, is unreachable. The recurrence for an open cell stays `above + left`.

### 6. What is the time and space, and where is the overflow risk?

**Answer.** `O(mn)` time, `O(mn)` space for the full table, `O(n)` space for one row. LeetCode 62’s signature returns a 32-bit integer and the tests stay inside that range, so adding two `int` cells is safe under the problem contract. If `m` and `n` were both near 100 with no such promise, `C(198, 99)` would overflow `int` and you would switch to a big integer or `long` only if the new bound actually fits.
