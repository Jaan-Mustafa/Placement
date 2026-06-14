# Minimum Coins (Coin Change)  🟡

**Reference:** https://leetcode.com/problems/coin-change/

---

## Problem

Given an array `coins` of distinct denominations and an `amount`, return the **fewest number of coins** needed to make up that amount. You may use each coin **unlimited** times. If the amount cannot be formed, return `-1`.

```
coins  = [1, 2, 5]
amount = 11
Answer = 3   // 5 + 5 + 1
```

---

## Intuition

This is an **unbounded** knapsack variant minimizing coin count.

- **State:** `dp[a]` = minimum number of coins to form amount `a`.
- **Recurrence:** `dp[a] = min over each coin c (c <= a) of (1 + dp[a - c])`.
- **Base case:** `dp[0] = 0`. Initialize others to a large sentinel (`amount + 1`) representing "unreachable".

Because coins can repeat, when we use coin `c` we look back at `dp[a - c]` in the **same** dp array (unbounded behavior).

---

## Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

int coinChange(const vector<int>& coins, int amount) {
    // Use amount+1 as "infinity": no solution needs more than `amount` coins
    vector<int> dp(amount + 1, amount + 1);
    dp[0] = 0;                                  // 0 coins to make amount 0

    for (int a = 1; a <= amount; ++a)
        for (int c : coins)
            if (c <= a)
                dp[a] = min(dp[a], 1 + dp[a - c]); // use one coin c, plus best for a-c

    return dp[amount] > amount ? -1 : dp[amount]; // still sentinel -> unreachable
}

int main() {
    vector<int> coins = {1, 2, 5};
    cout << coinChange(coins, 11) << "\n"; // 3
}
```

This 1-D formulation is already space-optimal. A 2-D `dp[i][a]` (items x amount) is equivalent but uses more memory; the order of the loops here naturally allows coin reuse.

---

## Complexity

| | |
|---|---|
| Time | O(amount * number_of_coins) |
| Space | O(amount) |

> ⚠️ Use a sentinel like `amount + 1` rather than `INT_MAX`; otherwise `1 + dp[a - c]` overflows. Remember to return `-1` when the amount is unreachable.
