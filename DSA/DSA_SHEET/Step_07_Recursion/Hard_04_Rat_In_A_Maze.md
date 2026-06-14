# Rat in a Maze  🔴

**Reference:** [GeeksforGeeks — Rat in a Maze](https://www.geeksforgeeks.org/rat-in-a-maze-backtracking-2/)

---

## Problem
An `N x N` grid has `1` = open cell, `0` = blocked. A rat starts at `(0,0)` and must reach `(N-1,N-1)`, moving **D**own, **L**eft, **R**ight, **U**p. Return all paths as direction strings in lexicographic order.

```
Input:  maze = [[1,0,0,0],
                [1,1,0,1],
                [1,1,0,0],
                [0,1,1,1]]
Output: ["DDRDRR","DRDDRR"]
```

---

## Intuition
DFS from the start. Try moves in alphabetical order `D, L, R, U` so results come out sorted. Mark a cell visited before recursing and unmark it after (backtrack). When the rat reaches the destination, record the accumulated path.

---

## Code (C++)
```cpp
void solve(int r, int c, vector<vector<int>>& m, int n,
           vector<vector<int>>& vis, string path, vector<string>& res) {
    if (r == n - 1 && c == n - 1) { res.push_back(path); return; }

    // directions in lexicographic order: D, L, R, U
    int dr[] = {1, 0, 0, -1}, dc[] = {0, -1, 1, 0};
    char dir[] = {'D', 'L', 'R', 'U'};

    vis[r][c] = 1;                          // mark current cell
    for (int k = 0; k < 4; k++) {
        int nr = r + dr[k], nc = c + dc[k];
        if (nr >= 0 && nr < n && nc >= 0 && nc < n &&
            !vis[nr][nc] && m[nr][nc] == 1) {     // in-bounds, unvisited, open
            solve(nr, nc, m, n, vis, path + dir[k], res);
        }
    }
    vis[r][c] = 0;                          // backtrack
}

vector<string> findPath(vector<vector<int>>& m, int n) {
    vector<string> res;
    vector<vector<int>> vis(n, vector<int>(n, 0));
    if (m[0][0] == 1) solve(0, 0, m, n, vis, "", res);
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(4^(N²)) worst case — four choices per cell |
| Space | O(N²) visited grid + recursion depth |

> ⚠️ Guard the start: if `maze[0][0] == 0` there is no path. Iterating directions in `D,L,R,U` order yields lexicographically sorted output for free.
