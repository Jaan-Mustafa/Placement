# Sum of Subarray Ranges  🔴

**Reference:** [LeetCode 2104 — Sum of Subarray Ranges](https://leetcode.com/problems/sum-of-subarray-ranges/)

---

## Problem
The *range* of a subarray is `max - min`. Return the sum of ranges over all contiguous subarrays.

```
Input:  [1, 2, 3]
Output: 4   (sum of (max-min) across all subarrays)
```

---

## Intuition
`sum of ranges = sum of all subarray maxes − sum of all subarray mins`. Each piece is a contribution problem solved by **monotonic stacks** (same technique as Sum of Subarray Minimums). For each element compute how many subarrays it is the max of (and separately the min of) via previous/next greater (and smaller) spans, multiply by the value, and subtract.

---

## Code (C++)
```cpp
long long sumSubarrayRanges(vector<int>& nums) {
    int n = nums.size();
    // helper: sum over all subarrays of element-as-extreme contribution
    auto contribution = [&](bool wantMin) -> long long {
        vector<int> left(n), right(n);
        stack<int> st;
        // left span
        for (int i = 0; i < n; i++) {
            while (!st.empty() &&
                  (wantMin ? nums[st.top()] >= nums[i]
                           : nums[st.top()] <= nums[i])) st.pop();
            left[i] = st.empty() ? i + 1 : i - st.top();
            st.push(i);
        }
        while (!st.empty()) st.pop();
        // right span (use strict opposite to avoid double counting)
        for (int i = n - 1; i >= 0; i--) {
            while (!st.empty() &&
                  (wantMin ? nums[st.top()] > nums[i]
                           : nums[st.top()] < nums[i])) st.pop();
            right[i] = st.empty() ? n - i : st.top() - i;
            st.push(i);
        }
        long long total = 0;
        for (int i = 0; i < n; i++)
            total += (long long)nums[i] * left[i] * right[i];
        return total;
    };
    long long maxSum = contribution(false);  // element as max
    long long minSum = contribution(true);   // element as min
    return maxSum - minSum;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(N) |

> ⚠️ Mind strict vs non-strict comparisons on the two spans — they must be asymmetric so subarrays with duplicate extremes are counted exactly once.
