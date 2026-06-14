# Minimum Window Subsequence  🔴

**Reference:** [LeetCode 727](https://leetcode.com/problems/minimum-window-subsequence/) · [GeeksforGeeks](https://www.geeksforgeeks.org/minimum-window-subsequence/)

---

## Problem
Given strings `s1` and `s2`, return the **minimum-length contiguous substring** `w` of `s1` such that `s2` is a **subsequence** of `w` (characters in order, not necessarily contiguous). If multiple, return the leftmost; if none, return `""`.

```
Input:  s1 = "abcdebdde", s2 = "bde"   Output: "bcde"
Input:  s1 = "jmeqksfrsdcmsiwvaovztaqenprpvnbstl", s2 = "u"   Output: ""
```

> Unlike Minimum Window *Substring* (LC 76, any order), here `s2` must appear **in order** as a subsequence — so a frequency window does not work.

---

## Intuition: forward match, then backward shrink (two pointers)
Sweep `s1` with two pointers:
1. **Forward:** advance through `s1`, matching characters of `s2` in order until all of `s2` is matched. The position where the last char matched is the window's right end.
2. **Backward:** from that right end, walk left re-matching `s2` in reverse to find the **tightest** start for this end.
3. Record the window if it's the shortest so far, then restart the forward scan from just after that start.

---

## Code (C++)
```cpp
string minWindow(string s1, string s2) {
    int n = s1.size(), m = s2.size();
    int bestLen = INT_MAX, bestStart = -1;
    int i = 0, j = 0;
    while (i < n) {
        // forward: match s2 as a subsequence
        if (s1[i] == s2[j]) {
            if (++j == m) {                  // matched all of s2
                int end = i;                 // inclusive right end
                // backward: shrink to tightest start
                int k = m - 1;
                while (k >= 0) {
                    if (s1[i] == s2[k]) k--;
                    i--;
                }
                i++;                         // i is now the tightest start
                if (end - i + 1 < bestLen) {
                    bestLen = end - i + 1;
                    bestStart = i;
                }
                i++;                         // continue search after this start
                j = 0;                       // reset to find next match
                continue;
            }
        }
        i++;
    }
    return bestStart == -1 ? "" : s1.substr(bestStart, bestLen);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(\|s1\| · \|s2\|) worst case (each forward match may trigger a backward walk) |
| Space | O(1) |

> ⚠️ This is **subsequence**, not substring: order matters but gaps are allowed, which is exactly why we can't use a character-count window like LC 76. A DP solution `dp[i][j]` is also O(n·m) and avoids the backward walk if preferred.
