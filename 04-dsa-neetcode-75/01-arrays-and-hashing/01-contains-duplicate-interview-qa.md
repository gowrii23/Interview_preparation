# 217. Contains Duplicate — Interview Q&A

## 1. What is the complexity, and why is it not always the early-exit time?

Worst-case time is expected `O(n)` because a duplicate-free array hashes every element. Space is `O(n)` for the same reason: the set holds every distinct value. Early exit helps only when a repeat appears before the end. Sorting is `O(n log n)` time and `O(1)` extra memory if you sort in place, which is the comparison-based lower bound for “are any two equal?” when you cannot hash.

## 2. Why a hash set instead of a hash map or a sort?

The only fact you need is presence. A map of counts works but stores an integer you never read. A map of indices is for two-sum, not for a yes/no duplicate check. A set’s `add` both records the value and reports whether it was new, which matches the question in one operation.

## 3. What bug does this code have?

```java
boolean[] seen = new boolean[1000];
for (int value : nums) {
    if (seen[value]) return true;
    seen[value] = true;
}
```

Values may be negative or larger than 999, so this throws or misses duplicates. A hash set has no value-domain assumption. Also, comparing `nums[i] == nums[i + 1]` without sorting only catches adjacent duplicates.

## 4. Follow-up: return the value that repeats, and there is exactly one. What changes?

The same set works: return the value when `add` fails. If you must return every duplicated value once, keep inserting and add the value to an answer list on the first collision only, so later copies of the same value are not reported again. Still `O(n)` expected time.

## 5. What if the constraints change and you must use `O(1)` extra memory, and the array may be modified?

Sort in place and scan once for `nums[i] == nums[i - 1]`. Time becomes `O(n log n)`. If the array is immutable and extra memory is forbidden, you cannot do better than a quadratic scan in the general comparison model. If values are bounded by a small `R`, a boolean array of size `R` is acceptable and uses `O(R)` memory.

## 6. What if the array contains one element, or the duplicate is the first and the last element?

One element cannot repeat, so return false. The first and last elements being equal is still a duplicate; the set remembers index 0 when it reaches the end. Do not assume duplicates are adjacent. An empty array is false. Integer overflow is irrelevant because you never add the values.
