# Best Time to Buy and Sell Stock III  🔴

**Reference:** https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/

---

## Problem

Given `prices` where `prices[i]` is the price on day `i`, find the maximum profit you can achieve with **at most two transactions**. You may not engage in multiple transactions at once (you must sell the stock before you buy again).

```
Input : prices = [3, 3, 5, 0, 0, 3, 1, 4]
Output: 6
Explanation: Buy at 0 (day 4), sell at 3 (day 6), profit 3; buy at 1 (day 7), sell at 4 (day 8), profit 3. Total = 6.
```

---

## Intuition

Extend the holding state with a **transaction-count cap**.

**DP state:** `dp[i][k][hold]` = max profit from day `i` onward when we have `k` transactions still available to complete, and `hold` ∈ {0 = not holding, 1 = holding}. Here a transaction is "spent" when we **buy** (cap = 2).

**Recurrence:**
```
dp[i][k][0] = max(dp[i+1][k][0],                  // rest
                  prices[i] + dp[i+1][k][1])      // sell (does not use a cap)
dp[i][k][1] = max(dp[i+1][k][1],                  // rest
                  -prices[i] + dp[i+1][k-1][0])   // buy (uses one transaction)
```
Base: everything at day `n`, or `k == 0`, is `0`. Answer = `dp[0][2][0]`.

---

## Code (C++)

```cpp
#include <vector>
#include <algorithm>
using namespace std;

int maxProfit(vector<int>& prices) {
    int n = prices.size();
    if (n == 0) return 0;
    const int K = 2; // at most two transactions

    // dp[i][k][hold]; sized n+1 and K+1 so indices i+1 and k-1 stay valid.
    vector<vector<vector<long long>>> dp(
        n + 1, vector<vector<long long>>(K + 1, vector<long long>(2, 0)));

    for (int i = n - 1; i >= 0; --i) {
        for (int k = 1; k <= K; ++k) { // k = 0 stays 0 (no transactions left)
            // Not holding: rest, or sell today (selling doesn't consume a cap).
            dp[i][k][0] = max(dp[i + 1][k][0],
                              (long long)prices[i] + dp[i + 1][k][1]);
            // Holding: rest, or buy today (buying consumes one transaction → k-1).
            dp[i][k][1] = max(dp[i + 1][k][1],
                              -(long long)prices[i] + dp[i + 1][k - 1][0]);
        }
    }
    return (int)dp[0][K][0];
}
```

> Space optimization: row `i` depends only on row `i+1`, so two `(K+1) x 2` layers (or even 4 rolling scalars: buy1, sell1, buy2, sell2) reduce space to O(1).

---

## Complexity

| | |
|---|---|
| Time | O(n · K) = O(2n) = O(n) |
| Space | O(n · K) for the table, O(1) when rolled |

> ⚠️ Decide consistently whether a transaction is consumed on **buy** or on **sell** — here it is on buy. Mixing the two leads to off-by-one transaction counts.
> ⚠️ Keep `k = 0` initialized to `0` and never decrement below it; `dp[i+1][k-1]` is only reached for `k >= 1`.
