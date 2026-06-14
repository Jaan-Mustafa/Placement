# Surrounded Regions  🟡

**Reference:** [LeetCode 130 — Surrounded Regions](https://leetcode.com/problems/surrounded-regions/)

---

## Problem
Given a board of `'X'` and `'O'`, flip every `'O'` that is completely surrounded by `'X'` to `'X'`. An `'O'` is **not** captured if it (or any `'O'` connected to it) touches the border.

```
Input: board = [['X','X','X'],['X','O','O'],['X','X','X']]
Output:        [['X','X','X'],['X','X','X'],['X','X','X']]
```

---

## Intuition
Rather than hunting for surrounded regions, flip the logic: any `'O'` reachable from the border can never be captured. So we DFS inward from every border `'O'`, marking the whole connected region as "safe" (using a temporary marker like `'#'`). After this pass, every remaining `'O'` must be enclosed, so we flip it to `'X'`; the safe `'#'` cells are restored to `'O'`.

---

## Code (C++)
```cpp
class Solution {
public:
    void dfs(int r, int c, vector<vector<char>>& board) {
        int n = board.size(), m = board[0].size();
        if (r < 0 || r >= n || c < 0 || c >= m || board[r][c] != 'O') return;
        board[r][c] = '#';                      // mark border-connected 'O' as safe
        dfs(r - 1, c, board); dfs(r + 1, c, board);
        dfs(r, c - 1, board); dfs(r, c + 1, board);
    }

    void solve(vector<vector<char>>& board) {
        int n = board.size(), m = board[0].size();

        // launch DFS from all border 'O's
        for (int i = 0; i < n; i++) {
            if (board[i][0] == 'O')     dfs(i, 0, board);
            if (board[i][m-1] == 'O')   dfs(i, m-1, board);
        }
        for (int j = 0; j < m; j++) {
            if (board[0][j] == 'O')     dfs(0, j, board);
            if (board[n-1][j] == 'O')   dfs(n-1, j, board);
        }

        // remaining 'O' are surrounded -> capture; '#' was safe -> restore
        for (int i = 0; i < n; i++)
            for (int j = 0; j < m; j++) {
                if (board[i][j] == 'O') board[i][j] = 'X';
                else if (board[i][j] == '#') board[i][j] = 'O';
            }
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(N*M) |
| Space | O(N*M) |

> Solve the complement: mark what's safe (border-connected) first, then everything left over is by definition surrounded.
