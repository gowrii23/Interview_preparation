# Interview Q&A: Number of Connected Components (LeetCode 323)

## Q1. Why does the component count start at n and go down?

**Answer:** Before any edge is applied, no two nodes have a reason to be in the same component, so there are n components, including nodes the edge list never names. An undirected edge is a possible merge. It is a real merge only when its endpoints currently have different roots. Each real merge turns two components into one, so the count drops by exactly one. An edge whose endpoints already share a root is not a merge. Decrementing anyway reports fewer components than exist. Incrementing on each edge reports something closer to a size or an edge count, not a component count. After the last edge, whatever was never merged is still in the total.

## Q2. Walk n = 5 and edges `[0-1, 1-2, 3-4]`. What does a repeated `0-1` change?

**Answer:** Union 0 with 1, count 4. Union 1 with 2. 1 already points at 0's component, 2 does not, count 3. Union 3 with 4, count 2. The components are `{0,1,2}` and `{3,4}`. A second edge `0-1` finds the same root for both ends, `union` returns false, and the count stays 2. That edge is the dotted prune in the diagram: it is real input, and it is not a new component operation. If the repeated edge had decremented the count, the answer would be 1, which claims a path between node 2 and node 4 that does not exist.

## Q3. How does this differ from counting islands on a grid?

**Answer:** Islands are components of an implicit grid graph, 4-connected land cells, and the natural algorithm is a flood because the edges are the neighbor rule rather than a list. This problem hands you n and an explicit undirected edge list, which is union-find's input shape. Both are "count components." I can translate this problem into a flood by building an adjacency list and DFS-ing every unvisited id. I can translate islands into union-find by unioning each land cell with the land above it and to its left. The answers match. What I do not do is run a grid bounds check on an abstract edge list, or ignore isolated node ids the way water cells are ignored. An isolated node here is a component. A water cell on the island grid is not a node.

## Q4. What do path compression and rank change about the answer, as opposed to the speed?

**Answer:** They change nothing about the final count if `union` still links different roots and refuses equal roots. A correct parent walk without compression returns the same connectivity. Rank only chooses which root becomes the parent. The count depends on whether the roots differed, not on which one won. I still write both optimizations because a long chain makes each `find` scan O(n) nodes and a large edge list becomes quadratic. With both, each operation is inverse-Ackermann, which I call nearly constant. I do not recompute the count at the end by scanning parents unless I want a sanity check. If I do scan, I must `find` each node first, because a non-root parent pointer is not a component id until it is compressed or walked.

## Q5. The graph has a cycle and is still one component. Is that a failure?

**Answer:** Not for this problem. The problem asks for components, and a cycle is one component if the cycle's nodes are linked and nobody else is attached... a cycle through all nodes is one component. `union` returning false on the closing edge is the success path for "we already connected these." The tree problem later is the one that turns that false into a failed tree test. I keep the two questions separate. Here I count how many times union returned true, subtract that from n, and I am done. I do not also require the edge count to equal n − 1. A complete graph on n nodes has one component and far more than n − 1 edges.

## Q6. Quote time, space, and the n = 0 or empty-edge cases.

**Answer:** Initialization is O(n). Each edge is a near-constant number of parent operations, so the edge loop is O(E α(n)). Space is O(n). No adjacency list is required. n = 0 returns 0 because the arrays are empty and the loop does not run. n = 1 with an empty edge list returns 1. n = 4 with an empty edge list returns 4. I do not treat "no edges" as "one empty component." Each node is a piece of the graph. If the platform promises n ≥ 1, the n = 0 case is still what the code does, and I would rather have that behavior than a special case that indexes `edges[0]`.
