# Minimum Insertions to Make String Palindrome  🟡

**Reference:** https://leetcode.com/problems/minimum-insertion-steps-to-make-a-string-palindrome/

---

## Problem

Given a string `s`, return the minimum number of character insertions needed (anywhere) to make `s` a palindrome.

```
s = "mbadm"
Insert to get "mdbabdm" or "mbdadbm"
Minimum insertions = 2
```

---

## Intuition

The characters that already form the longest palindromic subsequence (LPS) can stay in place. Every remaining character needs a matching insertion.

So:

```
answer = n - LPS(s)
```

where `LPS(s) = LCS(s, reverse(s))`. The same logic gives minimum *deletions* to make a palindrome — both equal `n - LPS`.

---

## Code (C++)

```cpp
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

int minInsertions(const string& s) {
    string r(s.rbegin(), s.rend()); // reverse
    int n = s.size();
    // dp[i][j]: LCS of s[0..i-1] and r[0..j-1] = longest palindromic subseq
    vector<vector<int>> dp(n + 1, vector<int>(n + 1, 0));

    for (int i = 1; i <= n; ++i)
        for (int j = 1; j <= n; ++j)
            if (s[i - 1] == r[j - 1])
                dp[i][j] = 1 + dp[i - 1][j - 1];
            else
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);

    int lps = dp[n][n];
    return n - lps; // characters not part of any palindrome must be inserted
}
```

> Space optimization: the LPS/LCS table can be reduced to O(n) with two rolling rows.

---

## Complexity

| | |
|---|---|
| Time | O(n²) |
| Space | O(n²), reducible to O(n) |

> ⚠️ The result is `n - LPS`, not `LPS` itself. Insertions and deletions to reach a palindrome are equal in count — both subtract the LPS from the length.
