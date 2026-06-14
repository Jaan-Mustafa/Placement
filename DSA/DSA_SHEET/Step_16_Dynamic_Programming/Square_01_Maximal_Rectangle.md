# Maximum Rectangle Area with All 1's  🔴

**Reference:** https://leetcode.com/problems/maximal-rectangle/

---

## Problem

Given a `rows x cols` binary matrix filled with `0`s and `1`s, find the largest rectangle containing only `1`s and return its area.

```
matrix =
1 0 1 0 0
1 0 1 1 1
1 1 1 1 1
1 0 0 1 0
Largest all-1 rectangle area = 6   // the 2x3 block of 1s in rows 2-3, cols 2-4
```

---

## Intuition

Reduce the 2-D problem to the well-known **largest rectangle in a histogram**, applied row by row.

Process the matrix one row at a time, maintaining a `heights` array:
- If `matrix[r][c] == 1`, then `heights[c] += 1` (the bar grows taller).
- If `matrix[r][c] == 0`, then `heights[c] = 0` (the bar is cut to zero).

After updating `heights` for row `r`, `heights` describes a histogram whose bars are the consecutive 1s ending at this row. The largest rectangle of 1s with its bottom on row `r` equals the largest rectangle in that histogram. Take the max over all rows.

The histogram step uses a **monotonic stack**: for each bar we find how far it can extend left and right while staying the limiting height, giving width × height in O(cols).

---

## Code (C++)

```cpp
#include <vector>
#include <stack>
#include <algorithm>
using namespace std;

// Largest rectangle area in a histogram, O(n) with a monotonic stack.
int largestRectangleArea(const vector<int>& h) {
    int n = h.size(), best = 0;
    stack<int> st;                         // indices of increasing heights
    for (int i = 0; i <= n; ++i) {
        int cur = (i == n) ? 0 : h[i];     // sentinel 0 flushes the stack
        while (!st.empty() && h[st.top()] >= cur) {
            int height = h[st.top()]; st.pop();
            int left = st.empty() ? -1 : st.top();
            int width = i - left - 1;      // bars strictly between left and i
            best = max(best, height * width);
        }
        st.push(i);
    }
    return best;
}

int maximalRectangle(vector<vector<char>>& matrix) {
    if (matrix.empty() || matrix[0].empty()) return 0;
    int rows = matrix.size(), cols = matrix[0].size();
    vector<int> heights(cols, 0);
    int best = 0;
    for (int r = 0; r < rows; ++r) {
        for (int c = 0; c < cols; ++c)
            heights[c] = (matrix[r][c] == '1') ? heights[c] + 1 : 0; // build histogram
        best = max(best, largestRectangleArea(heights));
    }
    return best;
}
```

> LeetCode passes `vector<vector<char>>` with `'1'`/`'0'`; for an `int` matrix compare against `1`. The `heights` array is reused across rows, so extra space is just O(cols).

---

## Complexity

| | |
|---|---|
| Time | O(rows · cols) |
| Space | O(cols) |

> ⚠️ Reset a bar to 0 (not "skip") on a `0` cell — the rectangle must be solid 1s. The histogram sentinel (`cur = 0` at `i == n`) is needed to pop and measure all remaining bars.
