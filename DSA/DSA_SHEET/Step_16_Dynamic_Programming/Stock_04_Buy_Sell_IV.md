# Best Time to Buy and Sell Stock IV  🔴

**Reference:** https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/

---

## Problem

Given an integer `k` and an array `prices` where `prices[i]` is the price on day `i`, return the maximum profit achievable with **at most `k` transactions**. You may not engage in multiple transactions simultaneously (sell before you buy again).

```
Input : k = 2, prices = [2, 4, 1]
Output: 2
Explanation: Buy at 2 (day 0), sell at 4 (day 1), profit = 2. The second transaction is unused.
```

---

## Intuition

This is the general form of Stock III — same machine, but `k` is a parameter.

**DP state:** `dp[i][j][hold]` = max profit from day `i` onward with `j` transactions still available and `hold` ∈ {0 = not holding, 1 = holding}. A transaction is consumed on **buy**.

**Recurrence:**
```
dp[i][j][0] = max(dp[i+1][j][0], prices[i] + dp[i+1][j][1])      // rest / sell
dp[i][j][1] = max(dp[i+1][j][1], -prices[i] + dp[i+1][j-1][0])   // rest / buy
```
Base: day `n` or `j == 0` → `0`. Answer = `dp[0][k][0]`.

**Optimization:** if `k >= n/2`, transactions are effectively unlimited (you can never make more than `n/2` non-overlapping trades), so fall back to the greedy Stock II solution.

---

## Code (C++)

```cpp
#include <vector>
#include <algorithm>
using namespace std;

int maxProfit(int k, vector<int>& prices) {
    int n = prices.size();
    if (n == 0 || k == 0) return 0;

    // If k is large enough, it's the unlimited-transactions case (Stock II).
    if (k >= n / 2) {
        int profit = 0;
        for (int i = 1; i < n; ++i)
            if (prices[i] > prices[i - 1])
                profit += prices[i] - prices[i - 1]; // grab every upswing
        return profit;
    }

    // dp[i][j][hold]: sized (n+1) x (k+1) x 2.
    vector<vector<vector<long long>>> dp(
        n + 1, vector<vector<long long>>(k + 1, vector<long long>(2, 0)));

    for (int i = n - 1; i >= 0; --i) {
        for (int j = 1; j <= k; ++j) {
            // Not holding: rest, or sell today.
            dp[i][j][0] = max(dp[i + 1][j][0],
                              (long long)prices[i] + dp[i + 1][j][1]);
            // Holding: rest, or buy today (uses one transaction → j-1).
            dp[i][j][1] = max(dp[i + 1][j][1],
                              -(long long)prices[i] + dp[i + 1][j - 1][0]);
        }
    }
    return (int)dp[0][k][0];
}
```

> Space optimization: row `i` depends only on row `i+1`; keep two `(k+1) x 2` layers → O(k) space.

---

## Complexity

| | |
|---|---|
| Time | O(n · k) |
| Space | O(n · k) for the table, O(k) when rolled |

> ⚠️ The `k >= n/2` shortcut both speeds things up **and** avoids allocating a huge table for large `k`.
> ⚠️ Handle `k == 0` and `n == 0` up front — otherwise the table and loops produce `0` only by luck of initialization.
