# LIS using Binary Search  🟡

**Reference:** https://leetcode.com/problems/longest-increasing-subsequence/

---

## Problem

Given an integer array `nums`, return the length of the longest strictly increasing subsequence — but in O(n log n) time using the patience-sorting / binary-search technique.

```
Input:  nums = [10, 9, 2, 5, 3, 7, 101, 18]
Output: 4
Explanation: LIS [2, 3, 7, 18] has length 4.
```

---

## Intuition

Maintain an array `tails` where, conceptually, `tails[k]` is the **smallest possible tail value** of any increasing subsequence of length `k+1` seen so far. `tails` is always sorted in increasing order, which lets us binary search.

For each `x = nums[i]`:
- Find the first element in `tails` that is `>= x` (use `lower_bound` for strictly increasing).
- If none exists, `x` extends the longest chain → append it (length grows).
- Otherwise replace that element with `x` — this keeps the tail as small as possible so future numbers have a better chance to extend.

The length of `tails` at the end is the LIS length. Note: `tails` itself is **not** a valid LIS, only its length is meaningful.

State: `tails[k]` = minimal tail of an increasing subsequence of length `k+1`. Transition: each element either appends or overwrites via binary search.

---

## Code (C++)

```cpp
#include <vector>
#include <algorithm>
using namespace std;

int lengthOfLIS(vector<int>& nums) {
    vector<int> tails;             // tails[k] = smallest tail of LIS of length k+1

    for (int x : nums) {
        // lower_bound -> first element >= x (strictly increasing LIS)
        auto it = lower_bound(tails.begin(), tails.end(), x);
        if (it == tails.end()) {
            tails.push_back(x);    // x extends the longest subsequence
        } else {
            *it = x;               // replace to keep tail minimal
        }
    }
    return (int)tails.size();      // length of tails == LIS length
}
```

For a **non-decreasing** (allows equal) LIS, swap `lower_bound` for `upper_bound`. Space is O(n) for the `tails` buffer; it cannot be reduced further but is the same order as the O(n^2) DP while running much faster.

---

## Complexity

| | |
|---|---|
| Time | O(n log n) |
| Space | O(n) |

> ⚠️ `tails` does **not** hold an actual LIS — it can be a mix of values. Use it only for the length. For strictly increasing use `lower_bound`; using `upper_bound` would (incorrectly) count equal elements.
