# Best Time to Buy and Sell Stock  🟢

**Reference:** https://leetcode.com/problems/best-time-to-buy-and-sell-stock/

---

## Problem

You are given an array `prices` where `prices[i]` is the price of a stock on day `i`. You may complete **at most one transaction** (buy once and sell once). Buy must happen before sell. Return the maximum profit; if no profit is possible, return `0`.

```
Input : prices = [7, 1, 5, 3, 6, 4]
Output: 5
Explanation: Buy on day 1 (price = 1), sell on day 4 (price = 6), profit = 6 - 1 = 5.
```

---

## Intuition

This is the simplest of the stock family — only one buy/sell pair is allowed, so a full DP table is overkill, but the DP idea is still useful.

**DP state:** `minSoFar` = minimum price seen among days `0..i`.
**Recurrence:** at each day `i`, the best profit if we sell today is `prices[i] - minSoFar`. The answer is the max of this over all days.

```
profit = max(profit, prices[i] - min(prices[0..i]))
```

We track the running minimum buy price and the running best profit in a single pass — effectively an O(1) "rolling DP".

---

## Code (C++)

```cpp
#include <vector>
#include <algorithm>
#include <climits>
using namespace std;

int maxProfit(vector<int>& prices) {
    int n = prices.size();
    if (n == 0) return 0;

    int minSoFar = prices[0]; // cheapest price to buy at, among days seen
    int best = 0;             // best profit achievable so far

    // Single pass: for each day, try selling today against cheapest past day.
    for (int i = 1; i < n; ++i) {
        best = max(best, prices[i] - minSoFar); // sell today
        minSoFar = min(minSoFar, prices[i]);    // update cheapest buy price
    }
    return best;
}
```

> Space optimization: already O(1) — we keep only two scalars (`minSoFar`, `best`) instead of any DP array.

---

## Complexity

| | |
|---|---|
| Time | O(n) |
| Space | O(1) |

> ⚠️ Return `0` (not a negative value) when prices are strictly decreasing — you are allowed to make **no** transaction.
> ⚠️ You must buy before you sell; update `minSoFar` *after* computing today's profit (as above) to respect ordering — though here both orders work since `prices[i]-prices[i]=0`.
