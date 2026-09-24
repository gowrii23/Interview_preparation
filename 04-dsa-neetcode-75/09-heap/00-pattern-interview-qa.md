# Interview Q&A: heap vs quickselect

## Q1. The interviewer says "find the kth largest." What is your first question, and how does the answer change the algorithm?

**Answer:** I ask whether the array is fixed or whether numbers keep arriving, and whether they need only the kth value or the k elements. A fixed array and a single value: quickselect, average O(n), or a size-k min-heap in O(n log k) if I want a bounded worst case and shorter code. A fixed array and the k elements in order: a size-k heap, then sort those k, or partial sort. A stream: a size-k heap, because quickselect would rescan history. A stream plus "the median so far" is specifically two heaps, not one heap of size n/2 that I rebuild. I also ask if I may reorder the input. Quickselect does. The heap scan does not.

## Q2. Why is quickselect average linear if partition looks like quicksort?

**Answer:** Quicksort sorts both sides, so T(n) = O(n) + T(left) + T(right), which is O(n log n) when the splits are balanced. Quickselect only continues into the side that contains the target index, so T(n) = O(n) + T(n/2) in a balanced case, and that sum is O(n). Even with uneven but random pivots, the expected cost of the successive partitions is still linear because each element has a constant expected number of comparisons before its side is discarded. The quadratic case is a pivot that only peels one element every time, the same degenerate quicksort. A random index as pivot makes that pattern depend on luck, not on a sorted input.

## Q3. In a min-heap of size k used for "k largest," why do you pop when the size exceeds k, and why is the root the answer?

**Answer:** The heap stores candidates for the top k. If it holds k + 1 values, the smallest of those cannot be among the k largest in the array, because at least k values in the heap are bigger than it. Popping the root throws that value away and restores size k. After the scan, every rejected value was smaller than the current root at the time it was seen, and later evictions only raise the bar or keep it. So every value outside the heap is smaller than or equal to every value that remains, in the sense that at most k - 1 array values are strictly larger than the root. The root is therefore the kth largest. The maximum is somewhere in the heap, not at the root. If I had used a max-heap of size k I would be keeping the wrong end.

## Q4. How do you simulate a max-heap in Python and in Java?

**Answer:** Python's `heapq` only supports min-heaps. I push `-value` and read the maximum as `-heap[0]`. That works for numeric priorities. If the priority is not negatable, I push a tuple `(reverse_key, tie_breaker, item)` where `reverse_key` gets smaller as the item gets more important, and I include a unique tie breaker so items never get compared with `<` and throw. Java's `PriorityQueue` is a min-heap of the natural order. `new PriorityQueue<>(Collections.reverseOrder())` is a max-heap of integers. I do not negate in Java unless I want one code shape in both languages. Negating `Integer.MIN_VALUE` overflows, so for a full int range the comparator is the safer Java max-heap.

## Q5. What does heap order actually guarantee, and what does a common bug look like?

**Answer:** In a min-heap, every parent is less than or equal to its children. There is no left-to-right order between siblings. The minimum is at index 0, and the second minimum might be either child. Printing the underlying list is not a sorted print. The usual bug is assigning into the list, appending with `list.append`, or changing a priority field without sifting. The shape still looks like a complete binary tree in the array, but the parent-child inequality is false, so the next `heappop` returns the wrong element. Another bug is using a heap to get a full sort and claiming O(n). Heapsort is O(n log n). The linear claim belongs to build-heap plus a single order statistic, not to popping n times.

## Q6. When would you still sort?

**Answer:** When the next step needs the elements in order anyway, when n is small enough that clarity dominates, or when I need many different order statistics on the same frozen array. One sort answers all of them. Quickselect's advantage disappears if I am going to sort the survivors immediately after. I would also sort if the language heap has a large constant and the interviewer is clearly testing something else. I would not sort inside a loop that inserts one element and asks for the new median; that is the two-heap problem, and sorting each time is the solution that times out.
