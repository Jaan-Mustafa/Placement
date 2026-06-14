# Target Sum  🟡

**Reference:** https://leetcode.com/problems/target-sum/

---

## Problem

Given an integer array `nums` and a target `S`, assign a `+` or `-` sign to each number so that the resulting expression evaluates to `S`. Return the **number of ways** to do so.

```
nums = [1, 1, 1, 1, 1]
S    = 3
Answer = 5
// -1+1+1+1+1, +1-1+1+1+1, +1+1-1+1+1, +1+1+1-1+1, +1+1+1+1-1
```

---

## Intuition

Split the numbers into a positive set `P` (with `+`) and a negative set `N` (with `-`):

```
sum(P) - sum(N) = S
sum(P) + sum(N) = total
=> sum(P) = (total + S) / 2
```

So this reduces to **counting subsets with sum `target = (total + S) / 2`**.

- **Validity:** `(total + S)` must be non-negative and even; also `abs(S) <= total`.
- **State:** `dp[i][t]` = number of subsets of the first `i` elements summing to `t`.
- **Recurrence:** `dp[i][t] = dp[i-1][t] + (nums[i-1] <= t ? dp[i-1][t - nums[i-1]] : 0)`.

---

## Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

int countSubsetsWithSum(const vector<int>& nums, int target) {
    int n = nums.size();
    vector<vector<int>> dp(n + 1, vector<int>(target + 1, 0));
    dp[0][0] = 1;                              // empty subset -> sum 0

    for (int i = 1; i <= n; ++i)
        for (int t = 0; t <= target; ++t) {    // t from 0 to handle zero elements
            dp[i][t] = dp[i - 1][t];
            if (nums[i - 1] <= t)
                dp[i][t] += dp[i - 1][t - nums[i - 1]];
        }
    return dp[n][target];
}

int findTargetSumWays(const vector<int>& nums, int S) {
    int total = accumulate(nums.begin(), nums.end(), 0);
    // target = (total + S) / 2 must be a valid non-negative integer
    if (total + S < 0 || (total + S) % 2 != 0) return 0;
    return countSubsetsWithSum(nums, (total + S) / 2);
}

int main() {
    vector<int> nums = {1, 1, 1, 1, 1};
    cout << findTargetSumWays(nums, 3) << "\n"; // 5
}
```

**Space optimization:** the count DP needs only the previous row, so a single `vector<int> dp(target+1)` iterated right-to-left per element gives `O(target)` space.

---

## Complexity

| | |
|---|---|
| Time | O(n * (total + S)/2) |
| Space | O(n * target), optimizable to O(target) |

> ⚠️ Handle the edge cases: if `total + S` is negative or odd, return `0`. Iterating `t` from `0` is essential so that `0`-valued elements double the count.
