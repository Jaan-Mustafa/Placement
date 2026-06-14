# Best Time to Buy and Sell Stock with Cooldown  🟡

**Reference:** https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/

---

## Problem

Given `prices` where `prices[i]` is the price on day `i`, find the maximum profit with **unlimited transactions**, subject to one rule: after you **sell**, you must wait **one full day** (cooldown) before you can buy again. You can hold at most one share at a time.

```
Input : prices = [1, 2, 3, 0, 2]
Output: 3
Explanation: buy(1) -> sell(2) -> cooldown -> buy(0) -> sell(2). Profit = (2-1) + (2-0) = 3.
```

---

## Intuition

Same holding machine as Stock II, but selling forces a one-day gap. The cleanest way to encode the gap: when we **buy**, we must come from a day where we were already free of any cooldown — i.e. buy uses `dp[i+2]` (skipping the cooldown day) or, equivalently, buy comes from the "rested, not-holding" state.

**DP state:** `dp[i][hold]` = max profit from day `i` onward, `hold` ∈ {0 = not holding, 1 = holding}.

**Recurrence:**
```
dp[i][0] = max(dp[i+1][0],                  // rest, stay free
               prices[i] + dp[i+1][1])      // sell today
dp[i][1] = max(dp[i+1][1],                  // rest, stay holding
               -prices[i] + dp[i+2][0])     // buy today, but next day is cooldown -> jump to i+2
```
Base: indices `>= n` give `0`. Answer = `dp[0][0]`.

---

## Code (C++)

```cpp
#include <vector>
#include <algorithm>
using namespace std;

int maxProfit(vector<int>& prices) {
    int n = prices.size();
    if (n == 0) return 0;

    // Size n+2 so dp[i+2] is always valid (cooldown skips a day after selling).
    vector<vector<long long>> dp(n + 2, vector<long long>(2, 0));

    for (int i = n - 1; i >= 0; --i) {
        // Not holding: rest, or sell today (gain prices[i], become not holding).
        dp[i][0] = max(dp[i + 1][0], (long long)prices[i] + dp[i + 1][1]);
        // Holding: rest, or buy today. After a future sell there is a cooldown,
        // so a buy effectively continues from the not-holding state at i+2.
        dp[i][1] = max(dp[i + 1][1], -(long long)prices[i] + dp[i + 2][0]);
    }
    return (int)dp[0][0];
}
```

> Space optimization: each row depends only on rows `i+1` and `i+2`, so a handful of rolling scalars (e.g. `notHold`, `notHoldNext`, `hold`) bring space down to O(1).

---

## Complexity

| | |
|---|---|
| Time | O(n) |
| Space | O(n) for the table, O(1) when rolled to rolling variables |

> ⚠️ The cooldown applies **after a sell, before the next buy** — that is why the cooldown jump (`i+2`) lives in the **buy** branch, not the sell branch.
> ⚠️ Allocate `n + 2` rows (not `n + 1`) so `dp[i+2]` never indexes out of bounds at `i = n-1` or `i = n-2`.
