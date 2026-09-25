# 34. Binary Tree Maximum Path Sum (LC 124)

**Link:** https://leetcode.com/problems/binary-tree-maximum-path-sum/

## Problem in your own words

A path in a binary tree is a chain of nodes where each step goes to a parent or a child and no node repeats. The path may start and end anywhere; it does not have to pass through the root, and it does not have to be a root-to-leaf path. Return the maximum sum of node values along any such path. Node values may be negative, so the best path might be a single node, and a negative child must not be forced into the sum.

## Easy analogy

Each person can offer their boss the best single route that starts at them and goes downward: their own score plus the better of the two routes their reports offer, and they refuse a report whose route is negative. Separately, the company records a "best day" that is allowed to combine both reports under the same person, because that path turns around there and never needs to go up to the boss. The answer is the best day anyone recorded, not the route that was offered upward.

## Diagram

```mermaid
flowchart TB
    A((-10)) --> B((9))
    A --> C((20))
    C --> D((15))
    C --> E((7))
    A -.->|upward gain is not the answer| G["best is 15+20+7"]
```

```
       -10
       /  \
      9    20
          /  \
         15   7

At 20, both children pay off. The turn-around path is 15 + 20 + 7 = 42.
The gain handed upward is 20 + 15 = 35, only one side.
At -10, left gain 9 and right gain 35, turn-around is 34, which loses to 42.
Answer 42. The dotted arrow is the upward offer, which is not what we return
to the caller. The caller wants the global best.
```

## Intuition

For each node, define `gain(node)` as the best sum of a downward path that **starts at this node** and goes only down, through one child chain. The parent can extend at most one side, so:

```
left  = max(0, gain(left child))
right = max(0, gain(right child))
gain(node) = node.val + max(left, right)
```

`max(0, ...)` drops a negative child path. You would never walk into it if your goal is the sum.

The best path that has this node as its highest point (the bend) is `node.val + left + right`. That path is legal: it goes down the left chain, through the node, and down the right chain. It cannot be extended upward, so it must be recorded in a global `best`, not returned to the parent.

A leaf with a negative value still updates `best` to that value, because both gains are 0 and the path of one node is allowed. Initialize `best` to a very small number so an all-negative tree does not stay at 0.

The function the problem asks for returns `best` after the walk, not the gain of the root.

## Step-by-step tiny walkthrough

Tree: `-10`, left `9`, right `20` with left `15` and right `7`. `best` starts at a tiny number.

1. `gain(9)`: both children 0. Bend `9`. `best = 9`. Return `9`.
2. `gain(15)`: bend `15`, `best = 15`. Return `15`.
3. `gain(7)`: bend `7`, `best = 15`. Return `7`.
4. `gain(20)`: left contribution `15`, right `7`. Bend `20 + 15 + 7 = 42`. `best = 42`. Return `20 + 15 = 35` (the better side only).
5. `gain(-10)`: left contribution `max(0, 9) = 9`, right `max(0, 35) = 35`. Bend `-10 + 9 + 35 = 34`. `best` stays `42`. Return `-10 + 35 = 25`.
6. The answer is `42`, not `25`.

All-negative check: tree is `-2` with left `-1`.

1. `gain(-1)`: contributions 0 and 0. Bend `-1`. `best = -1`. Return `-1`.
2. `gain(-2)`: `max(0, -1) = 0`, other side 0. Bend `-2`. `best` stays `-1`. Return `-2`.
3. Answer `-1`. You did not "improve" it to 0 by skipping every node. The empty path is not allowed; a path has at least one node.

## Java

```java
class Solution {
    class TreeNode { int val; TreeNode left, right; TreeNode(int x){val=x;} }

    private int best;

    public int maxPathSum(TreeNode root) {
        best = Integer.MIN_VALUE;
        gain(root);
        return best;
    }

    /** Best downward path that starts at node. Also records the best bend. */
    private int gain(TreeNode node) {
        if (node == null) return 0;
        int left = Math.max(0, gain(node.left));
        int right = Math.max(0, gain(node.right));
        best = Math.max(best, node.val + left + right);
        return node.val + Math.max(left, right);
    }
}
```

## Python

```python
class TreeNode:
    def __init__(self, x):
        self.val = x
        self.left = None
        self.right = None

class Solution:
    def maxPathSum(self, root):
        self.best = float("-inf")

        def gain(node):
            if not node:
                return 0
            left = max(0, gain(node.left))
            right = max(0, gain(node.right))
            self.best = max(self.best, node.val + left + right)
            return node.val + max(left, right)

        gain(root)
        return self.best
```

## Complexity

**Time `O(n)`.** Each node is visited once. The two `max` operations are constant work.

**Extra memory `O(h)`** for the call stack. The global `best` is `O(1)`. You do not store a path, only the sum. If the interviewer wants the actual nodes, you would store a copy of the best path when you update `best`, which can add `O(n)` memory.

## Pitfalls

- Returning `node.val + left + right` to the parent. That bend cannot continue upward: the parent would be a third arm, and a path cannot fork. The parent receives only one side.
- Forgetting the global and returning the root's gain. On the sample that returns `25` (or `-10 + 35`) instead of `42`.
- Initializing `best` to 0. An all-negative tree would return 0, and there is no empty path in this problem.
- Always adding a child even when its gain is negative. `9 + (-3)` is worse than `9` alone. `max(0, gain)` is the refusal.
- Treating `max(0, gain)` as "skip this node too" when both children are negative. You still add `node.val` into `best`. The 0 only drops a child branch.
- Confusing this with "max root-to-leaf" or with path-sum III (count of paths that sum to a target). Here any node-to-node path is legal, and you want the maximum sum, once.

## 2-minute interview script

"A path can't fork, but it can bend at one node. I'll compute, for each node, the best downward sum I could hand my parent: my value plus the better child chain, and I'll ignore a child whose chain sums to a negative number by taking max with zero. Separately I'll consider the path that bends here, which is my value plus both child contributions, and I'll keep the maximum bend in a global. The answer is that global, not the gain at the root, because the best path might live entirely in one subtree. I initialize the global to the most negative int so a tree of all negatives still returns the largest node. A null child contributes zero and doesn't update the global. Time is linear, stack is the height. On the sample, the bend at twenty is fifteen plus twenty plus seven, forty-two, and the root's bend is worse because of the negative ten. The bugs I watch are handing both children up to the parent, returning the root gain instead of the global, and resetting the answer to zero when every node is negative."
