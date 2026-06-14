# Count Subsets with Sum K  🟡

**Reference:** https://www.geeksforgeeks.org/perfect-sum-problem/

---

## Problem

Given an array `arr` of non-negative integers and a value `k`, count the number of subsets whose elements sum to exactly `k`.

```
arr = [1, 2, 3, 3]
k   = 6
Answer = 3
// {3,3}, {1,2,3}, {1,2,3}  (the two 3's are distinct positions)
```

---

## Intuition

Like subset sum, but instead of a boolean we **count** the number of ways.

- **State:** `dp[i][t]` = number of subsets of the first `i` elements summing to `t`.
- **Recurrence:** `dp[i][t] = dp[i-1][t] + (arr[i-1] <= t ? dp[i-1][t - arr[i-1]] : 0)`.
- **Base case:** `dp[0][0] = 1` (one way — the empty subset — to make sum 0).

> Zeros are tricky: a `0` can be either included or excluded, so each zero doubles the count of every target. The base-case handling below treats zeros correctly because `dp[i][t] += dp[i-1][t]` (exclude 0) `+ dp[i-1][t-0]` (include 0).

---

## Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MOD = 1e9 + 7;

int countSubsets(const vector<int>& arr, int k) {
    int n = arr.size();
    // dp[i][t] : number of subsets of first i elements with sum t
    vector<vector<int>> dp(n + 1, vector<int>(k + 1, 0));
    dp[0][0] = 1;                       // empty subset -> sum 0

    for (int i = 1; i <= n; ++i) {
        for (int t = 0; t <= k; ++t) {
            dp[i][t] = dp[i - 1][t];                          // exclude arr[i-1]
            if (arr[i - 1] <= t)
                dp[i][t] = (dp[i][t] + dp[i - 1][t - arr[i - 1]]) % MOD; // include it
        }
    }
    return dp[n][k];
}

int main() {
    vector<int> arr = {1, 2, 3, 3};
    cout << countSubsets(arr, 6) << "\n"; // 3
}
```

**Space optimization:** rows depend only on the previous row, so two 1-D arrays of size `k+1` (or one array iterated right-to-left) cut space to `O(k)`.

---

## Complexity

| | |
|---|---|
| Time | O(n * k) |
| Space | O(n * k), optimizable to O(k) |

> ⚠️ Start the inner loop at `t = 0` (not `t = 1`) so that elements equal to `0` correctly double the counts. Use a modulus when counts can overflow.
