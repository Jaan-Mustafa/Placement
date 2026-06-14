# Largest Divisible Subset  🟡

**Reference:** https://leetcode.com/problems/largest-divisible-subset/

---

## Problem

Given a set of **distinct** positive integers `nums`, return the largest subset such that for every pair `(a, b)` in it, either `a % b == 0` or `b % a == 0`. If multiple answers exist, return any.

```
Input:  nums = [1, 2, 4, 8]
Output: [1, 2, 4, 8]   (each divides the next)

Input:  nums = [1, 2, 3]
Output: [1, 2] or [1, 3]
```

---

## Intuition

This is LIS in disguise. **Sort the array first.** After sorting, a divisible chain only needs each element to be divisible by the previous chosen one — because divisibility, like LIS, is transitive when ordered (`a | b` and `b | c` ⇒ `a | c`), checking adjacent picks suffices.

Let `dp[i]` = size of the largest divisible subset **ending at index `i`** (sorted). Keep a back-pointer `hash[i]`.

```
dp[i]   = 1 + max over j<i (dp[j])  with nums[i] % nums[j] == 0
hash[i] = the chosen j
```

Find the index with the global max `dp`, then follow back-pointers and reverse.

---

## Code (C++)

```cpp
#include <vector>
#include <algorithm>
using namespace std;

vector<int> largestDivisibleSubset(vector<int>& nums) {
    int n = nums.size();
    if (n == 0) return {};

    sort(nums.begin(), nums.end());     // crucial: enables adjacent divisibility check

    vector<int> dp(n, 1), hash(n);
    for (int i = 0; i < n; i++) hash[i] = i;

    int maxLen = 1, lastIndex = 0;

    for (int i = 0; i < n; i++) {
        for (int j = 0; j < i; j++) {
            // since sorted, nums[i] >= nums[j]; only check nums[i] % nums[j]
            if (nums[i] % nums[j] == 0 && dp[j] + 1 > dp[i]) {
                dp[i]   = dp[j] + 1;
                hash[i] = j;
            }
        }
        if (dp[i] > maxLen) {
            maxLen    = dp[i];
            lastIndex = i;
        }
    }

    vector<int> res;
    res.push_back(nums[lastIndex]);
    while (hash[lastIndex] != lastIndex) {
        lastIndex = hash[lastIndex];
        res.push_back(nums[lastIndex]);
    }
    reverse(res.begin(), res.end());    // chain built backwards
    return res;
}
```

Space is O(n) for `dp` plus O(n) for the `hash` back-pointers used to rebuild the subset.

---

## Complexity

| | |
|---|---|
| Time | O(n^2)  (plus O(n log n) sort) |
| Space | O(n) |

> ⚠️ Sorting is mandatory — without it you would have to test both `a%b` and `b%a` for every pair and the chain logic breaks. The problem guarantees distinct integers, so no duplicate handling is needed.
