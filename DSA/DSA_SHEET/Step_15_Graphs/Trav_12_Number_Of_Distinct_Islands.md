# Number of Distinct Islands  🟡

**Reference:** [GeeksforGeeks — Find the Number of Distinct Islands in a 2D Matrix](https://www.geeksforgeeks.org/find-the-number-of-distinct-islands-in-a-2d-matrix/)

---

## Problem
Given a binary grid (`1` = land, `0` = water), count the number of **distinct** islands. Two islands are the same if one can be translated (no rotation/reflection) to match the other.

```
Input: grid = [[1,1,0,0,0],
               [1,1,0,0,0],
               [0,0,0,1,1],
               [0,0,0,1,1]]
Output: 1     // both islands have the same shape
```

---

## Intuition
Each island's shape is captured by the set of cell positions **relative to a fixed anchor** — typically the cell where DFS started. By storing every visited cell as an offset `(r - baseRow, c - baseCol)`, two islands with identical shapes produce identical offset lists regardless of where they sit in the grid. Collect each island's offset signature into a `set`; the set's size is the number of distinct shapes.

---

## Code (C++)
```cpp
class Solution {
public:
    void dfs(int r, int c, int baseR, int baseC, vector<vector<int>>& grid,
             vector<pair<int,int>>& shape) {
        int n = grid.size(), m = grid[0].size();
        if (r < 0 || r >= n || c < 0 || c >= m || grid[r][c] == 0) return;
        grid[r][c] = 0;                                  // mark visited
        shape.push_back({r - baseR, c - baseC});         // store relative offset
        dfs(r - 1, c, baseR, baseC, grid, shape);
        dfs(r + 1, c, baseR, baseC, grid, shape);
        dfs(r, c - 1, baseR, baseC, grid, shape);
        dfs(r, c + 1, baseR, baseC, grid, shape);
    }

    int countDistinctIslands(vector<vector<int>>& grid) {
        int n = grid.size(), m = grid[0].size();
        set<vector<pair<int,int>>> shapes;               // unique island signatures

        for (int i = 0; i < n; i++)
            for (int j = 0; j < m; j++)
                if (grid[i][j] == 1) {
                    vector<pair<int,int>> shape;
                    dfs(i, j, i, j, grid, shape);         // anchor at (i, j)
                    shapes.insert(shape);                 // dedupes equal shapes
                }
        return shapes.size();
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(N*M * log K) |
| Space | O(N*M) |

> DFS always visits cells in the same deterministic order, so identical shapes yield byte-for-byte identical offset vectors — that's what makes the `set` dedupe them.
