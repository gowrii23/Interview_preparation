# Interview Q&A: backtracking pattern

## Q1. What does "unchoose" actually undo, and what bug appears if you delete that line?

**Answer:** Unchoose restores the shared mutable state to the way it was before the current choice. For a combination problem that is `path.pop()`. For word search it is writing the original letter back onto the cell. The recursive call may have appended more and popped more; by the time it returns, the path should differ from its pre-call value by exactly the one choice this frame added. If I skip the pop, the next loop iteration appends onto a path that still contains the previous candidate, so I generate mixes that were never chosen together, and the base case records garbage. If I skip the board restore, every later path sees a permanently blocked cell.

## Q2. Why copy the path when you record an answer?

**Answer:** The path is one list object for the whole search. Recording it stores a reference. Later pops and appends mutate that same object, so every stored answer changes with it, and at the end they are all empty or all equal to the last path. `new ArrayList<>(path)` in Java and `path[:]` in Python snapshot the current contents. The copy costs linear time in the path length, and that factor belongs in the complexity. Copying only at the base case, not at every node, is correct when only complete paths are answers. Subsets are the exception: every node is an answer, so every node copies.

## Q3. How does the start index prevent duplicate combinations without a set?

**Answer:** I decide that every combination will be built in non-decreasing index order. The loop runs from `start` to the end, and the recursive call's new start is `i` if reuse is allowed or `i + 1` if it is not. A combination that would have been `[3, 2, 2]` is only built as `[2, 2, 3]`, because once `3` is chosen the later loop never looks back at `2`. I do not generate the permutations and filter them. If the input itself contains duplicate values and each index may be used once, the start index is not enough: `[1a, 1b]` and `[1b, 1a]` are different index sequences with the same values. That variant sorts the array and skips a candidate equal to the previous one when the previous one was not used in this frame.

## Q4. Where do you prune, and how is that different from unchoose?

**Answer:** Prune means "do not recurse." Unchoose means "undo a choice you did make, after the recurse returns." They sit on opposite sides of the call. In combination sum I prune when the candidate is already larger than the remaining target, and if the array is sorted I can `break` rather than `continue`, because later candidates are larger too. In word search I prune when the cell is out of bounds, already on the path, or the wrong letter. Those calls return immediately and there is nothing to unchoose, because I never marked them. The dotted edges in the diagrams are these refused branches. A common mistake is to recurse and only notice the overflow in the child. That is correct but wasteful; the parent already had the information.

## Q5. Compare the recursion argument for "use each element once" and "use it unlimited times."

**Answer:** Both loops start at `start` so the path stays non-decreasing. The difference is the argument passed down. Use-once passes `i + 1`, which removes `nums[i]` from the future pool and also removes everything before it. Unlimited reuse passes `i`, so the child may pick `nums[i]` again, and the loop at the child still will not pick `nums[i - 1]`. Passing `0` every time would allow every order and explode into duplicate permutations. Passing `i + 1` on an unlimited-reuse problem silently drops answers such as `[2, 2, 3]` for target 7. I say the parameter out loud when I write the call so the interviewer can see I chose it on purpose.

## Q6. What complexity do you claim, and what do you refuse to claim?

**Answer:** I claim a bound in terms of the branching factor and the depth, and I include the cost of copying answers. I do not claim O(n) or O(n log n) for a search that can emit exponentially many combinations. If the interviewer wants a tighter bound, I use the problem constraints: combination sum depth is at most `target / min(candidates)`, word-search depth is the word length, subsets depth is n with a binary include/exclude shape. Space besides the output is the recursion stack plus one path, O(depth), when the board or the path is mutated in place. The output itself can be as large as the time bound, so "extra space" and "total space" are different sentences.
