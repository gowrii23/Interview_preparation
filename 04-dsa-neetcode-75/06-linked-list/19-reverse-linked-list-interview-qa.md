# 19. Reverse Linked List — Interview Q&A

### 1. Complexity

**Q:** What are the time and extra-memory costs of the iterative reverse and the recursive reverse? Where does the memory go?

**A:** Both visit each node a constant number of times, so time is `O(n)`. The loop stores `prev`, `curr`, and `nxt` — `O(1)` extra memory. The recursive form makes one call per node and each frame holds `head` until the tail returns, so the call stack is `O(n)` extra memory. The nodes themselves are not copied in either version.

### 2. Invariant

**Q:** State an invariant that is true at the top of every iterative loop.

**A:** `prev` is the head of the already reversed prefix (or null if nothing has been reversed). `curr` is the head of the still-original suffix (or null if the suffix is empty). There is no edge from the prefix to the suffix. The nodes already visited have their `next` pointers pointing toward the old head. The loop body takes the first node of the suffix and pushes it onto the front of the prefix, which restores the invariant.

### 3. Off-by-one

**Q:** A candidate stops the loop with `while (curr.next != null)` instead of `while (curr != null)`. What does the list look like afterward?

**A:** The last node is never rewired. For `1 -> 2 -> 3`, the loop flips `1` and `2`, then `curr` is `3` and `curr.next` is null, so it stops with `prev` still `2`. Node `3` still points at `2`, and `2` points at `1`, so you have a cycle and you never return `3`. The correct test is on `curr` itself, because the last node is a real node that must point at the previous one, and only the null after it ends the walk.

### 4. Follow-up

**Q:** Reverse only the portion from position `left` to `right` (LC 92), one-indexed, in one pass.

**A:** Walk a pointer `prev` to the node just before `left` (a dummy makes position 1 easy). Remember `start = prev.next`, the first node of the window. Then run the ordinary three-pointer reverse for `right - left` steps, but keep the tail of the unreversed part attached. After the window is reversed, `prev.next` should be the old `right` node, and `start.next` should be the node that used to follow `right`. Example: `1 -> 2 -> 3 -> 4 -> 5`, left 2, right 4 becomes `1 -> 4 -> 3 -> 2 -> 5`. Time stays `O(n)`, memory `O(1)`.

### 5. Recursion vs iteration

**Q:** Walk through the recursive stitch on `1 -> 2 -> 3`. Why is `head.next.next = head` safe?

**A:** `reverse(3)` returns `3` immediately. `reverse(2)` then knows `2.next` is still `3`, so `2.next.next = 2` writes `3.next = 2`, and `2.next = null` breaks the old forward edge. `reverse(1)` writes `2.next = 1` and `1.next = null`. It is safe because the recursive call reversed the tail but did not modify `head.next` yet: that field still names the node that is now the tail of the reversed suffix, which is exactly the node that should point back at `head`. Both versions return the same new head. Ship the loop unless the interviewer asks for the recurrence.

### 6. Null cases

**Q:** What should happen for a null head, a single node, and a two-node list? What if some `val` is null-like (the value 0)?

**A:** A null head: the loop does not run and returns null. A single node: `nxt` is null, `node.next` becomes null, `prev` becomes that node, and you return it. Two nodes: first points at null, second points at the first, return the second. Node values are never read; `0` is a normal value and must not be treated as a missing node. The only null that matters is a null reference in `next`.
