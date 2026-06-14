# Trapping Rainwater  🔴

**Reference:** [LeetCode 42 — Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/)

---

## Problem
Given heights of bars of width 1, compute how much water is trapped after rain.

```
Input:  [0,1,0,2,1,0,1,3,2,1,2,1]
Output: 6
```

---

## Intuition
Water above bar `i` is `min(maxLeft, maxRight) - height[i]`. The **two-pointer** method computes this in O(1) space: move the pointer on the smaller side inward, because that side bounds the water. Track running `leftMax`/`rightMax`; whichever side is lower determines trapped water at that position. (A monotonic decreasing stack also solves this by accumulating water layer-by-layer between bars.)

---

## Code (C++)
```cpp
int trap(vector<int>& height) {
    int l = 0, r = height.size() - 1;
    int leftMax = 0, rightMax = 0, water = 0;
    while (l < r) {
        if (height[l] <= height[r]) {       // left side is the bottleneck
            if (height[l] >= leftMax) leftMax = height[l];
            else water += leftMax - height[l];
            l++;
        } else {                            // right side is the bottleneck
            if (height[r] >= rightMax) rightMax = height[r];
            else water += rightMax - height[r];
            r--;
        }
    }
    return water;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — single pass |
| Space | O(1) |

> The two-pointer insight: if `height[l] <= height[r]`, the left bar's water depends only on `leftMax` (the right side is guaranteed tall enough), so it's safe to finalise the left position.
