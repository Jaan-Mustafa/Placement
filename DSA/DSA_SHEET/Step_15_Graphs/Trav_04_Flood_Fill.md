# Flood Fill  🟡

**Reference:** [LeetCode 733 — Flood Fill](https://leetcode.com/problems/flood-fill/)

---

## Problem
Given an `image` grid, a starting pixel `(sr, sc)`, and a `color`, replace the color of the start pixel and all 4-directionally connected pixels of the same original color with the new color.

```
Input: image = [[1,1,1],[1,1,0],[1,0,1]], sr = 1, sc = 1, color = 2
Output: [[2,2,2],[2,2,0],[2,0,1]]
```

---

## Intuition
Classic grid DFS/BFS. Record the original color of the start pixel; we only want to repaint cells that share this color and are connected to the start. DFS outward in 4 directions, repainting each matching cell to the new color. The new color itself serves as the "visited" marker, so we add a guard: if the start color already equals the target color, do nothing to avoid infinite recursion.

---

## Code (C++)
```cpp
class Solution {
public:
    void dfs(int r, int c, vector<vector<int>>& img, int oldColor, int newColor) {
        int n = img.size(), m = img[0].size();
        if (r < 0 || r >= n || c < 0 || c >= m) return;  // out of bounds
        if (img[r][c] != oldColor) return;                // not part of region

        img[r][c] = newColor;                             // repaint (acts as visited)
        dfs(r - 1, c, img, oldColor, newColor);
        dfs(r + 1, c, img, oldColor, newColor);
        dfs(r, c - 1, img, oldColor, newColor);
        dfs(r, c + 1, img, oldColor, newColor);
    }

    vector<vector<int>> floodFill(vector<vector<int>>& image, int sr, int sc, int color) {
        int oldColor = image[sr][sc];
        if (oldColor == color) return image;              // avoid infinite loop
        dfs(sr, sc, image, oldColor, color);
        return image;
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(N*M) |
| Space | O(N*M) |

> The `oldColor == color` early return is essential: without it, repainting to the same color never changes the cell, so the "visited" guard never trips and DFS loops forever.
