# Spiral Traversal of a Matrix  🟡

**Reference:** [LeetCode 54 — Spiral Matrix](https://leetcode.com/problems/spiral-matrix/)

---

## Problem
Return all elements of an `m × n` matrix in spiral order (clockwise, starting top-left).

```
Input:  [[1,2,3],[4,5,6],[7,8,9]]
Output: [1,2,3,6,9,8,7,4,5]
```

---

## Intuition
Maintain four boundaries — `top`, `bottom`, `left`, `right`. Walk:
1. left→right along `top`, then `top++`
2. top→bottom along `right`, then `right--`
3. right→left along `bottom` (if rows remain), then `bottom--`
4. bottom→top along `left` (if cols remain), then `left++`

Repeat while `top ≤ bottom` and `left ≤ right`.

---

## Code (C++)
```cpp
vector<int> spiralOrder(vector<vector<int>>& m) {
    int top = 0, bottom = m.size() - 1;
    int left = 0, right = m[0].size() - 1;
    vector<int> res;
    while (top <= bottom && left <= right) {
        for (int j = left; j <= right; j++) res.push_back(m[top][j]);
        top++;
        for (int i = top; i <= bottom; i++) res.push_back(m[i][right]);
        right--;
        if (top <= bottom) {                          // a row remains
            for (int j = right; j >= left; j--) res.push_back(m[bottom][j]);
            bottom--;
        }
        if (left <= right) {                          // a column remains
            for (int i = bottom; i >= top; i--) res.push_back(m[i][left]);
            left++;
        }
    }
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(m × n) — each cell visited once |
| Space | O(1) extra (output excluded) |

> ⚠️ The two `if` guards prevent re-traversing a row/column when the matrix is non-square or collapses to a single line.
