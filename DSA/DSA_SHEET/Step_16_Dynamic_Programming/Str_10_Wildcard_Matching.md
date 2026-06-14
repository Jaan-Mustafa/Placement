# Wildcard Matching  🔴

**Reference:** https://leetcode.com/problems/wildcard-matching/

---

## Problem

Given an input string `s` and a pattern `p` containing the wildcards `?` (matches any single character) and `*` (matches any sequence of characters, including empty), return whether `p` matches the **entire** string `s`.

```
s = "adceb", p = "*a*b"
'*' matches ""  then 'a' matches 'a', '*' matches "dce", 'b' matches 'b'
Result: true
```

---

## Intuition

Let `dp[i][j]` = whether `s[0..i-1]` matches `p[0..j-1]`.

Recurrence:
- If `p[j-1] == s[i-1]` or `p[j-1] == '?'`: `dp[i][j] = dp[i-1][j-1]` (single-char match).
- If `p[j-1] == '*'`: `dp[i][j] = dp[i-1][j]` (`*` absorbs `s[i-1]`) **or** `dp[i][j-1]` (`*` matches empty).
- Else: `false`.

Base cases:
- `dp[0][0] = true` (both empty).
- `dp[0][j] = true` only while every pattern char so far is `'*'` (a leading run of `*` can match empty).
- `dp[i][0] = false` for `i > 0` (non-empty string vs empty pattern).

---

## Code (C++)

```cpp
#include <string>
#include <vector>
using namespace std;

bool isMatch(const string& s, const string& p) {
    int n = s.size(), m = p.size();
    // dp[i][j]: does s[0..i-1] match p[0..j-1]
    vector<vector<bool>> dp(n + 1, vector<bool>(m + 1, false));
    dp[0][0] = true; // empty matches empty

    // empty string matches a pattern of only '*' characters
    for (int j = 1; j <= m; ++j)
        if (p[j - 1] == '*')
            dp[0][j] = dp[0][j - 1];

    for (int i = 1; i <= n; ++i) {
        for (int j = 1; j <= m; ++j) {
            if (p[j - 1] == '*')
                // '*' = match empty (dp[i][j-1]) OR absorb s[i-1] (dp[i-1][j])
                dp[i][j] = dp[i][j - 1] || dp[i - 1][j];
            else if (p[j - 1] == '?' || p[j - 1] == s[i - 1])
                dp[i][j] = dp[i - 1][j - 1]; // single-char match
            // else stays false
        }
    }
    return dp[n][m];
}
```

> Space optimization: rows depend only on the previous row and the current row to the left, so a 1-D array of size `m+1` works (carry the diagonal in a temp). Reduces space to O(m).

---

## Complexity

| | |
|---|---|
| Time | O(n·m) |
| Space | O(n·m), reducible to O(m) |

> ⚠️ The `*` here matches any sequence on its own — unlike regex `.*`, it is not tied to a preceding character. Do not forget the `dp[0][j]` initialization for leading `*`s, or patterns like `"*"` against any string will fail.
