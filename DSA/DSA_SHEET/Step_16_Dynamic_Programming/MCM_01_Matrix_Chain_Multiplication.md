# Matrix Chain Multiplication  🔴

**Reference:** https://www.geeksforgeeks.org/matrix-chain-multiplication-dp-8/

---

## Problem

Given a chain of matrices, find the minimum number of scalar multiplications needed to multiply them all together. Matrix `i` has dimensions `arr[i-1] x arr[i]`, so a dimension array `arr` of size `n` describes `n-1` matrices.

```
arr = [10, 20, 30]   // 2 matrices: A(10x20), B(20x30)
Only one way: (A x B) = 10 * 20 * 30 = 6000 multiplications
Answer = 6000

arr = [40, 20, 30, 10, 30]  // 4 matrices
Answer = 26000
```

---

## Intuition

This is a classic **partition / interval DP**. For the matrices in the range `[i, j]`, we try every place `k` (with `i <= k < j`) to split the product into two halves `[i, k]` and `[k+1, j]`, solve each half, then pay the cost of multiplying the two resulting matrices together.

Define `f(i, j)` = minimum multiplications to multiply matrices `i..j`.

Recurrence (the **partition** over the cut point `k`):
- `f(i, j) = min over k in [i, j-1] of ( f(i, k) + f(k+1, j) + arr[i-1] * arr[k] * arr[j] )`
- The first half produces a matrix of size `arr[i-1] x arr[k]`, the second `arr[k] x arr[j]`, so combining costs `arr[i-1] * arr[k] * arr[j]`.

Base case: `f(i, i) = 0` (a single matrix needs no multiplication).

We compute the answer with **recursion + memoization** here; `MCM_02` shows the bottom-up version.

---

## Code (C++)

```cpp
#include <vector>
#include <climits>
#include <algorithm>
using namespace std;

// f(i, j): min cost to multiply matrices i..j (1-indexed over matrices)
int solve(int i, int j, const vector<int>& arr, vector<vector<int>>& dp) {
    if (i == j) return 0;                 // single matrix => no cost
    if (dp[i][j] != -1) return dp[i][j];  // memoized

    int best = INT_MAX;
    // try every partition point k: split into [i..k] and [k+1..j]
    for (int k = i; k < j; ++k) {
        int cost = solve(i, k, arr, dp)            // cost of left half
                 + solve(k + 1, j, arr, dp)        // cost of right half
                 + arr[i - 1] * arr[k] * arr[j];   // cost to merge the two
        best = min(best, cost);
    }
    return dp[i][j] = best;
}

int matrixChainMultiplication(const vector<int>& arr) {
    int n = arr.size();          // n-1 matrices, indices 1..n-1
    vector<vector<int>> dp(n, vector<int>(n, -1));
    return solve(1, n - 1, arr, dp);
}
```

> The memo table is `O(n^2)` states; each state does `O(n)` work for the partition loop. The bottom-up form (MCM_02) avoids recursion depth and stack overhead.

---

## Complexity

| | |
|---|---|
| Time | O(n³) |
| Space | O(n²) + O(n) recursion stack |

> ⚠️ Indices are over matrices, not the dimension array. Matrix `k` spans `arr[k-1] x arr[k]`, so the merge cost uses `arr[i-1] * arr[k] * arr[j]`. Start the top-level call at `(1, n-1)`, never `(0, n-1)`.
