# Longest Common Prefix  🟢

**Reference:** [LeetCode 14 — Longest Common Prefix](https://leetcode.com/problems/longest-common-prefix/)

---

## Problem
Find the longest common prefix shared by an array of strings. Return "" if there is none.

```
Input:  ["flower", "flow", "flight"]    Output: "fl"
Input:  ["dog", "racecar", "car"]       Output: ""
```

---

## Intuition
Sort the strings. After sorting, only the **first** and **last** strings can differ the most; their common prefix is the common prefix of the entire set. Compare those two character by character.

---

## Code (C++)
```cpp
string longestCommonPrefix(vector<string>& strs) {
    if (strs.empty()) return "";
    sort(strs.begin(), strs.end());
    string &a = strs.front(), &b = strs.back();
    int i = 0;
    while (i < a.size() && i < b.size() && a[i] == b[i]) i++;
    return a.substr(0, i);
}
```

### Alternative (no sort): vertical scanning
Compare character `i` across all strings; stop at the first mismatch or shortest string end — O(N · M).

---

## Complexity
| | |
|---|---|
| Time | O(N log N · M) with sort (M = avg length) |
| Space | O(1) extra |
