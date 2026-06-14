# Maximum Sum of Non-Adjacent Elements (House Robber I)  🟡

**Reference:** https://leetcode.com/problems/house-robber/ , https://www.geeksforgeeks.org/maximum-sum-such-that-no-two-elements-are-adjacent/

---

## Problem

Given an array `nums` where `nums[i]` is the money in house `i`, find the maximum amount you can rob **without robbing two adjacent houses** (robbing adjacent houses triggers the alarm).

```
Input : nums = [2, 7, 9, 3, 1]
Output: 12
Explanation: rob house 0 (2) + house 2 (9) + house 4 (1) = 12
```

---

## Intuition

At each house `i` you make a choice:
- **Pick** house `i`: take `nums[i]` and add the best you could do up to `i-2` (you must skip `i-1`).
- **Skip** house `i`: carry forward the best up to `i-1`.

Take the maximum of these two.

**DP state:** `dp[i]` = maximum sum robbing from houses `0..i` with no two adjacent.
**Recurrence:** `dp[i] = max(nums[i] + dp[i-2], dp[i-1])`.
**Base cases:** `dp[0] = nums[0]`, `dp[1] = max(nums[0], nums[1])`.

---

## Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

int rob(vector<int>& nums) {
    int n = nums.size();
    if (n == 0) return 0;
    if (n == 1) return nums[0];
    vector<int> dp(n, 0);
    dp[0] = nums[0];                          // base: only first house
    dp[1] = max(nums[0], nums[1]);            // base: best of first two
    for (int i = 2; i < n; ++i) {
        int pick = nums[i] + dp[i - 2];       // rob i, skip i-1
        int skip = dp[i - 1];                 // don't rob i
        dp[i] = max(pick, skip);
    }
    return dp[n - 1];
}

int main() {
    vector<int> nums = {2, 7, 9, 3, 1};
    cout << rob(nums) << "\n"; // 12
    return 0;
}
```

> note: `dp[i]` depends only on `dp[i-1]` and `dp[i-2]`, so this is easily reduced to `O(1)` space with two variables (`prev1`, `prev2`).

---

## Complexity

| | |
|---|---|
| Time | O(n) |
| Space | O(n) (reducible to O(1)) |

> ⚠️ When picking house `i`, add `dp[i-2]` (not `nums[i-2]`) — you want the best achievable sum up to `i-2`, not just that single house's money.
> ⚠️ Handle empty and single-element arrays before initializing `dp[1]`.
