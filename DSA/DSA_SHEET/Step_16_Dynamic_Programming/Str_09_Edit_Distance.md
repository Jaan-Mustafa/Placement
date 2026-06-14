# Edit Distance  🔴

**Reference:** https://leetcode.com/problems/edit-distance/

---

## Problem

Given two strings `word1` and `word2`, return the minimum number of operations to convert `word1` into `word2`. Allowed operations: **insert**, **delete**, or **replace** a single character.

```
word1 = "horse", word2 = "ros"
horse -> rorse (replace 'h'->'r')
rorse -> rose  (delete 'r')
rose  -> ros   (delete 'e')
Minimum operations = 3
```

---

## Intuition

Let `dp[i][j]` = min operations to convert `word1[0..i-1]` into `word2[0..j-1]`.

Recurrence:
- If `word1[i-1] == word2[j-1]`: `dp[i][j] = dp[i-1][j-1]` (no operation needed).
- Else take 1 + the cheapest of:
  - `dp[i-1][j-1]` → **replace**
  - `dp[i-1][j]`   → **delete** from `word1`
  - `dp[i][j-1]`   → **insert** into `word1`

Base cases: `dp[i][0] = i` (delete all `i` chars), `dp[0][j] = j` (insert all `j` chars).

---

## Code (C++)

```cpp
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

int minDistance(const string& w1, const string& w2) {
    int n = w1.size(), m = w2.size();
    // dp[i][j]: min ops to turn w1[0..i-1] into w2[0..j-1]
    vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));

    for (int i = 0; i <= n; ++i) dp[i][0] = i; // delete all
    for (int j = 0; j <= m; ++j) dp[0][j] = j; // insert all

    for (int i = 1; i <= n; ++i) {
        for (int j = 1; j <= m; ++j) {
            if (w1[i - 1] == w2[j - 1]) {
                dp[i][j] = dp[i - 1][j - 1];   // chars equal, no op
            } else {
                dp[i][j] = 1 + min({ dp[i - 1][j - 1],  // replace
                                     dp[i - 1][j],       // delete
                                     dp[i][j - 1] });     // insert
            }
        }
    }
    return dp[n][m];
}
```

> Space optimization: only the previous row is needed, so two 1-D arrays of size `m+1` reduce space to O(m).

---

## Complexity

| | |
|---|---|
| Time | O(n·m) |
| Space | O(n·m), reducible to O(m) |

> ⚠️ Do not skip the base-case initialization (`dp[i][0]=i`, `dp[0][j]=j`). Leaving them as 0 silently produces wrong answers. The `+1` applies only on a mismatch — matched characters add nothing.
