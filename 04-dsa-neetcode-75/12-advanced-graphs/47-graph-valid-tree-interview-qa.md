# Interview Q&A: Graph Valid Tree (LeetCode 261)

## Q1. Which three properties are you actually testing, and which pairs are enough?

**Answer:** The properties are n nodes, exactly n − 1 undirected edges, connectivity, and no cycle. The first is given. Any two of the other three imply the one you left out, for a simple undirected graph. I test n − 1 edges and one component. I could test n − 1 edges and "every union linked different roots." I could test "one component and no redundant edge" without a separate length check, because each redundant edge and each missing edge shows up as either a failed union or a component count other than 1. I cannot test only one of them. n − 1 edges alone accepts a triangle plus an isolated node. One component alone accepts a triangle. No-cycle alone accepts a forest of two edges on four nodes.

## Q2. Walk a 4-node graph with edges `[0-1, 1-2, 2-0]`. Where does it fail?

**Answer:** The length is 3, and n − 1 is 3, so the length check passes. Union 0 and 1, components 3. Union 1 and 2, components 2. Union 2 and 0: both ends already have the same root, because 0, 1, and 2 are one component. I return false before decrementing. Node 3 is still alone, which is the other face of the same failure: the wasted edge would have been the one that could have connected 3, but even if I only talked about cycles I already have a same-root edge. If the edges had been `[0-1, 1-2, 2-3]`, every union would merge, components would end at 1, and the answer would be true.

## Q3. Why does the DFS version need a parent pointer when union-find does not?

**Answer:** An undirected adjacency list stores each edge twice, once at each endpoint. DFS that marks nodes visited will look from child back to parent and see a visited node. That reverse step is the same edge, not a second path. The DFS cycle rule is "a visited neighbor that is not my parent." Union-find consumes the edge list, where each undirected edge appears once. The first time the edge is processed it links two roots. There is no second stored copy to process, so there is no false cycle. A later different edge between the same components is a real cycle, and `find` reports one root. I choose union-find on an edge-list input to avoid the parent bug. I choose DFS when I already have the adjacency list and I am also collecting the traversal for another reason.

## Q4. Is a connected DAG a tree? Why is topological sort the wrong tool here?

**Answer:** A DAG is directed. This problem's edges are undirected, so "before" and "after" are not defined and a topological order is not a concept the input has. If I orient each undirected edge in both directions and run Kahn, every edge becomes a 2-cycle and every non-empty graph looks cyclic, including a real tree. If I orient each edge once, arbitrarily, a tree becomes a DAG and so does a disconnected forest, and a cycle might become a DAG depending on how I oriented it. The test would answer a different question. I stay with the undirected criteria: n − 1 edges and one component. Directed trees, with roots and parent pointers, are a different interview problem and usually want indegrees of at most 1 plus connectivity.

## Q5. What about a self-loop, a duplicate edge, and n = 1?

**Answer:** n = 1 with no edges has length 0, which equals n − 1, and the component count stays 1. It is a tree. A self-loop `[0,0]` on that node makes the length 1, which is not 0, so I return false. A duplicate edge is two identical undirected pairs. On n = 2, edges `[0,1]` and `[0,1]` have length 2, not 1, so the length check fails. If I had not checked length, the second union would see one root and return false anyway. Both inputs are cycles in the undirected sense: a self-loop is a cycle of length 1, and a duplicate is a cycle of length 2. I do not have to remember those names if the two checks are in place.

## Q6. State the complexity after the early length check.

**Answer:** If the edge list is longer or shorter than n − 1, I return in O(1) after reading the length, assuming the array's length is known. If it equals n − 1, I initialize O(n) parent storage and run n − 1 unions. Each union is inverse-Ackermann with compression and rank, so the slow path is O(n α(n)) time and O(n) space. I do not allocate an adjacency list in this version. The DFS alternative builds that list in O(n) and then traverses O(n) edges, same big-O, with the parent exception discussed above. I mention that a very dense illegal graph is rejected by the length check before any union, which matters when someone passes a complete graph and expects you to walk every edge.
