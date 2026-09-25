# Interview Q&A: Word Search II (LeetCode 212)

## Q1. Why is "run Word Search I for every dictionary word" not the solution you want?

**Answer:** Word Search I is a board DFS for one string. Repeating it for every word restarts from every cell and re-walks shared prefixes. `oat`, `oath`, and `oaths` would each pay for the `oa` prefix from scratch. A trie searches that prefix once per board path: the walk follows one edge per cell, and sibling words are just different children. You also get an early exit the moment the current prefix is not in the dictionary at all. The board DFS complexity does not disappear — a path can still branch four ways — but the dictionary no longer multiplies the search as a separate outer factor of "number of words," except insofar as those words create trie branches the board can actually follow.

## Q2. Explain the two different prunes. Which one is undone?

**Answer:** The board prune is temporary. A cell on the current path is marked so the same path cannot re-enter it, and the letter is restored when that stack frame returns. The next neighbor, the next start cell, and a different word may all need that letter. The trie prune is permanent. After exploring a child's subtree, if that child no longer holds a word and has no children, the edge is deleted. Every word through that prefix has already been collected, so every future start should fail fast. Undoing the trie deletion would resurrect finished work. Doing the trie deletion before the recursive calls deletes longer words you have not searched yet.

## Q3. How can `oa` and `oath` both be reported? What exactly is stored on the nodes?

**Answer:** The node for `oa` stores the string `oa` (or an end bit plus a way to recover the string). It also has a child `t` leading to `h`, which stores `oath`. When the path reaches the `oa` node I append `oa` and clear only that stored string. The `t` child is still there, so the DFS continues into `t` and then `h` if the board allows it, and appends `oath`. On the way back, `h` may now be empty and get deleted, then `t`, but the `oa` node stays if it still has other children. If `oath` was the only continuation and `oa` was already cleared, the `oa` node becomes empty too and then its parent drops the `a` edge. Discovery order might list either word first depending on where the paths start. Both belong in the answer if both paths exist.

## Q4. Why is each word added at most once, even if the board contains two copies?

**Answer:** The word lives in one trie node. The first path that reaches that node with the right letters appends the string and sets the field to null (Java) or pops the `"#"` key (Python). A later path may still walk through those nodes if other words need them, but the field is already empty, so it does not append again. If the finished word was a leaf, pruning then removes the node, and later paths cannot even walk there. I do not need a separate result set for correctness under this scheme. A set would also dedupe, and it would hide a bug if I forgot to clear the marker and also forgot to prune.

## Q5. What does the visited mark have to do with a word like `aaa` on a two-cell board `aa`?

**Answer:** The trie will happily walk `a → a → a` because the dictionary word has three letters. The board only has two cells. After the path consumes both cells, both are marked, so every neighbor is either off-board or marked, and the third `a` never matches a cell. The recursion returns false for that continuation, restores both cells, and does not emit `aaa`. The same mechanism allows `aa`: two different cells, each used once. Forgetting the mark makes `aaa` true by standing on one cell three times, which violates the rule. Forgetting the unmark makes a later, legitimate path see a cell as blocked even though no active path is using it.

## Q6. Give the complexity you would say out loud, including space and the board mutation.

**Answer:** Trie construction is linear in the total number of dictionary characters, and that is also the extra space besides the recursion stack. The stack is at most the longest word, because each recursive call consumes one letter and the visit mark prevents cycles. Time is O(cells × 4^(longest word)) in the worst case, with a large practical reduction when the trie rejects a prefix or a finished branch has been deleted. I mutate the board to store the visit mark and I write the original letter back before returning, so when `findWords` returns, every cell holds its original character. If the interviewer forbids mutation, I use a boolean matrix or a set of coordinates and backtrack those the same way: add before the recursive calls, remove after. The trie pruning logic does not change.
