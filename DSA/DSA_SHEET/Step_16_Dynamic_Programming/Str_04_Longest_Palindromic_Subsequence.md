# Longest Palindromic Subsequence  🟡

**Reference:** https://leetcode.com/problems/longest-palindromic-subsequence/

---

## Problem

Given a string `s`, return the length of its longest palindromic subsequence (characters in order, not necessarily contiguous, that read the same forwards and backwards).

```
s = "bbbab"
Longest palindromic subsequence = "bbbb", length = 4
```

---

## Intuition

Key insight: the longest palindromic subsequence of `s` equals the LCS of `s` and its reverse `rev(s)`.

A subsequence common to `s` and `reverse(s)` reads the same in both directions, which is exactly a palindrome. So solve standard LCS on `(s, reverse(s))`.

Let `dp[i][j]` = LCS of `s[0..i-1]` and `r[0..j-1]` where `r = reverse(s)`:
- If `s[i-1] == r[j-1]`: `dp[i][j] = 1 + dp[i-1][j-1]`.
- Else: `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`.

---

## Code (C++)

```cpp
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

int longestPalindromeSubseq(const string& s) {
    string r(s.rbegin(), s.rend()); // reverse of s
    int n = s.size();
    // dp[i][j]: LCS of s[0..i-1] and r[0..j-1]
    vector<vector<int>> dp(n + 1, vector<int>(n + 1, 0));

    for (int i = 1; i <= n; ++i) {
        for (int j = 1; j <= n; ++j) {
            if (s[i - 1] == r[j - 1])
                dp[i][j] = 1 + dp[i - 1][j - 1]; // match
            else
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
        }
    }
    return dp[n][n];
}
```

> Space optimization: same as LCS — two rows of size `n+1` give O(n) space. (An alternative interval-DP formulation `dp[i][j]` over substrings also works but is not simpler.)

---

## Complexity

| | |
|---|---|
| Time | O(n²) |
| Space | O(n²), reducible to O(n) |

> ⚠️ Do not confuse longest palindromic *subsequence* (this problem, LCS-based) with longest palindromic *substring* (contiguous, expand-around-center). They are different problems.
