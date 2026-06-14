# Longest Substring with At Most K Distinct Characters  🔴

**Reference:** [LeetCode 340](https://leetcode.com/problems/longest-substring-with-at-most-k-distinct-characters/) · [GeeksforGeeks](https://www.geeksforgeeks.org/find-the-longest-substring-with-k-unique-characters-in-a-given-string/)

---

## Problem
Given a string `s` and integer `k`, return the length of the longest substring containing **at most `k` distinct characters**.

```
Input:  s = "eceba", k = 2   Output: 3   ("ece")
Input:  s = "aa", k = 1       Output: 2   ("aa")
```

---

## Intuition
**Variable-size sliding window with a frequency map.**
- Expand `right`, incrementing the count of `s[right]`.
- While the map holds more than `k` distinct keys, shrink from `left`, erasing a key when its count hits 0.
- Every step the window is valid, so update the best length.

---

## Code (C++)
```cpp
int lengthOfLongestSubstringKDistinct(string s, int k) {
    if (k == 0) return 0;
    unordered_map<char,int> cnt;
    int left = 0, best = 0;
    for (int right = 0; right < (int)s.size(); right++) {
        cnt[s[right]]++;                 // expand
        while ((int)cnt.size() > k) {    // too many distinct -> shrink
            if (--cnt[s[left]] == 0)
                cnt.erase(s[left]);
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
| Space | O(K) — map holds at most K+1 keys |

> ⚠️ Guard `k == 0` (no substring allowed) and always erase keys at count 0 so `cnt.size()` reflects true distinct count.
