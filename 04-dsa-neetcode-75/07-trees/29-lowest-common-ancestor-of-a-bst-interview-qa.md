# 29. Lowest Common Ancestor of a BST — Interview Q&A

### 1. Complexity

**Q:** Why is the BST walk `O(h)` and not `O(n)`? What do you pay if you ignore the BST property and use the general-tree LCA?

**A:** Each comparison throws away an entire subtree, so you follow one root-to-split path. The length of that path is at most the height `h`. Balanced BST: `O(log n)` time and `O(1)` extra memory in the loop. Skewed BST: `h = n`, so `O(n)` time, which matches "you might have to walk the whole spine." The general LCA visits every node in the worst case because a target can be in either child with no ordering to consult, so it is `Θ(n)` time and `O(h)` stack even on a balanced tree. Using it here is correct and slower than necessary. Parent pointers plus two ancestor sets are `O(n)` memory and also ignore the order.

### 2. Invariant

**Q:** What is true about `cur` each time the loop condition is checked?

**A:** Both `p` and `q` lie in `cur`'s subtree. That starts true because both exist in the whole tree. If both values are less than `cur.val`, the BST property puts both in the left subtree, so stepping left preserves the invariant. Same for the right. When neither "both left" nor "both right" holds, `cur` is still a common ancestor, and no child of `cur` has both: one target is in the left subtree and the other in the right, or one target is `cur` itself. That is the definition of the lowest common ancestor. The invariant plus the exit test is the whole proof.

### 3. Off-by-one

**Q:** The go-left test is written `p.val <= cur.val && q.val <= cur.val`. On the diagram, `p` is node `2` and `q` is node `4`. Where do you end up?

**A:** At `6` both values are `< 6`, you go left, fine. At `2`, `p.val <= 2` and `q.val` is `4`, which is not `<= 2`, so you do not go left. You also do not go right, because `p.val > 2` is false. You happen to return `2`. So this particular pair still works. The failure is `p` and `q` both equal to a duplicated value, or more sharply: suppose the loop is `<=` and both targets are in the left subtree including a descendant that equals nothing — the real break is when one target equals `cur` and the other is less. Example: `cur` is `2`, `p` is `2`, `q` is `0`. `0 <= 2` and `2 <= 2`, so `<=` steps left and leaves `p` behind. You then search a subtree that does not contain `p` and return the wrong node or null. The comparison must be strict. Equality means "this node is one of the targets; stop."

### 4. Follow-up

**Q:** The tree is no longer a BST (LC 236). What is the algorithm, in short, and which piece of today's code do you throw away?

**A:** Throw away the value comparisons. A postorder on a general binary tree returns a node if that subtree contains `p` or `q`. If the left recursion returns a node and the right recursion returns a node, the current node is the LCA. If only one side returns a node, pass that node up (it is either the LCA already found below, or the one target found on that side). If the current node is `p` or `q`, return it immediately so a target that is an ancestor of the other is not missed. Time `O(n)`, stack `O(h)`. You need the "both nodes exist" guarantee, or you must distinguish "found one target" from "found the LCA," because a single found target bubbles all the way to the root and looks like an answer even if the other target is missing.

### 5. Recursion vs iteration

**Q:** Write the recursive BST version. Why is the loop the one you default to?

**A:** If both values are smaller, return the recursive result on the left child; if both are larger, return the result on the right; otherwise return `root`. It is the invariant in three lines, and the stack is `O(h)` frames that do no extra work. The loop stores that path in one pointer. Same comparisons, `O(1)` memory, no risk of a skewed stack. I say the recursive sentence so the interviewer hears the definition, then I code the loop. There is nothing to backtrack: once you step left you never need the right child, so an explicit stack would be pure overhead.

### 6. Null cases

**Q:** Can `root`, `p`, or `q` be null? What if the walk falls off a child? What about a tree of one node where `p` and `q` are that node?

**A:** The LeetCode contract says `p` and `q` exist in the tree, so a correct walk does not hit a null child, and the final `return null` is only a compiler satisfier. A null `root` at the start returns null. If someone calls you with a missing target, the `<=` bug or a wrong tree makes `cur` null, and returning null is the honest "not found." One node that is both `p` and `q`: both comparisons fail (the value is not strictly less or greater), so you return that node. That matches "a node is an ancestor of itself." Do not special-case `p == q` beyond what the comparisons already do. Never treat value `0` as null; `0` can be a BST key and the walk compares it like any other int.
