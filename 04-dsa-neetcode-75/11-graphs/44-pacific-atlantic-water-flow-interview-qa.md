# Interview Q&A: Pacific Atlantic Water Flow (LeetCode 417)

## Q1. Why is searching from the oceans equivalent to searching from every cell?

**Answer:** Take a path `c0 → c1 → … → border` where each step moves to a neighbor of height less than or equal to the current cell, and the border touches the Pacific. Read that path backward. From the border, each step moves to a neighbor of height greater than or equal to the current cell, and it ends at `c0`. So `c0` can reach the Pacific exactly when an uphill (non-decreasing) walk from some Pacific border can reach `c0`. The same equivalence holds for the Atlantic. I compute those two reachable sets directly. Their intersection is the set of cells with both downhill paths. I do not lose cells that need a long winding river, because DFS follows every legal uphill step, and visited only skips cells whose reachability for this ocean is already known.

## Q2. Walk the heights `[[1,2],[4,3]]`. Which cell fails, and why?

**Answer:** Pacific seeds are the top row and the left column: `(0,0)=1`, `(0,1)=2`, `(1,0)=4`. Uphill from 1 reaches 2 and then 3, and 4 is itself a seed. Every cell reaches the Pacific. Atlantic seeds are the bottom row and the right column: `(1,0)=4`, `(1,1)=3`, `(0,1)=2`. From 4, both neighbors are shorter, so both steps prune. From 3, neighbors are 2 and 4; 4 is already marked and 2 is shorter, so no new cell. From 2, the uphill neighbor is 3, already marked, and 1 is shorter, so the step into 1 is refused. `(0,0)` has Pacific paint only. The answer is `(0,1), (1,0), (1,1)`. The tempting mistake is to include `(0,0)` because it borders the Pacific and "the island is small." Bordering one ocean is not bordering both, and its uphill neighbors do not connect it back to an Atlantic seed.

## Q3. What goes wrong with a shared visited matrix, or with the comparison `next <= current`?

**Answer:** A shared visited matrix means the Atlantic DFS treats Pacific-reached cells as finished. On a flat grid every cell is reachable from both oceans, but the Pacific search would mark them all first and the Atlantic search would mark nothing new. The intersection would be only the cells that happen to be Atlantic seeds, which is wrong: interior cells of a flat island drain to both oceans. The flipped comparison `next <= current` walks downhill from the beach. It answers "where could ocean water run inland," which is a different physical question. On a strictly increasing slope away from the Pacific, the forward-from-ocean walk stops at the border, and inland cells that easily drain to the Pacific never get painted. I want `next >= current` in the reverse search, including equality.

## Q4. Why are equal heights allowed, and what does a 1×1 grid return?

**Answer:** The flow rule says water moves to a neighbor that is lower or the same height. A plateau is one puddle, not a wall. If the plateau touches an ocean anywhere on its border, every cell of that plateau can reach that ocean by sliding across equal cells. A strict `>` would paint only the border cell and leave the interior false. A 1×1 grid is both a Pacific border and an Atlantic border. Both searches mark that single cell, and the intersection is `[[0,0]]`. I do not special-case it beyond the loops: row 0 and row `n-1` are the same row, column 0 and column `m-1` are the same column, and marking a cell twice in one ocean is a no-op because of visited.

## Q5. DFS versus multi-source BFS. Which do you pick, and what is the frontier?

**Answer:** Both are O(cells). Multi-source BFS pushes every Pacific border cell at the start, all marked, and that queue is the frontier. Each pop climbs into unmarked neighbors of greater or equal height and pushes them. The Atlantic is a second BFS with its own queue and its own matrix. I like BFS when the grid is large enough that a snake of increasing heights might blow the recursion stack. I like DFS in an interview when I want twenty fewer lines and the constraints are a few hundred cells. The answer set does not depend on the choice. The dotted edges are the same in both pictures: a downhill neighbor that is not enqueued and not recursed into.

## Q6. State the complexity and two constraints you use without restating them as code.

**Answer:** Time O(rows · cols), because each cell is entered at most once per ocean and each entrance inspects four neighbors. Space O(rows · cols) for the two matrices, plus stack or queue. I rely on rectangular input, non-negative heights only in the sense that the comparison is numeric and total, and on 4-directional movement. I do not assume heights are unique. I do not assume a cell belongs to only one ocean. The output order is the scan order of the intersection, row by row. If a judge compared lists without sorting, I would sort both sides in the test, not change the algorithm, because the problem accepts any order. I also do not run a third search from interior cells to "confirm." Two paints are the confirmation.
