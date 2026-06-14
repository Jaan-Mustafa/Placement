# Print LIS  🟡

**Reference:** https://www.geeksforgeeks.org/construction-of-longest-increasing-subsequence-using-dynamic-programming/
Related: https://leetcode.com/problems/longest-increasing-subsequence/

---

## Problem

Given an integer array `nums`, return one actual longest strictly increasing subsequence (not just its length).

```
Input:  nums = [10, 9, 2, 5, 3, 7, 101, 18]
Output: [2, 3, 7, 18]   (any valid LIS of maximum length is acceptable)
```

---

## Intuition

Build the same `dp[i]` as in the basic LIS (length of LIS ending at index `i`), but additionally store a **parent/back-pointer** `hash[i]` = the previous index `j` chosen when extending to `i`.

```
dp[i]   = 1 + max over j<i (dp[j])  with nums[j] < nums[i]
hash[i] = the j that gave that maximum (or i itself if none)
```

Track the index `lastIndex` where the global maximum `dp` value occurs. Then walk the back-pointers from `lastIndex` until `hash[i] == i`, collecting elements, and finally reverse to get increasing order.

---

## Code (C++)

```cpp
#include <vector>
#include <algorithm>
using namespace std;

vector<int> printLIS(vector<int>& nums) {
    int n = nums.size();
    if (n == 0) return {};

    vector<int> dp(n, 1);      // LIS length ending at i
    vector<int> hash(n);       // back-pointer to previous index
    for (int i = 0; i < n; i++) hash[i] = i;  // self = chain start

    int maxLen = 1, lastIndex = 0;

    for (int i = 0; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (nums[j] < nums[i] && dp[j] + 1 > dp[i]) {
                dp[i]   = dp[j] + 1;
                hash[i] = j;            // remember where we came from
            }
        }
        if (dp[i] > maxLen) {           // track best ending index
            maxLen    = dp[i];
            lastIndex = i;
        }
    }

    // Reconstruct by following back-pointers
    vector<int> lis;
    lis.push_back(nums[lastIndex]);
    while (hash[lastIndex] != lastIndex) {
        lastIndex = hash[lastIndex];
        lis.push_back(nums[lastIndex]);
    }
    reverse(lis.begin(), lis.end());   // built backwards, so reverse
    return lis;
}
```

Space is O(n) for both `dp` and `hash`; the `hash` array is the extra cost of reconstruction over plain length computation.

---

## Complexity

| | |
|---|---|
| Time | O(n^2) |
| Space | O(n) |

> ⚠️ Initialize `hash[i] = i` so the reconstruction loop terminates at a chain start. Don't forget the final `reverse`, since elements are collected from the end backwards.
