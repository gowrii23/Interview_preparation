# Interview Q&A: BFS, DFS, and visited

## Q1. When do you choose BFS, and when do you choose DFS?

**Answer:** I choose BFS when distance matters and every edge has the same cost. The queue guarantees that the first time I reach a node I reached it with the fewest edges. I choose DFS when I want to finish a whole component, when the state is a path I will undo (backtracking on a grid), or when I am detecting a directed cycle with colors. For "is there a path?" and "how many components?" either is correct. I do not use DFS and then claim the path I found is shortest. I also do not use recursive DFS on a graph that might be a single path of 10^5 nodes unless I know the runtime stack can hold it. Then I switch to an explicit stack or to BFS, same visited rules.

## Q2. Why mark visited when you enqueue, not when you dequeue?

**Answer:** The queue is the frontier of discovered-but-not-expanded nodes. If I wait until dequeue to mark, every edge into a node enqueues another copy before the first copy is popped. A dense graph then processes the same node many times, and the O(V + E) bound becomes false in practice. Marking at enqueue means the second edge sees the node as already discovered and does not push it. The dotted edges in a BFS diagram are exactly those rejected rediscoveries. The same rule in DFS is "mark before recursing on the children," which is usually the first line of the recursive function. Marking after the recursive calls lets a cycle re-enter the node.

## Q3. What does a boolean visited set get wrong on a directed graph if the question is "is there a cycle?"

**Answer:** A boolean set only says "I have been here before." In a directed graph, reaching a node that already finished is often a cross edge or a forward edge, and it is legal in a DAG. Example: node 0 points to 1 and 2, and 1 points to 2. DFS finishes 2, then 1 sees 2 already visited. That is not a cycle. A cycle is a back edge into a node that is still on the active recursion stack, an ancestor of the current node. Three colors encode that. White means unseen, gray means on the stack, black means finished. Only a gray neighbor proves a cycle. Kahn's algorithm sidesteps colors: if a topological peel cannot emit every node, something had a remaining indegree and that something sits on a cycle.

## Q4. How do you represent a grid as a graph without building an adjacency list?

**Answer:** The cell `(r, c)` is the node. Its edges are the four coordinate deltas `(1,0), (-1,0), (0,1), (0,-1)`, filtered by the bounds and by whatever extra rule the problem adds, such as "neighbor is land" or "neighbor height is at least this height." I do not allocate a `Map` from cells to lists. The bounds check is the missing-edge test. Visited is a matrix of the same shape, or a mutation of the cell when the problem allows it. Diagonal moves are extra edges and I do not add them unless the problem says "8-connected." Time is still O(number of cells) for a full flood, because each cell is enqueued at most once and has a constant degree.

## Q5. A recursive DFS works in the interview room and fails on a long input. What happened?

**Answer:** The call stack is one frame per recursive edge. On a path graph, or a snake-shaped island, that is O(n) frames. Language stacks are often smaller than the heap, so a graph that fits in memory can still crash the recursion. The algorithm is not wrong; the implementation's hidden stack is. I rewrite it as BFS, or as DFS with an explicit stack of nodes (and, if I need parent information or colors, an explicit stack of states). I do not "fix" it by enlarging the visited array. I also watch for the opposite bug: an explicit stack that pushes a neighbor every time it is seen, without a mark, which recreates the infinite loop the recursion had on a cycle.

## Q6. What is the time and space you quote for a standard traversal, and what do you count as E on a grid?

**Answer:** For an adjacency list, time is O(V + E) and extra space is O(V) for visited plus the queue or stack. I count an undirected edge twice if it is stored on both endpoints, and that constant 2 disappears into the O. On a grid, V is `rows * cols` and E is at most `4 * rows * cols`, so both time and visited space are O(rows · cols). I do not say O(V + E) and leave the interviewer to guess what V is. If the search stops early, the bound is still the worst case, and I mention that the best case can return at the first cell. Output space is separate when the output is a list of coordinates or a cloned graph.
