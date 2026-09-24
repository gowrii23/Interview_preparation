# 26. Maximum Depth of Binary Tree — Interview Q&A

### 1. Complexity

**Q:** DFS and BFS are both linear time. When is one actually cheaper on memory, and can you stop early?

**A:** Time is `Θ(n)` for both if you truly need the maximum, because the deepest leaf could be the last node you would have skipped. You cannot stop early with a correct answer unless you already have another bound. Memory: DFS is `O(h)` and wins on a wide, shallow tree (a perfect tree has `h = log n` but the BFS queue holds `O(n)` nodes on the last level). BFS is `O(w)` and wins on a very deep skinny tree where `h` is `n` and `w` is 1, if the recursion stack would overflow and your queue stays small. Worst case over all shapes, both can use `O(n)` extra memory.

### 2. Invariant

**Q:** State the invariant of the recursive function in one sentence. What does the BFS counter mean after each level?

**A:** `maxDepth(node)` returns the number of nodes on the longest downward path that starts at `node` and ends at a leaf in its subtree, and it returns 0 when `node` is null. After the recursive calls, the invariant applied to the children plus one for `node` gives the answer, because every root-to-leaf path in this subtree goes through exactly one of the children (or stops here if both are null). In BFS, after `d` successful level iterations, every node still in the queue has depth `d + 1`, and `d` is the number of levels fully processed. When the queue empties, `d` is the depth.

### 3. Off-by-one

**Q:** You initialize `depth = -1` for BFS, or you write `return Math.max(left, right)` without the `+ 1`. What do you report for a single-node tree and for the tree whose longest path has 3 nodes?

**A:** Without the `+ 1`, a leaf returns `max(0, 0) = 0`, and the 3-node-deep tree returns 2. You counted edges, or you counted nothing at the leaves. LeetCode's single node must return 1, and that example must return 3. The BFS bug `depth = -1` then `depth++` once per level happens to return the right number if you also increment before processing — if you start at `-1` and increment at the end you are short by one on every tree, including reporting 0 for a single node. Pick one convention and test the leaf: the answer is 1, the empty tree is 0.

### 4. Follow-up

**Q:** Minimum depth (LC 111) looks identical. Why is `1 + min(left, right)` wrong, and what about counting leaves only?

**A:** Minimum depth is the shortest root-to-leaf path. A null child is not a leaf. `1 + min(depth(left), depth(right))` treats a missing child as depth 0 and then reports 1 for any node that has only one child, even if the other side is a long chain. The fix: if one child is null, the minimum path must go through the child that exists; only when both exist do you take the min. BFS is often cleaner for minimum depth because the first time you dequeue a leaf you can return its level. "Count nodes that have no children" is a different number, the number of leaves, and it is not a depth.

### 5. Recursion vs iteration

**Q:** Rewrite the recursion as an explicit stack of `(node, depth)` pairs. What bug does that make obvious?

**A:** Push `(root, 1)`. Pop, and let `best = max(best, depth)`. Push each existing child with `depth + 1`. If you push children with the same depth you forgot the plus one, and a chain of three nodes reports 1. If you start the root at 0 you are back to the edge-count convention and a leaf reports 0. The explicit stack makes the number you store the thing you have to get right, which the recursive `1 + max(...)` hides in the return. Use recursion in the interview unless they ask for the stack. Both are DFS.

### 6. Null cases

**Q:** Null root, a root with only a left child who is a leaf, and a node whose value is 0. What do you return, and do you read `.val`?

**A:** Null root returns 0. Root plus one left leaf: left returns 1, right returns 0, answer is `1 + max(1, 0) = 2`. You never read `val`. A node with value 0 is a real node and contributes to depth like any other. Do not write `if (root == null || root.val == 0)`. The right-null side returning 0 is correct only because `max` ignores it when the left side is positive; that same 0 would be a bug in minimum depth, which is the follow-up, not this problem.
