# Linked-list patterns — Interview Q&A

### 1. Complexity

**Q:** Why is a heap merge of k sorted lists `O(N log k)` and not `O(N log N)`? When is divide-and-conquer the same bound?

**A:** N is the total number of nodes across all lists. The heap never holds more than one node per list, so its size is at most k. Each node is inserted once and removed once, and each heap operation is `O(log k)`. Divide-and-conquer pairs the lists and merges each pair with the linear two-list merge. Every node takes part in `O(log k)` merge rounds (the depth of the pairing tree), so the total is also `O(N log k)`, with a smaller constant and no heap, at the cost of `O(log k)` recursion stack or an explicit list of intermediate heads.

### 2. Invariant

**Q:** What invariant makes the "gap of n" technique correct for deleting the nth node from the end?

**A:** After the opening sprint, `fast` is exactly `n + 1` nodes ahead of `slow` (counting the dummy as a node). Both then move one step at a time, so the distance never changes. When `fast` becomes null it has walked off the last real node, which means `slow` is standing on the predecessor of the nth-from-end node. `slow.next = slow.next.next` unlinks that node and the invariant tells you there is no other bookkeeping to do.

### 3. Off-by-one

**Q:** You start `fast` and `slow` on the dummy and move `fast` only `n` steps instead of `n + 1`. What goes wrong?

**A:** `slow` stops on the node you meant to delete, not on the node before it. A singly linked list cannot unlink a node unless you hold its predecessor, so you either skip the wrong node or you have to keep a `prev` that you forgot to introduce. The extra step is exactly the predecessor. The symmetric bug is walking `n + 1` when both pointers started on `head` rather than the dummy: `fast` can fall off the end during the sprint when `n` equals the length.

### 4. Follow-up

**Q:** How do you return the node where a cycle begins, not just a boolean? How do you find the middle and then put the list back the way it was?

**A:** Floyd still applies (LC 142). After `slow` and `fast` meet, park one pointer on `head` and walk both one step at a time; they meet at the entrance. Proof sketch: the meeting point is `mu + t` steps into the list where `mu` is the stem length, and the remaining distance around the loop equals `mu` modulo the cycle length. For a middle that you must not destroy, remember `slow.next` was the cut; reversing the second half twice restores the original order. If the caller needs the list unchanged, copy nothing — just reverse, use, reverse again.

### 5. Recursion vs iteration

**Q:** When is a recursive list solution acceptable, and when should you insist on iteration?

**A:** Recursion is natural when the answer for a node is "the answer for `next`, then a constant-time stitch" — reverse, merge two lists, and a recursive k-way merge all look like that. The stack is `O(n)` or `O(log k)` and a list of 10^5 nodes can blow the call stack. Iteration with an explicit `prev` / `tail` / heap uses `O(1)` or `O(k)` heap memory and is what you want in production. In an interview, give the iterative version first and mention the recursive one as the same recurrence unrolled by the call stack.

### 6. Null cases

**Q:** Which null checks are load-bearing, and which are noise?

**A:** Before `fast.next.next`, both `fast` and `fast.next` must be non-null or you throw. Before `curr.next = prev` you only need `curr` itself. A dummy's `next` is allowed to be null; that is the empty-list result, not an error. When merging, a null list is an empty sequence: skip it when seeding the heap, and if one side of a two-list merge is null, splice the other side on in one assignment. Do not special-case "both null" beyond `dummy.next`, which is already null. Cycle detection on a null head or a single node with `next == null` returns false without entering the loop.
