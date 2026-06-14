# Longest Palindromic Substring  🟡

**Reference:** [LeetCode 5 — Longest Palindromic Substring](https://leetcode.com/problems/longest-palindromic-substring/)

---

## Problem
Return the longest contiguous substring that is a palindrome.

```
Input:  "babad"    Output: "bab"  (or "aba")
Input:  "cbbd"     Output: "bb"
```

---

## Intuition: expand around center
Every palindrome has a center. There are `2N-1` possible centers (N single characters + N-1 gaps between characters, for odd/even-length palindromes). From each center, expand outward while both sides match, tracking the longest palindrome found.

---

## Code (C++)
```cpp
string longestPalindrome(string s) {
    if (s.empty()) return "";
    int start = 0, maxLen = 1;

    auto expand = [&](int l, int r) {
        while (l >= 0 && r < s.size() && s[l] == s[r]) {
            if (r - l + 1 > maxLen) { start = l; maxLen = r - l + 1; }
            l--; r++;
        }
    };

    for (int i = 0; i < s.size(); i++) {
        expand(i, i);       // odd-length palindrome (center = i)
        expand(i, i + 1);   // even-length palindrome (center = i, i+1)
    }
    return s.substr(start, maxLen);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N²) — N centers × O(N) expansion |
| Space | O(1) |

> O(N²) DP also works but uses O(N²) space. Manacher's algorithm achieves O(N) but is rarely required in interviews — expand-around-center is the expected answer.
