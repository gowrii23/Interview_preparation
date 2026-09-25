# Counting Bits — Interview Q&A

### 1. Why is `ans[i >> 1]` already known when you are filling `ans[i]`?

**Answer.** `i >> 1` is `floor(i / 2)`, which is strictly less than `i` for every `i >= 1`. A loop that increases `i` has already written every smaller index, including that parent. The recurrence is a tree of right-shifts rooted at 0, and increasing order is a valid topological order. Going downward would read zeros and return a table of last-bits only.

### 2. What does the formula do for an even `i` and for an odd `i`?

**Answer.** Even: the last bit is 0, so `ans[i] = ans[i / 2]`. Adding a trailing zero does not change the population. Odd: the last bit is 1, so the population is one more than `i >> 1`, which is the number you get by clearing that trailing one. Examples: `4` (`100`) copies `ans[2] = 1`; `5` (`101`) is `ans[2] + 1 = 2`.

### 3. There is a second recurrence `ans[i] = ans[i & (i - 1)] + 1`. Why is it also `O(n)`?

**Answer.** `i & (i - 1)` clears the lowest set bit, so it is strictly smaller than `i` and already stored. Every positive `i` has at least one set bit, so you add exactly 1 to a previous answer. The dependency still points at a smaller index, and one left-to-right pass works. It is another way to say “remove one 1, look up the rest.” The shift form is simpler to explain; both are linear.

### 4. What is `ans` for `n = 0` and for `n = 1`?

**Answer.** `n = 0` allocates a one-element array and the loop does not run: `[0]`. `n = 1` writes `ans[1] = ans[0] + 1 = 1`, so `[0, 1]`. These are the base of the recurrence. An off-by-one in the array length shows up immediately here: a length-`n` array cannot hold index `n`.

### 5. Why not call the Hamming-weight function on every `i`?

**Answer.** It is correct. On 32-bit words it is also `O(n)` time. The interviewer’s follow-up is a tighter linear pass that reuses earlier answers instead of rescanning bits. The DP does one addition and one shift per integer. Mention the popcount version as the baseline, then write the recurrence. Extra memory is the output either way.

### 6. Does this need a 32-bit mask in Python or Java?

**Answer.** No. `n` is non-negative, every `i` in `0..n` is non-negative, and `i >> 1` and `i & 1` do not overflow and do not depend on wraparound. Java’s arithmetic shift and logical shift agree on these values because the sign bit is clear. The mask conversation belongs to reverse-bits and to add-without-plus, not to this table.
