# Shortest Common Supersequence  🔴

**Reference:** https://leetcode.com/problems/shortest-common-supersequence/

---

## Problem

Given two strings `a` and `b`, return the shortest string that has **both** `a` and `b` as subsequences. If multiple answers exist, return any.

```
a = "abac", b = "cab"
SCS = "cabac"  (length 5)
"abac" is a subsequence (a,b,a,c) and "cab" is a subsequence (c,a,b)
```

---

## Intuition

The shortest common supersequence keeps the LCS once and adds the leftover characters of each string around it. Its length is:

```
len(SCS) = len(a) + len(b) - LCS(a, b)
```

To build the string itself, fill the LCS table, then **backtrack** from `dp[n][m]`:
- If characters match, write the character once and move diagonally.
- Else move toward the larger neighbor, writing the character we step away from (it must appear in the supersequence).
- After reaching an edge, append the remaining prefix of whichever string is left.

Because we walk end-to-start, reverse the built string at the end.

---

## Code (C++)

```cpp
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

string shortestCommonSupersequence(const string& a, const string& b) {
    int n = a.size(), m = b.size();
    // dp[i][j]: LCS of a[0..i-1] and b[0..j-1]
    vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));
    for (int i = 1; i <= n; ++i)
        for (int j = 1; j <= m; ++j)
            if (a[i - 1] == b[j - 1])
                dp[i][j] = 1 + dp[i - 1][j - 1];
            else
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);

    // Backtrack to construct the supersequence
    string res;
    int i = n, j = m;
    while (i > 0 && j > 0) {
        if (a[i - 1] == b[j - 1]) {        // common char: take once
            res.push_back(a[i - 1]);
            --i; --j;
        } else if (dp[i - 1][j] >= dp[i][j - 1]) {
            res.push_back(a[i - 1]);       // unique to a
            --i;
        } else {
            res.push_back(b[j - 1]);       // unique to b
            --j;
        }
    }
    while (i > 0) res.push_back(a[--i]);   // leftover of a
    while (j > 0) res.push_back(b[--j]);   // leftover of b

    reverse(res.begin(), res.end());       // built back-to-front
    return res;
}
```

> Space optimization: the *length* alone needs only O(m), but reconstructing the string requires the full O(n·m) table to backtrack.

---

## Complexity

| | |
|---|---|
| Time | O(n·m) |
| Space | O(n·m) |

> ⚠️ When characters match, append the character **only once**. Forgetting to drain the leftover prefixes after the loop (the two trailing `while`s) produces an incomplete supersequence.
