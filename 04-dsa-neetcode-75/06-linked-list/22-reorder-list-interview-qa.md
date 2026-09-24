# 22. Reorder List — Interview Q&A

### 1. Complexity

**Q:** Three passes sounds like a lot. What is the actual complexity, and how does the array approach compare?

**A:** Each pass touches each node a constant number of times: find middle `O(n)`, reverse `O(n)`, zip `O(n)`. Total time `O(n)`, extra memory `O(1)`. Loading every node into an `ArrayList` and then linking index `i` to `n - 1 - i` with a careful alternating scheme is also `O(n)` time but `O(n)` memory. Both are correct. The pointer version is the one that matches the "do it in place" follow-up.

### 2. Invariant

**Q:** During the zip, what is true about `first` and `second`?

**A:** `first` is the next still-unconsumed node of the original left half, and the nodes before it are already in final order. `second` is the next still-unconsumed node of the reversed right half, which is the next original node from the end that the pattern wants. The left remainder is at least as long as the right remainder, and the left remainder already ends in null because of the cut. Each iteration consumes one node from each side and leaves that invariant intact. When `second` is null, the left side has zero or one node left and that node already terminates the list.

### 3. Off-by-one

**Q:** On `1 -> 2 -> 3 -> 4`, your middle loop is `while (fast != null && fast.next != null)`. Where does `slow` stop, and what does a zip that only checks `second != null` do?

**A:** Start `slow` and `fast` at `1`. After two iterations `fast` is null and `slow` is `3` (the right middle). The cut yields `1 -> 2 -> 3` and second half `4`. Reverse is just `4`. Zip once: `1 -> 4 -> 2 -> 3`. That happens to be correct for this input. Now try a zip that assumes the second half can be longer, or a cut that leaves `slow` on `3` while you also reverse from `slow` inclusive: you either duplicate `3` or you drop `2`. The off-by-one is which node is the last node of the first half. With the stricter guard `fast.next != null && fast.next.next != null`, `slow` stops on `2`, halves are `1 -> 2` and `4 -> 3` after reverse, and the zip produces `1 -> 4 -> 2 -> 3` with the loop condition matching the longer-left policy. State the policy before you code.

### 4. Follow-up

**Q:** Reorder into groups of k, or rotate the list left by k. Which pieces of this solution survive?

**A:** The middle-finding and the zip are specific to "ends inward." Reverse and the dummy/cut technique survive. Rotating right by k (LC 61) is: walk to the tail while counting `n`, connect tail to head to make a ring, then walk `n - k % n` steps and cut. That is closer to "nth from the end" than to reorder. Reordering in blocks of k is the reverse-nodes-in-k-group problem (LC 25): reverse each window of k with the same three-pointer reverse, and stop when fewer than k nodes remain. The lesson to say out loud is that reorder-list is a pipeline of middle, reverse, and merge, and each stage is reusable on its own.

### 5. Recursion vs iteration

**Q:** A recursive solution takes the head and the tail and fills inward. Why is the three-pass loop preferable?

**A:** Recursively: reorder the middle list `head.next ... predecessor(tail)`, then set `head.next = tail` and `tail.next` to the old second node. Finding the predecessor of the tail is `O(n)` if you do not already have it, so a naive recursion is `O(n^2)`. You can pass the tail pointer down, but the bookkeeping is easier to get wrong than three straight loops, and the stack is `O(n)`. The iterative pipeline is `O(n)` time, `O(1)` memory, and each stage is a pattern you can test alone: print the list after the cut, after the reverse, and after the zip.

### 6. Null cases

**Q:** What about null, one node, two nodes, and three nodes? Does the zip dereference a null `first`?

**A:** Null and one node: return immediately, nothing to weave. Two nodes `1 -> 2`: `slow` stays on `1` because `fast.next.next` is null. Second half is `2`, reversed `2`, cut makes `1.next` null. Zip sets `1 -> 2 -> null`. Correct, the list is unchanged. Three nodes `1 -> 2 -> 3`: `slow` stops on `2`, reverse `3`, zip `1 -> 3 -> 2`. `first` is never null inside the loop because the left half is longer or equal and we stop on `second`. After the last splice, `first` may become null only once `second` is also null, and the `while` then exits before the next read.
