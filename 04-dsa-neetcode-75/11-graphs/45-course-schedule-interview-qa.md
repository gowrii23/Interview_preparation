# Interview Q&A: Course Schedule (LeetCode 207)

## Q1. Why is "can finish" the same question as "is the prerequisite graph a DAG?"

**Answer:** Each pair `[a, b]` is a constraint that `b` precedes `a`. A total order of courses exists if and only if these directed constraints have no cycle. If a cycle exists, every course on it has a predecessor that is also on it, so there is no course on the cycle that you are allowed to take first, and you cannot finish those courses. If no cycle exists, a topological order is a legal semester-by-semester plan, even if many courses share prerequisites. Courses outside the pairs have no constraints and can sit anywhere. So the boolean is "does a topological order of all `numCourses` nodes exist?" I do not need the order itself until the follow-up.

## Q2. Walk `numCourses = 2` with `[[1,0]]`, then with `[[1,0],[0,1]]`.

**Answer:** The pair `[1,0]` means 0 before 1. I add the edge `0 → 1` and set indegree `[0, 1]`. The queue starts with course 0. I pop it, decrement course 1 to 0, and push 1. I pop 1. Two pops equal two courses, so the answer is true. Adding `[0,1]` adds the edge `1 → 0` and raises course 0's indegree to 1. Now both indegrees are 1, the queue is empty, and I take zero courses. The answer is false. I do not need a special two-node check. The empty frontier is the cycle report. A self-loop `[0,0]` is the same picture with one node: indegree 1, nothing to pop, false.

## Q3. Why do isolated courses have to start in the queue?

**Answer:** `numCourses` counts every label from 0 to n - 1, not merely the labels that appear in pairs. A course with no edges has indegree 0 and is legal to take immediately. If I only seeded the queue from endpoints of edges, then `numCourses = 3` and prerequisites `[[1,0]]` would pop 0 and 1, stop at taken = 2, and return false even though course 2 is free. The fix is a loop over every course id. Isolated courses are pushed up front. They contribute to `taken` even though their adjacency lists are empty. The same loop is why an empty prerequisite list returns true for any `numCourses`, including 0 if the platform passes it.

## Q4. Compare Kahn and DFS colors. When does a boolean visited set lie?

**Answer:** Kahn peels sources and counts how many nodes it can remove. DFS tries to color the graph: white unseen, gray on the current path, black fully explored. Seeing a gray neighbor means the current path has walked back to an ancestor, which is a cycle. Seeing a black neighbor means that node was finished through some other path. In a DAG that is normal: two courses can both be prerequisites of a third, or both depend on one earlier course. A boolean "I have seen this node" treats that second arrival as a cycle and rejects a schedule that is fine. Example: edges `0 → 2` and `1 → 2`. Node 2 is visited twice and there is no cycle. Gray versus black is what separates "still on my stack" from "already done." Both correct algorithms are O(V + E). I default to Kahn when I may need the order next, because the pop list is the order.

## Q5. You reverse every edge by mistake. What still works, and what fails in the follow-up?

**Answer:** If every edge is reversed, a cycle exists in the new graph exactly when it existed in the original, so `canFinish` still returns the right boolean. Isolated nodes stay isolated. Course Schedule II is where the mistake shows up. The pop order is then a topological order of the reversed constraints, which takes a course before its prerequisite. The reverse of that full list is a valid order of the original graph, so a complete flip can be repaired by reversing the output. A partial flip, where only some edges point backward, is not repaired that way. I avoid the repair puzzle by building each edge as prerequisite → course in the adjacency list, and I record the pop order from that same graph.

## Q6. What is the complexity, and how do duplicate prerequisites or a disconnected graph behave?

**Answer:** Time O(V + E) and extra space O(V + E) for lists, indegree, and the queue. Each edge is relaxed once. The graph may be disconnected; Kahn does not care, because the queue can hold sources from many components at once and the DFS version loops over every unvisited node. Duplicate pairs are not part of the usual input promise. If they appeared, my code would increment indegree twice for one logical constraint and might leave a node stuck after a single real predecessor is taken. The defensive version inserts edges through a set so a repeated pair does not change indegree. I mention that only as a follow-up. For the stated problem I treat each pair as a distinct given edge and rely on the promise that duplicates are absent.
