# Rotate Matrix by 90°  🟡

**Reference:** [LeetCode 48 — Rotate Image](https://leetcode.com/problems/rotate-image/)

---

## Problem
Rotate an `n × n` matrix 90° clockwise, in place.

```
Input:  [[1,2,3],[4,5,6],[7,8,9]]
Output: [[7,4,1],[8,5,2],[9,6,3]]
```

---

## Intuition
A 90° clockwise rotation = **transpose** (swap across the main diagonal) **then reverse each row**.
- Transpose turns rows into columns.
- Reversing each row flips them into clockwise position.

---

## Code (C++)
```cpp
void rotate(vector<vector<int>>& m) {
    int n = m.size();
    // 1. transpose: swap m[i][j] with m[j][i]
    for (int i = 0; i < n; i++)
        for (int j = i + 1; j < n; j++)
            swap(m[i][j], m[j][i]);
    // 2. reverse each row
    for (int i = 0; i < n; i++)
        reverse(m[i].begin(), m[i].end());
}
```

---

## Why it works
```
[1 2 3]   transpose   [1 4 7]   reverse rows   [7 4 1]
[4 5 6]   --------->   [2 5 8]   ----------->   [8 5 2]
[7 8 9]                [3 6 9]                  [9 6 3]
```

## Complexity
| | |
|---|---|
| Time | O(n²) |
| Space | O(1) in place |

> For **counter-clockwise**: transpose then reverse each **column** (or reverse rows first, then transpose).
