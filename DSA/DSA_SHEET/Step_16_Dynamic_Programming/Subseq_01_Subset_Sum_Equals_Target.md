# Subset Sum Equals Target  🟡

**Reference:** https://www.geeksforgeeks.org/subset-sum-problem-dp-25/

---

## Problem

Given an array of non-negative integers `arr` and a value `target`, determine whether there exists a subset of `arr` whose elements sum to exactly `target`.

```
arr    = [3, 34, 4, 12, 5, 2]
target = 9
Answer = true   // subset {4, 5} sums to 9
```

---

## Intuition

For every element we have a choice: **pick it** (if it does not exceed the remaining target) or **not pick it**.

- **State:** `dp[i][t]` = `true` if some subset of the first `i` elements (`arr[0..i-1]`) sums to `t`.
- **Recurrence:**
  - Don't take: `dp[i-1][t]`
  - Take (only if `arr[i-1] <= t`): `dp[i-1][t - arr[i-1]]`
  - `dp[i][t] = notTake || take`
- **Base case:** `dp[i][0] = true` (empty subset gives sum 0), `dp[0][t>0] = false`.

---

## Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

bool subsetSum(const vector<int>& arr, int target) {
    int n = arr.size();
    // dp[i][t] : can we form sum t using first i elements
    vector<vector<bool>> dp(n + 1, vector<bool>(target + 1, false));

    // Sum 0 is always achievable with the empty subset
    for (int i = 0; i <= n; ++i) dp[i][0] = true;

    for (int i = 1; i <= n; ++i) {
        for (int t = 1; t <= target; ++t) {
            bool notTake = dp[i - 1][t];
            bool take = false;
            if (arr[i - 1] <= t)                 // element fits in remaining target
                take = dp[i - 1][t - arr[i - 1]];
            dp[i][t] = notTake || take;
        }
    }
    return dp[n][target];
}

int main() {
    vector<int> arr = {3, 34, 4, 12, 5, 2};
    cout << (subsetSum(arr, 9) ? "true" : "false") << "\n"; // true
}
```

**Space optimization:** each row depends only on the previous row, so two 1-D arrays (`prev`, `curr`) of size `target+1` suffice, reducing space to `O(target)`. Iterating a single array from right to left also works for the 0/1 style.

---

## Complexity

| | |
|---|---|
| Time | O(n * target) |
| Space | O(n * target), optimizable to O(target) |

> ⚠️ This assumes **non-negative** integers. With negative numbers the sum range can be negative and a simple 0..target table no longer indexes correctly.
