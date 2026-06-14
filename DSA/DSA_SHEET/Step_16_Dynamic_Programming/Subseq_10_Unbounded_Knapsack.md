# Unbounded Knapsack  🟡

**Reference:** https://www.geeksforgeeks.org/unbounded-knapsack-repetition-items-allowed/

---

## Problem

Given `n` items with `weight[i]` and `value[i]`, and a knapsack of capacity `W`, maximize the total value. Unlike 0/1 knapsack, **each item may be chosen any number of times**.

```
weight = [2, 4, 6]
value  = [5, 11, 13]
W      = 10
Answer = 27   // item0 (w=2) x5 -> 5*5 = 25? -> better: item1 x2 + item0 x1 = 11+11+5 = 27
```

---

## Intuition

Same as 0/1 knapsack, but when we take an item we may take it **again**, so we stay on the same item index.

- **State:** `dp[i][w]` = best value using item types `0..i-1` with capacity `w`.
- **Recurrence:**
  - Skip item `i`: `dp[i-1][w]`
  - Take item `i` (stay on row `i`): `value[i-1] + dp[i][w - weight[i-1]]`
  - `dp[i][w] = max(skip, take)`
- **Base case:** `dp[0][w] = 0`.

---

## Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

int unboundedKnapsack(const vector<int>& weight, const vector<int>& value, int W) {
    int n = weight.size();
    // dp[i][w] : best value using first i item types with capacity w
    vector<vector<int>> dp(n + 1, vector<int>(W + 1, 0));

    for (int i = 1; i <= n; ++i) {
        for (int w = 0; w <= W; ++w) {
            dp[i][w] = dp[i - 1][w];                          // skip item i-1
            if (weight[i - 1] <= w)
                dp[i][w] = max(dp[i][w],
                               value[i - 1] + dp[i][w - weight[i - 1]]); // reuse item i-1
        }
    }
    return dp[n][W];
}

int main() {
    vector<int> weight = {2, 4, 6};
    vector<int> value  = {5, 11, 13};
    cout << unboundedKnapsack(weight, value, 10) << "\n"; // 27
}
```

**Space optimization:** use one `vector<int> dp(W+1)` and iterate `w` from **low to high** (left-to-right). This left-to-right order is exactly what allows an item to be reused — contrast with 0/1 knapsack which iterates right-to-left:

```cpp
vector<int> dp(W + 1, 0);
for (int i = 0; i < n; ++i)
    for (int w = weight[i]; w <= W; ++w)
        dp[w] = max(dp[w], value[i] + dp[w - weight[i]]);
```

---

## Complexity

| | |
|---|---|
| Time | O(n * W) |
| Space | O(n * W), optimizable to O(W) |

> ⚠️ The "take" term reads `dp[i][...]` (current row), not `dp[i-1][...]`. In the 1-D version iterate `w` **left-to-right** — the opposite of 0/1 knapsack.
