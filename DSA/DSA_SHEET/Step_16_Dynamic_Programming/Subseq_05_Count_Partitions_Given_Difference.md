# Count Partitions with Given Difference  🔴

**Reference:** https://www.geeksforgeeks.org/count-of-subsets-with-sum-equal-to-x/ (Striver "Partitions with given difference")

---

## Problem

Given an array `arr` of non-negative integers and a number `D`, count the number of ways to partition the array into two subsets `S1` and `S2` (with `sum(S1) >= sum(S2)`) such that `sum(S1) - sum(S2) = D`.

```
arr = [5, 2, 6, 4]
D   = 3
Answer = 1
// {5, 2} sum=7 , {6, 4}... actually {6} vs rest -> the one valid split giving diff 3
```

---

## Intuition

Let `total` be the sum of all elements and `s2 = sum(S2)`. Then `sum(S1) = total - s2`, and:

```
sum(S1) - sum(S2) = D
(total - s2) - s2 = D
s2 = (total - D) / 2
```

So the answer is the **number of subsets whose sum equals `target = (total - D) / 2`**.

- **Validity:** `(total - D)` must be non-negative and even, else the answer is `0`.
- **State:** `dp[i][t]` = number of subsets of first `i` elements with sum `t`.
- **Recurrence:** `dp[i][t] = dp[i-1][t] + (arr[i-1] <= t ? dp[i-1][t - arr[i-1]] : 0)`.

---

## Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MOD = 1e9 + 7;

int countSubsetsWithSum(const vector<int>& arr, int target) {
    int n = arr.size();
    vector<vector<int>> dp(n + 1, vector<int>(target + 1, 0));
    dp[0][0] = 1;                            // empty subset -> sum 0

    for (int i = 1; i <= n; ++i)
        for (int t = 0; t <= target; ++t) {  // start at 0 to handle zero elements
            dp[i][t] = dp[i - 1][t];
            if (arr[i - 1] <= t)
                dp[i][t] = (dp[i][t] + dp[i - 1][t - arr[i - 1]]) % MOD;
        }
    return dp[n][target];
}

int countPartitions(const vector<int>& arr, int D) {
    int total = accumulate(arr.begin(), arr.end(), 0);
    // s2 = (total - D) / 2 must be a valid non-negative integer
    if (total - D < 0 || (total - D) % 2 != 0) return 0;
    return countSubsetsWithSum(arr, (total - D) / 2);
}

int main() {
    vector<int> arr = {5, 2, 6, 4};
    cout << countPartitions(arr, 3) << "\n"; // 1
}
```

**Space optimization:** the count DP needs only the previous row, so a single `vector<int> dp(target+1)` iterated right-to-left per element gives `O(target)` space.

---

## Complexity

| | |
|---|---|
| Time | O(n * (total - D)/2) |
| Space | O(n * target), optimizable to O(target) |

> ⚠️ Guard against `total - D < 0` and odd `(total - D)` — both must return `0`. Iterate `t` from `0` so zero-valued elements double counts correctly, and apply `MOD` to avoid overflow.
