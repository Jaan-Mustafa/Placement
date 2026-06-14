# Maximum Consecutive Ones  🟢

**Reference:** [LeetCode 485 — Max Consecutive Ones](https://leetcode.com/problems/max-consecutive-ones/)

---

## Problem
Given a binary array, return the maximum number of consecutive 1s.

```
Input:  [1, 1, 0, 1, 1, 1]    Output: 3
```

---

## Intuition
Keep a running count of the current streak of 1s. Reset it to 0 on every 0. Track the best streak seen.

---

## Code (C++)
```cpp
int findMaxConsecutiveOnes(vector<int>& nums) {
    int count = 0, best = 0;
    for (int x : nums) {
        if (x == 1) count++;        // extend streak
        else        count = 0;      // streak broken
        best = max(best, count);
    }
    return best;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |
