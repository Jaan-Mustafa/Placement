# Longest Repeating Character Replacement  🟡

**Reference:** [LeetCode 424](https://leetcode.com/problems/longest-repeating-character-replacement/)

---

## Problem
Given a string `s` (uppercase letters) and integer `k`, you may replace **at most `k` characters** with any letter. Return the length of the longest substring of a single repeated letter you can obtain.

```
Input:  s = "ABAB", k = 2     Output: 4   (replace both A's or both B's)
Input:  s = "AABABBA", k = 1  Output: 4   ("AABA" -> "AAAA")
```

---

## Intuition
A window is valid if `(window length − count of the most frequent char) ≤ k`, i.e. the number of characters we'd need to change fits within `k`.
- Track the frequency of each letter and `maxFreq` (the highest count seen in the window).
- If `windowLen − maxFreq > k`, shrink from the left.
- The window only needs to grow to find the answer, so the result is the largest valid window.

---

## Code (C++)
```cpp
int characterReplacement(string s, int k) {
    vector<int> freq(26, 0);
    int left = 0, maxFreq = 0, best = 0;
    for (int right = 0; right < (int)s.size(); right++) {
        maxFreq = max(maxFreq, ++freq[s[right] - 'A']);  // expand
        // changes needed = windowLen - maxFreq
        while ((right - left + 1) - maxFreq > k) {       // too many changes -> shrink
            freq[s[left] - 'A']--;
            left++;
        }
        best = max(best, right - left + 1);
    }
    return best;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) — fixed 26-letter table |

> ⚠️ We never recompute `maxFreq` downward. That's fine: the answer can only improve when `maxFreq` grows, and a stale (slightly high) `maxFreq` never produces a window larger than a previously found valid one.
