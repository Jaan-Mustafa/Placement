# Count Substrings with At Most K Distinct Characters  🟡

**Reference:** [GeeksforGeeks — Count substrings with exactly K distinct](https://www.geeksforgeeks.org/count-number-of-substrings-with-exactly-k-distinct-characters/) · [LeetCode 992](https://leetcode.com/problems/subarrays-with-k-different-integers/)

---

## Problem
Count substrings containing **exactly** K distinct characters.

```
Input:  s = "aba", k = 2    Output: 3   ("ab", "ba", "aba")
Input:  s = "abc", k = 2    Output: 2   ("ab", "bc")
```

---

## Intuition: exactly K = atMost(K) − atMost(K−1)
Counting "exactly K" directly is awkward, but "at most K distinct" is a clean sliding window. Then:

`exactly(K) = atMost(K) − atMost(K−1)`

For **at most K**: expand the window, and when distinct count exceeds K, shrink from the left. Every valid window of size `len` ending at `right` contributes `right - left + 1` substrings.

---

## Code (C++)
```cpp
int atMostK(string& s, int k) {
    if (k < 0) return 0;
    unordered_map<char,int> cnt;
    int left = 0, res = 0;
    for (int right = 0; right < s.size(); right++) {
        cnt[s[right]]++;
        while (cnt.size() > k) {                 // too many distinct
            if (--cnt[s[left]] == 0) cnt.erase(s[left]);
            left++;
        }
        res += right - left + 1;                 // substrings ending at right
    }
    return res;
}

int countExactlyK(string s, int k) {
    return atMostK(s, k) - atMostK(s, k - 1);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(K) |

> The "atMost(K) − atMost(K−1)" trick converts many "exactly K" problems into two easy sliding-window passes.
