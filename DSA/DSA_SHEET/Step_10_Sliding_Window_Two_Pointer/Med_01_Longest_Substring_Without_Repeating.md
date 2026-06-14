# Longest Substring Without Repeating Characters  🟡

**Reference:** [LeetCode 3](https://leetcode.com/problems/longest-substring-without-repeating-characters/) · [GeeksforGeeks](https://www.geeksforgeeks.org/length-of-the-longest-substring-without-repeating-characters/)

---

## Problem
Given a string `s`, find the length of the longest substring with **no repeating characters**.

```
Input:  s = "abcabcbb"   Output: 3   ("abc")
Input:  s = "bbbbb"      Output: 1   ("b")
Input:  s = "pwwkew"     Output: 3   ("wke")
```

---

## Intuition
**Variable-size sliding window.** Keep a window `[left, right]` that always contains distinct characters.
- Store the **last seen index** of each character.
- When `s[right]` was seen *inside* the current window, jump `left` to just past that previous occurrence.
- The answer is the max window size over all `right`.

---

## Code (C++)
```cpp
int lengthOfLongestSubstring(string s) {
    vector<int> last(256, -1);          // last index where each char appeared
    int left = 0, best = 0;
    for (int right = 0; right < (int)s.size(); right++) {
        char c = s[right];
        // if seen and inside the window, move left past it
        if (last[c] >= left)
            left = last[c] + 1;
        last[c] = right;                // update last seen
        best = max(best, right - left + 1);
    }
    return best;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — each index visited once |
| Space | O(1) — fixed 256-size table (or O(min(N, charset))) |

> ⚠️ Use `last[c] >= left` (not just `last[c] != -1`); a repeat *outside* the current window must NOT shrink it.
