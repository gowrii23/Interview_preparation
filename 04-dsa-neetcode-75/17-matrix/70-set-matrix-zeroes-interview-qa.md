# Set Matrix Zeroes — Interview Q&A

### 1. Why do you need two booleans if the first row and first column are already the marker arrays?

**Answer.** `matrix[0][0]` is the marker cell for column 0 and also the marker cell for row 0. Those answers differ. Example: a zero at `(0, 1)` must clear row 0 and must not, by itself, clear column 0. If you store “row 0 is dirty” by writing `matrix[0][0] = 0`, the column-0 pass will also wipe column 0. Read both original borders into booleans first, and never recover those two facts from the corner after you have started writing markers.

### 2. What is the required order of the passes, and which order is fatal?

**Answer.** (1) Snapshot the two borders. (2) Mark from interior zeros into the borders. (3) Zero the interior from the markers. (4) Zero the borders from the booleans. Swapping (3) and (4) wipes the column markers in row 0 before the interior is filled, so columns that should die stay alive, and a first row that was all nonzero can also be cleared too early and zero every interior column. The snapshot has to be step 1 even if the later writes would have “rediscovered” some zeros, because step 2 overwrites the corner.

### 3. Walk `[[0,1,2,0],[3,4,5,2],[1,3,1,5]]`.

**Answer.** `firstRowZero` is true (zeros at columns 0 and 3). `firstColZero` is true (zero at row 0). Interior zeros: there is no interior zero; the zeros sit on the border. Markers in the interior pass do not change. The interior fill sees column marker `matrix[0][3] == 0`, so column 3 of rows 1 and 2 becomes 0. Row markers `matrix[1][0]` and `matrix[2][0]` are still nonzero, and `matrix[0][1]`, `matrix[0][2]` are still nonzero, so `3,4,5` and `1,3,1` stay, except column 3. Then the booleans wipe row 0 and column 0. Result: `[[0,0,0,0],[0,4,5,0],[0,3,1,0]]`.

### 4. What happens on `[[1]]`, `[[0]]`, a single row, and a single column?

**Answer.** `[[1]]`: both flags false, interior loops empty, matrix unchanged. `[[0]]`: both flags true, then both final loops write 0, still `[[0]]`. A single row has no interior. `firstRowZero` is true iff any entry is 0, and the final loop zeros the whole row, which also zeros every column. A single column is the same with `firstColZero`. You do not need a special case; the empty interior loops are the special case.

### 5. A zero is written into an interior cell during the fill. Why doesn’t that accidentally clear another row?

**Answer.** The fill pass only reads markers. It does not call the marker logic again. And the markers were taken from the original interior zeros, before any fill write. So a freshly written zero is an output, not a seed. Combining “if I see a zero, mark and also clear now” in one loop is how that bug appears.

### 6. What complexity do you quote for the `O(m+n)` version and the `O(1)` version?

**Answer.** Both are `O(mn)` time. The comfortable version stores two boolean arrays, `O(m+n)` extra memory, and is a good thing to say first so the invariant is clear. The follow-up replaces those arrays with row 0, column 0, and two booleans, which is `O(1)` extra memory. You cannot do better than `O(mn)` time in the worst case because every cell can affect the answer and every cell may need to be written.
