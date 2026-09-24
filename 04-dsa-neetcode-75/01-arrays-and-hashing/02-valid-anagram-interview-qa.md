# 242. Valid Anagram — Interview Q&A

## 1. What is the complexity of the count solution versus sorting?

The count array is `O(n)` time and `O(1)` extra space for a fixed 26-letter alphabet. Sorting both strings is `O(n log n)` time and `O(n)` space for the copies Timsort or `Arrays.sort` on objects needs. The 26-slot scan at the end is constant work. Unicode with a hash map is expected `O(n)` time and `O(k)` space for distinct characters.

## 2. Why increment one string and decrement the other in one array?

Both strings must describe the same multiset. Addition records what `s` supplies. Subtraction records what `t` consumes. A zero array means supply equals consumption for every letter. Two separate arrays also work; one array uses half the bookkeeping and makes the final check a search for a nonzero slot. A hash set of characters is wrong because it forgets duplicates: `"aab"` and `"abb"` would look similar if you only stored presence.

## 3. What bug does this code have?

```java
if (s.length() != t.length()) return false;
int[] count = new int[26];
for (int i = 0; i < s.length(); i++) {
    count[s.charAt(i) - 'a']++;
}
return true;
```

It counts `s` and ignores `t` after the length check. `"ab"` and `"cd"` have the same length and would return true. Another bug is using the array for uppercase or punctuation: `'A' - 'a'` is negative and throws. A third bug is returning true when any slot is zero instead of when every slot is zero.

## 4. Follow-up: how would you find every anagram of `p` inside a longer string `s`?

That is the sliding-window anagram problem. Keep a window of length `p` and the same 26-count difference. As the window moves, decrement the character that leaves and increment the character that enters, and test the zero-array condition in `O(1)` or `O(26)`. Brute force that sorts every window is `O(n k log k)`.

## 5. What if the constraints change and strings can hold any Unicode code point, with `n` up to 10^5?

Drop the 26-array. Use a hash map from character to count, still adding `s` and subtracting `t`. Time is expected linear. Space grows with distinct characters, which is at most `n` and at most the Unicode set you actually see. Sorting remains correct and may be simpler to code under time pressure; state the `O(n log n)` trade.

## 6. What if one string is empty, or both strings are equal already?

Two empty strings are anagrams: length 0, and the count loop never runs, so the array stays zero. A string is an anagram of itself because counts match even though no rearrangement is required. Do not special-case identical strings; the count check already accepts them. Case sensitivity matters: under the usual problem statement `"a"` and `"A"` are different characters.
