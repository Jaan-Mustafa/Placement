# Word Break  🔴

**Reference:** [LeetCode 139 — Word Break](https://leetcode.com/problems/word-break/)

---

## Problem
Given a string `s` and a dictionary `wordDict`, return `true` if `s` can be segmented into a space-separated sequence of one or more dictionary words. Words may be reused.

```
Input:  s = "leetcode", wordDict = ["leet","code"]
Output: true        ("leet" + "code")
```

---

## Intuition
Recursion on a `start` index: try every prefix `s[start..i]`; if it's in the dictionary, recurse on the rest. Pure recursion is exponential because the same `start` is recomputed many times — **memoize** `dp[start]` so each suffix is solved once, giving polynomial time.

---

## Code (C++)
```cpp
bool solve(int start, const string& s,
           unordered_set<string>& dict, vector<int>& dp) {
    if (start == (int)s.size()) return true;     // consumed whole string
    if (dp[start] != -1) return dp[start];        // memoized result

    for (int end = start; end < (int)s.size(); end++) {
        string w = s.substr(start, end - start + 1);
        if (dict.count(w) && solve(end + 1, s, dict, dp))
            return dp[start] = 1;                 // prefix valid + rest valid
    }
    return dp[start] = 0;
}

bool wordBreak(string s, vector<string>& wordDict) {
    unordered_set<string> dict(wordDict.begin(), wordDict.end());
    vector<int> dp(s.size(), -1);                 // -1 = unvisited
    return solve(0, s, dict, dp);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N² · L) — N start states × N splits × substring/hash cost L |
| Space | O(N) memo + recursion depth (dictionary excluded) |

> ⚠️ Without memoization this is exponential. `dp[start]` caches "can the suffix from `start` be segmented?" — the suffix answer never depends on how you reached `start`.
