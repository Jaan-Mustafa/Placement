# Number of LIS  🔴

**Reference:** https://leetcode.com/problems/number-of-longest-increasing-subsequence/

---

## Problem

Given an integer array `nums`, return the **number of** longest strictly increasing subsequences (count of distinct LIS, all of maximum length).

```
Input:  nums = [1, 3, 5, 4, 7]
Output: 2
Explanation: The two LIS of length 4 are [1,3,4,7] and [1,3,5,7].

Input:  nums = [2, 2, 2, 2, 2]
Output: 5
Explanation: LIS length is 1; each of the 5 elements is one.
```

---

## Intuition

Extend the standard LIS DP with a count array.

- `len[i]` = length of the longest increasing subsequence ending at `i`.
- `cnt[i]` = number of such longest subsequences ending at `i`.

For each `j < i` with `nums[j] < nums[i]`:
- If `len[j] + 1 > len[i]`: we found a strictly longer chain → set `len[i] = len[j] + 1` and **reset** `cnt[i] = cnt[j]`.
- If `len[j] + 1 == len[i]`: another way to reach the same best length → **accumulate** `cnt[i] += cnt[j]`.

Finally, find the global max length and sum `cnt[i]` over all `i` whose `len[i]` equals that maximum.

---

## Code (C++)

```cpp
#include <vector>
#include <algorithm>
using namespace std;

int findNumberOfLIS(vector<int>& nums) {
    int n = nums.size();
    if (n == 0) return 0;

    vector<int> len(n, 1), cnt(n, 1);   // each element: length 1, one way
    int maxLen = 1;

    for (int i = 0; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (nums[j] < nums[i]) {
                if (len[j] + 1 > len[i]) {
                    len[i] = len[j] + 1;
                    cnt[i] = cnt[j];      // reset: new longer length
                } else if (len[j] + 1 == len[i]) {
                    cnt[i] += cnt[j];     // accumulate: same length, more ways
                }
            }
        }
        maxLen = max(maxLen, len[i]);
    }

    // sum counts over all chains achieving the maximum length
    int total = 0;
    for (int i = 0; i < n; i++)
        if (len[i] == maxLen) total += cnt[i];

    return total;
}
```

Space is O(n) for the `len` and `cnt` arrays.

---

## Complexity

| | |
|---|---|
| Time | O(n^2) |
| Space | O(n) |

> ⚠️ The key subtlety is the reset-vs-accumulate distinction: on a strictly longer chain you **overwrite** `cnt[i]` with `cnt[j]`, but on an equal-length chain you **add**. Mixing these up is the most common bug. For large inputs the counts could overflow; use `long long` if the problem allows huge arrays (LeetCode guarantees it fits in `int`).
