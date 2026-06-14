# Largest Rectangle in Histogram  🔴

**Reference:** [LeetCode 84 — Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/)

---

## Problem
Given bar heights of width 1, find the area of the largest rectangle that fits within the histogram.

```
Input:  [2, 1, 5, 6, 2, 3]
Output: 10   (bars 5 and 6 -> width 2 x height 5)
```

---

## Intuition
For each bar, the widest rectangle of that height extends until a **shorter** bar appears on either side. Using a **monotonic increasing stack of indices**, when the incoming bar is shorter than the stack top, we pop and finalise that bar's rectangle: its height is the popped value, and its width spans from just after the new stack top to the current index. A sentinel `0` at the end flushes everything.

---

## Code (C++)
```cpp
int largestRectangleArea(vector<int>& heights) {
    heights.push_back(0);                   // sentinel to flush the stack
    stack<int> st;                          // indices, increasing heights
    int best = 0;
    for (int i = 0; i < (int)heights.size(); i++) {
        while (!st.empty() && heights[st.top()] > heights[i]) {
            int h = heights[st.top()];      // height of the rectangle
            st.pop();
            int leftBoundary = st.empty() ? -1 : st.top();
            int width = i - leftBoundary - 1;
            best = max(best, h * width);
        }
        st.push(i);
    }
    heights.pop_back();                     // restore input
    return best;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — each index pushed/popped once |
| Space | O(N) |

> When a bar is popped, the new stack top is its previous-smaller bar and `i` is its next-smaller bar, so width = `i - st.top() - 1`. The trailing `0` sentinel guarantees every bar is eventually popped.
