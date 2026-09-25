# 347. Top K Frequent Elements — Interview Q&A

## 1. Why is bucket sort `O(n)` here, and when is the heap slower?

A frequency is an integer from 1 to `n`, so one array indexed by frequency orders the values without comparisons. Counting, filling, and scanning that array are all linear in `n` plus the number of distinct keys. A binary heap of the top `k` costs `O(log k)` per distinct key, so `O(n log k)` after the count. The heap uses less auxiliary structure when you only store `k` entries, but it is not linear unless `k` is constant.

## 2. Why index buckets by frequency instead of putting values in a tree map?

The rank key is a small integer, which is exactly what counting sort and bucket sort are for. A balanced tree of frequencies adds a `log n` factor you do not need. The hash map is still required first, because equal values are scattered through the array; the bucket array cannot count them until identical values have been collapsed.

## 3. What bug does this code have?

```python
buckets = [[]] * (len(nums) + 1)
for value, f in freq.items():
    buckets[f].append(value)
```

The multiplication copies one list reference `n + 1` times. Every append lands in every bucket, so the downward scan returns the same values many times and can overflow `k` with duplicates. Build a new list per index. Another bug is sizing the array to `max(freq)` without counting, or sizing it to `k`, which is too small when one value appears more than `k` times.

## 4. Follow-up: return the `k` most frequent words, ties broken by alphabetical order. What changes?

Bucket sort alone does not order ties. Inside each bucket, sort the words, and read buckets from high frequency to low, taking words in alphabetical order. Time becomes `O(n log n)` in the worst tie. A heap of `(-freq, word)` also works if the word order matches the tie rule. The uniqueness promise of the number problem is gone, so say the tie break out loud.

## 5. What if the constraints change and `n` is 10^7, `k` is 10, and several values share the same frequency?

The problem’s “unique answer” promise may be dropped. Agree on a tie break (smaller value, or any). The heap of size `k` is attractive because `log k` is tiny and you do not allocate `n` bucket lists. Bucket sort is still linear and fine if memory for `n + 1` list heads is acceptable. If frequencies can exceed a 32-bit counter, use a 64-bit count; with `n` at 10^7 a 32-bit int still holds the count.

## 6. What if `k` equals the number of distinct values, or the array is all one value?

If `k` equals the number of distinct values, the answer is every distinct value. The bucket scan simply drains every non-empty bucket. If every element is the same, one bucket at index `n` holds that single value, and `k` must be 1. Do not return the value `n` times; the map already collapsed copies into one key.
