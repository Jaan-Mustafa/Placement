# Max Consecutive Ones III  🟡

**Reference:** [LeetCode 1004](https://leetcode.com/problems/max-consecutive-ones-iii/)

---

## Problem
Given a binary array `nums` and an integer `k`, return the length of the longest subarray containing only `1`s after flipping **at most `k` zeros**.

```
Input:  nums = [1,1,1,0,0,0,1,1,1,1,0], k = 2   Output: 6
Input:  nums = [0,0,1,1,1,0,0,1,1,1,0,1,0], k = 3 Output: 10
```

---

## Intuition
Reframe: find the **longest window containing at most `k` zeros**.
- Expand `right`, counting zeros inside the window.
- While zeros `> k`, shrink from `left` (decrementing the zero count when an actual zero leaves).
- Track the largest window length.

---

## Code (C++)
```cpp
int longestOnes(vector<int>& nums, int k) {
    int left = 0, zeros = 0, best = 0;
    for (int right = 0; right < (int)nums.size(); right++) {
        if (nums[right] == 0) zeros++;      // expand window
        while (zeros > k) {                 // too many zeros -> shrink
            if (nums[left] == 0) zeros--;
            left++;
        }
        best = max(best, right - left + 1);
    }
    return best;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> A common O(N) optimization never shrinks below the best size found (window only grows or slides), but the `while`-shrink version above is the clearest and still O(N).
