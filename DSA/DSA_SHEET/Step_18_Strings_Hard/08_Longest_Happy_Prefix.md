# Longest Happy Prefix  🔴

**Reference:** [LeetCode 1392 — Longest Happy Prefix](https://leetcode.com/problems/longest-happy-prefix/)

---

## Problem
A **happy prefix** is a non-empty prefix that is also a suffix (excluding the whole string itself). Return the longest happy prefix of `s`; return `""` if none exists.

```
Input:  "level"     Output: "l"
Input:  "ababab"    Output: "abab"
Input:  "leetcodeleet"  Output: "leet"
Input:  "a"         Output: ""
```

---

## Intuition: it's just the last LPS value
"Longest proper prefix that is also a suffix" is *exactly* the definition of the KMP **LPS / failure function** evaluated at the last index.

Build the LPS array of `s`. Then `lps[n-1]` is the length of the longest happy prefix, and the answer is `s.substr(0, lps[n-1])`.

No separate matching pass is needed — computing the failure function alone solves the problem in linear time.

---

## Code (C++)
```cpp
string longestPrefix(string s) {
    int n = s.size();
    vector<int> lps(n, 0);             // lps[i] = longest proper prefix == suffix of s[0..i]
    int len = 0;
    for (int i = 1; i < n; ) {
        if (s[i] == s[len]) {
            lps[i++] = ++len;          // extend current prefix-suffix
        } else if (len > 0) {
            len = lps[len - 1];        // fall back without advancing i
        } else {
            lps[i++] = 0;              // no prefix-suffix at this position
        }
    }
    return s.substr(0, lps[n - 1]);    // longest happy prefix
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — single LPS construction |
| Space | O(N) — the LPS array |

> ⚠️ The happy prefix must be a *proper* prefix (shorter than `s`), which the LPS definition already guarantees — that's why we never set `lps[i] = i+1`. A rolling-hash solution also works but risks collisions; KMP is exact.
