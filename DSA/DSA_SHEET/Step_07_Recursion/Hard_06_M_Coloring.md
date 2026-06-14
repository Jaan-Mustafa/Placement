# M-Coloring Problem  🔴

**Reference:** [GeeksforGeeks — M Coloring Problem](https://www.geeksforgeeks.org/m-coloring-problem-backtracking-5/)

---

## Problem
Given an undirected graph (adjacency matrix) and `m` colors, decide whether every vertex can be colored using at most `m` colors so that **no two adjacent vertices share a color**.

```
Input:  V = 4, m = 3, edges = (0-1),(1-2),(2-3),(3-0),(0-2)
Output: true        (e.g. 0→1, 1→2, 2→3, 3→2 ... a valid 3-coloring exists)
```

---

## Intuition
Color vertices one at a time. For the current vertex try each of the `m` colors; a color is allowed only if no already-colored neighbour uses it. Recurse to the next vertex; if a dead end is hit, undo the color (backtrack) and try the next one. Success when all vertices are colored.

---

## Code (C++)
```cpp
bool isSafe(int v, vector<vector<int>>& g, vector<int>& color, int c, int V) {
    for (int u = 0; u < V; u++)
        if (g[v][u] && color[u] == c) return false;   // adjacent same color
    return true;
}

bool solve(int v, vector<vector<int>>& g, int m, vector<int>& color, int V) {
    if (v == V) return true;                            // all vertices colored
    for (int c = 1; c <= m; c++) {
        if (isSafe(v, g, color, c, V)) {
            color[v] = c;
            if (solve(v + 1, g, m, color, V)) return true;
            color[v] = 0;                               // backtrack
        }
    }
    return false;
}

bool graphColoring(vector<vector<int>>& graph, int m, int V) {
    vector<int> color(V, 0);                            // 0 = uncolored
    return solve(0, graph, m, color, V);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(m^V) — m color choices per vertex, pruned by `isSafe` |
| Space | O(V) color array + recursion depth |

> ⚠️ Check adjacency against **already assigned** colors only; uncolored neighbours (`color == 0`) never block a choice.
