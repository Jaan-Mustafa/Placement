# Matrix Chain Multiplication (Bottom-Up)  🔴

**Reference:** https://www.geeksforgeeks.org/matrix-chain-multiplication-dp-8/

---

## Problem

Same as `MCM_01`: given a dimension array `arr` describing `n-1` matrices (matrix `i` is `arr[i-1] x arr[i]`), find the minimum scalar multiplications to multiply the whole chain. Here we solve it **bottom-up (tabulation)** to remove recursion entirely.

```
arr = [40, 20, 30, 10, 30]   // 4 matrices
Answer = 26000
```

---

## Intuition

Convert the recursion `f(i, j) = min_k ( f(i,k) + f(k+1,j) + arr[i-1]*arr[k]*arr[j] )` into a table `dp[i][j]`.

Key observation about **order of evaluation**: `dp[i][j]` depends on smaller intervals (`dp[i][k]` and `dp[k+1][j]`). So we must fill the table by **increasing interval length**, or equivalently iterate `i` from high to low and `j` from low to high.

- `dp[i][i] = 0` (single matrix).
- For each interval `[i, j]`, try every cut `k in [i, j-1]` and take the minimum.

The answer is `dp[1][n-1]`.

---

## Code (C++)

```cpp
#include <vector>
#include <climits>
#include <algorithm>
using namespace std;

int matrixChainMultiplicationBottomUp(const vector<int>& arr) {
    int n = arr.size();          // n-1 matrices
    // dp[i][j]: min cost to multiply matrices i..j
    vector<vector<int>> dp(n, vector<int>(n, 0)); // dp[i][i] = 0 already

    // i goes high -> low, j goes low -> high so smaller intervals are ready
    for (int i = n - 1; i >= 1; --i) {
        for (int j = i + 1; j < n; ++j) {
            int best = INT_MAX;
            for (int k = i; k < j; ++k) {       // partition point
                int cost = dp[i][k]             // left half [i..k]
                         + dp[k + 1][j]         // right half [k+1..j]
                         + arr[i - 1] * arr[k] * arr[j]; // merge cost
                best = min(best, cost);
            }
            dp[i][j] = best;
        }
    }
    return dp[1][n - 1];
}
```

> Equivalent formulation: loop on interval length `len` from 2..n-1, then `i` from 1..n-len, with `j = i + len - 1`. Both fill cells only after their sub-intervals are computed.

---

## Complexity

| | |
|---|---|
| Time | O(n³) |
| Space | O(n²) |

> ⚠️ The traversal order is the whole point: a naive `for i; for j` from low to high reads `dp[k+1][j]` before it is filled. Iterate `i` downward (or by interval length) so dependencies are satisfied.
