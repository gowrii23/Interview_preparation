# 31. Validate Binary Search Tree — Interview Q&A

### 1. Complexity

**Q:** Time and stack for the bounds DFS. How does "collect inorder into an array and check it is sorted" compare?

**A:** Bounds DFS visits each node once: `O(n)` time, `O(h)` stack, `O(1)` extra heap memory. Collecting the inorder sequence is `O(n)` time and `O(n)` extra memory for the list, then a linear scan. The streaming inorder (keep only `prev`) is `O(n)` time and `O(h)` stack with `O(1)` extra heap, same as bounds. None of these is `O(log n)` unless the tree is known valid and you are searching; validation must be ready to look at every node because any node could be the one that breaks.

### 2. Invariant

**Q:** State the invariant of `valid(node, low, high)` and show why the illegal `4` fails it.

**A:** If `valid` returns true, `node`'s subtree is a BST and every value in it lies strictly inside `(low, high)`. The root call uses a window that contains every `int`, so the only constraints are the ones the tree itself creates. At `5`, the right subtree is called with `(5, +inf)`. At `6`, the left subtree is called with `(5, 6)`. The call on `4` requires `5 < 4 < 6`, which is false, so the function returns false and the invariant's premise ("if it returns true") does not apply. A parent-only check never builds the lower bound `5` for that grandchild, so it has no invariant that mentions ancestors.

### 3. Off-by-one

**Q:** The failure test is `node.val < low || node.val > high`, and the left child is called with `high = node.val`. What duplicate do you accept, and what sentinel bug is the sibling of this mistake?

**A:** A left child equal to its parent has `val == high` and is not rejected by strict `<` / `>`. Duplicates must be invalid. The fix is `<=` and `>=` against exclusive bounds. The sibling bug is seeding an `int` window at `Integer.MIN_VALUE` and `Integer.MAX_VALUE` and then rejecting `val <= low` for a tree whose only node is `Integer.MIN_VALUE`: the sentinel collides with a legal key, which is an off-by-one at the edge of the type. Widen the window to `long`, or use null as "no bound" and only compare when the bound object is non-null.

### 4. Follow-up

**Q:** The interviewer allows duplicate keys on the right, or asks you to repair the tree. What changes in the bound test?

**A:** If duplicates are legal only on the right, the right child may inherit `low = val` but the comparison becomes `val < low` is bad and `val > high` is bad, while `val == low` is allowed on the right and `val == high` is still bad on the left. Say the policy before you code; the LeetCode policy is "no duplicates." Repairing is not a local swap: a single bad node can need to move across an ancestor, so the honest repair is to dump a sorted unique inorder and rebuild a balanced BST. Do not "swap the offending child upward" and claim the tree is fixed.

### 5. Recursion vs iteration

**Q:** Iterative inorder validation versus recursive bounds. When do you pick each?

**A:** Recursive bounds are the clearest statement of the ancestor rule and the one I code by default. Iterative inorder uses an explicit stack, walks left, and compares each popped node to `prev`. I pick it when the tree may be skewed and recursion depth is a concern, or when I am already writing iterative inorder for the kth-smallest problem and want one traversal. Both are `O(n)` time. The iterative version's bug is forgetting to set `prev` after the comparison, or seeding `prev` inside the int range. I do not unroll the bounds recursion into a stack of `(node, low, high)` triples unless asked; it is the same algorithm with more noise.

### 6. Null cases

**Q:** Null root, a node with a null left and a real right, a node equal to `Integer.MIN_VALUE` with a right child, and a null bound versus a null child.

**A:** Null root returns true immediately: an empty tree is a BST. A null child returns true and does not tighten anything further. A node may have only a right child; the left call is the null base case and the right call still carries `low = node.val`. `Integer.MIN_VALUE` as a key is legal when the low bound is `Long.MIN_VALUE`, and its right child must be strictly greater. Do not treat a null child as value 0. Do not treat a missing bound as 0 either: 0 can be a key, so "no bound" has to be a null `Integer` or an out-of-range `long`, not the number zero.
