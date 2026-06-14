# Count Palindromic Subsequences  🔴

**Reference:** [GeeksforGeeks — Count all palindromic subsequences in a given string](https://www.geeksforgeeks.org/count-palindromic-subsequence-given-string/) · [LeetCode 730 — Count Different Palindromic Subsequences](https://leetcode.com/problems/count-different-palindromic-subsequences/)

---

## Problem
Count the number of **palindromic subsequences** in a string (subsequences need not be contiguous; here we count by position, so different index sets count separately even if the text is identical).

```
Input:  "abcb"
Palindromic subsequences: "a","b","c","b","bcb","bb"  ->  6
```

---

## Intuition: interval DP
Let `dp[i][j]` = number of palindromic subsequences within `s[i..j]`. Build up by interval length.

For the range `[i, j]`:
- Subsequences entirely inside `[i+1, j]` → `dp[i+1][j]`
- Subsequences entirely inside `[i, j-1]` → `dp[i][j-1]`
- The overlap counted in both is `[i+1, j-1]` → subtract `dp[i+1][j-1]`

So far: `dp[i][j] = dp[i+1][j] + dp[i][j-1] - dp[i+1][j-1]`.

Now the endpoints:
- **If `s[i] == s[j]`**: every palindrome in `[i+1, j-1]` can be wrapped by `s[i]…s[j]` to form a new palindrome, plus the two-character palindrome `s[i]s[j]` itself. That adds `dp[i+1][j-1] + 1`. Combined with the formula above, the `-dp[i+1][j-1]` cancels:
  `dp[i][j] = dp[i+1][j] + dp[i][j-1] + 1`.
- **If `s[i] != s[j]`**: no new wrapping is possible, keep the subtraction:
  `dp[i][j] = dp[i+1][j] + dp[i][j-1] - dp[i+1][j-1]`.

Base case: `dp[i][i] = 1` (each single character is a palindrome).

---

## Code (C++)
```cpp
int countPalindromicSubseq(const string& s) {
    int n = s.size();
    if (n == 0) return 0;
    // dp[i][j] = count of palindromic subsequences in s[i..j]
    vector<vector<long long>> dp(n, vector<long long>(n, 0));

    for (int i = 0; i < n; i++) dp[i][i] = 1;          // single chars

    // fill by increasing interval length
    for (int len = 2; len <= n; len++) {
        for (int i = 0; i + len - 1 < n; i++) {
            int j = i + len - 1;
            if (s[i] == s[j]) {
                // wrapping cancels the inclusion-exclusion overlap term
                dp[i][j] = dp[i + 1][j] + dp[i][j - 1] + 1;
            } else {
                // inclusion-exclusion: subtract the doubly-counted inner range
                dp[i][j] = dp[i + 1][j] + dp[i][j - 1] - dp[i + 1][j - 1];
            }
        }
    }
    return (int)dp[0][n - 1];
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N²) — all `O(N²)` intervals, O(1) work each |
| Space | O(N²) — the DP table |

> ⚠️ This counts subsequences by **position** (so `"bb"` from indices {1,3} is one count). If the problem asks for *distinct* palindromic strings (LeetCode 730), the recurrence changes and you must take results modulo `1e9+7`. Use `long long` for the table since the count grows exponentially with `N`.
