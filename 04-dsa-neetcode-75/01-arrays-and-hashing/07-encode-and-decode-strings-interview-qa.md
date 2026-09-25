# 271. Encode and Decode Strings — Interview Q&A

## 1. What is the complexity, including the cost of building strings?

Both directions are `O(L)` time and `O(L)` space, where `L` is the total number of characters. In Java, `StringBuilder` appends are amortized constant per character. In Python, `"".join` is linear, while repeated `encoded += chunk` can copy the growing string and become quadratic. Each decoded slice copies its payload once; that copy is the output, so it is necessary work, not an extra hidden factor beyond `O(L)`.

## 2. Why a length prefix instead of a delimiter or an escape character?

Any single delimiter character can appear inside a string, so `split` cannot know which copies are real. Escaping works (double every delimiter, then wrap fields) but the decoder must scan and un-escape, and you have to prove the escape is reversible. A length prefix lets the decoder jump: read a number, take that many characters, repeat. The payload is opaque. The `#` only terminates the header because lengths are written in decimal digits, which do not include `#`.

## 3. What bug does this code have?

```python
def decode(self, s: str) -> list[str]:
    return s.split("#")
```

`"a#b"` encodes as `3#a#b`. Splitting on `#` yields `["3", "a", "b"]` instead of `["a#b"]`. Another bug is encoding with `word.length()` after converting to a byte array while decoding by UTF-16 code units, so the count and the cursor disagree. A third bug is using `indexOf('#')` from the start of the string on every chunk, which finds an earlier mark or a mark inside a previous payload if you forgot to move the start.

## 4. Follow-up: strings are raw bytes that may include any byte, including the byte of `#` and digits. What changes?

Keep the same frame on a byte buffer: write a 4-byte big-endian length, then that many bytes. You no longer need `#` at all, because the length field has a fixed width. Decimal plus `#` also still works if `#` is only the header terminator and the length is a character count, not a “read until hash” of the payload. Fixed-width headers avoid parsing digits.

## 5. What if the constraints change and a single string can be longer than `Integer.MAX_VALUE`, or the encoded blob can be truncated?

A Java `String` cannot be that long; you would switch to a chunked reader and a 64-bit length (`long`). On truncation, `start + length` can run past the buffer. Check the bounds and throw or return an error instead of slicing off the end. Also reject a length that does not parse. The interview problem assumes `encode`’s output is what `decode` receives, so those checks are defensive.

## 6. What if the list is empty, or it contains one empty string between two words?

An empty list encodes to `""` and decodes to `[]` because the cursor loop never runs. `["ab", "", "c"]` encodes to `2#ab0#1#c`. The middle `0#` appends an empty string and leaves the cursor on `1`. Forgetting to allow length 0 would stick the cursor on `#` or merge the next word. Test that case before you finish.
