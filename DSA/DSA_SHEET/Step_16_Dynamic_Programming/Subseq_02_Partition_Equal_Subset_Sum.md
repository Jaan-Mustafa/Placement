# Partition Equal Subset Sum  🟡

**Reference:** https://leetcode.com/problems/partition-equal-subset-sum/

---

## Problem

Given a non-empty array `nums` of positive integers, determine whether it can be partitioned into two subsets such that the sums of the two subsets are equal.

```
nums = [1, 5, 11, 5]
Answer = true   // {1, 5, 5} and {11} both sum to 11
```

---

## Intuition

If the total sum is **odd**, equal partition is impossible. Otherwise each subset must sum to `total / 2`. The problem reduces to **Subset Sum** with `target = total / 2`.

- **State:** `dp[i][t]` = `true` if a subset of the first `i` elements sums to `t`.
- **Recurrence:** `dp[i][t] = dp[i-1][t] || (nums[i-1] <= t && dp[i-1][t - nums[i-1]])`.
- **Base case:** `dp[i][0] = true`.

---

## Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

bool canPartition(const vector<int>& nums) {
    int total = accumulate(nums.begin(), nums.end(), 0);
    if (total % 2 != 0) return false;        // odd total -> cannot split equally
    int target = total / 2;
    int n = nums.size();

    vector<vector<bool>> dp(n + 1, vector<bool>(target + 1, false));
    for (int i = 0; i <= n; ++i) dp[i][0] = true; // sum 0 via empty subset

    for (int i = 1; i <= n; ++i) {
        for (int t = 1; t <= target; ++t) {
            dp[i][t] = dp[i - 1][t];                       // don't take nums[i-1]
            if (nums[i - 1] <= t)
                dp[i][t] = dp[i][t] || dp[i - 1][t - nums[i - 1]]; // take it
        }
    }
    return dp[n][target];
}

int main() {
    vector<int> nums = {1, 5, 11, 5};
    cout << (canPartition(nums) ? "true" : "false") << "\n"; // true
}
```

**Space optimization:** use a single `vector<bool> dp(target+1)` and iterate `t` from `target` down to `nums[i-1]`, giving `O(target)` space.

---

## Complexity

| | |
|---|---|
| Time | O(n * sum/2) |
| Space | O(n * sum/2), optimizable to O(sum/2) |

> ⚠️ Always check `total % 2 != 0` first and return `false` immediately — otherwise `target` becomes a truncated half and the answer is wrong.
