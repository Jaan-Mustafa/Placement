# Best Time to Buy and Sell Stock with Transaction Fee  🟡

**Reference:** https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/

---

## Problem

Given `prices` where `prices[i]` is the price on day `i`, and an integer `fee`, find the maximum profit with **unlimited transactions**. A transaction fee `fee` is charged **once per completed transaction** (you may charge it on either buy or sell — just be consistent). You can hold at most one share at a time.

```
Input : prices = [1, 3, 2, 8, 4, 9], fee = 2
Output: 8
Explanation: buy(1) -> sell(8): profit 8-1-2 = 5; buy(4) -> sell(9): profit 9-4-2 = 3. Total = 8.
```

---

## Intuition

Identical to Stock II's two-state machine, but each completed transaction costs `fee`. We charge the fee on the **sell** side (charging on buy works too — just don't double-charge).

**DP state:** `dp[i][hold]` = max profit from day `i` onward, `hold` ∈ {0 = not holding, 1 = holding}.

**Recurrence:**
```
dp[i][0] = max(dp[i+1][0],                       // rest
               prices[i] - fee + dp[i+1][1])     // sell today, pay the fee here
dp[i][1] = max(dp[i+1][1],                       // rest
               -prices[i] + dp[i+1][0])          // buy today
```
Base: day `n` → `0`. Answer = `dp[0][0]`.

---

## Code (C++)

```cpp
#include <vector>
#include <algorithm>
using namespace std;

int maxProfit(vector<int>& prices, int fee) {
    int n = prices.size();
    if (n == 0) return 0;

    // dp[i][hold]; built bottom-up from day n down to 0.
    vector<vector<long long>> dp(n + 1, vector<long long>(2, 0));

    for (int i = n - 1; i >= 0; --i) {
        // Not holding: rest, or sell today and pay the fee (charged once per transaction).
        dp[i][0] = max(dp[i + 1][0],
                       (long long)prices[i] - fee + dp[i + 1][1]);
        // Holding: rest, or buy today (no fee here, since fee is on sell).
        dp[i][1] = max(dp[i + 1][1],
                       -(long long)prices[i] + dp[i + 1][0]);
    }
    return (int)dp[0][0];
}
```

> Space optimization: each row needs only row `i+1`, so two scalars `notHold`, `hold` collapse the table to O(1) space.

---

## Complexity

| | |
|---|---|
| Time | O(n) |
| Space | O(n) for the table, O(1) when rolled to two variables |

> ⚠️ Charge `fee` exactly **once** per round-trip — pick the buy branch *or* the sell branch, never both, or you will subtract it twice.
> ⚠️ Use a wide enough type (`long long` here) for intermediate sums when prices and fee are large to avoid overflow before the final cast.
