# 23. Remove Nth Node From End of List — Interview Q&A

### 1. Complexity

**Q:** Compare the one-pass gap method with "count the length, then walk again." Is either asymptotically better?

**A:** Both are `O(L)` time and `O(1)` extra memory. The two-pass version walks about `L` nodes to count, then about `L - n` nodes to reach the predecessor. The one-pass version walks `n + 1` plus `L - n` steps, which is also about `L + 1`. The difference is a constant factor and the fact that one pass never stores `L`. Neither needs a stack. Recursion that deletes on the way back is `O(L)` time and `O(L)` stack, which is strictly worse on memory.

### 2. Invariant

**Q:** After the opening sprint and at every later step, what is the invariant?

**A:** `fast` is exactly `n + 1` nodes ahead of `slow`, counting along `next`, and both started on the dummy so the dummy counts as a position. Equivalently, the node `n` steps in front of `slow` is the node `fast` was on one step ago — the deletion target once `fast` has become null. Moving both pointers one step preserves the distance. When `fast == null`, `slow.next` is the nth node from the end. The dummy is what makes this true even if that node is the original head: `slow` can be the dummy and `slow.next` is still a real node.

### 3. Off-by-one

**Q:** You loop `for (i = 0; i < n; i++)` from the dummy, then move until `fast` is null, then unlink `slow.next`. On `1 -> 2 -> 3 -> 4 -> 5` with `n = 2`, which node do you delete, and which did you want?

**A:** You wanted `4`. The short sprint leaves `fast` on `2` (two hops: `1`, then `2`) with `slow` on dummy. Synchronized walk: `(fast, slow)` goes `(3, 1)`, `(4, 2)`, `(5, 3)`, `(null, 4)`. Unlink `4.next`, which drops `5`. You deleted the last node, i.e. the 1st from the end, because the gap was `n` rather than `n + 1`. The predecessor of the 2nd-from-end is `3`, and a gap of 3 from the dummy is what parks `slow` there. The fix is the bound `i < n + 1`, not a special case after the fact.

### 4. Follow-up

**Q:** `n` might be larger than the length, or you must remove the nth node from the *start* as well. What changes?

**A:** If `n` can exceed the length, the opening sprint must stop if `fast` becomes null, and you return the list unchanged (or throw, if that is the contract). Do not assume the LeetCode guarantee outside LeetCode. Removing the nth from the start is a single pointer: walk `n - 1` steps from a dummy and unlink. Removing every nth node, or the nth from each end in one pass, is a different problem; the reusable piece is still "dummy plus a measured gap," not the specific number `n + 1`.

### 5. Recursion vs iteration

**Q:** How does a recursive delete-from-the-end work, and why is the pointer version better here?

**A:** Recurse to the tail. The base case returns 0 or 1 as "nodes below me." On the way back, if the count of nodes after this one is `n`, you are the predecessor... actually the frame whose returned depth equals `n` is the target, and the caller skips it by returning `node.next` instead of `node` after stitching. It works, and the call stack implicitly stores the predecessors you wished you had. Cost is `O(L)` stack and a null-head base case that is easy to mishandle when `n` equals the length. The dummy two-pointer loop is the same predecessor search with `O(1)` memory and an obvious loop bound. Prefer it.

### 6. Null cases

**Q:** One node and `n = 1`. Two nodes and `n = 2`. What is null, and what must not be null?

**A:** One node, `n = 1`: dummy's next is that node. `fast` walks two steps and is null. `slow` stays on dummy. `dummy.next = node.next`, which is null. Return null. Two nodes `1 -> 2`, `n = 2`: `fast` walks three steps to null, `slow` stays on dummy, `dummy.next` becomes `2`. Return `2`. After a correct gap, `slow` is non-null and `slow.next` is the target, also non-null, because `n` is in range. `slow.next.next` may be null; assigning it is how you delete the tail. The original `head` reference may point at a deleted node; the caller must use the returned head.
