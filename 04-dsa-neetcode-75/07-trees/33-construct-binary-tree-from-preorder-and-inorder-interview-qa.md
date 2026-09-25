# 33. Construct Binary Tree from Preorder and Inorder — Interview Q&A

### 1. Complexity

**Q:** Why is the map version linear, and where does the quadratic version spend the extra time? What is the stack cost on a left-skewed tree?

**A:** Each of the `n` nodes is created once and performs a constant-time hash lookup, and the map is built in one `O(n)` pass. Total time `O(n)`. Extra heap memory is `O(n)` for the map; the output tree is separate. If instead you scan the current inorder window to find the root, a left-skewed tree (every node is the rightmost... actually a left spine: inorder is the nodes in reverse of preorder) makes the root sit at the end of a window of length `n, n - 1, ...`, so the scans cost `O(n^2)`. The recursion depth equals the number of nodes on a spine, so a skewed tree uses `O(n)` stack even with the linear map. Balanced input uses `O(log n)` stack. Slicing Python lists on every call is another quadratic copy even if the search itself is a map.

### 2. Invariant

**Q:** What is true of the preorder cursor and the inorder window at the start of `build`?

**A:** The current window `inorder[inLeft..inRight]` is exactly the set of values in the subtree being built, in inorder. The next `inRight - inLeft + 1` unused preorder values are exactly that subtree in preorder, and the cursor points at the first of them (the subtree root), unless the window is empty. Building the left child consumes precisely the next `mid - inLeft` preorder values, because that is the size of the left window. Therefore when the left call returns, the cursor points at the root of the right window. The invariant is why left-before-right works and why you must not consume a value when `inLeft > inRight`.

### 3. Off-by-one

**Q:** The base case is `if (inLeft >= inRight) return null`. What tree do you build for preorder `[1, 2]` and inorder `[2, 1]`? What about a one-node tree?

**A:** A one-node tree calls `build(0, 0)`. `0 >= 0` returns null, so you build nothing and, if you returned before reading preorder, you also leave the only value unconsumed. You should have produced a leaf. For `[1, 2]` / `[2, 1]`: root should be `1` with left child `2`. The top call has window `[0, 1]`, which is not `>=`, so you take `1`, mid is `1`, left window is `[0, 0]`. That call hits `0 >= 0` and returns null instead of the leaf `2`. The right window `[2, 1]` is empty and would have been null anyway. You return a bare `1` and never read `2`. The fix is `inLeft > inRight`. Equal indices are one element, a real node, whose child windows are empty.

### 4. Follow-up

**Q:** You are given inorder and postorder (LC 106), or preorder and postorder. What still works?

**A:** Inorder plus postorder: the root is the last value of the postorder range, not the first of preorder. Split inorder the same way, but build the right subtree before the left if you consume postorder from the end, because postorder emits left, right, then root — walking backward, you see the root, then the right subtree, then the left. The map and the window invariant stay. Preorder plus postorder is not enough for a unique binary tree when nodes can have a single child: you cannot tell a left child from a right child without null markers or a convention. If the problem promises every node has 0 or 2 children, the split sizes become determined and you can rebuild. Do not claim preorder and postorder always uniquely define a binary tree.

### 5. Recursion vs iteration

**Q:** Can you rebuild with an explicit stack? Why is the recursive window the version to write first?

**A:** Iterative preorder rebuild: the stack holds the path of ancestors waiting for a right child. You create the next preorder node and attach it to the left as long as its inorder index is still left of the parent; when the next inorder value says you have finished a left subtree, you pop until you find the ancestor that should own the next node as a right child. It is `O(n)` and `O(h)` and easy to get the pop condition wrong. The recursive version is the definition: next preorder value, split the window, left window, right window. I write that. I mention the stack only if they ban recursion. Either way the map (or the next inorder value you expect) is what tells you the window boundary.

### 6. Null cases

**Q:** Both arrays empty, a window that is empty on the left only, and a missing value that is not in the map. How do null children appear in the finished tree?

**A:** Empty arrays: the first call is `build(0, -1)`, `inLeft > inRight`, return null. You must not read `preorder[0]`. A node with no left child is an empty left window (`mid - 1 < inLeft`); that call returns null and does not advance the cursor, so the next preorder value becomes the right child. There is no explicit null in the input arrays; null children are exactly the empty windows. If a preorder value were absent from the map, `get` would return null and unboxing would throw. Under the problem constraints every value appears once, so that throw means the arrays do not describe the same tree. Do not treat value `0` as null. `0` is a legal node value and a fine hash key.
