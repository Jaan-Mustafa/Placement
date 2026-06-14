# Longest Increasing Subsequence  🟡

**Reference:** https://leetcode.com/problems/longest-increasing-subsequence/

---

## Problem

Given an integer array `nums`, return the length of the longest strictly increasing subsequence. A subsequence is derived by deleting zero or more elements without changing the order of the remaining elements.

```
Input:  nums = [10, 9, 2, 5, 3, 7, 101, 18]
Output: 4
Explanation: One LIS is [2, 3, 7, 18] (length 4).
```

---

## Intuition

Let `dp[i]` = length of the longest strictly increasing subsequence **ending exactly at index `i`** (and including `nums[i]`).

Recurrence: every element alone is a subsequence of length 1. For each `i`, look at every previous `j < i`. If `nums[j] < nums[i]`, then `nums[i]` can extend the subsequence ending at `j`:

```
dp[i] = 1 + max(dp[j])   for all j < i with nums[j] < nums[i]
dp[i] = 1                if no such j exists
```

The answer is `max(dp[i])` over all `i`, because the LIS can end at any index.

---

## Code (C++)

```cpp
#include <vector>
#include <algorithm>
using namespace std;

int lengthOfLIS(vector<int>& nums) {
    int n = nums.size();
    if (n == 0) return 0;

    // dp[i] = length of LIS ending at index i
    vector<int> dp(n, 1);          // every single element is an LIS of length 1
    int best = 1;                  // overall answer

    for (int i = 1; i < n; i++) {
        for (int j = 0; j < i; j++) {
            // strictly increasing: nums[j] must be smaller
            if (nums[j] < nums[i]) {
                dp[i] = max(dp[i], dp[j] + 1);
            }
        }
        best = max(best, dp[i]);   // LIS can end anywhere
    }
    return best;
}
```

This O(n^2) tabulation uses O(n) extra space for `dp`. Space cannot be reduced below O(n) here because computing `dp[i]` needs every earlier `dp[j]`. For an O(n log n) approach see the binary-search variant (LIS_03).

---

## Complexity

| | |
|---|---|
| Time | O(n^2) |
| Space | O(n) |

> ⚠️ The subsequence must be **strictly** increasing, so use `<` not `<=`. Initialize the whole `dp` array to `1`, not `0`, since a single element already forms a valid subsequence.
