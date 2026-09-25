# 125. Valid Palindrome — Interview Q&A

## 1. What is the complexity of the two-pointer version versus building a cleaned string?

Both are `O(n)` time. The pointer version is `O(1)` extra space. The cleaned string stores up to `n` characters, so it is `O(n)` extra space, and reversing it with a slice copies those characters again. The inner skip loops do not change the bound: across the whole scan, `left` and `right` still move at most `n` times total.

## 2. Why two pointers instead of reversing the string and comparing?

Reversing the raw string fails because punctuation and case sit in different places: `"ab"` reversed is `"ba"`, and `"A,b,a"` reversed is not equal to itself even though the letters are `a b a`. You would still have to clean first, then reverse. Pointers compare the cleaned sequence from both ends without building it. The structure matches “read inward.”

## 3. What bug does this code have?

```java
while (left < right) {
    if (!Character.isLetterOrDigit(s.charAt(left))) left++;
    else if (!Character.isLetterOrDigit(s.charAt(right))) right--;
    else if (Character.toLowerCase(s.charAt(left)) != Character.toLowerCase(s.charAt(right)))
        return false;
    else { left++; right--; }
}
```

This version is actually careful. The bug to avoid is the single-if form that always increments `left` and decrements `right` after a skip check that only moved one side, which can skip a real character or compare a letter to punctuation. Another bug is `s.charAt(left) == s.charAt(right)` without case folding, so `"Aa"` returns false. A third is using `||` in the skip condition so a pointer runs off the end when the other side is already alphanumeric.

## 4. Follow-up: you may delete at most one character and still want a palindrome. What changes?

That is valid palindrome II. On the first mismatch, branch: check the inside range with the left character removed, or with the right character removed. Do not delete more than once. The helper is the same two-pointer compare with no extra skips beyond the original alphanumeric rule, if that rule still applies. Time stays `O(n)` because each branch scans the string at most once.

## 5. What if the constraints change and the string can be 10^7 Unicode characters, and case folding must be locale-aware?

`O(n)` time and `O(1)` extra space still holds if you can classify and case-fold a code point in `O(1)`. Locale-aware case folding is not always one code unit (German ß). Then you cannot compare `char` to `char`; normalize to a sequence of folded code points, which may use `O(n)` memory. Ask whether ASCII is enough before you take that on. Indexing by UTF-16 units in Java can split a surrogate pair; iterate code points if the input is real Unicode.

## 6. What if the string is empty, or it is a single digit surrounded by commas?

Both are palindromes. The pointers cross immediately, or they land on the same digit and stop, so you return true without a failing comparison. Do not require a letter. Do not return false for the empty cleaned sequence. A string of one alphanumeric character has nothing to mismatch with.
