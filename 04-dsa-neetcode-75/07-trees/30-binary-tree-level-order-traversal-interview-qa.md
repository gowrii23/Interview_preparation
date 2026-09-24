# 30. Binary Tree Level Order Traversal — Interview Q&A

### 1. Complexity

**Q:** Time, queue memory, and output memory. Why is the queue not `O(1)`?

**A:** Each node enters and leaves the queue once, so time is `O(n)` and the output has `n` values, `Θ(n)` space that the caller asked for. Extra memory is the queue. After you finish a level of width `w` you may have pushed up to `2w` children, so the peak is proportional to the widest level. A complete last level holds `(n + 1) / 2` nodes, hence `O(n)` worst-case extra memory. A skewed tree uses `O(1)` queue slots and `O(n)` stack if you had used DFS. Quote both: time `O(n)`, extra `O(w)`.

### 2. Invariant

**Q:** What is true right after you snapshot `size`, and what is true when the inner loop finishes?

**A:** Right after the snapshot, the queue contains exactly the nodes of the current level, left to right, and nothing else. The inner loop pops precisely those nodes. Every offer during the loop is a child of this level, so it belongs to the next level and sits behind the nodes still to be popped, because a queue adds at the back. When the inner loop ends, the queue contains exactly the next level, left to right, and `level` contains the values of the level you just finished. The outer loop restores the snapshot situation. Empty queue means no next level.

### 3. Off-by-one

**Q:** The inner loop is `for (int i = 0; i <= size; i++)`. What happens on the example `3 / \ 9 20`?

**A:** Snapshot at the root is `size = 1`. The loop runs twice. First pop takes `3` and pushes `9` and `20`. Second pop takes `9`, which belongs to the next level, and may push `9`'s children into this same row. The row becomes `[3, 9]` and `20` is stranded in a later row or further scrambled. One extra iteration steals the first node of the next level. The bound is `i < size`, a half-open count of the nodes that were present at the snapshot. The same bug in Python is `range(len(q) + 1)` or, worse, `while q` inside the level with no frozen length, which never separates levels.

### 4. Follow-up

**Q:** Zigzag (LC 103), bottom-up levels (LC 107), and "connect next-right pointers." What do you reuse?

**A:** The same BFS with a frozen size. Zigzag: keep a `leftToRight` flag and insert each value at the front or the back of a deque row, or reverse the row when the flag is false. Do not reverse the queue itself or you will mess up the children. Bottom-up: run ordinary level order and reverse the outer list at the end, or insert each row at index 0. Next-right pointers: while you pop a level, the previously popped node on this level gets `next = current`, and the last node's `next` stays null. All of these are still `O(n)` time. The piece you do not reinvent is the snapshot.

### 5. Recursion vs iteration

**Q:** Write the DFS that fills `result.get(depth)`. Why is BFS still the default answer?

**A:** `dfs(node, depth)`: if null, return; if `depth == result.size()`, append a new row; add `node.val` to that row; recurse left then right at `depth + 1`. Left-before-right makes each row left to right, and deeper nodes append later within a row only after shallower calls... actually a right node at depth 2 is visited after the entire left subtree, so values at the same depth still append left to right because every node in the left subtree is left of every node in the right subtree. It works. Stack is `O(h)`, which can be better than `O(w)` on a wide tree. BFS is the default because the problem says "level order" and the queue is the definition. Give DFS if they ask for less memory on a wide shallow tree or forbid a queue.

### 6. Null cases

**Q:** Null root, a root with only a right child, and a node value of 0. Should null children occupy a slot in the row?

**A:** Null root: return an empty list before you touch the queue. Do not append an empty level. Root with only a right child: the second level is a one-element list containing that child. The missing left child is not a null entry in the output; this problem's rows contain real nodes only. (Serializing a tree for LC 297 is different and does record nulls.) Value `0` is a real value and must appear in the row. Never skip `if (node.val == 0)`. Skip only null references when offering children.
