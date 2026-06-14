# Minimum Insertions/Deletions to Convert A to B  🟡

**Reference:** https://www.geeksforgeeks.org/minimum-number-deletions-insertions-transform-one-string-another/

---

## Problem

Given strings `a` and `b`, find the minimum number of **deletions** (from `a`) and **insertions** (into `a`) required to transform `a` into `b`. Only insert and delete operations are allowed (no substitution).

```
a = "heap", b = "pea"
Delete 'h' and 'p'  -> "ea"   (2 deletions)
Insert 'p' at front -> "pea"  (1 insertion)
Deletions = 2, Insertions = 1, Total = 3
```

---

## Intuition

The longest common subsequence of `a` and `b` is the part that can be kept. Everything else in `a` must be deleted, and everything else in `b` must be inserted.

Let `L = LCS(a, b)`:
- Deletions = `len(a) - L` (extra characters in `a`).
- Insertions = `len(b) - L` (missing characters needed for `b`).
- Total operations = `len(a) + len(b) - 2·L`.

---

## Code (C++)

```cpp
#include <string>
#include <vector>
#include <algorithm>
#include <utility>
using namespace std;

// returns {deletions, insertions}; total = deletions + insertions
pair<int,int> minInsertDelete(const string& a, const string& b) {
    int n = a.size(), m = b.size();
    // dp[i][j]: LCS length of a[0..i-1] and b[0..j-1]
    vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));

    for (int i = 1; i <= n; ++i)
        for (int j = 1; j <= m; ++j)
            if (a[i - 1] == b[j - 1])
                dp[i][j] = 1 + dp[i - 1][j - 1];
            else
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);

    int lcs = dp[n][m];
    int deletions  = n - lcs; // remove extras from a
    int insertions = m - lcs; // add what b needs
    return {deletions, insertions};
}
```

> Space optimization: the LCS table reduces to O(m) with two rolling rows since only counts are needed.

---

## Complexity

| | |
|---|---|
| Time | O(n·m) |
| Space | O(n·m), reducible to O(m) |

> ⚠️ Deletions subtract from `len(a)` and insertions from `len(b)` — keep them straight. This problem forbids substitution; that variant is Edit Distance (Str_09).
