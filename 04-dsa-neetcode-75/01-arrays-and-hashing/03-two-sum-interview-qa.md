# 1. Two Sum — Interview Q&A

## 1. What are the time and space bounds, and how do they compare with sorting?

The hash scan is expected `O(n)` time and `O(n)` space. The nested loop is `O(n^2)` time and `O(1)` extra space. Sorting value-index pairs is `O(n log n)` time and `O(n)` space, then the two-pointer sweep is linear. Hashing wins when extra memory is allowed. Sorting wins when you must avoid hash tables or when you need values in order anyway.

## 2. Why store the index in the map instead of only the value in a set?

A set would tell you the partner exists and would not tell you where it is. The problem asks for indices. The map’s value is the earlier index. When several copies of a value exist, keeping the latest index is enough for the classic problem because any one valid pair is accepted. Overwriting the index is fine as long as the lookup happens first.

## 3. What bug does this code have?

```python
def twoSum(self, nums, target):
    index_of = {}
    for i, value in enumerate(nums):
        index_of[value] = i
        if target - value in index_of:
            return [index_of[target - value], i]
```

On input `[3, 2, 4]` and target `6` it still happens to work, but on `[3]` and target `6` it returns `[0, 0]` if that single element is checked after insert. On `[3, 3]` and target `6` the first element inserts itself and immediately returns `[0, 0]`, which reuses one index. Lookup must run before `index_of[value] = i`.

## 4. Follow-up: the array is already sorted. Do you still want a hash map?

No. Two pointers at the ends are enough: if the sum is too small, move the left pointer right; if too large, move the right pointer left. That is `O(n)` time and `O(1)` extra space, and it needs the sorted order. If you still need original indices, the array must have been sorted as pairs. This sorted variant is “two sum II.”

## 5. What if the constraints change and there may be zero or many answers, and values can overflow a 32-bit sum?

Return a list of pairs, and decide whether pairs must be unique by value or by index. Compute `need` in 64-bit arithmetic (`long` in Java) so `target - nums[i]` does not wrap. If “no solution” is allowed, return an empty result instead of assuming the loop always hits. If `n` is huge and memory is tight, an external sort of `(value, index)` uses less random-access memory than a hash map.

## 6. What if the same value must be used twice, or three numbers must sum to the target?

Using a value twice requires two distinct indices with that value. The lookup-before-insert order already supports `[3, 3]`. Three numbers is 3Sum: sort, fix one index, and run two pointers on the rest, skipping duplicates if the caller wants unique triplets. Do not nest a hash map three deep unless the constraints are tiny.
