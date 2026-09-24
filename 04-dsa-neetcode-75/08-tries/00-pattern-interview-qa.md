# Interview Q&A: trie pattern

## Q1. Why use a trie instead of a hash set of the words?

**Answer:** A hash set answers exact membership in expected O(length) time, and that is enough when every query is a finished string. A trie pays the same linear cost for an exact query and also stops early on a missing prefix, lists the legal next letters in O(1) or O(alphabet) time, and lets many words share the nodes for a common start. Wildcard search and board search are walks that branch on the trie's children. A hash set forces you to either materialize every possible substitution or scan the whole dictionary at every step. Use the hash set when the only operation is "is this complete word present?" Use the trie when the next character is a choice.

## Q2. What is the difference between a node that exists and a node that ends a word?

**Answer:** Existence means at least one inserted word passes through this prefix. The end bit means at least one inserted word stops here. After inserting only `apple`, the node for `app` exists and its end bit is false, so search fails and starts-with succeeds. Inserting `app` flips that bit and does not remove the children that spell `le`. Deleting a word, if you add that operation, clears the end bit and only removes nodes that are no longer an end and have no children, walking from the leaf back toward the root.

## Q3. How do you choose between an array of 26 children and a hash map?

**Answer:** Lowercase English letters make a `Node[26]` natural: child lookup is an index calculation, and the code stays branch-predictable. The cost is 26 references on every node, including nodes with one child. A hash map stores only the letters that actually occur, which wins when the alphabet is Unicode or when most nodes are sparse. Asymptotically both walks are still O(length). In an interview I name the alphabet out loud and pick the array when the problem says lowercase letters. I do not mix the two in one trie without a reason.

## Q4. A wildcard `.` can be any letter. What changes in the walk, and what is the worst case?

**Answer:** A concrete letter still follows one child, or fails if that child is missing. A dot loops over every existing child and accepts the word if any of those recursive walks accepts the suffix. The `.` does not match "empty"; it matches exactly one character, so the index advances by one in every branch. With a pattern of `L` dots and a dense trie, the search is Θ(26^L) node visits. Real dictionaries are sparse, so the branching factor is the number of children actually stored, not always 26. This is still a search explosion, which is why the trie matters: you only branch into letters that some inserted word actually used.

## Q5. During a board search you both mark cells and delete trie nodes. Why are those two different undos?

**Answer:** The cell mark is local to the current path. The same cell may be used by a different path later, so DFS must unmark it on the way back, exactly like backtracking. The trie deletion is global and permanent: once the only word through a node has been recorded, that prefix can never contribute a new answer, from this cell or from any later start cell. Unmarking the trie would make later cells rediscover a word you already emitted. Deleting a trie node before the recursive exploration of its children is the bug; children are explored first, and the node is removed only when it has no end-word left and no children left.

## Q6. What do you say if the interviewer asks for the complexity of "building the trie, then querying"?

**Answer:** Building is linear in the total number of characters inserted, because each character creates or follows one edge. One exact query or prefix query is linear in the query length, independent of the dictionary size except for the constant-time (or map) child lookup. The space is the number of nodes, which is at most one per inserted character and less when prefixes overlap. I then add the query-specific blow-up if there is one: wildcards can be exponential in the number of dots, and a board DFS is exponential in the word length times the number of cells, with trie pruning cutting branches that no longer contain words. I do not quote a single big-O that hides which phase it refers to.
