# Sliding Window Maximum  🔴

**Reference:** [LeetCode 239 — Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/)

---

## Problem
Given an array and window size `k`, return the maximum of each contiguous window as it slides left to right.

```
Input:  nums = [1,3,-1,-3,5,3,6,7], k = 3
Output: [3, 3, 5, 5, 6, 7]
```

---

## Intuition
Use a **monotonic decreasing deque of indices**. The front always holds the index of the current window maximum. Before pushing a new index, pop smaller values from the back (they can never be a max while the new element is in the window). Pop the front when it slides out of the window (index ≤ `i - k`). Record the front once the first window is full.

---

## Code (C++)
```cpp
vector<int> maxSlidingWindow(vector<int>& nums, int k) {
    deque<int> dq;                          // indices, decreasing values
    vector<int> res;
    for (int i = 0; i < (int)nums.size(); i++) {
        // drop indices that fell out of the window
        if (!dq.empty() && dq.front() <= i - k) dq.pop_front();
        // maintain decreasing order
        while (!dq.empty() && nums[dq.back()] <= nums[i]) dq.pop_back();
        dq.push_back(i);
        if (i >= k - 1) res.push_back(nums[dq.front()]);   // window complete
    }
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — each index enters/leaves the deque once |
| Space | O(k) |

> A deque (not a plain stack) is needed because we evict from **both** ends: stale indices from the front, smaller values from the back. The front is always the window max.
