# Longest Consecutive Sequence  🟡

**Reference:** [LeetCode 128 — Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence/)

---

## Problem
Given an unsorted array, find the length of the longest run of **consecutive** integers (order in the array doesn't matter). Required: O(N).

```
Input:  [100, 4, 200, 1, 3, 2]    Output: 4   (1,2,3,4)
```

---

## Intuition
Put all numbers in a hash set for O(1) lookups. A number starts a sequence only if `x-1` is **not** in the set. From each such start, count upward (`x+1, x+2, ...`) while present. Each number is visited at most twice, so it's O(N) overall — not O(N²).

---

## Code (C++)
```cpp
int longestConsecutive(vector<int>& nums) {
    unordered_set<int> s(nums.begin(), nums.end());
    int best = 0;
    for (int x : s) {
        if (!s.count(x - 1)) {              // x is a sequence start
            int cur = x, len = 1;
            while (s.count(cur + 1)) { cur++; len++; }
            best = max(best, len);
        }
    }
    return best;
}
```

---

## Why it's O(N), not O(N²)
The inner `while` only runs from sequence **starts**, and across all starts it visits each element exactly once. The `if (!s.count(x-1))` guard is what prevents re-walking the same sequence.

## Complexity
| | |
|---|---|
| Time | O(N) average |
| Space | O(N) |

> Sorting then scanning is O(N log N) — acceptable but slower than the hash-set approach.
