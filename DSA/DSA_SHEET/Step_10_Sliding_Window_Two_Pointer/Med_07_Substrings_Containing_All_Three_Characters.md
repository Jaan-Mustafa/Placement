# Number of Substrings Containing All Three Characters  🟡

**Reference:** [LeetCode 1358](https://leetcode.com/problems/number-of-substrings-containing-all-three-characters/)

---

## Problem
Given a string `s` made of only `a`, `b`, `c`, count the substrings that contain **at least one of each** of the three characters.

```
Input:  s = "abcabc"   Output: 10
Input:  s = "aaacb"    Output: 3   ("aacb", "acb", "aaacb")
Input:  s = "abc"      Output: 1
```

---

## Intuition
For each `right`, find the **smallest valid left** such that `s[left..right]` still contains all three characters. Once we have such a `left`, *every* substring ending at `right` that starts at index `0..left` is valid — that's `left + 1` substrings (when `left` is the largest valid start).

Equivalently: track the **last seen index** of `a`, `b`, `c`. A substring ending at `right` is valid iff it starts at or before `min(lastA, lastB, lastC)`. So add `1 + min(lastA, lastB, lastC)` to the answer.

---

## Code (C++)
```cpp
int numberOfSubstrings(string s) {
    int last[3] = {-1, -1, -1};          // last index of a, b, c
    long long res = 0;
    for (int i = 0; i < (int)s.size(); i++) {
        last[s[i] - 'a'] = i;            // update last seen
        // substrings ending at i that contain all three:
        // they may start anywhere in [0 .. min(last)]
        res += 1 + min({last[0], last[1], last[2]});
    }
    return (int)res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — single pass |
| Space | O(1) — three tracked indices |

> ⚠️ When any of `a`/`b`/`c` hasn't appeared yet, its last index is `-1`, so `min(...) = -1` and `1 + (-1) = 0` is correctly added (no valid substring yet).
