# Burst Balloons  🔴

**Reference:** https://leetcode.com/problems/burst-balloons/

---

## Problem

Given `n` balloons each with a number painted on it (`nums`), bursting balloon `i` gives `nums[i-1] * nums[i] * nums[i+1]` coins, where out-of-range neighbours are treated as `1`. After bursting, the neighbours become adjacent. Return the maximum coins obtainable by bursting all balloons.

```
nums = [3, 1, 5, 8]
Optimal order yields 167 coins
```

---

## Intuition

The trap: if we think about which balloon to burst **first**, the array keeps splitting and sub-problems are not independent (neighbours change). The fix is to think about which balloon to burst **last** in an interval.

Pad the array with `1` on both ends: `arr = [1, nums..., 1]`. Consider the open interval `(i, j)` — burst everything strictly between markers `i` and `j`. If balloon `k` is the **last** one burst in `(i, j)`, then at that moment its surviving neighbours are exactly `arr[i]` and `arr[j]` (everything else inside is already gone).

Define `f(i, j)` = max coins from bursting all balloons strictly between `i` and `j`.

Recurrence (partition over the **last** balloon `k`):
- `f(i, j) = max over k in (i, j) of ( arr[i] * arr[k] * arr[j] + f(i, k) + f(k, j) )`

Base case: `j - i == 1` (nothing between) gives `0`. Answer = `f(0, n+1)`.

---

## Code (C++)

```cpp
#include <vector>
#include <algorithm>
using namespace std;

int maxCoins(vector<int>& nums) {
    int n = nums.size();
    vector<int> arr(n + 2);
    arr[0] = arr[n + 1] = 1;                 // virtual boundary balloons
    for (int i = 0; i < n; ++i) arr[i + 1] = nums[i];

    int m = n + 2;
    // dp[i][j]: max coins bursting balloons strictly between i and j
    vector<vector<int>> dp(m, vector<int>(m, 0));

    // increasing interval length
    for (int len = 2; len < m; ++len) {
        for (int i = 0; i + len < m; ++i) {
            int j = i + len;
            int best = 0;
            for (int k = i + 1; k < j; ++k) {     // k = LAST balloon burst
                int coins = arr[i] * arr[k] * arr[j] // k's neighbours are i, j
                          + dp[i][k] + dp[k][j];
                best = max(best, coins);
            }
            dp[i][j] = best;
        }
    }
    return dp[0][m - 1];
}
```

> The "burst last" reformulation is what makes the halves independent. Bursting "first" does not, so the obvious greedy/forward partition fails.

---

## Complexity

| | |
|---|---|
| Time | O(n³) |
| Space | O(n²) |

> ⚠️ Think LAST, not first. The coin term multiplies `arr[i] * arr[k] * arr[j]` using the *interval boundaries* as neighbours, because everything strictly inside `(i, j)` is gone by the time `k` bursts.
