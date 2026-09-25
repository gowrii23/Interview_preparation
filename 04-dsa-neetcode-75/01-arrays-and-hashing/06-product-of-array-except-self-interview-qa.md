# 238. Product of Array Except Self — Interview Q&A

## 1. What are the time and space bounds, and does the output array count?

Time is `O(n)` because each index is written a constant number of times. Extra memory is `O(1)`: one running suffix and a few indices. The returned array is `O(n)` and is not counted as extra space under the usual follow-up. A prefix array plus a suffix array is still `O(n)` time but `O(n)` extra memory, which fails that follow-up while remaining a correct explanation.

## 2. Why is the running product enough, and why is division the wrong tool?

`answer[i]` depends only on elements strictly left and strictly right of `i`. Those two ranges are prefixes and suffixes, which a scan can maintain. Division needs a total product and then `total / nums[i]`. One zero makes the total zero and every quotient zero, including the slot that should exclude that zero. Two zeros make every answer zero, which division can get right only with extra zero-counting logic the problem told you not to use.

## 3. What bug does this code have?

```java
int right = 1;
for (int i = n - 1; i >= 0; i--) {
    right *= nums[i];
    answer[i] *= right;
}
```

The suffix includes `nums[i]` because the multiply into `right` happens before it is applied. Every answer is then the product of a prefix and a suffix that both include `i`, which is the full-array product. Swap the two lines. Another bug is initializing `answer[0]` to `nums[0]` instead of 1, which inserts the excluded element into every later prefix.

## 4. Follow-up: also return the product of every element except the two neighbors, or handle an array that contains zeros with division allowed. What changes?

If division becomes allowed, count zeros. Zero zeros: divide the total by `nums[i]`. One zero: every answer is 0 except at the zero’s index, which is the product of the rest. Two or more zeros: every answer is 0. For “except a window of neighbors,” adjust the excluded range; prefix and suffix products still combine as `prefix[L] * suffix[R]`.

## 5. What if the constraints change and products can exceed 64-bit integers, with `n` up to 10^5?

Java `long` is not enough. Use `BigInteger`, and accept a higher cost per multiplication as integers grow. Python already has unbounded integers; say that each multiplication can cost more than `O(1)` as the bit length grows, so the practical time is `O(n * M(b))` where `b` is the bit size of the running product. If they only want the answer modulo a prime, multiply modulo that prime and use modular inverses, which brings division back in a form that works with zeros only if you still special-case them.

## 6. What if the array has one element, or it contains a negative number?

One element: there is nothing else to multiply, and the empty product is 1, so return `[1]`. Negatives need no special case; the running product carries the sign. Two negatives in the excluded region make a positive contribution. Zero still dominates any sign. Do not take absolute values.
