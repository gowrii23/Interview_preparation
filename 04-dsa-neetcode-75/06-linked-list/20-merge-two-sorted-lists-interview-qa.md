# 20. Merge Two Sorted Lists — Interview Q&A

### 1. Complexity

**Q:** Why is the merge linear, and why is extra memory constant even though the output has `n + m` nodes?

**A:** Every node is attached exactly once, and each attachment is a pointer write, so time is `O(n + m)`. The output nodes are the input nodes with new `next` pointers. Beyond the dummy and `tail`, you allocate nothing, so auxiliary memory is `O(1)`. If the interviewer forbids mutating the inputs, you allocate `n + m` new nodes and copy values; time stays linear and extra memory becomes `O(n + m)`.

### 2. Invariant

**Q:** What is true about `tail` and the two remaining suffixes at the start of each loop?

**A:** The list from `dummy.next` through `tail` is sorted and contains exactly the nodes already consumed. Every remaining node in `a` and in `b` is greater than or equal to `tail.val` (or the result is still empty). Both remaining suffixes are themselves sorted. The loop attaches the smaller of the two suffix heads, which is the next value in the merged order, and the invariant holds again.

### 3. Off-by-one

**Q:** The loop is written `while (a.next != null && b.next != null)`. What breaks?

**A:** You stop when either list has one node left, before that node is compared. The final splice then attaches one entire leftover, including a node that might be larger than the other list's last node. Example: `1 -> 4` and `2 -> 3`. You might attach `1`, then see `a.next` still non-null and `b` at `2`, attach `2`, then `b` is `3` and `b.next` is null so you stop and splice `4` in front of a leftover `3`, producing `1 -> 2 -> 4 -> 3`. The loop condition has to allow a list that still has its last node, i.e. test the pointers themselves, not their `next` fields.

### 4. Follow-up

**Q:** Merge k sorted lists. Do you call this routine, or do you need a new idea?

**A:** This routine is the combine step. Divide and conquer: pair the k lists, merge each pair, and repeat. There are `log k` rounds and each round touches every node, so time is `O(N log k)`. A min-heap of the current head of each list is the same bound: each of the N nodes is pushed and popped once at `O(log k)`. Repeatedly merging list `i` into a growing result is `O(k N)` and is the solution to reject. See `24-merge-k-sorted-lists`.

### 5. Recursion vs iteration

**Q:** Write the recursive merge in one sentence and say why the loop is the one you ship.

**A:** If either list is null, return the other; otherwise the smaller head's `next` is the merge of the rest, and you return that head. That is correct and easy to say. Each call waits on a list that is one node shorter, so the stack is `O(n + m)`. The iterative dummy version does the identical stitch with `O(1)` extra memory, so it is the default answer. Use the recursive sentence only if the interviewer asks for the recurrence.

### 6. Null cases

**Q:** What do you return if `a` is null, if `b` is null, if both are null, and if both lists have one equal value?

**A:** `a` null: the loop does not run, `tail.next = b`, return `b`. `b` null: return `a`. Both null: `dummy.next` is null, return null. One node each with equal values: `<=` attaches `a` first, then the splice attaches `b`, so the result is `a` then `b`. No extra null check is required inside the loop because the `while` already guarantees both pointers are live before you read `.val` or `.next`.
