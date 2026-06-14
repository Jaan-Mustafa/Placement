# Minimum Path Sum in Triangle  🟡

**Reference:** https://leetcode.com/problems/triangle/

---

## Problem

Given a `triangle` (row `i` has `i+1` elements), find the minimum path sum from the top to the bottom. From index `j` on a row you may move to index `j` or index `j+1` on the next row.

```
triangle =
   [[2],
   [3, 4],
  [6, 5, 7],
 [4, 1, 8, 3]]

Best path: 2 -> 3 -> 5 -> 1 = 11
Answer = 11
```

---

## Intuition

State: `dp[i][j]` = minimum path sum from cell `(i,j)` down to the **last row**.

Computing bottom-up makes the moves natural — from `(i,j)` you may step to `(i+1,j)` or `(i+1,j+1)`:

```
dp[i][j] = triangle[i][j] + min( dp[i+1][j], dp[i+1][j+1] )
```

Base case: the last row is its own cost, `dp[n-1][j] = triangle[n-1][j]`. The answer is `dp[0][0]` (the apex).

---

## Code (C++)

```cpp
#include <bits/stdc++.h>
using namespace std;

int minimumTotal(vector<vector<int>>& triangle) {
    int n = triangle.size();
    // Start from the bottom row; it is its own minimal cost.
    vector<int> dp(triangle[n-1]);    // copy of last row

    // Move upward; row i has i+1 elements.
    for (int i = n - 2; i >= 0; i--) {
        for (int j = 0; j <= i; j++) {
            // From (i,j) we can go to (i+1,j) or (i+1,j+1).
            dp[j] = triangle[i][j] + min(dp[j], dp[j+1]);
        }
        // Entries beyond index i are now stale but never read again.
    }
    return dp[0];                     // apex holds the global minimum
}

int main() {
    vector<vector<int>> t = {{2},{3,4},{6,5,7},{4,1,8,3}};
    cout << minimumTotal(t) << "\n"; // 11
}
```

Space optimization: already applied — instead of a full `n x n` table we reuse a single 1D `dp` array of size `n`, updating in place from bottom to top, giving O(n) space.

---

## Complexity

| | |
|---|---|
| Time | O(n²) (sum of row lengths) |
| Space | O(n) (single rolling row) |

> ⚠️ This is a **fixed start, free end** problem — start is always the apex but you may finish at any bottom cell. Going bottom-up keeps the index transition `j -> j or j+1` clean and avoids needing the awkward `j-1`/`j` mapping required by a top-down formulation.
