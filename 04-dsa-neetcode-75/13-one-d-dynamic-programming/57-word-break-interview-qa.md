# 57. Word Break — Interview Q&A

## 1. Why this state?

`dp[i]` is a boolean for one prefix. A segmentation is a chain of cuts, and the last word ends at `i` and starts at some `j`. The characters before `j` are a smaller instance of the same question. Two different segmentations of that shorter prefix do not change whether the last word fits, so you store a boolean rather than the list of sentences. The dictionary membership test is outside the state: it is the transition filter.

## 2. How do you optimize space?

You need a boolean per prefix in the form above, which is `O(n)` and already one dimension. You cannot roll this down to a constant number of booleans, because a word can start at any earlier true index, not only at `i - 1` or `i - 2`. Capping the lookback by the maximum word length shrinks the inner loop, not the array. A top-down memo on the start index is the same `O(n)` booleans plus stack. If every word has length 1 or 2 only, then it collapses toward the decode-ways lookback and two booleans would suffice; that is a special case, not the problem.

## 3. Variant: return all segmentations, or words may be used only once, or you also have a length budget. What changes?

All sentences is Word Break II. Keep this boolean row so you do not explore dead prefixes, then DFS from 0, appending a word when `dp` (or a direct dictionary check) says the cut is real. Warn that the output size dominates. One use per word: the state grows by the set of used words, which is exponential; only do that for a tiny dictionary. A length budget or a “at most k words” limit adds that counter to the state, so the row becomes `dp[i][k]`. Say which of those you were actually asked before you code the boolean version.

## 4. Why is `"catsandog"` false even though several words match inside it?

`cat`, `cats`, `sand`, and `and` all occur as slices, and `cat|sand` covers a true prefix of length 7 (`"catsand"`). The remaining `"og"` is not a word, and the other cut `cats|and` leaves the same `"og"`. `"dog"` matches the letters near the end only if you start at the `'d'` in `"andog"`, but that start is not a boundary between a segmented prefix and a dictionary word in a way that consumes the `'o'` of `sand` correctly: starting `"dog"` at the `'d'` would require the prefix `"catsan"` to be segmented, and it is not. The DP never sets the last cell. Matching “somewhere in the string” is not the same as tiling the string.

## 5. What bug does this code have?

```python
dp = [False] * (n + 1)
for i in range(1, n + 1):
    for word in wordDict:
        if i >= len(word) and s[i - len(word):i] == word and dp[i - len(word)]:
            dp[i] = True
```

The loop body is actually a correct transition. The bug is the missing `dp[0] = True`. A word that matches at the very beginning checks `dp[0]`, which is false, so a string equal to a single dictionary word returns false, and every longer tiling fails with it. Set the empty prefix to true before the loops. A second practical bug is leaving this as written when the dictionary is a list of duplicates and you do not cap lengths: it is still correct, but a set plus a max-length window is what you want under a tight limit.

## 6. What are the time and space bounds you should say out loud?

“`O(n)` booleans. For each of `n` ends I try at most `M` starts, where `M` is the longest word, and each check hashes `O(M)` characters, so `O(n M^2)` time, plus building a set of the dictionary. If I do not use `M`, quote `O(n^3)`.” Also say expected hash time. If they ban hash sets, a trie of the dictionary changes the inner check to a walk of at most `M` edges and the same DP row; the state does not change.
