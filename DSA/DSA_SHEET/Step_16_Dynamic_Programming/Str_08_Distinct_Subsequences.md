# Distinct Subsequences  🔴

**Reference:** https://leetcode.com/problems/distinct-subsequences/

---

## Problem

Given strings `s` and `t`, count the number of distinct subsequences of `s` that equal `t`. The answer fits in a 32-bit signed integer.

```
s = "rabbbit", t = "rabbit"
Distinct ways to pick "rabbit" from "rabbbit" = 3
(choose which of the three 'b's to drop)
```

---

## Intuition

Let `dp[i][j]` = number of distinct subsequences of `s[0..i-1]` that equal `t[0..j-1]`.

Recurrence:
- If `s[i-1] == t[j-1]`: we may either **use** `s[i-1]` to match `t[j-1]` (`dp[i-1][j-1]`) **or skip** it (`dp[i-1][j]`). Sum them: `dp[i][j] = dp[i-1][j-1] + dp[i-1][j]`.
- Else: we must skip `s[i-1]`: `dp[i][j] = dp[i-1][j]`.

Base cases:
- `dp[i][0] = 1` for all `i` (empty `t` is matched exactly one way — pick nothing).
- `dp[0][j] = 0` for `j > 0` (non-empty `t` cannot come from empty `s`).

---

## Code (C++)

```cpp
#include <string>
#include <vector>
using namespace std;

int numDistinct(const string& s, const string& t) {
    int n = s.size(), m = t.size();
    // use unsigned long long to avoid overflow during accumulation
    vector<vector<unsigned long long>> dp(n + 1, vector<unsigned long long>(m + 1, 0));

    for (int i = 0; i <= n; ++i) dp[i][0] = 1; // empty t: one way

    for (int i = 1; i <= n; ++i) {
        for (int j = 1; j <= m; ++j) {
            if (s[i - 1] == t[j - 1])
                dp[i][j] = dp[i - 1][j - 1] + dp[i - 1][j]; // use or skip
            else
                dp[i][j] = dp[i - 1][j];                    // must skip
        }
    }
    return (int)dp[n][m]; // guaranteed to fit in 32-bit signed int
}
```

> Space optimization: each row depends only on the previous row, so a single 1-D array of size `m+1` works if you iterate `j` from high to low (right to left).

---

## Complexity

| | |
|---|---|
| Time | O(n·m) |
| Space | O(n·m), reducible to O(m) |

> ⚠️ Intermediate counts can overflow 32-bit ints even when the final answer fits. Accumulate in a wider type (`long long`/`unsigned long long`). When using the 1-D optimization, iterate `j` from right to left so you read the old `dp[j-1]`.
