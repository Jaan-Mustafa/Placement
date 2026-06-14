# N-Queens  🔴

**Reference:** [LeetCode 51 — N-Queens](https://leetcode.com/problems/n-queens/)

---

## Problem
Place `N` queens on an `N x N` board so that no two attack each other (no shared row, column, or diagonal). Return all distinct board configurations.

```
Input:  n = 4
Output: [[".Q..","...Q","Q...","..Q."],
         ["..Q.","Q...","...Q",".Q.."]]
```

---

## Intuition
Place exactly one queen per **column**, left to right. For a candidate row in the current column, check three attack lines: same row, the "↗" diagonal (`row + col`), and the "↘" diagonal (`row - col + n - 1`). Using boolean marker arrays makes each safety check O(1) instead of O(N).

---

## Code (C++)
```cpp
void solve(int col, int n, vector<string>& board,
           vector<int>& rowUsed, vector<int>& d1, vector<int>& d2,
           vector<vector<string>>& res) {
    if (col == n) { res.push_back(board); return; }   // all columns filled
    for (int row = 0; row < n; row++) {
        // d1 = upper diagonal (row+col), d2 = lower diagonal (row-col+n-1)
        if (rowUsed[row] || d1[row + col] || d2[row - col + n - 1]) continue;
        board[row][col] = 'Q';
        rowUsed[row] = d1[row + col] = d2[row - col + n - 1] = 1;
        solve(col + 1, n, board, rowUsed, d1, d2, res);
        board[row][col] = '.';                         // backtrack
        rowUsed[row] = d1[row + col] = d2[row - col + n - 1] = 0;
    }
}

vector<vector<string>> solveNQueens(int n) {
    vector<vector<string>> res;
    vector<string> board(n, string(n, '.'));
    vector<int> rowUsed(n, 0), d1(2 * n - 1, 0), d2(2 * n - 1, 0);
    solve(0, n, board, rowUsed, d1, d2, res);
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N!) — pruned column-by-column placement |
| Space | O(N) markers + O(N²) per board snapshot |

> ⚠️ Diagonal indices must be offset to stay non-negative: `row - col` ranges from `-(n-1)` to `n-1`, so add `n - 1` before indexing `d2`.
