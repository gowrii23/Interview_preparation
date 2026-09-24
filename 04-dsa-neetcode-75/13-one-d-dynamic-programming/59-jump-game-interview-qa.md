# 59. Jump Game — Interview Q&A

## 1. Why this state?

A full DP state is `reach[i]`, true when some earlier reachable index can cover `i`. That state is correct and redundant. Because a jump covers a whole interval, the true indices are exactly `0..farthest` for a single integer `farthest`. The integer is a compression of the boolean row, not a different problem. You update it only while the scan index is still inside the prefix; an index outside it cannot lend its jump length, which is why the algorithm returns false instead of reading `nums[i]` past the frontier.

## 2. How do you optimize space?

The boolean row is `O(n)` memory and, naively, `O(n^2)` time. The farthest integer is already `O(1)` memory and `O(n)` time. The right-to-left `goal` index is the same bounds: one integer, one pass. There is no second rolling variable to invent. If you need the actual jump list, you will store predecessors and the memory goes back to `O(n)`; the decision version does not.

## 3. Variant: minimum number of jumps to the end. What changes?

Reachability is not enough, because many reachable indices have different jump counts. This is Jump Game II. The clean greedy still uses a frontier, but the frontier is a BFS layer: scan from the current layer’s left to its right, track the farthest index any jump in that layer can touch, and each time you finish a layer you add one jump and set the next layer’s end to that farthest. You stop when the layer covers `n - 1`. Time stays `O(n)`. A DP `minJumps[i] = 1 + min(minJumps[j])` over `j` that reach `i` is correct and quadratic. Do not answer the boolean problem with the jump counter, and do not answer the counter with a pure yes/no farthest check.

## 4. Why is there never a hole behind `farthest`?

Suppose `F` is reachable via a jump from some `j ≤ F` with `j + nums[j] ≥ F`. Every index between `j` and `F` is a legal landing of that same jump, because the move may be shorter than `nums[j]`. Indices before `j` are reachable by the same argument applied to however `j` was reached, down to index 0 which is reachable by standing still. So a reported farthest index drags the entire prefix with it. The invariant of the loop is: at the start of iteration `i`, every index `≤ farthest` is reachable, and no index beyond it is. The update `farthest = max(farthest, i + nums[i])` extends the prefix using a newly confirmed reachable start.

## 5. What bug does this code have?

```java
int farthest = 0;
for (int i = 0; i < nums.length; i++) {
    farthest = Math.max(farthest, i + nums[i]);
}
return farthest >= nums.length - 1;
```

It never checks that `i` itself was reachable. On `[3, 2, 1, 0, 4]` the last cell sets `farthest` to `4 + 4 = 8` and the method returns true. Index 4 cannot be used as a launch pad. The fix is to return false when `i > farthest` before you read `nums[i]` as a jump. The early return when `farthest` already covers the end is optional; the hole check is not.

## 6. What do you say for `[0]`, `[1, 0]`, and `[0, 1]`?

`[0]`: already there, true. `[1, 0]`: from 0 you reach 1, which is the end, true, even though the last value is 0. `[0, 1]`: index 0 cannot launch, farthest stays 0, index 1 is a hole, false. These three separate “value 0 on the last cell” from “value 0 blocking the only path.” Also say you may jump past the end in the sense that `i + nums[i] >= n - 1` is success; you do not need a landing spot beyond the array.
