# Interview Q&A: Design Add and Search Words (LeetCode 211)

## Q1. Why is a trie the right structure once `.` is allowed?

**Answer:** A dot means "any letter that some stored word actually uses here," and the rest of the pattern still has to match from there. The trie's children are exactly that candidate set. A hash set of whole words would make you expand every dot into 26 letters and probe each possibility, which is the dense worst case even when the dictionary is tiny, or scan every word on every search. The trie DFS only enters branches that `addWord` created. Shared prefixes are still stored once, so `bad`, `dad`, and `mad` do not duplicate the logic of "match the suffix `ad`"; each branch just happens to have its own `a-d` nodes because the first letters differ.

## Q2. Trace `search("b..")` after adding `bad`. Where does the function branch, and where does it stop?

**Answer:** Index 0 is `b`, a concrete letter, so there is one call into the `b` child. Index 1 is `.`. The only child under `b` is `a`, so there is one recursive call, not 26. Index 2 is `.` again. The only child under `a` is `d`, and that call advances to index 3. The pattern is finished, and `d.end` is true, so the search returns true up the stack. If `bad` had not been a complete word, the same walk would return false at the end check. If another word `baq` existed, the second dot would try `d` and then `q` until one of them was an end at the right depth.

## Q3. Does `.` match the empty string, or several letters? What about a pattern longer than every word?

**Answer:** `.` matches exactly one character. It does not match zero characters, and one dot does not swallow the rest of a word. A pattern longer than a word fails when the walk asks for a child below a leaf: the child is missing, so that branch is false. A pattern shorter than a word fails the end-bit check unless some other word actually ends at that shallower node. After adding only `bad`, search(`ba`) is false and search(`b.d`) is true. Length is part of the match.

## Q4. What is the worst-case time, and why do real dictionaries usually beat it?

**Answer:** Add is linear in the word. Search with a pattern of length `L` that is all dots can visit every node in the first `L` levels. If every node has 26 children, that is on the order of 26^L visits. Stored dictionaries are sparse: most child slots are null, and the Java loop over the array skips those nulls in constant time per slot. The practical branching factor is the average number of children, which is small when words share structure or the dictionary is modest. I still mention the exponential bound so the interviewer knows I did not claim O(length) for a dotted query.

## Q5. How do you avoid treating the end marker as a child in a hash-map trie?

**Answer:** I either keep children and the end bit in different fields (a node object, which is what the Java array version does), or I use a reserved key such as `"#"` and skip it when the dot iterates keys. Recursing into `"#"` would pass a boolean into a function that expects a dict, or it would look up letters on the wrong object. The base case reads that marker only when the pattern index is done. A letter lookup never uses the marker key because letters and `"#"` are different keys.

## Q6. Could you answer this with BFS instead of DFS? What changes?

**Answer:** Yes. The queue holds pairs of `(node, index into the pattern)`. A letter enqueues one child if it exists. A dot enqueues every existing child, each with `index + 1`. If any dequeued state has `index == len(pattern)` and the node is an end, return true. If the queue drains, return false. The complexity is the same. DFS is shorter and can return on the first hit without generating sibling branches. BFS uses explicit memory proportional to the frontier, which spikes on a run of dots. I choose DFS unless the interviewer asks for the shortest matching word, which this boolean problem does not.
