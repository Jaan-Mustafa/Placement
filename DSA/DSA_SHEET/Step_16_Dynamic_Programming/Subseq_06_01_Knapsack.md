# 0/1 Knapsack  🟡

**Reference:** https://www.geeksforgeeks.org/0-1-knapsack-problem-dp-10/

---

## Problem

Given `n` items, each with a `weight[i]` and a `value[i]`, and a knapsack of capacity `W`, choose a subset of items (each item at most once) to maximize total value without exceeding capacity `W`.

```
weight = [1, 3, 4, 5]
value  = [1, 4, 5, 7]
W      = 7
Answer = 9   // pick items with weights {3, 4} -> value 4 + 5 = 9
```

---

## Intuition

For each item we either **take it** (if it fits) or **skip it** — the "0/1" means each item is used at most once.

- **State:** `dp[i][w]` = max value using the first `i` items with capacity `w`.
- **Recurrence:**
  - Skip: `dp[i-1][w]`
  - Take (if `weight[i-1] <= w`): `value[i-1] + dp[i-1][w - weight[i-1]]`
  - `dp[i][w] = max(skip, take)`
- **Base case:** `dp[0][w] = 0` (no items -> no value).

---

## Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

int knapsack01(const vector<int>& weight, const vector<int>& value, int W) {
    int n = weight.size();
    // dp[i][w] : best value using first i items with capacity w
    vector<vector<int>> dp(n + 1, vector<int>(W + 1, 0));

    for (int i = 1; i <= n; ++i) {
        for (int w = 0; w <= W; ++w) {
            dp[i][w] = dp[i - 1][w];                         // skip item i-1
            if (weight[i - 1] <= w)
                dp[i][w] = max(dp[i][w],
                               value[i - 1] + dp[i - 1][w - weight[i - 1]]); // take it
        }
    }
    return dp[n][W];
}

int main() {
    vector<int> weight = {1, 3, 4, 5};
    vector<int> value  = {1, 4, 5, 7};
    cout << knapsack01(weight, value, 7) << "\n"; // 9
}
```

**Space optimization:** since `dp[i]` depends only on `dp[i-1]`, use one `vector<int> dp(W+1)` and iterate `w` from `W` **down to** `weight[i-1]`. The right-to-left order guarantees each item is used at most once. This gives `O(W)` space.

---

## Complexity

| | |
|---|---|
| Time | O(n * W) |
| Space | O(n * W), optimizable to O(W) |

> ⚠️ In the 1-D optimization the inner loop **must** run from high `w` to low `w`. Looping left-to-right would reuse the same item multiple times — that is the **unbounded** knapsack, not 0/1.
