# 11. Container With Most Water — Interview Q&A

## 1. What is the complexity, and why can you ignore most pairs?

Time is `O(n)` and extra space is `O(1)`. You evaluate one container per pointer move, and there are `n - 1` moves. The pairs you skip are pairs that keep the current shorter line and use a closer partner: those have smaller width and a height still capped by that shorter line, so their area cannot beat the area you already computed for that line at the current wider position. That is the proof, not a heuristic.

## 2. Why move the shorter line instead of always moving the right pointer or binary searching?

The width only shrinks. The max area for a fixed limiting height is achieved at the greatest width, which is why you start outside and only abandon a line once it has been the limiter at the current width. Always moving one side misses a tall line on the other side. Binary search does not apply because the area is not monotonic in the index: it depends on a min of two heights and a width.

## 3. What bug does this code have?

```java
int area = Math.min(height[left], height[right]) * (right - left);
if (height[left] < height[right]) right--;
else left++;
```

Two bugs. The multiply can overflow a 32-bit `int` when heights and width are large; compute it in a `long`. The move is backwards: when the left line is shorter, the code moves `right`, which discards the taller side and can never raise the limit set by `left`. A third bug is using `right - left + 1` as the width. The distance between indices `0` and `8` is `8`, not `9`.

## 4. Follow-up: return the indices as well, or solve trapping rain water. What changes?

For indices, remember `bestLeft` and `bestRight` whenever the area improves. Still `O(n)`. Trapping rain water is a different total: water above each index is `min(maxLeft, maxRight) - height[i]`, summed over the array. Two pointers still work, but you add water at the shorter side and advance it, using the max height seen on each side. Do not reuse the container formula for that sum.

## 5. What if the constraints change and heights can be `10^9` with `n` at `10^5`, and you need the exact area?

The product can be `10^14`, which fits in a 64-bit `long` and does not fit in a 32-bit `int`. Return `long`. If heights can be arbitrary big integers, use a big-integer type; in Python the same code already returns the exact area. Time stays linear. If they also want every optimal pair, you may have to store several answers, but you still should not fall back to all pairs unless the output size forces you to list them.

## 6. What if two lines have equal height, or the array has only two lines?

Equal heights: both are the limiter. Moving either one is correct for the maximum area, because any better container does not need both of these equal lines at this width. The `<=` branch moves left. Two lines: the loop runs once, the area is `min(h0, h1) * 1`, and then the pointers meet. That single pair is the answer. An array of one line cannot form a container; the constraints guarantee at least two.
