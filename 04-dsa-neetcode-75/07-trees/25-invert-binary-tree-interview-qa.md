# 25. Invert Binary Tree — Interview Q&A

### 1. Complexity

**Q:** Time and extra memory for the recursive invert and the BFS invert. Do you allocate `n` new nodes?

**A:** Both visit each node once, so time is `O(n)`. Recursive extra memory is the call stack, `O(h)`, which is `O(n)` on a skewed tree. BFS extra memory is the queue, `O(w)`, up to `O(n)` on a bushy level. The in-place swap allocates no tree nodes. A version that constructs a new node per call is still `O(n)` time but `O(n)` heap memory for the copy, on top of the stack.

### 2. Invariant

**Q:** What is true when `invertTree(node)` returns?

**A:** The returned reference is `node` (or null). Every node in that subtree has had its left and right children exchanged, including nodes that were originally in the left subtree and are now reachable through the right pointer, and vice versa. Nodes outside this subtree are untouched. That is why the root call inverts the whole tree and why a recursive call on a child is allowed to finish before the parent swaps: the child's internal mirror is already done, and the parent swap only changes which side that mirrored subtree hangs on.

### 3. Off-by-one

**Q:** This problem has no index. Where is the analogous fencepost mistake?

**A:** Swapping only down to depth `d - 1`, or looping `for` a level and also swapping children that were already swapped when they were pushed. In the BFS code, you swap a node when you pop it, exactly once. If you swap when you push and also when you pop, every node except the root is swapped twice and ends up back where it started. That is the tree version of an off-by-one: one extra swap undoes the work. Recursively, calling invert on the children after you have swapped, and also having swapped inside the children, is fine — that is one swap per node. Calling the parent swap twice is not.

### 4. Follow-up

**Q:** Check whether one tree is the mirror of another, or invert an n-ary tree. What changes?

**A:** Mirror-check is the same recursion as "same tree" with the children crossed: `a.left` compares to `b.right` and `a.right` compares to `b.left`, plus equal values and matching nulls. You do not need to invert a copy first. An n-ary invert reverses the children list of every node and recurses into each child. Time stays linear in the number of nodes. If the tree is threaded or has parent pointers, you update those too; a plain `TreeNode` does not have them.

### 5. Recursion vs iteration

**Q:** Show that the queue version visits each node once and never swaps twice. Why might you still prefer recursion in the interview?

**A:** Each node is offered at most once, by its parent, when the parent has just swapped and is enqueueing the new children. The root is offered once at the start. Each poll swaps that node exactly once. Children are read after the swap, so you enqueue the post-swap children, but those children have not been swapped themselves yet; their own swap happens when they are polled. Recursion is shorter and matches "mirror(left), mirror(right), swap." I write the recursive version first. I switch to the queue if the interviewer asks for an explicit traversal or worries about a skewed stack.

### 6. Null cases

**Q:** Null root, a root whose left is null and right is a leaf, and a leaf. Any chance you call `.val` on null?

**A:** Null root returns null immediately. A leaf swaps two nulls and returns itself. A root with only a right child: after the recursive calls, left is null and right is the inverted child; the swap puts the child on the left and null on the right. The code reads `.left` and `.right` only after the null check on the current node. It never reads `.val` at all, so a value of `0` is irrelevant. Missing children are null references, not nodes with a special value.
