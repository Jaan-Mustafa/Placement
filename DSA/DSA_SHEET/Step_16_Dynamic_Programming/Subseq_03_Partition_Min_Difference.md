# Partition Set into Two Subsets with Minimum Difference  🔴

**Reference:** https://www.geeksforgeeks.org/partition-a-set-into-two-subsets-such-that-the-difference-of-subset-sums-is-minimum/

---

## Problem

Given an array `arr` of non-negative integers, partition it into two subsets `S1` and `S2` so that the absolute difference `|sum(S1) - sum(S2)|` is minimized. Return that minimum difference.

```
arr = [1, 6, 11, 5]
Answer = 1
// {1, 6, 5} sum=12 , {11} sum=11 -> |12 - 11| = 1
```

---

## Intuition

Let `total` be the sum of all elements. If one subset has sum `s1`, the other has `total - s1`, and the difference is `|total - 2*s1|`. To minimize it we want `s1` as close to `total/2` as possible.

- First compute **all achievable subset sums** using the subset-sum DP.
- **State:** `dp[i][s]` = `true` if some subset of the first `i` elements sums to `s`.
- Then scan every achievable `s1` in `[0, total/2]` and minimize `total - 2*s1`.

---

## Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

int minSubsetSumDifference(const vector<int>& arr) {
    int n = arr.size();
    int total = accumulate(arr.begin(), arr.end(), 0);

    // dp[i][s] : can a subset of first i elements achieve sum s
    vector<vector<bool>> dp(n + 1, vector<bool>(total + 1, false));
    for (int i = 0; i <= n; ++i) dp[i][0] = true;

    for (int i = 1; i <= n; ++i)
        for (int s = 1; s <= total; ++s) {
            dp[i][s] = dp[i - 1][s];                     // skip arr[i-1]
            if (arr[i - 1] <= s)
                dp[i][s] = dp[i][s] || dp[i - 1][s - arr[i - 1]]; // include it
        }

    int best = INT_MAX;
    // Only need to check up to total/2; s2 = total - s1 mirrors the other half
    for (int s1 = 0; s1 <= total / 2; ++s1)
        if (dp[n][s1])
            best = min(best, total - 2 * s1);            // |s2 - s1| = total - 2*s1

    return best;
}

int main() {
    vector<int> arr = {1, 6, 11, 5};
    cout << minSubsetSumDifference(arr) << "\n"; // 1
}
```

**Space optimization:** only the last row of `dp` is needed for the final scan, so a single `vector<bool> dp(total+1)` (filled with right-to-left iteration per element) reduces space to `O(total)`.

---

## Complexity

| | |
|---|---|
| Time | O(n * total) |
| Space | O(n * total), optimizable to O(total) |

> ⚠️ Scan `s1` only up to `total/2`; for each achievable `s1` the difference is `total - 2*s1` (already non-negative there), so no `abs()` is required.
