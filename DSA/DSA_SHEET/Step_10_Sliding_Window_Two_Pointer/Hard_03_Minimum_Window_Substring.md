# Minimum Window Substring  🔴

**Reference:** [LeetCode 76](https://leetcode.com/problems/minimum-window-substring/)

---

## Problem
Given strings `s` and `t`, return the **smallest substring of `s`** that contains every character of `t` (including duplicates). Return `""` if none exists.

```
Input:  s = "ADOBECODEBANC", t = "ABC"   Output: "BANC"
Input:  s = "a", t = "a"                  Output: "a"
Input:  s = "a", t = "aa"                 Output: ""   (not enough a's)
```

---

## Intuition
**Expand-then-shrink sliding window.**
- Count needed characters of `t` in `need[]`. Track how many distinct requirements are still unmet via `required`.
- Expand `right`; when a char's count in the window reaches the needed amount, decrement `formed`/`required`.
- Once all requirements are met, **shrink from `left`** as much as possible while still valid, recording the smallest window.

---

## Code (C++)
```cpp
string minWindow(string s, string t) {
    if (s.empty() || t.empty()) return "";
    vector<int> need(128, 0);
    for (char c : t) need[c]++;

    int required = t.size();             // total chars still needed (with multiplicity)
    int left = 0, bestLen = INT_MAX, bestStart = 0;

    for (int right = 0; right < (int)s.size(); right++) {
        // if this char was still needed, consuming it reduces required
        if (need[s[right]]-- > 0) required--;
        // window now contains all of t -> try to shrink
        while (required == 0) {
            if (right - left + 1 < bestLen) {
                bestLen = right - left + 1;
                bestStart = left;
            }
            // release s[left]; if it becomes needed again, window breaks validity
            if (++need[s[left]] > 0) required++;
            left++;
        }
    }
    return bestLen == INT_MAX ? "" : s.substr(bestStart, bestLen);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(\|s\| + \|t\|) |
| Space | O(1) — fixed 128-size table |

> ⚠️ `need[]` can go negative — that records *surplus* characters in the window. A char only counts toward `required` again when `++need[..]` rises above 0, i.e. we removed one that was actually essential.
