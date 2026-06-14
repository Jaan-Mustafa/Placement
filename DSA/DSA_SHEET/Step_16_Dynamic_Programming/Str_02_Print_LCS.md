# Print LCS  🟡

**Reference:** https://www.geeksforgeeks.org/printing-longest-common-subsequence/

---

## Problem

Given two strings `a` and `b`, build and return the actual longest common subsequence string (not just its length). If multiple LCS exist, returning any one valid LCS is acceptable.

```
a = "abcde", b = "ace"
Output: "ace"
```

---

## Intuition

First fill the standard LCS table `dp[i][j]`. Then **backtrack** from `dp[n][m]` to reconstruct the string:

- If `a[i-1] == b[j-1]`: this character is part of the LCS; prepend it and move diagonally to `dp[i-1][j-1]`.
- Else move toward the larger neighbor: if `dp[i-1][j] >= dp[i][j-1]` go up (`i--`), otherwise go left (`j--`).

Because we walk from the end to the start, characters are collected in reverse, so we reverse the result at the end.

---

## Code (C++)

```cpp
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

string printLCS(const string& a, const string& b) {
    int n = a.size(), m = b.size();
    vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));

    // Step 1: fill the LCS length table (bottom-up)
    for (int i = 1; i <= n; ++i)
        for (int j = 1; j <= m; ++j)
            if (a[i - 1] == b[j - 1])
                dp[i][j] = 1 + dp[i - 1][j - 1];
            else
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);

    // Step 2: backtrack to reconstruct the subsequence
    string lcs;
    int i = n, j = m;
    while (i > 0 && j > 0) {
        if (a[i - 1] == b[j - 1]) {
            lcs.push_back(a[i - 1]); // part of LCS
            --i; --j;                // move diagonally
        } else if (dp[i - 1][j] >= dp[i][j - 1]) {
            --i;                     // came from above
        } else {
            --j;                     // came from the left
        }
    }
    reverse(lcs.begin(), lcs.end()); // built back-to-front
    return lcs;
}
```

> Space optimization: the O(m) rolling-array trick gives the LCS *length*, but printing the sequence requires the full O(n·m) table to backtrack, so keep the 2-D table here.

---

## Complexity

| | |
|---|---|
| Time | O(n·m) |
| Space | O(n·m) |

> ⚠️ You must keep the full 2-D table for reconstruction — the rolling-array space trick discards the path information needed to backtrack.
