# 35. Serialize and Deserialize Binary Tree — Interview Q&A

### 1. Complexity

**Q:** How many tokens do you emit, and what are the time and memory costs of both directions? Why is the string `O(n)` and not `O(n log n)` or worse?

**A:** A binary tree with `n` nodes has `n + 1` null child pointers if you count every missing child (inductively: a single node has two; adding a node replaces one null with a node that brings two new nulls, net +1 null). The preorder writer emits each node once and each null once, so `2n + 1` tokens. Each token is a comma plus a number whose character length is `O(1)` for fixed-width ints, or `O(log |v|)` if you count bits; under the usual word-cost model the string is `O(n)`. Both walks do constant work per token: time `O(n)`. Memory is the string, the token deque `O(n)`, and the recursion stack `O(h)`, which is `O(n)` on a spine. Building the string by repeated immutable concatenation in a loop can become quadratic; `StringBuilder` or a list joined once keeps it linear.

### 2. Invariant

**Q:** What does one `read()` call consume, and why can left and right share one queue?

**A:** `read()` consumes exactly the token sequence of one subtree: either a single `#`, or a value token plus the entire left subtree sequence plus the entire right subtree sequence. That matches what `write` emitted for that subtree. Because the left sequence is a prefix that `read` fully consumes before it returns, the next token is the first token of the right subtree. No index arithmetic is required. The invariant fails if either side writes right-before-left or skips a null, because then the number of tokens `read` expects is not what remains in the queue. At the end of the root call the queue is empty. If it is not, the string was not produced by this writer.

### 3. Off-by-one

**Q:** You write a comma after every token, including the last, and in Java you `split(",")`. You also treat an empty token as a null. Where does a leaf go wrong?

**A:** `write` of a leaf `1` produces `1,#,#,` if you always append a comma. `String.split(",")` with limit 0 discards the trailing empty string, so the tokens are `["1", "#", "#"]` and you happen to survive. Now encode null as an empty token instead of `#`: a leaf is `1,,,` which splits to `["1"]` after trailing empties are dropped. `read` builds `1`, then the left `read` polls nothing, returns null, and the right `read` does the same. You still get a leaf, by accident, but a node whose right child is missing and whose left child exists can lose the empty marker that separated them, and two consecutive nulls at the end disappear. The off-by-one is "one missing token at the boundary." Fix it by using a non-empty marker and not relying on a trailing comma, or by `split(",", -1)` which keeps empties. Do not define null as "no characters."

### 4. Follow-up

**Q:** Serialize a BST so the string is smaller, or support an n-ary tree, or make the format human-readable level order.

**A:** A BST can be serialized from preorder values alone if you also store the bounds: the next value belongs in this subtree only when it lies inside `(low, high)`, otherwise you return and leave the value for an ancestor. You can omit null markers. That uses the BST property and is wrong for a general binary tree, which is this problem. An n-ary tree needs a child count in front of the children, or an end-of-children marker, because there is no fixed "left then right." Level-order format is the queue version: write values left to right, write `#` for missing children, and on the way back use a queue of parents and attach two tokens per parent. It is equally `O(n)`. I only switch formats if both methods change together.

### 5. Recursion vs iteration

**Q:** The recursive reader uses `O(h)` stack. How do you deserialize iteratively, and when does recursion actually fail?

**A:** Iterative preorder is awkward because you must remember whether the next token is a left child or a right child of which ancestor. Level-order iteration is the natural loop: parse all tokens, the first real token is the root, push it, and then each dequeued node consumes the next two tokens as left and right, enqueueing only real nodes. Stack and queue are `O(n)` worst case either way. Recursion fails in practice on a spine of length near the platform stack limit, often a few thousand to tens of thousands of frames, even though `n` itself is acceptable. If the constraints allow a long chain, I mention the iterative level-order pair. For an interview on a balanced example I write the recursive preorder, because the invariant is one sentence.

### 6. Null cases

**Q:** Null root, a leaf, a node with only a right child, value `0`, and value `Integer.MIN_VALUE`. Which of these is a `#`?

**A:** Null root: the whole string is `#`, and `read` returns null without creating a node. A leaf `v`: `v,#,#`. The two hashes are the children, not a second encoding of `v`. Only a right child: `v,#,right...` so the left `read` consumes one `#` and the right `read` consumes the subtree. Value `0` is the token `0`, never `#`. `Integer.MIN_VALUE` is the token `-2147483648`, which `parseInt` accepts, including the leading minus; it must not be split into a sign and a number. A `#` appears only where a child pointer is null. If `deserialize` is handed an empty string, returning null is a reasonable guard, but this writer never produces an empty string: it produces at least `#`.
