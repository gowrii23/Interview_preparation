# 32. Kth Smallest Element in a BST — Interview Q&A

### 1. Complexity

**Q:** Justify `O(h + k)` rather than always `O(n)`. What is the memory, and what changes with subtree-size augmentation?

**A:** Each node is pushed and popped at most once. The algorithm pops exactly `k` nodes and then returns, so at most the ancestors of those nodes plus the nodes themselves are pushed. Reaching the minimum costs `O(h)` to push the left spine; each later successor costs amortized `O(1)` stack operations across the `k` pops. A clean bound is `O(h + k)` time and `O(h)` stack memory. When `k = n` or `h = n` this is `O(n)`, so the worst case over inputs is linear. Augmenting every node with `leftSize` costs `O(n)` preprocessing and `O(n)` memory, after which each query compares `k` to `leftSize + 1` and descends one child, `O(h)` time, and updates along the insertion path stay `O(h)` if the tree is otherwise a normal BST.

### 2. Invariant

**Q:** What is true about the stack and about `k` just before you pop?

**A:** The stack, from bottom to top, is a chain of ancestors whose left subtrees are not yet finished, and every node still in the stack has a value greater than every node already popped. The nodes already popped are the `visited` smallest values, in order, and `k` is the original k minus `visited` (in the code, `k` has already been reduced by the previous pops). The next pop is the smallest value not yet emitted, because its entire left subtree has been processed — that is why you pushed the left spine before popping. When the decremented `k` hits 0, the just-popped value is the original kth. Nodes in right subtrees you have not entered are all larger than the current pop, so skipping them when you return is safe.

### 3. Off-by-one

**Q:** You increment a counter after the pop and return when `count == k`, but you seeded `count = 1` before any pop. For `k = 1` and `k = 3` on the sample tree, what do you return?

**A:** The first pop is the minimum, value `1`, and you set `count` to 2 if you increment from a seed of 1, so `count == k` fails for `k = 1`. You continue and return a later node. For `k = 3` you return the 2nd smallest, because the counter is one ahead of the number of pops. The seed and the test disagree by one. Either start `count` at 0 and return when it reaches `k` after incrementing, or decrement the input `k` and return when it hits 0. Do not do both. Also, a full-list approach must index with `k - 1`.

### 4. Follow-up

**Q:** Find the kth largest, or support insert and delete between queries.

**A:** Kth largest is inorder from the right: push the right spine, pop, then go left. Or run ordinary kth-smallest with `k' = n - k + 1` if you know `n`. With updates, store `leftSize` on each node (or subtree size). Insert and delete update the counts on the search path in `O(h)`. The query walks from the root using those counts and never scans `k` nodes, which matters when `k` is large and queries are frequent. If the tree must stay balanced you need a balanced BST; the counting trick does not by itself limit `h`. I mention augmentation only after the inorder solution is solid.

### 5. Recursion vs iteration

**Q:** Recursive inorder with a shared counter. Can it return immediately, and what does it cost compared with the stack?

**A:** `int walk(node)`: if node is null or k is already consumed, return the answer you stored. Recurse left. If k hits 0 during the left call, return. Decrement k, and if it is 0 store `node.val` and return. Otherwise return the right recursion. It is correct. The call stack is `O(h)` and you do not start the right subtree once k is 0, but the frames above the kth node still unwind, so the latency includes that unwind. The iterative version returns from `kthSmallest` at the pop, which is the same number of nodes touched and easier to stop cleanly. I code the iterative one. I describe the recursive one if they ask "isn't inorder just recursion?"

### 6. Null cases

**Q:** Null root, `k = 1` on a single node, a node whose left is null so the node itself is the next smallest, and value 0.

**A:** A null root with a valid `k` cannot happen under the constraints; the loop does not run and the method returns the sentinel. A single node and `k = 1`: the inner while pushes that node, the left child is null, you pop it, decrement k to 0, and return its value. You never touch `.left` on a null current because the inner while guards `cur != null`. When a popped node has a null left, that was already handled by the spine walk: the pop happens because `cur` became null. Value 0 is a legal smallest element; do not use 0 as a "not found" answer. The `-1` sentinel is safe only because this problem's statement says k is in range and, if values can be `-1`, you would rather throw than return a colliding sentinel.
