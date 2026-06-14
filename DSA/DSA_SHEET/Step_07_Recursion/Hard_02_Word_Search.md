# Word Search  🔴

**Reference:** [LeetCode 79 — Word Search](https://leetcode.com/problems/word-search/)

---

## Problem
Given an `m x n` grid of characters and a `word`, return `true` if the word exists by connecting **adjacent** (up/down/left/right) cells. The same cell may **not** be reused within one word.

```
Input:  board = [["A","B","C","E"],
                 ["S","F","C","S"],
                 ["A","D","E","E"]], word = "ABCCED"
Output: true
```

---

## Intuition
Try each cell as a starting point. DFS matches `word[idx]` against the current cell, then explores its four neighbours for `word[idx+1]`. Mark the cell as visited (overwrite with a sentinel) before recursing and restore it afterward so other paths can reuse it.

---

## Code (C++)
```cpp
bool dfs(vector<vector<char>>& b, const string& w, int r, int c, int idx) {
    if (idx == (int)w.size()) return true;                 // matched all chars
    int m = b.size(), n = b[0].size();
    if (r < 0 || r >= m || c < 0 || c >= n) return false;  // out of bounds
    if (b[r][c] != w[idx]) return false;                   // char mismatch

    char tmp = b[r][c];
    b[r][c] = '#';                                         // mark visited
    bool found = dfs(b, w, r + 1, c, idx + 1) ||
                 dfs(b, w, r - 1, c, idx + 1) ||
                 dfs(b, w, r, c + 1, idx + 1) ||
                 dfs(b, w, r, c - 1, idx + 1);
    b[r][c] = tmp;                                         // restore (backtrack)
    return found;
}

bool exist(vector<vector<char>>& board, string word) {
    for (int r = 0; r < (int)board.size(); r++)
        for (int c = 0; c < (int)board[0].size(); c++)
            if (dfs(board, word, r, c, 0)) return true;
    return false;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(M · N · 4^L) — start cells × branching for word length L |
| Space | O(L) recursion depth |

> ⚠️ Restore the cell (`b[r][c] = tmp`) on the way out — forgetting this corrupts the board for later starting points and breaks legitimate paths.
