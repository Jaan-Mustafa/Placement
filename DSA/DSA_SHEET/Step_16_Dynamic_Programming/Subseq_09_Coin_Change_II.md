# Coin Change II (Number of Ways)  🟡

**Reference:** https://leetcode.com/problems/coin-change-ii/

---

## Problem

Given an array `coins` of distinct denominations and an `amount`, return the **number of combinations** that make up that amount. Each coin may be used an **unlimited** number of times. Combinations differing only in order count once.

```
coins  = [1, 2, 5]
amount = 5
Answer = 4
// {5}, {2,2,1}, {2,1,1,1}, {1,1,1,1,1}
```

---

## Intuition

This is an **unbounded** knapsack where we count distinct combinations (order does not matter).

- **State:** `dp[i][a]` = number of ways to make amount `a` using the first `i` coin types.
- **Recurrence:**
  - Don't use coin `i`: `dp[i-1][a]`
  - Use coin `i` (unlimited, so stay on row `i`): `dp[i][a - coins[i-1]]`
  - `dp[i][a] = notTake + take`
- **Base case:** `dp[i][0] = 1` (one way — pick nothing — to make amount 0).

The key to avoiding permutation double-counting is iterating **coins in the outer loop** and amount in the inner loop.

---

## Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

int change(int amount, const vector<int>& coins) {
    int n = coins.size();
    // dp[i][a] : #ways to make amount a using first i coin types
    vector<vector<long long>> dp(n + 1, vector<long long>(amount + 1, 0));
    for (int i = 0; i <= n; ++i) dp[i][0] = 1; // one way to make 0: take nothing

    for (int i = 1; i <= n; ++i) {
        for (int a = 0; a <= amount; ++a) {
            dp[i][a] = dp[i - 1][a];                       // skip coin i-1
            if (coins[i - 1] <= a)
                dp[i][a] += dp[i][a - coins[i - 1]];       // reuse coin i-1 (unbounded)
        }
    }
    return (int)dp[n][amount];
}

int main() {
    vector<int> coins = {1, 2, 5};
    cout << change(5, coins) << "\n"; // 4
}
```

**Space optimization:** because the "take" term uses the current row, a single `vector<long long> dp(amount+1)` with the coin loop outside and amount loop inside (left-to-right) suffices — `O(amount)` space:

```cpp
vector<long long> dp(amount + 1, 0); dp[0] = 1;
for (int c : coins)
    for (int a = c; a <= amount; ++a) dp[a] += dp[a - c];
```

---

## Complexity

| | |
|---|---|
| Time | O(number_of_coins * amount) |
| Space | O(number_of_coins * amount), optimizable to O(amount) |

> ⚠️ Loop **coins outside, amount inside** to count combinations. Swapping the loops counts ordered sequences (permutations) instead. Use a 64-bit type since counts can exceed `int`.
