# Interview Q&A: Number of Islands (LeetCode 200)

## Q1. What is the graph, in one sentence, and why is the answer a component count?

**Answer:** Each land cell is a node, and each pair of land cells that share a side is an undirected edge. Water cells are absent from the graph. An island is a connected component, and the function returns how many there are. Area would be the size of a component. Perimeter would be a property of its boundary. The scan-and-flood algorithm counts a component when it discovers the first unmarked land cell, then marks every cell reachable from it so the scan will not discover that component again. The order of the scan does not change the number of times a new component is discovered.

## Q2. Why sink the cell to water before recursing?

**Answer:** The neighbor relationship is symmetric. If I recurse into the right-hand land before marking the current cell, that neighbor's left-hand move comes straight back, and the two calls alternate until the stack overflows. Marking first breaks the cycle: the return edge sees water and prunes. Marking first is the grid version of "mark visited when you discover the node." Doing the four recursive calls and only then writing `'0'` is the bug. Writing `'0'` also means I do not need a second matrix, with the consequence that the caller's grid is destroyed. I say that trade out loud.

## Q3. How would you write the BFS version, and what is the frontier?

**Answer:** When the scan finds land, I increment the count, mark that cell, and push it. The queue holds land cells that belong to this island and whose neighbors I have not expanded. Each pop looks at four neighbors. A neighbor that is in bounds and still land gets marked and pushed. Marking at push time, not at pop time, keeps a cell off the frontier twice. When the queue empties, the component is done and the scan continues. The dotted edges in the study diagram are cells on that frontier, or neighbor directions that were rejected as water. BFS and DFS return the same count. BFS uses the heap for the queue, which avoids a deep call stack on a large snake island.

## Q4. Two land cells touch only at a corner. How many islands, and what would you change to make it one?

**Answer:** Under the 4-directional rule it is two islands. My delta list is only up, down, left, and right, so the corner cell is never generated as a neighbor and the flood cannot cross. The scan finds it later and increments again. If the interviewer changes the rule to 8-connected, I add the four diagonal deltas and nothing else. I do not "approximately" connect corners by running the flood twice. I would also restate the visited rule, because a diagonal step can still walk back to the cell I just left, and the mark still has to happen before the recursive calls.

## Q5. Can union-find solve this, and when is it a better choice than flood fill?

**Answer:** Yes. Give every cell an index `r * cols + c`. For each land cell, union it with the land cell above and the land cell to the left if those exist. Initialize the component count to the number of land cells, and decrement on every successful union. The final count is the number of islands. Time is nearly O(cells) with path compression and union by rank. I prefer this when the follow-up is dynamic: land appears or disappears and I must answer the count many times. A static flood would rerun from scratch. For one static query, flood fill is less code and the same big-O. I would not build union-find and also flood; one component algorithm is enough.

## Q6. What are the complexity and the empty-input behavior?

**Answer:** Time is O(rows · cols). Each cell is visited by the outer scan, and the flood writes each land cell once and then refuses to enter it. Space is O(rows · cols) worst-case stack or queue. A seen matrix, if I cannot mutate, is that much extra heap space on top of the stack. An empty board, or a board with rows but a zero column count, returns 0 before any indexing. A board of all water returns 0 because the increment never runs. A board of one land cell returns 1. I do not special-case a full rectangle of land beyond the flood; the answer is 1, and the algorithm already produces it.
