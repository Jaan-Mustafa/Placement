# Longest Bitonic Subsequence  🔴

**Reference:** https://www.geeksforgeeks.org/longest-bitonic-subsequence-dp-15/

---

## Problem

Given an array `nums`, find the length of the longest **bitonic** subsequence: a subsequence that first strictly increases and then strictly decreases. A purely increasing or purely decreasing subsequence also counts as bitonic.

```
Input:  nums = [1, 11, 2, 10, 4, 5, 2, 1]
Output: 6
Explanation: [1, 2, 10, 4, 2, 1] increases then decreases (length 6).
```

---

## Intuition

A bitonic subsequence has a "peak" index. For each index `i` as the peak, the best bitonic length is:

```
(LIS ending at i, going left-to-right) + (LDS starting at i, going right-to-left) - 1
```

We subtract 1 because the peak element `i` is counted in both halves.

So compute two DP arrays:
- `lis[i]` = length of longest strictly increasing subsequence ending at `i` (scan left→right).
- `lds[i]` = length of longest strictly decreasing subsequence starting at `i` (scan right→left, i.e. increasing from the right).

Answer = `max(lis[i] + lds[i] - 1)`.

---

## Code (C++)

```cpp
#include <vector>
#include <algorithm>
using namespace std;

int longestBitonicSubsequence(vector<int>& nums) {
    int n = nums.size();
    if (n == 0) return 0;

    vector<int> lis(n, 1), lds(n, 1);

    // lis[i]: longest increasing subsequence ending at i
    for (int i = 0; i < n; i++)
        for (int j = 0; j < i; j++)
            if (nums[j] < nums[i])
                lis[i] = max(lis[i], lis[j] + 1);

    // lds[i]: longest decreasing subsequence starting at i
    // (scan from the right; increasing when read right-to-left)
    for (int i = n - 1; i >= 0; i--)
        for (int j = n - 1; j > i; j--)
            if (nums[j] < nums[i])
                lds[i] = max(lds[i], lds[j] + 1);

    int best = 0;
    for (int i = 0; i < n; i++)
        best = max(best, lis[i] + lds[i] - 1);  // peak counted once

    return best;
}
```

Space is O(n) for the two arrays. Both DP passes are the standard O(n^2) LIS recurrence applied in opposite directions.

---

## Complexity

| | |
|---|---|
| Time | O(n^2) |
| Space | O(n) |

> ⚠️ Subtract 1 for the shared peak, otherwise it is double-counted. The strict comparisons (`<`) match the strictly-increasing-then-strictly-decreasing definition; the classic GfG version requires both sides to be non-empty in some variants — here a strictly monotone sequence is also accepted as bitonic.
