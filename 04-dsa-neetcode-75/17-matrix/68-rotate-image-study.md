# 48. Rotate Image

## Problem in my own words

You are given an `n` by `n` matrix of integers. Rotate it 90 degrees clockwise, and do the rotation inside the same matrix. You may use a few scalar variables, but not another `n` by `n` board. Every cell moves to a new index except the center cell when `n` is odd, which stays where it is.

## Easy analogy

A square sticky note is rotated a quarter turn to the right on the desk. The top edge becomes the right edge, the right edge becomes the bottom edge, and so on. You can do that by picking up four corners at a time and cycling them, one ring of the note after another, from the outside in.

## Diagram

One cycle on the outer ring. The dotted arrow is the saved top value landing in its final cell; that cell is done for this cycle.

```mermaid
flowchart LR
  leftCell["left side cell"] --> topCell["top cell"]
  bottomCell["bottom cell"] --> leftCell
  rightCell["right cell"] --> bottomCell
  saved["saved old top"] -.-> rightCell
```

For the corner of a 3×3, that cycle is `7 → top-left`, `9 → bottom-left`, `3 → bottom-right`, and the saved `1 → top-right`.

## Intuition

Cell `(r, c)` moves to `(c, n - 1 - r)` on a clockwise quarter turn. Doing that write directly overwrites a value you still need, so take the four cells in the cycle together:

- `(first, i)` top
- `(i, last)` right
- `(last, last - offset)` bottom
- `(last - offset, first)` left

where `offset = i - first` and `last = n - 1 - layer`.

Save the top, copy left into top, bottom into left, right into bottom, and the saved top into right. That is one quarter turn for those four positions. Advance `i` until the cell just before `last`. Including `last` would start the next cycle on a cell this ring already finished.

Repeat for each layer `0 .. n/2 - 1`.

An equivalent in-place method is to transpose across the main diagonal and then reverse every row. It is worth naming if you blank on the cycle. The code below is the cycle, which makes each cell’s destination obvious.

## Tiny walkthrough

```
1 2
3 4
```

`layer = 0`, `first = 0`, `last = 1`, only `i = 0`.

- Save `1`.
- Top-left becomes left cell `3`.
- Bottom-left becomes bottom-right `4`.
- Bottom-right becomes top-right `2`.
- Top-right becomes saved `1`.

```
3 1
4 2
```

Which is the clockwise rotation. On a 3×3 the same ring runs two offsets, and the center `5` is outside `n/2` layers so it stays.

## Java

```java
class Solution {
    public void rotate(int[][] matrix) {
        int n = matrix.length;
        for (int layer = 0; layer < n / 2; layer++) {
            int first = layer;
            int last = n - 1 - layer;
            for (int i = first; i < last; i++) {
                int offset = i - first;
                int top = matrix[first][i];
                matrix[first][i] = matrix[last - offset][first];
                matrix[last - offset][first] = matrix[last][last - offset];
                matrix[last][last - offset] = matrix[i][last];
                matrix[i][last] = top;
            }
        }
    }
}
```

## Python

```python
def rotate(matrix: list[list[int]]) -> None:
    n = len(matrix)
    for layer in range(n // 2):
        first = layer
        last = n - 1 - layer
        for i in range(first, last):
            offset = i - first
            top = matrix[first][i]
            matrix[first][i] = matrix[last - offset][first]
            matrix[last - offset][first] = matrix[last][last - offset]
            matrix[last][last - offset] = matrix[i][last]
            matrix[i][last] = top
```

## Complexity

Each cell is moved a constant number of times, so time is `O(n^2)`, which is optimal because every cell may change. Extra memory is `O(1)`: the layer indexes and one temporary `top`. The transpose-and-reverse alternative has the same bounds.

## Pitfalls

- Looping `i <= last`. The last index of the edge belongs to the next corner’s cycle, which this iteration has already written. The ring comes back unchanged or scrambled depending on how many extra steps you take.
- Cycling in the wrong direction and producing a counterclockwise board: left into right, and so on. Check a 2×2 before you defend the code.
- Reversing each row and then transposing. That is counterclockwise. Clockwise is transpose first, then reverse each row.
- Allocating `int[][] rotated = new int[n][n]` when the prompt says in place. Mention it only as the easy version you are not submitting.
- Forgetting the inner layers. A 4×4 has two rings; rotating only the outside leaves the middle 2×2 unmoved.

## Interview script

“I’ll rotate layer by layer from the outside in. For each offset along the top of the layer I hold the top cell in a temp and cycle left → top, bottom → left, right → bottom, temp → right. The inner loop stops before the far corner so I don’t rotate the same four cells twice. Odd centers stay put. That’s `O(n^2)` time and constant extra memory. If I get stuck I can transpose and reverse each row, which is the same permutation.”
