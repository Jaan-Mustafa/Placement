# Longest Common Subsequence (LCS)  🟡

**Reference:** https://leetcode.com/problems/longest-common-subsequence/

---

## Problem

Given two strings `text1` and `text2`, return the length of their longest common subsequence. A subsequence keeps the relative order of characters but need not be contiguous. If there is no common subsequence, return 0.

```
text1 = "abcde", text2 = "ace"
LCS = "ace", length = 3
```

---

## Intuition

Let `dp[i][j]` = length of the LCS of the first `i` characters of `text1` and the first `j` characters of `text2`.

Recurrence:
- If `text1[i-1] == text2[j-1]`: `dp[i][j] = 1 + dp[i-1][j-1]` (match both, extend diagonal).
- Else: `dp[i][j] = max(dp[i-1][j], dp[i][j-1])` (drop one character from either string).

Base case: `dp[0][*] = dp[*][0] = 0` (empty string has no common subsequence).

---

## Code (C++)

```cpp
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

int longestCommonSubsequence(const string& a, const string& b) {
    int n = a.size(), m = b.size();
    // dp[i][j]: LCS of a[0..i-1] and b[0..j-1]
    vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));

    for (int i = 1; i <= n; ++i) {
        for (int j = 1; j <= m; ++j) {
            if (a[i - 1] == b[j - 1])
                dp[i][j] = 1 + dp[i - 1][j - 1]; // characters match
            else
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]); // skip one
        }
    }
    return dp[n][m];
}
```

> Space optimization: each row only depends on the previous row, so two 1-D arrays of size `m+1` (or a single array with a temp for the diagonal) reduce space to O(m).

---

## Complexity

| | |
|---|---|
| Time | O(n·m) |
| Space | O(n·m), reducible to O(m) |

> ⚠️ Remember the index offset: `dp` is sized `(n+1)×(m+1)` and string indices are `i-1`/`j-1`. Off-by-one here is the most common bug.
