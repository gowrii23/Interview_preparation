# 28. Subtree of Another Tree — Interview Q&A

### 1. Complexity

**Q:** Why is the straightforward DFS `O(n m)` and not `O(n + m)`? When does the string method actually become linear?

**A:** In the worst case every one of the `n` nodes has the same value as `subRoot`'s root, and `isSame` walks `Θ(m)` nodes before finding a mismatch near the leaves. That is `Θ(n m)` comparisons. Average trees fail faster, but you quote the worst case. Serialization visits each tree once, `O(n + m)`, and builds two strings of that length. A naive `String.contains` is not guaranteed linear. Running KMP, or an equivalent linear substring search, on those strings is `O(n + m)`. The serialization must include null markers and separators. Without them the string method is linear and wrong.

### 2. Invariant

**Q:** Separate the invariant of `isSame` from the invariant of `isSubtree`.

**A:** `isSame(a, b)` is true only when the two rooted trees are identical, including absent children. It never "searches." `isSubtree(root, subRoot)` is true when at least one node in `root`'s tree is the root of a tree identical to `subRoot`. The code maintains that by returning true on an exact match at the current node, otherwise combining the same claim on the left child and the right child with or. A false `isSame` at the current node does not refute the outer claim, because the witness can sit lower. A false `isSame` does refute the inner claim immediately.

### 3. Off-by-one

**Q:** `isSame` returns true when one side is null and the other is a leaf's missing child, but your search treats "values equal" as success and does not compare children. What trees do you get wrong?

**A:** Any time `subRoot` is a proper "prefix" of a branch. Example: `root` is `4` with left `1` and right `2`, `subRoot` is `4` with only left `1`. Values at the candidate match, you return true, and you never notice the extra right child. That is the structural off-by-one: you stopped one level too soon. The full `isSame` walks until both sides are null together. Another fencepost is a serializer that omits the final null marker, so a subtree string is a prefix of a larger node's string and `contains` returns a false positive. Compare every child pair, or emit every null.

### 4. Follow-up

**Q:** Count how many times `subRoot` occurs, or allow the match to be an unordered tree. What changes?

**A:** Count: replace the early `return true` with a sum. Add 1 when `isSame` succeeds, then always add the counts from the left and right. Do not stop at the first hit. Overlapping occurrences are fine; each starting node is independent, and `isSame` does not consume nodes. If "subtree" means "there exists a subset of children that match" you have a different, harder problem (tree pattern matching). Say that the LeetCode definition is exact and ordered. If duplicates of the same shape are nested, the count version counts each starting node that fully matches, which may be more than one along a chain.

### 5. Recursion vs iteration

**Q:** How would you iterate, and why is recursion a better fit than BFS here?

**A:** You can push every node of `root` on a stack or queue and run an iterative same-tree check against `subRoot`. That is correct and still `O(n m)` time. BFS does not help asymptotically because the match is not about levels; a matching subtree can sit at any depth and you still may have to test every node. Recursion matches the two definitions directly and shares the call stack with the tree shape. Use iteration if `n` can be large enough to blow the stack, with an explicit stack of nodes still to test as candidate roots.

### 6. Null cases

**Q:** `subRoot` null, `root` null, both null, `subRoot` a single node whose value appears as a leaf, and `subRoot` a single node whose value appears as an internal node with children.

**A:** `subRoot` null: true, including when `root` is also null. `root` null and `subRoot` non-null: false. A single-node `subRoot` matches a leaf with that value because `isSame` sees equal values and both children null on both sides. The same single-node `subRoot` does not match an internal node with that value if the internal node has a child: `isSame` hits one-null versus a real child and returns false. Then the search continues and may still match a leaf elsewhere. Value `0` is a normal value. Only a null reference is an empty tree.
