# Arrays and Hashing — Interview Q&A

## 1. What are the time and space bounds of the hash-index pattern, and why?

Expected time is linear: one pass over `n` items, and each insert or lookup is expected constant time. Space is linear in the number of distinct keys you keep. A 26-slot count array is constant extra memory because the alphabet does not grow with `n`. Hashing a string key costs the length of that string, so grouping `n` strings of length `k` is `O(nk)` with a count signature, not `O(n)`. Worst-case hash degradation is `O(n)` per operation; say “expected `O(1)`” unless the interviewer asks about adversarial keys.

## 2. Why a hash map instead of sorting or a nested loop?

A nested loop checks every pair and is `O(n^2)`. Sorting puts equals or complements next to each other and is `O(n log n)`, which is the right backup when a hash table is forbidden, but it reorders indices and does extra comparison work. The map answers “have I seen the partner?” at the moment you see the current element, so you never revisit earlier elements. The structure matches the question: membership, index, or frequency, not order.

## 3. What bug does this two-sum sketch have?

```text
map.put(nums[i], i);
if (map.containsKey(target - nums[i])) return ...;
```

The insert happens before the lookup. When `2 * nums[i] == target`, the element finds itself. The fix is to look up the complement first, and only then store the current index. The same bug appears as “count this character before comparing two strings of equal length” if you accidentally count both strings into one array in the wrong direction.

## 4. Follow-up: the input can contain many duplicates and you must return every index pair. What changes?

Keep a map from value to a list of indices, or scan with a frequency map and then emit combinations. Time stays proportional to `n` plus the number of pairs you output. Do not claim `O(n)` if the output itself is quadratic. Still insert a value only after you have paired it with earlier indices, so you do not pair an index with itself.

## 5. What if the constraints change and `n` is 10^7 while values span the full 32-bit range?

A hash map still works and uses memory proportional to distinct values. A boolean array indexed by value does not, because the value domain is huge. If they also forbid extra memory beyond a few words, you must sort a copy (or sort in place if order can be destroyed) and solve with two pointers, accepting `O(n log n)` time. If they only need existence of a duplicate, a set is enough; you can return as soon as an insert fails and you do not need indices.

## 6. How do you choose between a set, a map of counts, and a map of lists?

- Set: the only fact is “seen or not” (duplicate detection, consecutive-sequence starts).
- Map of counts: the fact is “how many” (anagrams, top-k, sliding-window frequencies).
- Map of lists: the fact is “who belongs with this signature” (group anagrams).

State that choice in one sentence before coding. If two designs both work, prefer the one whose key is the thing the problem asks you to match.
