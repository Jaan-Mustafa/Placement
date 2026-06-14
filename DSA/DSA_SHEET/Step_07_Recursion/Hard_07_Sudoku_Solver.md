# Sudoku Solver  🔴

**Reference:** [LeetCode 37 — Sudoku Solver](https://leetcode.com/problems/sudoku-solver/)

---

## Problem
Fill a partially completed `9 x 9` Sudoku board (empty cells are `'.'`) so that every row, every column, and every `3 x 3` box contains the digits `1-9` exactly once. A unique solution is guaranteed.

```
Input:  board with '.' for blanks
Output: the same board fully filled with a valid solution
```

---

## Intuition
Scan for the first empty cell. Try digits `1-9`; a digit is valid if it isn't already in that cell's row, column, or `3 x 3` box. Place it, recurse — if the recursion eventually fills the whole board, propagate `true`. Otherwise undo the digit (backtrack) and try the next one.

---

## Code (C++)
```cpp
bool isValid(vector<vector<char>>& b, int r, int c, char ch) {
    for (int i = 0; i < 9; i++) {
        if (b[r][i] == ch) return false;                 // row
        if (b[i][c] == ch) return false;                 // column
        int br = 3 * (r / 3) + i / 3;                    // 3x3 box mapping
        int bc = 3 * (c / 3) + i % 3;
        if (b[br][bc] == ch) return false;               // box
    }
    return true;
}

bool solve(vector<vector<char>>& b) {
    for (int r = 0; r < 9; r++) {
        for (int c = 0; c < 9; c++) {
            if (b[r][c] == '.') {                         // first empty cell
                for (char ch = '1'; ch <= '9'; ch++) {
                    if (isValid(b, r, c, ch)) {
                        b[r][c] = ch;
                        if (solve(b)) return true;        // solved downstream
                        b[r][c] = '.';                    // backtrack
                    }
                }
                return false;                             // no digit fits here
            }
        }
    }
    return true;                                          // no empties left
}

void solveSudoku(vector<vector<char>>& board) { solve(board); }
```

---

## Complexity
| | |
|---|---|
| Time | O(9^(empty cells)) worst case — heavily pruned by `isValid` |
| Space | O(1) extra (in-place) + recursion depth |

> ⚠️ The box index `3*(r/3)+i/3`, `3*(c/3)+i%3` walks all 9 cells of the enclosing block — getting this mapping wrong is the most common bug.
