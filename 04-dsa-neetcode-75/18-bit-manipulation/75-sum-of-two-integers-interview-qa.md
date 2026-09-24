# Sum of Two Integers — Interview Q&A

### 1. What do XOR and AND mean inside one bit of an adder?

**Answer.** XOR is the sum bit: 0+0 → 0, 0+1 → 1, 1+0 → 1, 1+1 → 0. AND is the carry out of that same column: it is 1 only for 1+1. The carry does not belong in this column; it belongs one position higher, so the word-level carry is `(a & b) << 1`. The next round adds that carry back in with the same two operations. When the AND is 0, XOR was already the full sum.

### 2. Why does two’s complement make the same loop correct for negative numbers?

**Answer.** A negative value is stored as the pattern you get by inverting the bits of its magnitude and adding one. Addition hardware does not have a separate “minus” path; it adds those patterns and ignores the carry off the end of the word. This loop is that hardware. `-1` is all ones. Adding `1` carries through every column, leaves zeros in the word, and the leftover carry falls off bit 31. The visible result is 0. `-2 + 3` similarly produces the pattern for `1`. You never branch on the sign.

### 3. Show why the unmasked Python loop does not terminate on `-1 + 1`.

**Answer.** Python’s `-1` has infinitely many leading ones. First round: XOR is `-2` (all ones except the low bit), AND is `1`, carry is `2`. Next round the low bits of `-2` and `2` produce XOR `-4` and carry `4`. Each round the carry is a single bit one position further left, and it always lands on another 1 from the infinite sign extension, so the carry never becomes 0. Masking both the XOR and the carry with `0xFFFFFFFF` cuts the word at 32 bits. Within at most 32 rounds the carry shifts out of that window and the loop stops at 0.

### 4. How do you recover a negative Python result, and what are the two identities to check?

**Answer.** After the masked loop, `a` is in `0 .. 4294967295`. Bit 31 clear means `a <= 0x7FFFFFFF` and the signed value is `a` itself. Bit 31 set means the signed value is `a - 2^32`. `~(a ^ 0xFFFFFFFF)` equals that difference: XORing the mask flips the low 32 bits, and Python’s `~x = -x - 1`. Identities: `0xFFFFFFFF` → `-1`, and `0x80000000` → `-2147483648`. Also check the loop-skipping case `getSum(-2, 0) == -2`, where `a` is still a native negative int and must not be run through the “pattern > 0x7FFFFFFF” conversion.

### 5. What is the Java loop condition, and what breaks if you write `b > 0`?

**Answer.** `while (b != 0)`. A carry can occupy bit 31, which makes the signed `int` negative, for example while computing a sum that overflows into the sign bit. `b > 0` then exits and drops that carry, returning a XOR that is not the sum. `!= 0` keeps going until the pattern is actually clear. Java needs no final sign fix because the `int` you have is already the wrapped two’s-complement result.

### 6. What is the iteration bound, and which bugs still use a forbidden `+`?

**Answer.** Each round shifts the carry left by at least one, so a 32-bit word needs at most 32 rounds. Time and extra memory are constant. Forbidden bugs: `carry = (a & b) + (a & b)` to simulate the shift, `a += carry`, or unary minus to flip a sign. `<< 1` is the legal double. Compute `carry` from the old `a` and `b` before you overwrite `a` with `a ^ b`; if you XOR first, the AND no longer sees the original pair unless you saved them.
