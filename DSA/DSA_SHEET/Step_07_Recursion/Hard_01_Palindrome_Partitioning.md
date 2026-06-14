# Palindrome Partitioning  🔴

**Reference:** [LeetCode 131 — Palindrome Partitioning](https://leetcode.com/problems/palindrome-partitioning/)

---

## Problem
Partition string `s` such that **every substring** of the partition is a palindrome. Return all possible partitionings.

```
Input:  s = "aab"
Output: [["a","a","b"], ["aa","b"]]
```

---

## Intuition
Backtracking with a moving `start` index. At each position try every prefix `s[start..i]`; if that prefix is a palindrome, fix it as one piece and recurse on the rest starting at `i+1`. When `start == n` we've consumed the whole string, so the current list is a valid partition.

---

## Code (C++)
```cpp
bool isPalin(const string& s, int l, int r) {
    while (l < r) if (s[l++] != s[r--]) return false;   // two-pointer check
    return true;
}

void solve(int start, const string& s,
           vector<string>& cur, vector<vector<string>>& res) {
    if (start == (int)s.size()) {        // whole string partitioned
        res.push_back(cur);
        return;
    }
    for (int i = start; i < (int)s.size(); i++) {
        if (isPalin(s, start, i)) {      // only cut where prefix is palindrome
            cur.push_back(s.substr(start, i - start + 1));
            solve(i + 1, s, cur, res);   // recurse on remaining suffix
            cur.pop_back();              // backtrack
        }
    }
}

vector<vector<string>> partition(string s) {
    vector<vector<string>> res;
    vector<string> cur;
    solve(0, s, cur, res);
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(2^N · N) — 2^N ways to cut, O(N) per palindrome check/copy |
| Space | O(N) recursion depth (output excluded) |

> ⚠️ Only recurse on a prefix **after** confirming it is a palindrome — checking inside the loop prunes invalid branches early.
