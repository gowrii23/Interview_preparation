# 21. Linked List Cycle — Interview Q&A

### 1. Complexity

**Q:** Prove the tortoise and hare finish in linear time, both with and without a cycle. How much memory does the hash-set alternative use?

**A:** No cycle: `fast` visits every node at most once and stops when it, or the node before a one-step hop, is null. That is at most `n` iterations. With a cycle of length `C` after a stem of length `S`: both pointers reach the cycle after `O(S)` steps. Inside the cycle their positions differ by a multiple of `C`. Each step reduces the gap from `fast` back to `slow` by one modulo `C`, so they meet after at most `C` further iterations. Total time `O(S + C) = O(n)`. Memory is `O(1)`. A set of seen nodes is `O(n)` time and `O(n)` memory and also correct; use it only if constant memory was not required.

### 2. Invariant

**Q:** What invariant explains why they must meet inside the cycle?

**A:** After both pointers have entered the cycle, consider the clockwise distance from `slow` to `fast` along the cycle, in `0 .. C - 1`. A full iteration moves `slow` by `+1` and `fast` by `+2`, so that distance increases by `1` modulo `C` (equivalently, `fast` catches up by one). A quantity that increases by 1 mod `C` hits `0` within `C` steps. Distance `0` means they reference the same node. Before they are both in the cycle the invariant does not apply, and that is fine: you only return true on a meeting, which cannot happen on a null-terminated chain if you moved at least once.

### 3. Off-by-one

**Q:** You write `while (fast.next != null && fast.next.next != null)` and start by comparing before any move. Name two failures.

**A:** First, a null head crashes on `fast.next` before the loop guard can save you; the guard must test `fast` itself. Second, comparing before moving returns true immediately because both references are `head`, including a one-node list with no cycle. Even with a correct guard, checking at the top of the loop has the same false positive. Move, then compare. Also, `fast.next.next` in the guard is the wrong shape: you need `fast` and `fast.next` non-null so the body may legally read `fast.next.next`.

### 4. Follow-up

**Q:** Detect the node where the cycle begins (LC 142) and find the cycle length. Can you do both in `O(1)` memory?

**A:** Run Floyd until `slow == fast`. The meeting node is some point in the cycle, not necessarily the entrance. Then set `p = head` and walk `p` and `slow` one step at a time; the node where they meet is the entrance. Reason: if the stem has length `mu` and the meeting point is `x` steps past the entrance, Floyd's meeting satisfies `x ≡ mu (mod C)`, so walking `mu` steps from the meeting point lands on the entrance, and walking `mu` steps from the head does too. Cycle length: from the meeting node, walk until you return, counting steps. All of this is `O(n)` time and `O(1)` memory.

### 5. Recursion vs iteration

**Q:** Could you detect a cycle by recursion? Why is that a bad fit?

**A:** A recursive walk still needs a set of nodes on the current stack, otherwise a cycle is infinite recursion. That set is `O(n)` memory, the same as the hash-set solution, plus a stack overflow risk exactly on the inputs that contain a cycle — the case you most need to survive. There is no natural "subtract one and recurse" that terminates on a loop. Iteration with two pointers is the algorithm that matches the shape of the problem. Recursion is the wrong tool here, not a stylistic variant.

### 6. Null cases

**Q:** What do you return for null, a single node with `next == null`, a single node whose `next` is itself, and two nodes pointing at each other?

**A:** Null: the loop guard fails, return false. Single node, `next` null: `fast.next` is null, return false. Single node pointing at itself: one iteration moves `slow` and `fast` both back to that node (`next.next` is itself), they compare equal, return true. Two-node cycle `1 -> 2 -> 1`: first iteration `slow` is `2`, `fast` is `1`; second iteration both are `1` (or they meet on the next step). Return true. Never read `.val`. Identity is the whole test.
