# Missing Number — Interview Q&A

### 1. Why does XORing `0..n` with every array element leave exactly the missing number?

**Answer.** XOR is associative, commutative, and its own inverse: `x ^ x = 0`, and `0` is the identity. In the combined expression each present number appears twice, once from the ideal range and once from the array, so those pairs collapse to 0. The missing number appears once. The whole expression reduces to `0 ^ missing`, which is the missing number. Order of the array does not matter.

### 2. Where does `n` enter the XOR, and what bug appears if you leave it out?

**Answer.** The indexes only cover `0..n-1`. The legal values also include `n`, and `len(nums)` is `n`. Seed the accumulator with `nums.length` before the loop, or XOR it once afterward. If you forget, and the missing number is `n`, every other value cancels and you return 0, which is wrong whenever 0 is actually present. Example: `[0, 1]` must return 2, not 0.

### 3. Show the Gauss formula and the Java overflow.

**Answer.** `missing = n*(n+1)/2 - sum(nums)`. The algebra is exact. In Java, `n` and the sum are often `int`. The multiplication `n*(n+1)` overflows 32 bits for large legal `n` (values up to the usual `10^4` bound are safe, a follow-up bound near `10^5` is not if you are careless, and `2^31` overflow starts once the product exceeds about `2.1e9`, i.e. `n` around `65536`). Compute the product in `long`, or skip the issue and XOR. Python’s `int` does not overflow, so the formula is safe there; still mention the Java trap.

### 4. What do you return for `[0]`, `[1]`, and an empty array if it were allowed?

**Answer.** `[0]` has `n = 1` and contains 0, so the hole is 1. The loop does `1 ^ 0 ^ 0 = 1`. `[1]` has `n = 1` and contains 1, so the hole is 0: `1 ^ 0 ^ 1 = 0`. An empty array would mean `n = 0` and the only possible value 0 is missing; the loop does not run and the seed `0` is already the answer. The usual constraint is `n >= 1`, but the code happens to handle `n = 0`.

### 5. How is this different from “every element appears twice except one”?

**Answer.** The single-number problem XORs only the array: pairs cancel and the unique element remains. Here the array has no duplicates at all. You manufacture the pairs yourself by also XORing the complete range `0..n`. If you only XOR the array, nothing cancels and you get a useless mix of every present value. The idea “XOR cancels pairs” is shared; the source of the pairs is not.

### 6. Can you do it with constant memory if XOR is disallowed, and what do you give up?

**Answer.** Yes, the sum formula with a wide integer is constant extra memory and linear time. In-place cyclic sort (put value `v` at index `v` while `v < n`, then scan for the index whose value is not `i`) is also linear time and constant extra memory, but it mutates the array and the index logic around `n` is easier to get wrong. Sorting is simpler and slower, `O(n log n)`. XOR is the clean constant-space answer that leaves the array unchanged.
