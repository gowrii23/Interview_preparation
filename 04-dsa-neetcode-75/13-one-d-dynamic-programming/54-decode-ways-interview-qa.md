# 54. Decode Ways — Interview Q&A

## 1. Why this state?

`dp[i]` counts decodings of a prefix of length `i`. A decoding is a sequence of cuts, and the last cut has length 1 or 2, so every decoding of the prefix is an extension of a decoding of a shorter prefix. Two decodings that reach the same index with a valid parse are interchangeable for the suffix. You do not store the letters. Zero is not a second state variable; it is a cut you refuse. If you defined the state as “ways from index `i` to the end,” that is the same recurrence pointed backward, and the answer sits at index 0 instead of index `n`.

## 2. How do you optimize space?

The cell reads only `dp[i - 1]` and `dp[i - 2]`, so two integers replace the array. Time stays `O(n)`, extra memory `O(1)`. Keep a third local `cur` so you do not overwrite `prev1` before the pair term has used the old `prev2`. A backward recursion with memo is `O(n)` memory because of the map and the stack. Prefer the forward roll unless the interviewer asks for top-down.

## 3. Variant: a character may be `'*'`, meaning 1..9, or you must return the decodings modulo `10^9 + 7`. What changes?

Decode Ways II widens the single-digit and two-digit cases into counts of interpretations, not just yes/no. A `'*'` as a single contributes 9 times `dp[i - 1]`. A pair containing a star contributes as many values in 10..26 as the pattern allows (`'*'` in the ones place after a `'1'` is 9, after a `'2'` is 6, two stars is 15, and so on). The state is still “ways for this prefix.” Take every product modulo `10^9 + 7` if they ask, including the additions. Do not modulo only at the end if the language wraps a 32-bit int. The zero rules remain: a concrete `'0'` is still not a single letter.

## 4. Which strings are zero, one, or more than one?

`"0"`, `"06"`, `"30"`, `"100"` are 0. `"10"` and `"27"` are 1 (`27` cannot be a pair). `"12"` is 2. `"226"` is 3. `"11106"` is 2. A useful spoken check: a `'0'` must be the second digit of a `10` or a `20`, and it consumes the previous digit so that previous digit cannot also be used as a single. If that pair is illegal, the whole string is 0 from that point forward unless an earlier split saves it, which it cannot once the zero is orphaned.

## 5. What bug does this code have?

```java
if (s.charAt(i) != '0') cur += prev1;
int two = Integer.parseInt(s.substring(i - 1, i + 1));
if (two <= 26) cur += prev2;
```

`Integer.parseInt("01")` is `1`, and `1 <= 26` is true, so a pair that starts with zero is treated as a letter. On `"101"` the last step adds both the single `'1'` and the fake pair `"01"`, returning 2. The only real decoding is `10|1`, so the answer is 1. The pair test has to be `two >= 10 && two <= 26`. A second, related bug is checking only `two > 0`, which still accepts `01` through `09`.

## 6. Why does the empty prefix have one way?

`dp[0] = 1` means “there is one way to decode nothing,” which is the start of every non-empty decode. It is not a claim that the empty string was part of the input. Without it, a two-digit string such as `"12"` cannot count the split that takes both digits at once, because that split stands on the empty prefix. `"10"` would also become 0, which is wrong. If the whole input is empty, the classic problem does not ask; a guard on a leading zero is the case that matters.
