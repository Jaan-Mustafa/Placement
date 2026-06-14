# Maximal Rectangle  🔴

**Reference:** [LeetCode 85 — Maximal Rectangle](https://leetcode.com/problems/maximal-rectangle/)

---

## Problem
Given a binary matrix of `'0'`/`'1'`, find the area of the largest rectangle containing only `1`s.

```
Input:
1 0 1 0 0
1 0 1 1 1
1 1 1 1 1
1 0 0 1 0
Output: 6
```

---

## Intuition
Process the matrix **row by row**, maintaining a histogram where `heights[j]` is the number of consecutive `1`s ending at the current row in column `j`. Reset to 0 on a `'0'`. For each row, the answer is the **largest rectangle in that histogram** — exactly the previous problem. Sweep all rows and take the max.

---

## Code (C++)
```cpp
int largestRectangleArea(vector<int>& heights) {
    heights.push_back(0);
    stack<int> st;
    int best = 0;
    for (int i = 0; i < (int)heights.size(); i++) {
        while (!st.empty() && heights[st.top()] > heights[i]) {
            int h = heights[st.top()]; st.pop();
            int left = st.empty() ? -1 : st.top();
            best = max(best, h * (i - left - 1));
        }
        st.push(i);
    }
    heights.pop_back();
    return best;
}

int maximalRectangle(vector<vector<char>>& matrix) {
    if (matrix.empty()) return 0;
    int cols = matrix[0].size();
    vector<int> heights(cols, 0);
    int best = 0;
    for (auto& row : matrix) {
        for (int j = 0; j < cols; j++)
            heights[j] = (row[j] == '1') ? heights[j] + 1 : 0;  // build histogram
        best = max(best, largestRectangleArea(heights));
    }
    return best;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(R · C) — each row's histogram solved in O(C) |
| Space | O(C) |

> The key reduction: each row turns the 2D problem into a 1D "largest rectangle in histogram". A `'0'` resets that column's height to 0 because the run of 1s is broken.
