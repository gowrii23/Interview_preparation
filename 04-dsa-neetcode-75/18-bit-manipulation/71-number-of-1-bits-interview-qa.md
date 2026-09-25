# Number of 1 Bits — Interview Q&A

### 1. Why does `n & (n - 1)` clear exactly one bit?

**Answer.** If the lowest set bit of `n` sits at position `k`, then `n` looks like `...xxx100..0` with `k` zeros. `n - 1` borrows through those zeros and looks like `...xxx011..1`. AND keeps the `xxx` prefix, because those bits match, and clears bit `k` and everything below it. Nothing above `k` changes. One 1 disappears. Repeat until none remain.

### 2. What does the Java loop return for `0`, for `1`, and for a value with only bit 31 set?

**Answer.** `0` never enters the loop and returns 0. `1` clears itself in one step and returns 1. Bit 31 alone is `Integer.MIN_VALUE`, which is negative. `while (n != 0)` still runs. `n - 1` is `Integer.MAX_VALUE`, the AND is 0, and the count is 1. `while (n > 0)` would have returned 0, which is wrong.

### 3. How does the “test bit 0, then shift” version stay correct for a negative Java int?

**Answer.** You cannot use arithmetic `>>`, which fills from the left with the sign bit and never reaches 0. Use a counted loop of 32 and an unsigned `>>>`, or test `(n >>> i) & 1` for `i` from 0 to 31. Kernighan’s loop avoids the shift question entirely: it only uses subtraction and AND, and it reaches 0 after `k` steps even when the sign bit is the bit you clear.

### 4. What is the complexity, and when is the fixed 32-step loop better?

**Answer.** Kernighan is `O(k)` bit operations, worst case 32. The fixed loop is always 32. They are both `O(1)` on this problem’s word size. The fixed loop has more predictable timing. Kernighan is faster for sparse words and is the one that shows you understand the lowest-set-bit trick. Extra memory is a single counter either way.

### 5. Is Python’s unlimited integer a problem for this specific function?

**Answer.** Not on the problem’s inputs, which fit in 32 bits and are non-negative. `n &= n - 1` strictly reduces a non-negative `n` until it hits 0, so the loop ends. If someone passed a negative Python int, the two’s-complement pattern is infinite ones and the loop would not mean “32-bit weight” anymore. Mask once with `0xFFFFFFFF` if you need that contract. You do not mask inside the loop the way the adder must, because this loop is shrinking, not shifting a carry outward.

### 6. How would you count 1 bits for every integer from 0 to n, using this idea?

**Answer.** That is Counting Bits. You could call this function on each `i`, which is correct and `O(n)` with a bigger constant. The tighter recurrence is `bits[i] = bits[i >> 1] + (i & 1)`: the population of `i` is the population of `i` without its last bit, plus that last bit. Each value is then `O(1)` after the previous ones, still `O(n)` overall, with a very small constant and no inner loop.
