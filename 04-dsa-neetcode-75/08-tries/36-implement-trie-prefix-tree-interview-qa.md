# Interview Q&A: Implement Trie (LeetCode 208)

## Q1. Walk through `insert("apple")`, then `search("app")` and `startsWith("app")`.

**Answer:** Insert creates five nodes under the root and sets `end` only on the node that corresponds to the full string `apple`. The node for the prefix `app` exists because the walk passed through it, but nothing ever stopped there, so `end` is false. `search("app")` successfully walks three edges and returns false because of that bit. `startsWith("app")` returns true because a missing child is the only failure mode for a prefix. A second `insert("app")` does not allocate nodes; it sets `end` on the node that is already there, and then search returns true.

## Q2. Why is the root node not allowed to represent a letter?

**Answer:** The root is the empty prefix, which every word shares before its first character. If the root itself were the first letter, words that start with different letters would need different roots and the structure would no longer be one tree. Operations start at the root and consume zero characters so far. The end bit on the root would mean "the empty word was inserted," which this problem's tests do not require, but the same code handles it if `insert("")` is called.

## Q3. Can you support `search` and `startsWith` with one private helper without mixing their meaning?

**Answer:** Yes. The helper returns the node reached by the string, or null if any edge is missing. `startsWith` returns `node != null`. `search` returns `node != null && node.end`. The bug to avoid is folding the end check into the helper, which makes a prefix query unimplementable, or forgetting it, which makes search accept prefixes. Returning the node also leaves room for a delete operation or a "list words with this prefix" operation later: the caller already sits on the right subtree.

## Q4. What is the time and space, and where does prefix sharing show up?

**Answer:** Each operation looks at each character of its argument once and does a constant-time child jump, so time is O(length) regardless of how many words are stored. Space is proportional to the number of nodes. Inserting `apple` then `apply` allocates the first word's five letters and then one extra node for `y`, not five new nodes. The worst case is still the sum of the lengths, when no two words share a prefix. A size-26 array does not change the time bound; it increases the memory per node by a constant factor of 26 references.

## Q5. How would you delete a word, and what must you not delete?

**Answer:** Walk to the end node. If it does not exist, or its end bit is already false, there is nothing to delete. Clear the end bit. Then, walking back toward the root (recursion makes this natural), remove a node only when its end bit is false and it has no children. Deleting `apple` while `app` remains must leave the `app` node in place because its end bit is true. Deleting `apple` when it was the only word may remove `e, l, p, p, a` all the way up. Deleting `app` while `apple` remains only clears the end bit on `app`; the `l` child keeps that node alive.

## Q6. The input promise is lowercase English letters. What do you say if the interviewer widens it?

**Answer:** I would replace `Node[26]` and `ch - 'a'` with a map from character to child, and keep the end bit. The walk, the null-means-miss rule, and the difference between search and starts-with do not change. I would not hash the entire word inside the trie node; that throws away the prefix structure. If they also want case-insensitive search, I normalize at insert and at query time rather than storing two edges per letter. If they want counts ("how many times was this word inserted?"), the end bit becomes an integer and starts-with still ignores it.
