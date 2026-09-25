# 34. Binary Tree Maximum Path Sum — Interview Q&A

### 1. Complexity

**Q:** Why is one DFS enough, and what would a "try every pair of nodes" solution cost? Memory?

**A:** Every legal path has a unique highest node (the bend, or the top of a straight downward path). The DFS considers that path exactly when it is standing on the highest node, using the already-computed downward gains of the children. Each node is visited once, so time is `O(n)`. Extra memory is the recursion stack `O(h)`, plus one integer for the best sum. Enumerating pairs of nodes and checking whether the path between them is unique is `O(n^2)` pairs and `O(n)` work to sum each path if you are careless, far too slow, and it rediscovers the same bends. You do not need parent pointers.

### 2. Invariant

**Q:** What does `gain(node)` return, and what extra fact is true about `best` after the call?

**A:** `gain(node)` returns the maximum sum of any downward path that starts at `node` and goes only through descendants, including the path that is just `node`. It is always at least `node.val`, because child contributions are clamped at 0 before they are added. It is not the maximum path inside the subtree: that number may bend below `node` or bend at `node` using both children, and it lives in `best`. After `gain(node)` returns, `best` is the maximum path sum among all paths whose nodes lie entirely inside this subtree. The root call therefore leaves `best` equal to the answer for the whole tree. The clamp `max(0, child)` preserves the return contract: a negative child gain would only decrease a path that is allowed to stop at `node`.

### 3. Off-by-one

**Q:** You clamp with `max(0, gain)` but you also skip updating `best` when `node.val` is negative. What does `-2` with a left child `-1` return? What about a single node `-5`?

**A:** At `-1`, if you skip the update because the value is negative, `best` never sees `-1`. At `-2` you also skip. `best` stays at its initial value. A single node `-5` returns the initial value too. The clamp is not permission to ignore the node; it only ignores a child branch. The path of one negative node is legal and is the answer when every node is negative. Update `best` unconditionally with `node.val + left + right` after the clamp, and start `best` at `Integer.MIN_VALUE`, not 0. Starting at 0 is the same family of bug: it invents an empty path whose sum is 0, which this problem does not allow. A single `-5` must return `-5`.

### 4. Follow-up

**Q:** Also return one path that achieves the sum, or restrict the path to root-to-leaf, or count paths that sum to a target (LC 437).

**A:** To recover a path, whenever you update `best`, store the two downward chains you used (or enough parent info to rebuild them) plus this node in the middle. Do it only on improvement so you keep one witness. Root-to-leaf maximum is a different recurrence: you must pick a child if one exists, you do not clamp a negative child away if it is the only route to a leaf, and you do not combine both children. LC 437 counts downward paths that sum to a target, usually with prefix sums on the DFS stack, and a path there is rootward-to-descendant, not an arbitrary bend. Say which definition you are solving before you reuse this code. The bend-plus-global pattern does not count paths and does not force a leaf.

### 5. Recursion vs iteration

**Q:** Why is postorder required, and what goes wrong if you process the node before the children?

**A:** `gain` of a node is defined from the gains of its children, so both recursive calls must return before you update `best` or compute the upward value. That is postorder. If you add `node.val` into `best` before the children, you can still record the single-node path, but you cannot yet add the child contributions, so you either miss the bend or you add it later with stale zeros and never revisit the node. An explicit stack can simulate the same postorder with a "children done" state; it is the same algorithm and easier to get wrong. Recursion is the version to write. There is no BFS formulation that keeps the downward gain as cleanly, because the gain depends on the deeper nodes first.

### 6. Null cases

**Q:** What does a null child return, what about a null root, and what about a leaf whose value is 0 or negative?

**A:** A null child returns 0 and does not touch `best`. That 0 means "do not extend," and `max(0, 0)` stays 0, so a leaf's bend is exactly its value. The problem's root is non-null; if you were called on null before initialization you would return the sentinel, so the public method should not call `gain` on a null root unless `best` has a defined empty-tree policy. This problem always has at least one node. A leaf `0`: both contributions are 0, `best` becomes 0, return 0. That 0 is a real path, not the clamp. A leaf `-3`: `best` becomes `-3`, return `-3`. Do not convert a null child into a node with value 0; a fake node would be an extra path vertex the tree does not have. The clamp already represents "stop here" without adding a vertex.
