# Rod Cutting Problem  🟡

**Reference:** https://www.geeksforgeeks.org/cutting-a-rod-dp-13/

---

## Problem

Given a rod of length `N` and a `price[]` array where `price[i]` is the value of a piece of length `i+1`, cut the rod into pieces to **maximize total selling value**. Any length piece may be used any number of times.

```
price = [1, 5, 8, 9, 10, 17, 17, 20]   // length 1..8
N     = 8
Answer = 22   // cut into lengths 2 + 6 -> 5 + 17 = 22
```

---

## Intuition

This is an **unbounded knapsack** in disguise: the "items" are piece lengths `1..N`, each piece of length `L` has weight `L` and value `price[L-1]`, and capacity is `N`. A piece length can be reused, so we stay on the same length when we take it.

- **State:** `dp[i][len]` = max value using piece lengths `1..i` to fill a rod of length `len`.
- **Recurrence:**
  - Don't use length `i`: `dp[i-1][len]`
  - Use length `i` (reuse allowed): `price[i-1] + dp[i][len - i]`
  - `dp[i][len] = max(...)`
- **Base case:** `dp[i][0] = 0`.

---

## Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

int rodCutting(const vector<int>& price, int N) {
    // length index i (1..N) -> piece of length i with value price[i-1]
    // dp[i][len] : best value cutting a rod of length `len` using lengths 1..i
    vector<vector<int>> dp(N + 1, vector<int>(N + 1, 0));

    for (int i = 1; i <= N; ++i) {
        for (int len = 0; len <= N; ++len) {
            dp[i][len] = dp[i - 1][len];                       // skip length i
            if (i <= len)
                dp[i][len] = max(dp[i][len],
                                 price[i - 1] + dp[i][len - i]); // cut a piece of length i (reuse)
        }
    }
    return dp[N][N];
}

int main() {
    vector<int> price = {1, 5, 8, 9, 10, 17, 17, 20}; // lengths 1..8
    cout << rodCutting(price, 8) << "\n"; // 22
}
```

**Space optimization:** as with unbounded knapsack, collapse to one `vector<int> dp(N+1)` and iterate `len` **left-to-right** so each length can be reused:

```cpp
vector<int> dp(N + 1, 0);
for (int i = 1; i <= N; ++i)
    for (int len = i; len <= N; ++len)
        dp[len] = max(dp[len], price[i - 1] + dp[len - i]);
```

---

## Complexity

| | |
|---|---|
| Time | O(N^2) |
| Space | O(N^2), optimizable to O(N) |

> ⚠️ A piece of length `i` corresponds to `price[i-1]` (price array is 0-indexed). Because reuse is allowed, the "take" term reads the current row `dp[i][len - i]` and the 1-D loop must go left-to-right.
