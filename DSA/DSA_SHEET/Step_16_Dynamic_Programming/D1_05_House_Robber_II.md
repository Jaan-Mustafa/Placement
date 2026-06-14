# House Robber II  🟡

**Reference:** https://leetcode.com/problems/house-robber-ii/

---

## Problem

Same rules as House Robber I (no two adjacent houses), but now the houses are arranged in a **circle** — the first house is adjacent to the last house. So you can never rob both `nums[0]` and `nums[n-1]`. Find the maximum amount you can rob.

```
Input : nums = [2, 3, 2]
Output: 3
Explanation: houses 0 and 2 are adjacent (circle), so you cannot rob both.
             Best is to rob house 1 alone = 3.
```

---

## Intuition

Because the array is circular, the first and last houses cannot both be robbed. Split into **two independent linear problems** and take the better:

1. Consider houses `[0 .. n-2]` (exclude the last house).
2. Consider houses `[1 .. n-1]` (exclude the first house).

Each subproblem is exactly **House Robber I** on a linear range. The answer is the maximum of the two.

**DP state:** for a linear segment, `dp[i]` = max non-adjacent sum up to index `i`.
**Recurrence:** `dp[i] = max(nums[i] + dp[i-2], dp[i-1])`.
**Final answer:** `max(robLinear(0, n-2), robLinear(1, n-1))`.

---

## Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

// House Robber I on the sub-array nums[start..end] (inclusive)
int robLinear(vector<int>& nums, int start, int end) {
    int prev2 = 0;                 // best up to i-2
    int prev1 = 0;                 // best up to i-1
    for (int i = start; i <= end; ++i) {
        int pick = nums[i] + prev2;        // rob i
        int skip = prev1;                  // skip i
        int cur  = max(pick, skip);
        prev2 = prev1;                     // slide window
        prev1 = cur;
    }
    return prev1;                  // best for the whole segment
}

int rob(vector<int>& nums) {
    int n = nums.size();
    if (n == 0) return 0;
    if (n == 1) return nums[0];            // single house: no circle issue
    // Case A: skip last house | Case B: skip first house
    int caseA = robLinear(nums, 0, n - 2);
    int caseB = robLinear(nums, 1, n - 1);
    return max(caseA, caseB);
}

int main() {
    vector<int> a = {2, 3, 2};
    cout << rob(a) << "\n"; // 3
    vector<int> b = {1, 2, 3, 1};
    cout << rob(b) << "\n"; // 4
    return 0;
}
```

> note: `robLinear` already uses the `O(1)` space-optimized form (two rolling variables) instead of a full `dp[]` array.

---

## Complexity

| | |
|---|---|
| Time | O(n) (two linear passes) |
| Space | O(1) |

> ⚠️ Handle `n == 1` separately: with a single house there is no first/last adjacency, and the `[0..n-2]` range would be empty.
> ⚠️ Do **not** run the linear robber over the full circular array — you must explicitly exclude either the first or the last house, otherwise both could be picked illegally.
