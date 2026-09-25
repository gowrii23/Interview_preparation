# Interview Q&A: Combination Sum (LeetCode 39)

## Q1. Why does the recursive call pass `i` instead of `i + 1`?

**Answer:** `i + 1` means this index has been consumed and may not be chosen again. That is correct for a problem that allows each element once. Here a candidate may appear any number of times, so the child frame must still be allowed to choose index `i`. The loop in the child starts at `i` and can pick `candidates[i]` immediately, which is how `[2, 2, 2, 2]` is formed. Uniqueness does not come from that argument. It comes from the loop never starting before `i`. Once 3 is on the path, 2 is no longer a legal choice, so the permutation `[3, 2, 2]` is not generated. `[2, 2, 3]` is.

## Q2. Walk through candidates `[2, 3, 6, 7]` and target `7`. Which branches record an answer?

**Answer:** Sort is already in order. Start at 2, remain 7. Two more 2s lead to path `[2, 2, 2]` and remain 1. Every remaining candidate is at least 2, so that branch breaks. Back at `[2, 2]` with remain 3, choosing 3 hits remain 0 and records `[2, 2, 3]`. Back at `[2]` with remain 5, the start index moves to 3. 3, 6, and 7 do not build a second combination from there: 3 leaves 2, and the search is not allowed to go back to candidate 2. The first-pick 3 branch never hits zero. First-pick 6 leaves 1 and dies. First-pick 7 hits zero and records `[7]`. Those are the only two answers.

## Q3. Why sort? Is the algorithm wrong without it?

**Answer:** The start-index rule already prevents duplicate permutations whether or not the array is sorted. Sorting is not what makes correctness work. Sorting makes the prune correct and sharp: if `candidates[i] > remain`, every later index is larger, so `break` is safe. Without sorting I must `continue`, because a later unsorted value might be small enough to fit. I still sort in an interview because the prune is worth one line and the combinations come out in a predictable order. I do not sort inside the recursion. I sort once before the search.

## Q4. What is the difference between this problem and the version where each number may be used once and the input may contain duplicates?

**Answer:** In the use-once version the recursive start becomes `i + 1`. If the input also contains equal values, different indices can still spell the same combination, so after sorting I skip `candidates[i]` when it equals `candidates[i - 1]` and `i` is not the start of this frame. That "previous duplicate was not used" test is how `[1, 1, 6]` is kept once when the input has two 1s, without also deleting a legitimate pair of 1s. This problem's candidates are distinct and reuse is allowed, so that skip is unnecessary and the recursive index stays `i`. Mixing the two fixes — skip logic plus `i + 1`, or reuse plus a set — is how people fail the follow-up.

## Q5. Why must the stored answer be a copy, and where is the unchoose?

**Answer:** `path` is one list. `answers.add(path)` would store that reference. The following `path.remove` would edit a combination that is already in the output, and by the time the search finishes every stored list would be empty. Java copies with `new ArrayList<>(path)`. Python copies with `path[:]`. The unchoose is the remove or pop immediately after the recursive call, still inside the loop, before the next candidate is appended. The pop belongs to the choice that was just explored, not to the base case. The base case returns without pushing.

## Q6. Give a complexity bound that uses the target, and name the assumption you need for termination.

**Answer:** Each chosen number is at least `M`, the minimum candidate, and the target is `T`, so no path is longer than `T/M`. Each node of the recursion tries at most `n` candidates. That yields a bound on the order of `n^(T/M)` nodes, times O(`T/M`) work if I include copying a path. Sorting is O(n log n) once. Stack space is O(`T/M`) besides the output. Termination needs positive candidates. If a candidate were 0, choosing it would leave `remain` unchanged and the same index would recurse forever. The problem gives positive integers, and I would state that if the interviewer relaxes the input I add an explicit guard rather than trusting the prune.
