# 121. Best Time to Buy and Sell Stock — Interview Q&A

## 1. What is the complexity, and how does it compare with checking every pair?

One pass is `O(n)` time and `O(1)` space. Every pair of days is `O(n^2)` time. Both are correct. The linear scan works because the best buy for a fixed sell day is the minimum earlier price, and that minimum is available from the prefix without storing the whole prefix.

## 2. Why a running minimum instead of a stack or a sliding window of fixed size?

The feasible buy is “any earlier day,” not “the last `k` days.” The only statistic that matters from the past is the minimum. A stack of candidates would still expose that same minimum at the bottom. A fixed-size window would ignore a cheap day just outside the window, which is a legal buy. The running minimum is the window state, with the left edge implicitly sitting on the cheapest day so far.

## 3. What bug does this code have?

```java
int minPrice = prices[0];
int best = 0;
for (int price : prices) {
    best = Math.max(best, price - minPrice);
    minPrice = Math.min(minPrice, price);
}
```

This version is actually correct if `prices` is non-empty: selling on the buy day yields 0, then the minimum updates. The bug appears when the array is empty: `prices[0]` throws. Another real bug is updating the minimum first and then taking `price - minPrice` on the same iteration after that update, which is fine numerically (it adds 0) — the harmful bug is remembering the minimum of the whole array including the future, for example by scanning twice and subtracting the global min from the global max. That pairs a later cheap day with an earlier expensive sell. On `[7, 1]` it would report `6` if you sold at 7 and bought at 1.

## 4. Follow-up: you may complete as many buy/sell pairs as you want, but not overlapping. What changes?

Add every positive difference `prices[i] - prices[i - 1]`. That sums every upward slope, which is the best you can do with unlimited non-overlapping transactions. Still `O(n)` and `O(1)`. If a fee `f` is charged per transaction, the state becomes two running values: max profit if you currently hold a share, and max profit if you do not. That is the stock-with-fee dynamic program, not a single minimum.

## 5. What if the constraints change and prices are 64-bit, `n` is 10^7, and you must return the buy and sell indices?

Keep `minIndex` along with `minPrice`, and remember `buyIndex` and `sellIndex` when the profit improves. Still one pass, `O(1)` memory, which matters at `n = 10^7`. Compute the profit in a wide integer. If prices can be negative, the same formula works: profit is still `price - minPrice`, and a negative price can be a good buy. If they require at least one transaction even at a loss, drop the `best = 0` floor and track the maximum difference even when it is negative.

## 6. What if the array has one day, or the best sell is the last day and the best buy is the first?

One day: you cannot sell later, so the answer is 0. The loop sets the minimum and never records a positive profit. Buy on day 0 and sell on the last day is allowed and falls out of the same loop if that difference is the best; you do not special-case the ends. Equal prices throughout return 0.
