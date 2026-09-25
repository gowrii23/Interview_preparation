# 49. Group Anagrams — Interview Q&A

## 1. What is the complexity of the count-key approach versus sorting each word?

Count keys: `O(nk)` time, because each of `n` words of length up to `k` is scanned once and the 26-length signature is built in constant time relative to `k`. Sort keys: `O(nk log k)` time. Extra space beyond the output is the keys and the hash table, still `O(nk)` in the worst case because the groups store every character. Output size is part of the space bound; you cannot group the strings without writing them somewhere.

## 2. Why is the map key a count signature instead of the first string in the group?

You need to recognize an anagram before you know which group it joins. The first string is an arbitrary representative and comparing against it is `O(k)` only after you have a candidate group; finding that group naively is quadratic. The signature is a canonical name for the multiset, so the hash map jumps straight to the group. A tuple or a delimited string is hashable; a raw `int[]` in Java is not a value key, because arrays hash by identity.

## 3. What bug does this code have?

```java
String key = "";
for (int c : count) key += c;
```

Counts `1` and `11` concatenate to the same characters as `11` and `1`. `"ab"` (one a, one b → `1` and `1`) can also collide with other small patterns depending on encoding, and the missing separator is the classic bug. Another bug is sorting the word and using that sorted string as the value you return, which destroys the original spelling the caller expects.

## 4. Follow-up: group strings that are anagrams only after you may change at most one character. What changes?

The strict signature no longer defines the groups, because “one edit from an anagram” is not an equivalence relation you can hash in one key (it is not transitive in a simple way). For small `k`, generate all signatures within one count change and union the strings, carefully so you do not merge too much. State that the clean hash grouping relied on exact equality of counts.

## 5. What if the constraints change to Unicode strings and `n = 10^5`, `k = 100`?

A 26-array is wrong. Use a map from character to count, then freeze it as a sorted list of pairs for the key, which is `O(k log k)` per word in the worst case when every character is distinct. Memory grows with distinct characters per word. If they only want groups of size at least 2, still build the full map and drop singleton lists at the end; that filter is linear in the number of groups.

## 6. What if the input contains empty strings or the same string many times?

All empty strings share the all-zero signature and form one group. Identical strings are anagrams of each other and belong in the same group; you append every occurrence, you do not deduplicate unless the interviewer asks. A single string is a group of one. Do not skip length 0; `computeIfAbsent` still creates the list.
