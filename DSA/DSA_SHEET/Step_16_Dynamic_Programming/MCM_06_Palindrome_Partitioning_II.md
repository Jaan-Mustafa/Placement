# Palindrome Partitioning II  🔴

**Reference:** https://leetcode.com/problems/palindrome-partitioning-ii/

---

## Problem

Given a string `s`, partition it so that every substring of the partition is a palindrome. Return the **minimum number of cuts** needed.

```
s = "aab"
"aa" | "b"  -> both palindromes, 1 cut
Answer = 1
```

---

## Intuition

This is **front partition DP**. Define `dp[i]` = minimum cuts needed for the suffix `s[i..n-1]`. We try every end index `j >= i`; if `s[i..j]` is a palindrome, we may cut after `j` and recurse on the rest.

Recurrence (partition over the first palindrome piece `s[i..j]`):
- If `s[i..j]` is a palindrome: candidate = `1 + dp[j+1]` (or `0` extra if `j == n-1`, i.e. the rest is empty).
- `dp[i] = min over valid j of that candidate`.

Base case: `dp[n] = 0` (empty suffix needs no cut).

To check palindromes fast, precompute `isPal[i][j]` with the standard interval DP: `isPal[i][j] = (s[i]==s[j]) && (j-i<2 || isPal[i+1][j-1])`. The final answer is `dp[0]`.

A common framing returns cuts directly; here `dp[i]` already counts cuts, so subtract 1 is NOT needed.

---

## Code (C++)

```cpp
#include <string>
#include <vector>
#include <algorithm>
#include <climits>
using namespace std;

int minCut(const string& s) {
    int n = s.size();

    // Precompute palindrome table: isPal[i][j] true if s[i..j] is palindrome
    vector<vector<bool>> isPal(n, vector<bool>(n, false));
    for (int i = n - 1; i >= 0; --i)
        for (int j = i; j < n; ++j)
            isPal[i][j] = (s[i] == s[j]) && (j - i < 2 || isPal[i + 1][j - 1]);

    // dp[i]: min cuts for suffix s[i..n-1]; dp[n] = 0
    vector<int> dp(n + 1, 0);
    for (int i = n - 1; i >= 0; --i) {
        int best = INT_MAX;
        for (int j = i; j < n; ++j) {
            if (isPal[i][j]) {                 // s[i..j] is a palindrome piece
                int cost = (j == n - 1) ? 0    // rest empty => no extra cut
                                        : 1 + dp[j + 1];
                best = min(best, cost);
            }
        }
        dp[i] = best;
    }
    return dp[0];
}
```

> The palindrome table makes each substring check O(1), turning the overall solution into O(n²). Without it the recurrence would re-scan and become O(n³).

---

## Complexity

| | |
|---|---|
| Time | O(n²) |
| Space | O(n²) for the palindrome table + O(n) for dp |

> ⚠️ Handle the "rest is empty" case: when `j == n-1` the suffix is fully consumed, so add 0 cuts, not 1. Forgetting this overcounts by one cut.
