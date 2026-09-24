# 24. Merge k Sorted Lists — Interview Q&A

### 1. Complexity

**Q:** Derive `O(N log k)` for both the heap and the pairwise merge. When does the heap version degrade, and what is the memory?

**A:** Heap: at most `k` live heads, so each push and pop is `O(log k)`. Every node is pushed at most once (when it becomes a head) and popped once. Time `O(N log k)`, extra memory `O(k)`. Pairwise: a balanced round merges disjoint pairs and touches each node `O(1)` times, so a round is `O(N)`. The number of lists halves each round, so there are `O(log k)` rounds. Same time bound. Extra memory is `O(1)` per two-list merge plus `O(k)` to hold the current array of heads, or `O(log k)` if the split is recursive. The heap degrades toward `O(N log N)` only if you incorrectly push all nodes at once and `k` is about `N` with many one-node lists — still `O(N log k)` with `k = N`. The bound that actually gets worse is the unbalanced "fold into one list" loop: list `i` of size `n_i` is walked `k - i` times, which is `O(k N)` when sizes are even.

### 2. Invariant

**Q:** What does the heap represent after each pop, and why is the output sorted?

**A:** The heap contains exactly the current head of every list that still has nodes, and nothing else. Each individual list remains sorted, so its head is its minimum remaining value. The minimum of those heads is the minimum remaining value in the whole input. Appending it and replacing it with its successor (or removing the list if it has no successor) restores the invariant. By induction every appended value is the next in sorted order. The dummy's tail is the last appended node, so the result has no holes.

### 3. Off-by-one

**Q:** You seed the heap by walking each list and pushing every node, then you also push `node.next` when you pop. What goes wrong numerically and structurally?

**A:** Structurally you insert nodes twice, so the output contains duplicates and the `next` pointers form a mess because the same node is appended more than once. Numerically you do about `2N` heap operations and the heap can hold `O(N)` entries, so the cost becomes `O(N log N)` even when `k` is 2. The index you thought was "one node per list" is off by the entire suffix. Push a node only when every node before it in that list has already been popped, which means: push the original head, and thereafter push only `popped.next`.

### 4. Follow-up

**Q:** The lists are streams that you cannot rewind, or they are so long they do not fit in memory, or `k` is huge but most lists are empty. What do you keep?

**A:** The heap algorithm already streams: it only stores `k` nodes, not `N`, and it never rewinds because each list is consumed through `next`. Empty lists are skipped at seed time so they do not occupy heap slots. If `k` is huge and the heap of pointers is the memory bottleneck, an external merge still wants a heap of size `k`; you cannot do better than `Ω(N log k)` comparisons in the worst case for comparison merging. If you need stability among equal keys, record the list index and pop the smaller index on ties (the Python tuple already can). Divide and conquer is worse for true streams if it needs to hold whole merged intermediate lists before the next round; the heap emits the result online.

### 5. Recursion vs iteration

**Q:** How do you write divide-and-conquer recursively, and how does its stack compare with a recursive k-way "pick the min head and recurse"?

**A:** Recursive split: `merge(lists, lo, hi)` returns `lists[lo]` when `lo == hi`, and otherwise merges the result of the left half with the result of the right half using the iterative two-list merge. Depth is `O(log k)`, and each level across all calls does `O(N)` work, so time stays `O(N log k)`. A recursion that scans all `k` heads, chooses the min, and recurses on `N` is `O(k N)` time and `O(N)` stack. That is the recurrence to avoid. The heap loop is iteration with an explicit priority queue standing in for "the recursive state of k cursors."

### 6. Null cases

**Q:** What do you return for `lists == null`, `lists` empty, every list null, one non-null list, and two heads with the same value?

**A:** Null or empty array: the heap stays empty, `dummy.next` is null, return null. All null heads: nothing is offered, return null. One real list: the heap pops each node and pushes the successor, rebuilding the same chain; you return that head. Equal head values: both stay in the heap, `Integer.compare` (or the tuple's index) orders them, and both are appended. Neither is dropped. Do not treat value `0` as empty. Only a null reference means "this list is finished" or "this slot was empty."
