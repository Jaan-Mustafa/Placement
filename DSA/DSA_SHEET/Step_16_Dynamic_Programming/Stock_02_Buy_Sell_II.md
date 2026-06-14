# Best Time to Buy and Sell Stock II  🟡

**Reference:** https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/

---

## Problem

Given `prices` where `prices[i]` is the price on day `i`, you may complete **as many transactions as you like** (buy then sell, repeatedly), but you can hold **at most one share** at a time. You may buy and sell on the same day. Return the maximum profit.

```
Input : prices = [7, 1, 5, 3, 6, 4]
Output: 7
Explanation: Buy at 1, sell at 5 (profit 4); buy at 3, sell at 6 (profit 3). Total = 7.
```

---

## Intuition

With unlimited transactions, the classic DP carries a "holding" state.

**DP state:** `dp[i][hold]` = max profit at the start of day `i` where
- `hold = 0` → we currently hold **no** stock,
- `hold = 1` → we currently **hold** one stock.

**Recurrence:**
```
dp[i][0] = max(dp[i+1][0],                 // do nothing
               prices[i] + dp[i+1][1])     // sell today
dp[i][1] = max(dp[i+1][1],                 // do nothing
               -prices[i] + dp[i+1][0])    // buy today
```
Base: `dp[n][0] = dp[n][1] = 0`. Answer = `dp[0][0]` (start holding nothing).

(Equivalent greedy: sum every positive `prices[i]-prices[i-1]`, but DP generalizes to the harder variants.)

---

## Code (C++)

```cpp
#include <vector>
#include <algorithm>
using namespace std;

int maxProfit(vector<int>& prices) {
    int n = prices.size();
    if (n == 0) return 0;

    // dp[i][hold] computed bottom-up from day n down to 0.
    vector<vector<long long>> dp(n + 1, vector<long long>(2, 0));

    for (int i = n - 1; i >= 0; --i) {
        // Not holding: either stay, or buy today (pay prices[i], become holding).
        dp[i][0] = max(dp[i + 1][0], -(long long)prices[i] + dp[i + 1][1]);
        // Holding: either stay, or sell today (gain prices[i], become not holding).
        dp[i][1] = max(dp[i + 1][1], (long long)prices[i] + dp[i + 1][0]);
    }
    return (int)dp[0][0];
}
```

> Space optimization: only row `i+1` is needed for row `i`, so two scalars `notHold`, `hold` reduce space to O(1).

---

## Complexity

| | |
|---|---|
| Time | O(n) |
| Space | O(n) for the table, O(1) when rolled to two variables |

> ⚠️ "At most one share at a time" — you must sell before buying again; the two-state machine enforces this.
> ⚠️ Note I named state `dp[i][0]` = *not holding* and `dp[i][1]` = *holding*; mixing up the indices is the most common bug here.
