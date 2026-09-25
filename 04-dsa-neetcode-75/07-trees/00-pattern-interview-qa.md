# Tree patterns — Interview Q&A

### 1. Complexity

**Q:** A recursive DFS visits every node once and does `O(1)` work. Why do people still say space can be `O(n)`? How is BFS different?

**A:** Time is `O(n)` because each node is entered once and each edge is followed once. The recursion stack holds one frame per ancestor, so the extra memory is `O(h)`, the height. A linked-list-shaped tree has `h = n`, so the worst-case stack is `O(n)` even though a balanced tree uses `O(log n)`. BFS stores a frontier. On a perfect tree the last level has about `n / 2` nodes, so the queue is `O(n)` in the worst case and `O(w)` in general. Neither structure's memory is automatically `O(1)`. If you need `O(1)` extra memory you are usually in the iterative Morris / parent-pointer world, which these problems do not require.

### 2. Invariant

**Q:** What invariant do exclusive BST bounds maintain that a parent comparison does not?

**A:** On entry to a node, every value that is still legal in this subtree lies strictly between `low` and `high`, and every ancestor constraint has already been folded into those two numbers. The left child may use values in `(low, node.val)` and the right child may use `(node.val, high)`. A parent-only check keeps a weaker invariant: "I am on the correct side of my parent," which says nothing about grandparents. The node `4` under `6` under `5` satisfies `4 < 6` and still violates `4 > 5`, and the bound invariant catches it because `6`'s left subtree inherited `low = 5`.

### 3. Off-by-one

**Q:** Your BST test is `val < low || val > high`, and you pass `high = node.val` to the left child. What duplicate or boundary bug is that?

**A:** The bounds are documented as exclusive (`low < val < high`) but the comparison treats them as inclusive. A left child equal to its parent passes `val < high` when `high` is the parent and the test uses `>=` incorrectly... specifically: if you reject only with strict `< low` and `> high`, then `val == high` is accepted. In a BST, a child equal to the bound (the parent value) is not allowed under the usual strict ordering. The off-by-one is an equality on the boundary. Use `<= low` or `>= high` as failure when the parent value was passed as the bound. The same class of bug is iterating a level with `for (i = 0; i <= size; i++)` and popping one node from the next level.

### 4. Follow-up

**Q:** The interviewer changes "binary tree" to "n-ary tree," or asks for the lowest common ancestor of a general binary tree rather than a BST. What survives?

**A:** DFS and BFS survive; children become a list. The level-size snapshot still marks a level. BST bounds and inorder-sorted reasoning do not survive, because there is no order. The general LCA (LC 236) is a postorder: if a subtree contains both targets, the answer is already decided below you; if the left subtree contains one and the right contains the other, you are the LCA; if you are one of the targets and the other side finds the second, you are the LCA. You no longer branch left or right by comparing values. Say that explicitly so you do not force the BST shortcut onto a general tree.

### 5. Recursion vs iteration

**Q:** When do you rewrite a tree DFS as an explicit stack, and when is recursion the answer you want?

**A:** Recursion is the right default when the answer is a function of the two children and the height is expected to be logarithmic, or when `n` is the usual interview size. It matches the definition and it is harder to mix up preorder and postorder if you place the work in the obvious spot. Switch to an explicit stack when the tree can be skewed and the platform has a small stack, or when you must pause the traversal (kth smallest can stop after k pops). BFS is already iterative; do not recurse "level by level." Inorder kth-smallest is a good iterative stack because you can return as soon as the counter hits k and you never build the full list.

### 6. Null cases

**Q:** How should null roots and null children behave for depth, equality, BST validation, and serialization?

**A:** A null root has depth 0, is a valid BST, serializes to a single null marker, and is equal only to another null root. A null child is not a node you visit; it is the base case that stops the recursion. Equality fails when one side is null and the other is not, even if you never read `.val`. BST validation returns true on null without consulting the bounds. Do not treat the integer 0 or `Integer.MIN_VALUE` as null. Those are real keys, which is why bounds should be `long` or nullable rather than "the smallest int means no bound."
