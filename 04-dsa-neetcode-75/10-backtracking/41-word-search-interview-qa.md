# Interview Q&A: Word Search (LeetCode 79)

## Q1. What state does each recursive call need, and what does it deliberately not need?

**Answer:** It needs the current cell and the index of the next character to match. The path itself does not need to be stored, because the word is given and the index says how far along it the path already is. The visit state is either the sentinel sitting in the grid or a set of cells on the current path. I do not pass a copy of the board down. Children see the parent's mark through the shared grid, and they see the mark disappear only after the parent restores it, which is after the children have returned. Passing a fresh board copy would be correct and would blow the memory budget.

## Q2. Why is restoring the letter required if the search is about to return false anyway?

**Answer:** False from this cell means "this path does not work," not "this cell can never be part of any path." A different neighbor of the parent, or a completely different start cell, may still need the original letter. The classic failure is `ABCB` on the small letter board: the `B` is used legally near the front of a path and must be free again for a different attempt. If I only restore on failure paths but leak marks on success, a caller that inspects the board after a true result sees sentinels. I restore on the way out of every frame, and I place that restore before the frame's own return, so both outcomes put the letter back.

## Q3. How do you stop a path from reusing a cell, and how is that different from blocking the cell for the whole search?

**Answer:** The mark is pushed before the recursive calls and popped after them, so it lives exactly as long as the cell is an ancestor of the current call. That is backtracking, and it forbids cycles and reuse inside one path. A global visited set that is never cleared would mean "this cell was tried in some failed path, never use it again," which is wrong. Two disjoint attempts may both need the same letter. Word Search II sometimes deletes trie nodes permanently because the dictionary word has already been found. That permanent delete is about the dictionary, not about the grid cell. The grid cell is still unmarked when the path ends.

## Q4. Trace why `ABCB` is false on the usual three-row sample, even though those letters exist.

**Answer:** The sample board has one `B`. Any path that has consumed `A`, `B`, and `C` has the `B` cell on its stack, marked with the sentinel. The fourth character is `B`. The original `B` cell is a neighbor of `A` and of `C` depending on the route, but it is marked, so the character comparison sees `#` and fails. There is no second `B` elsewhere. Starts that do not begin at the `A` fail the first character. So every branch returns false. If the mark were missing, the path `A → B → C → B` would stand on the same `B` twice and wrongly return true.

## Q5. What is the complexity, and which prune actually helps on normal boards?

**Answer:** Worst case O(`R * C * 4^L`). Each of the `R * C` starts can, in a grid full of the same letter and a word of that letter, branch four ways at every character. Average boards prune at the first mismatched letter, which often happens at depth 1: the start cell's letter is wrong, and the DFS returns before any neighbor call. Checking the first character in the outer loop is the same prune made obvious. It does not improve the worst-case bound. Space is O(`L`) stack frames. I mention both bounds, because an interviewer who has seen only the matching prune sometimes thinks the algorithm is linear in the board size. It is linear only when the word almost never matches a prefix.

## Q6. How does this function change when the follow-up is a list of words?

**Answer:** I stop walking a bare string index and walk a trie node instead. Each board step must be a child edge labeled with the cell's letter. When the trie node says a word ends, I record that word and clear the end marker so I will not record it twice. The grid mark and restore stay exactly as they are here. After a trie node has no end-word and no children left, I delete that edge so later start cells do not rediscover a finished prefix. That deletion is permanent, unlike the grid restore. The outer loop is still "every cell," but it searches all words in one pass instead of calling `exist` once per word.
