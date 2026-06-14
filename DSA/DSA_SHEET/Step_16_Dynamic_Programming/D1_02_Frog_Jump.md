# Frog Jump  🟡

**Reference:** https://www.geeksforgeeks.org/geek-jump/ , https://www.codingninjas.com/studio/problems/frog-jump_3621012

---

## Problem

A frog starts at stone `0` and wants to reach stone `n-1`. `height[i]` is the height of the i-th stone. From stone `i` the frog can jump to stone `i+1` or `i+2`. The **cost** of a jump from `i` to `j` is `abs(height[i] - height[j])`. Find the **minimum total cost** to reach the last stone.

```
Input : height = [10, 20, 30, 10]
Output: 20
Explanation:
  0 -> 2 cost |10-30| = 20
  2 -> 3 cost |30-10| = 20  -> total 40
  0 -> 1 -> 3 cost |10-20| + |20-10| = 10 + 10 = 20  (better)
Minimum = 20
```

---

## Intuition

To stand on stone `i`, the frog arrived either from `i-1` (one jump) or from `i-2` (two jumps). Take the cheaper of the two options plus the jump cost.

**DP state:** `dp[i]` = minimum cost to reach stone `i` from stone `0`.
**Recurrence:**
```
oneStep = dp[i-1] + abs(height[i] - height[i-1])
twoStep = dp[i-2] + abs(height[i] - height[i-2])   // only if i >= 2
dp[i]   = min(oneStep, twoStep)
```
**Base case:** `dp[0] = 0`.

---

## Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

int frogJump(vector<int>& height) {
    int n = height.size();
    if (n == 1) return 0;
    vector<int> dp(n, 0);
    dp[0] = 0;                                   // base: starting stone
    for (int i = 1; i < n; ++i) {
        int oneStep = dp[i - 1] + abs(height[i] - height[i - 1]);
        int twoStep = INT_MAX;
        if (i > 1)                               // two-step only valid from i-2
            twoStep = dp[i - 2] + abs(height[i] - height[i - 2]);
        dp[i] = min(oneStep, twoStep);
    }
    return dp[n - 1];
}

int main() {
    vector<int> h = {10, 20, 30, 10};
    cout << frogJump(h) << "\n"; // 20
    return 0;
}
```

> note: `dp[i]` depends only on `dp[i-1]` and `dp[i-2]`, so this can run in `O(1)` space using two rolling variables.

---

## Complexity

| | |
|---|---|
| Time | O(n) |
| Space | O(n) (reducible to O(1)) |

> ⚠️ The two-step jump is only valid when `i >= 2`; guard against accessing `dp[i-2]` for `i = 1`.
> ⚠️ The cost is the **absolute difference of heights**, not the difference in indices — a common misread.
