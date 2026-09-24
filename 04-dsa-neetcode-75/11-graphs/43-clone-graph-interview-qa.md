# Interview Q&A: Clone Graph (LeetCode 133)

## Q1. Why is a map from original to clone required, instead of a visited set of values?

**Answer:** A visited set can tell you not to recurse, but it cannot tell you which new object should receive the edge. The neighbor list of the copy has to point at clones, not at originals, and not at a second clone of a node you already copied. The map answers both questions: the key means "already cloning this object," and the value is the object to link. Because the graph can cycle, that lookup happens while the original node is mid-visit, not only after its component is finished. A set of values happens to be unique in the stated constraints (values are 1 through n once each), and I still map objects to objects so I am not one constraint change away from gluing two different nodes together.

## Q2. What breaks if you insert the clone into the map after processing neighbors?

**Answer:** Take two nodes that point at each other. Start at node 1, allocate `1'`, and recurse to node 2 before recording the map entry. Node 2 allocates `2'` and recurses back to node 1. Node 1 is not in the map, so you allocate another clone and recurse to node 2 again. The calls never hit a base case. The stack overflows, and even a depth limit would produce multiple clones of the same vertex. Inserting `clones[node] = copy` before the loop means the back edge from 2 to 1 returns the in-progress `1'` and appends it. The neighbor lists are complete when both calls return. The same ordering fixes a longer cycle. Nothing about the bug is special to length 2.

## Q3. How do you treat a null input and a one-node graph?

**Answer:** A null reference has no value and no neighbors. The copy is null. I return before allocating the map entry. A one-node graph has an empty neighbor list. I allocate one new node, store it, skip the loop, and return it. I should be able to say that the returned object is not the input object, and that its neighbor list is a different empty list. If I returned the input unchanged, every local test that only checks `.val` would pass and the deep-copy requirement would still be wrong. In an interview I mention both edge cases before the cyclic one, because they are easy to forget once the discussion moves to the map.

## Q4. Does BFS change the map rule?

**Answer:** The rule is the same: the moment a node is discovered, allocate its clone and store the mapping, then push the original so its edges expand later. Discovering a neighbor that is already in the map means "append the existing clone," not "allocate again." If I pushed neighbors before storing the clone, a cycle would push the start node a second time and I would build two copies. BFS avoids a deep recursion on a long path. Time is still O(V + E), and the queue plus the map is O(V) extra space. I pick DFS when I want the shorter code and BFS when the graph can be a single long corridor. The copy is indistinguishable either way: same values, same undirected connections, no shared node objects.

## Q5. The graph is undirected. Where does that show up in the copy?

**Answer:** Undirected means each edge is stored on both endpoints. I do not add a special "also link the reverse" step. When DFS is sitting on A it appends clone(B) because B is in A's list, and when it is sitting on B it appends clone(A) because A is in B's list. If I only copied one direction I would invent a directed graph the input did not have. If the input list is inconsistent — A lists B but B does not list A — the copy preserves that inconsistency, which is what a deep copy should do. I do not "repair" the graph. I also do not sort neighbor lists unless a checker requires it. Order follows the input order because I iterate the lists as given.

## Q6. What complexity do you quote, and what would you change if the graph were disconnected and you were not handed every component?

**Answer:** From the given node I visit each reachable node once and each reachable edge once, O(V + E) time, O(V) map space, O(V) stack or queue. The problem states the graph is connected, so one start clones everything. If the follow-up handed me a list of nodes and the graph might be disconnected, I would iterate that list and call the same DFS for any node not yet in the map, then return all of the clones. I would not restart with a fresh map per component, or edges between them would be cloned twice. If I were only given one node of a disconnected graph, the unreachable component is not observable and cannot be cloned. I would say that limit rather than invent nodes.
