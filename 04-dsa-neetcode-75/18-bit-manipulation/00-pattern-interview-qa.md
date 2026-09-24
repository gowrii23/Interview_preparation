# Bit Manipulation — Interview Q&A

### 1. What does XOR give you that addition does not, and what does it throw away?

**Answer.** On each bit, XOR is the sum modulo 2. It throws away the carry, which is exactly the bits that were 1 in both inputs. Those shared bits are `a & b`, and the carry belongs one position to the left, so `(a & b) << 1`. Repeating “XOR, then add the carry” is ordinary addition. `x ^ x = 0` and `x ^ 0 = x` are why XOR also finds a missing or a single unique number: every paired value cancels.

### 2. Why does the same adder terminate in Java and spin in Python unless you mask?

**Answer.** A Java `int` is 32 bits. A carry that shifts out of bit 31 disappears, and `b` becomes 0 within 32 iterations. A Python `int` grows. For any negative pattern the top bits are an infinite string of ones, so there is always another column where both bits are 1 and the carry never runs out of word. `value & 0xFFFFFFFF` after every XOR and every shift pretends the word is 32 bits wide. That is not optional polish; it is what makes the loop finite.

### 3. How do you turn a masked 32-bit pattern back into a Python signed integer?

**Answer.** If the pattern is at most `0x7FFFFFFF`, bit 31 is clear and the number is already the right non-negative value. If it is larger, bit 31 is set and the two’s-complement value is `pattern - 2^32`. `~(pattern ^ 0xFFFFFFFF)` computes that: XOR with the mask flips the low 32 bits, and Python’s `~x = -x - 1` finishes the conversion. Check `0xFFFFFFFF → -1` and `0x80000000 → -2147483648` before you trust it.

### 4. What is `n & (n - 1)`, and what is `n & -n`?

**Answer.** Subtracting one flips every trailing 0 to 1 and the first 1 to 0. AND with `n` therefore clears the lowest set bit and leaves the others. Isolating that bit is `n & -n`, because negation in two’s complement flips bits and adds one, which rebuilds only the lowest 1. Hamming weight can loop on `n &= n - 1` and count the iterations. It is `O(k)` in the number of ones and it works for the all-high-bit pattern, since each pass removes one 1 and eventually hits 0.

### 5. When is a mask required besides the Python adder?

**Answer.** Whenever the problem promises a fixed width and the language does not have one. Reverse-bits must run exactly 32 steps so high zeros in the input become low zeros in the output; masking the input to 32 bits first protects you if a Python caller passed a wider integer. Counting bits and missing-number XOR on non-negative values do not need a mask. Java’s unsigned `>>>` is the local mask for “shift in zeros from the left” when the sign bit is set.

### 6. What overflow does Gauss have that XOR does not, on the missing-number problem?

**Answer.** The sum `0 + 1 + ... + n` is `n * (n + 1) / 2`. In Java, if you multiply two `int`s first, `n` near `2^16` already overflows before the division and the subtraction. You can cast to `long`, or you can XOR `0..n` together with every `nums[i]`. XOR never adds magnitudes, so the only bits in the accumulator are the bits of the missing value. Prefer XOR in the interview unless they ask for the formula, and if you show the formula, show the wider type.
