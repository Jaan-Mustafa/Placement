# Sum of Beauty of All Substrings  🟡

**Reference:** [LeetCode 1781 — Sum of Beauty of All Substrings](https://leetcode.com/problems/sum-of-beauty-of-all-substrings/)

---

## Problem
The **beauty** of a string = (max character frequency) − (min nonzero character frequency). Return the sum of beauty over all substrings.

```
Input:  "aabcb"    Output: 5
Input:  "aabcbaa"  Output: 17
```

---

## Intuition
There are O(N²) substrings. Fix a starting index `i`, then extend the end `j`, maintaining a running frequency array. After adding each `s[j]`, compute (max freq − min nonzero freq) over the 26 letters and add it. This reuses the frequency array as the window grows, so each substring's beauty is O(26).

---

## Code (C++)
```cpp
int beautySum(string s) {
    int total = 0, n = s.size();
    for (int i = 0; i < n; i++) {
        int freq[26] = {0};
        for (int j = i; j < n; j++) {
            freq[s[j] - 'a']++;                  // extend substring [i..j]
            int mx = 0, mn = INT_MAX;
            for (int k = 0; k < 26; k++) {
                if (freq[k] > 0) {
                    mx = max(mx, freq[k]);
                    mn = min(mn, freq[k]);
                }
            }
            total += mx - mn;
        }
    }
    return total;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N² · 26) |
| Space | O(26) = O(1) |

> Recomputing max/min over 26 letters per substring keeps it simple; the alphabet constant (26) makes it effectively O(N²).
