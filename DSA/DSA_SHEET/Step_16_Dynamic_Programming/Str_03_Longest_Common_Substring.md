# Longest Common Substring  🟡

**Reference:** https://www.geeksforgeeks.org/longest-common-substring-dp-29/

---

## Problem

Given two strings `a` and `b`, return the length of the longest **contiguous** substring common to both. Unlike a subsequence, a substring must be a continuous run of characters.

```
a = "abcde", b = "abfce"
Common substrings: "ab" (len 2), "ce" appears as 'c','e' but not contiguous in a
Longest common substring = "ab", length = 2
```

---

## Intuition

Let `dp[i][j]` = length of the longest common substring that **ends** exactly at `a[i-1]` and `b[j-1]`.

Recurrence:
- If `a[i-1] == b[j-1]`: `dp[i][j] = 1 + dp[i-1][j-1]` (extend the run).
- Else: `dp[i][j] = 0` (substring must be contiguous, so a mismatch breaks the run).

The answer is the **maximum value anywhere** in the table, not `dp[n][m]`, because the best run can end at any position.

---

## Code (C++)

```cpp
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

int longestCommonSubstring(const string& a, const string& b) {
    int n = a.size(), m = b.size();
    // dp[i][j]: length of common substring ending at a[i-1], b[j-1]
    vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));
    int best = 0;

    for (int i = 1; i <= n; ++i) {
        for (int j = 1; j <= m; ++j) {
            if (a[i - 1] == b[j - 1]) {
                dp[i][j] = 1 + dp[i - 1][j - 1]; // extend run
                best = max(best, dp[i][j]);      // track global max
            } else {
                dp[i][j] = 0;                    // mismatch breaks run
            }
        }
    }
    return best;
}
```

> Space optimization: only the previous row is needed, so two 1-D arrays of size `m+1` reduce space to O(m).

---

## Complexity

| | |
|---|---|
| Time | O(n·m) |
| Space | O(n·m), reducible to O(m) |

> ⚠️ The answer is the maximum cell in the table, NOT `dp[n][m]`. On a mismatch reset to 0 — do not carry over a `max` of neighbors as you would for subsequences.
