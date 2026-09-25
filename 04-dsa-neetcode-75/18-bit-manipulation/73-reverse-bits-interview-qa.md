# Reverse Bits — Interview Q&A

### 1. Why is the loop bound 32 rather than “until `n` becomes 0”?

**Answer.** The input is a 32-bit word, and zeros above the highest 1 are part of that word. Reversing `0...0001` must produce `1000...0`, which is `1 << 31`, not `1`. Stopping when `n` is 0 never shifts those leading zeros into the low end of the result. A counted loop of 32 copies every position, zero or not.

### 2. What is wrong with `n >> 1` in Java, and when does `>>>` matter?

**Answer.** `>>` is an arithmetic shift: it fills the vacated high bit with a copy of the old sign bit. After bit 31 of `n` has been shifted down into a low position and you keep going, the high side becomes all ones, and `n & 1` starts reading those ones as if they were original bits. `>>>` fills with zeros, which is what an unsigned 32-bit word does. It matters as soon as the original bit 31 is 1. For inputs that stay non-negative for all 32 shifts, the two operators match, but you should still write `>>>`.

### 3. The Java method returns a negative `int`. Did the reverse fail?

**Answer.** Not by itself. Java has no unsigned 32-bit return type in this signature, so a reversed word with bit 31 set is a negative `int` and the bit pattern is the answer. That happens exactly when the original word was odd, because the original bit 0 moves to bit 31. Print it with `Integer.toUnsignedString(result, 2)` if you want to see the bits without the sign confusing you.

### 4. What does the Python mask accomplish, and what do you return for the known sample?

**Answer.** `n &= 0xFFFFFFFF` throws away any stray high bits so you reverse a 32-bit quantity. Masking `result` each step keeps it inside 32 bits even if the loop bound were wrong. Thirty-two shifts from 0 already fit in 32 bits, so the output mask is defensive. The public sample `43261596` (`00000010100101000001111010011100`) reverses to `964176192` (`00111001011110000010100101000000`). That pair is the check to run.

### 5. How do you reverse by swapping pairs instead of rebuilding?

**Answer.** For `i` from 0 to 15, read bit `i` and bit `31 - i`, and if they differ, flip both. You can flip a bit with XOR against `(1 << i)`. Do each pair once or you swap back. This makes the permutation obvious and is also 16 constant-time iterations. The rebuild loop is shorter to write; the swap is a good second explanation. Both must use a width of 32, and Java still wants an unsigned treatment of the high bit when you construct `1 << 31` (that literal is already the sign bit; shifting a `1` into it is fine, comparing it is where signedness bites).

### 6. Is a 256-entry byte table worth mentioning?

**Answer.** Yes, as a follow-up. Split the word into four bytes, reverse the bits inside each byte with a table of 256 answers, and place the reversed byte 0 into byte 3 of the result, byte 1 into byte 2, and so on. Still `O(1)` time, with 256 bytes of table. It is the same algorithm with a wider “take the next chunk” step. Do not lead with it unless they ask how to go faster than 32 shifts.
