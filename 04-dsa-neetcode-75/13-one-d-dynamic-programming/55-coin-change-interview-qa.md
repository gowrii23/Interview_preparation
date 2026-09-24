# 55. Coin Change — Interview Q&A

## 1. Why this state?

`dp[a]` is the fewest coins that sum to `a`. Because copies are unlimited, the set of coins you have left after making `a - c` is the same set you started with. The remaining amount is a complete summary. Adding a “number of coins used so far” into the state would only recreate the search depth. Adding the last coin index is unnecessary for a minimum: every coin is still legal afterward. You would add that index, or iterate coins on the outside in a careful order, only if you were counting distinct combinations rather than minimizing the number of coins.

## 2. How do you optimize space?

The uncompressed state is `dp[i][a]`: fewest coins using only the first `i` denominations to make `a`. The unbounded transition reads the same row (`dp[i][a - c]`), so you drop `i` and keep one array of length `amount + 1`. You must walk `a` upward; walking downward would read a value that has not yet taken this coin, which is the 0/1 pattern. You cannot drop the amount axis. Time stays `O(amount · |coins|)`, memory `O(amount)`.

## 3. Variant: count the number of combinations, or each coin may be used once, or return the coins you picked. What changes?

Combinations (the “coin change II” counting problem): `dp[a] += dp[a - coin]` with `dp[0] = 1`, and iterate coins on the outside, amounts on the inside upward, so that order of coins inside a combination is not reshuffled. Permutations would put the amount loop outside. One use per coin: iterate amounts downward. Reconstructing coins: store the predecessor coin next to each `dp[a]` and walk from the target down to 0, or accept that the min problem does not require it. If no combination exists, counting returns 0, while this problem returns `-1`. Same capacity, different combine function, different “impossible” value.

## 4. Why is largest-first greedy wrong, and when is it fine?

Denominations `[1, 3, 4]`, amount `6`. Largest-first takes `4`, then two `1`s, three coins. Two `3`s are better. Canonical coin systems such as US denominations happen to be greedy-safe, but the problem gives arbitrary positive integers. BFS from 0, adding one coin per edge, also computes the fewest coins and is the same DP in disguise: the first time you reach `amount` is the minimum depth. The array version is easier to write without a queue.

## 5. What bug does this code have?

```java
int[] dp = new int[amount + 1];
Arrays.fill(dp, Integer.MAX_VALUE);
dp[0] = 0;
for (int a = 1; a <= amount; a++) {
    for (int c : coins) {
        if (c <= a) {
            dp[a] = Math.min(dp[a], dp[a - c] + 1);
        }
    }
}
```

When `dp[a - c]` is `Integer.MAX_VALUE`, adding 1 overflows to `Integer.MIN_VALUE`, and the min latches onto that negative. Later answers are garbage, and the final “still MAX_VALUE means -1” check never sees the cells that overflowed. Use sentinel `amount + 1`, and skip cells that are still at the sentinel if you want the invariant “every finite `dp[a]` is a real count.”

## 6. What do you return for amount 0, for a coin larger than the amount, and for `[2]` making 3?

Amount 0: `dp[0]` is already 0, the loops do not run, return 0. A coin larger than the amount is skipped by `c <= a`; if every coin is larger and amount is positive, the row stays at the sentinel and you return `-1`. `[2]` and amount `3`: `dp` is `[0, sentinel, 1, sentinel]`, return `-1`. Say these three before you claim the solution handles impossibility.
