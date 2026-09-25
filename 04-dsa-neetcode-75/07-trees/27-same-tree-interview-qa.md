# 27. Same Tree — Interview Q&A

### 1. Complexity

**Q:** Worst-case time and stack space when both trees have `n` nodes. Does a mismatch at the root change the bound?

**A:** Worst case the trees are identical, so you visit every node of both: `Θ(n)` time if `n` is the size of each, or `Θ(n1 + n2)` in general. The stack is `O(h)` and `h` can be `n` on a skewed tree. A mismatch at the root returns in `O(1)` time after the null and value checks, before any child call. So the tight worst case is linear, and the best case is constant. Do not claim early exit improves the worst case.

### 2. Invariant

**Q:** What does a `true` return mean, exactly? Why is "same multiset of values" a weaker claim?

**A:** `isSameTree(p, q)` is true exactly when the two rooted trees are identical as ordered binary trees: the same nodes exist in the same child positions and corresponding nodes store the same values. It is false as soon as any position differs. A multiset of values ignores positions, so a left-leaning `1 -> 2` and a right-leaning `1 -> 2` would look the same and are not. The invariant of the recursion is that the children are only consulted after this pair of nodes has been shown to be a structural match (both present and equal, or both absent).

### 3. Off-by-one

**Q:** You write the null test as `if (p == null || q == null) return p == q;` and then compare children. Is that off, or is a nearby variant off?

**A:** That one-liner is correct: if either is null, they are the same only when both are null. It folds the two base cases together. The off-by-one cousin is checking only one level, or writing `return p.val == q.val` and forgetting the children, which reports true for any two roots with the same value. Another fencepost: serializing and comparing arrays but dropping the last null marker, so a leaf and a node with a null-valued encoding get confused at the end of the list. In the recursive form the bug to avoid is returning true when one side still has an uncompared child.

### 4. Follow-up

**Q:** How do you change this to "mirror images," and how is LC 572 (subtree) different from calling this once on the two roots?

**A:** Mirror: `isSameTree(a.left, b.right) && isSameTree(a.right, b.left)`, with the same null and value checks. Subtree is not "the whole trees are equal." You must try this predicate at every node of the larger tree, because the matching shape may hang under some descendant. If any candidate returns true, the answer is true. Short-circuit the search once you find one. Comparing only the two roots solves same-tree, not subtree.

### 5. Recursion vs iteration

**Q:** Two stacks, popping in lockstep. What do you push for a null child, and why?

**A:** Push both roots. While both stacks are non-empty, pop one node from each. If both are null, continue. If one is null or the values differ, return false. Push left children, then right children, including nulls, so the stacks stay aligned. If you skip null pushes, a missing left child shifts the right child into the left's slot and you can accept different shapes. If one stack empties before the other, return false. If both empty, return true. This is `O(n)` time and `O(n)` worst-case stack memory. Recursion is the version to write first; the lockstep stacks are the iterative twin and the null pushes are the whole point.

### 6. Null cases

**Q:** Both null, one null, a leaf versus a node with value 0 and two null children, two nodes with the same value but one has a left child.

**A:** Both null: true, and this is a real input, not only a base case. One null: false, and you must not read `.val` on the null side. A leaf and a node `0` with null children: if the leaf's value is also 0, they are the same node shape; value 0 does not mean null. If you encoded null as 0 in a list you would confuse them, which is why the pointer recursion is safer. Same root value but only one has a left child: the child call hits "exactly one null" and returns false. That child call is required; the root check alone is not enough.
