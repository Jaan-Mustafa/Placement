# Best Time to Buy and Sell Stock  🟡

**Reference:** [LeetCode 121 — Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)

---

## Problem
Given daily prices, maximize profit from a **single** buy followed by a later sell. Return 0 if no profit is possible.

```
Input:  [7, 1, 5, 3, 6, 4]    Output: 5   (buy at 1, sell at 6)
Input:  [7, 6, 4, 3, 1]       Output: 0   (prices only fall)
```

---

## Intuition
To sell on day `i` for max profit, you'd have bought at the **minimum price seen so far**. Track that running minimum; at each day compute `price - minSoFar` and keep the best.

---

## Code (C++)
```cpp
int maxProfit(vector<int>& prices) {
    int minPrice = INT_MAX, profit = 0;
    for (int p : prices) {
        minPrice = min(minPrice, p);        // cheapest buy so far
        profit = max(profit, p - minPrice); // best sell today
    }
    return profit;
}
```

---

## Dry run
`[7,1,5,3,6,4]`: min→7,1,1,1,1,1; profit→0,0,4,4,**5**,5. Answer **5**.

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> This is the single-transaction version. Multiple transactions, cooldowns, and fees are covered in Step 16 (DP on Stocks).
