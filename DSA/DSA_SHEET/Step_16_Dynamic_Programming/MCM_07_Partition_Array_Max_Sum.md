# Partition Array for Maximum Sum  🟡

**Reference:** https://leetcode.com/problems/partition-array-for-maximum-sum/

---

## Problem

Given an integer array `arr` and an integer `k`, partition the array into contiguous subarrays of length **at most `k`**. After partitioning, every element of a subarray becomes the maximum value of that subarray. Return the largest possible sum of the resulting array.

```
arr = [1, 15, 7, 9, 2, 5, 10], k = 3
Best partition -> [15,15,15, 9,9, 10,10] (one valid split)
Answer = 84
```

---

## Intuition

This is **front partition DP**. Define `dp[i]` = maximum sum obtainable from the suffix `arr[i..n-1]`. From index `i` we choose the length `len` of the first subarray (`1 <= len <= k` and not past the end). That subarray contributes `len * (max of its elements)`, then we recurse on the rest.

Recurrence (partition over the first block length):
- `dp[i] = max over len in [1, min(k, n-i)] of ( len * maxInWindow + dp[i + len] )`
- where `maxInWindow` is the running max of `arr[i .. i+len-1]`.

Base case: `dp[n] = 0` (empty suffix). Answer = `dp[0]`.

---

## Code (C++)

```cpp
#include <vector>
#include <algorithm>
using namespace std;

int maxSumAfterPartitioning(vector<int>& arr, int k) {
    int n = arr.size();
    // dp[i]: best sum for suffix arr[i..n-1]; dp[n] = 0
    vector<int> dp(n + 1, 0);

    for (int i = n - 1; i >= 0; --i) {
        int curMax = 0, best = 0;
        // try first block of length len, extend the window incrementally
        for (int len = 1; len <= k && i + len <= n; ++len) {
            curMax = max(curMax, arr[i + len - 1]);   // running window max
            int cand = len * curMax + dp[i + len];
            best = max(best, cand);
        }
        dp[i] = best;
    }
    return dp[0];
}
```

> Keeping a running `curMax` as `len` grows avoids re-scanning the window, so each `i` costs O(k) instead of O(k²). Space is O(n) and is not further reducible since `dp[i]` looks ahead up to `k` cells.

---

## Complexity

| | |
|---|---|
| Time | O(n·k) |
| Space | O(n) |

> ⚠️ Subarray length is *at most* k, so loop `len` from 1 and stop at `min(k, n-i)`. Multiply by the window max times the actual length (`len * curMax`), not by k.
