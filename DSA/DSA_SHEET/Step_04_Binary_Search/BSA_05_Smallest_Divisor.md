# Smallest Divisor Given a Threshold  🟡

**Reference:** [LeetCode 1283 — Find the Smallest Divisor Given a Threshold](https://leetcode.com/problems/find-the-smallest-divisor-given-a-threshold/)

---

## Problem
Find the smallest divisor `d` such that the sum of `ceil(a[i] / d)` over all elements is ≤ `threshold`.

```
Input:  nums = [1, 2, 5, 9], threshold = 6    Output: 5
Input:  nums = [44, 22, 33, 11, 1], threshold = 5    Output: 44
```

---

## Intuition: binary search on the divisor
A larger divisor makes each `ceil(a[i]/d)` smaller, so the total sum is **monotonically decreasing** in `d`. Search `d` in `[1, max(nums)]` for the smallest value whose sum ≤ threshold.

---

## Code (C++)
```cpp
long long sumByDivisor(vector<int>& nums, int d) {
    long long sum = 0;
    for (int x : nums)
        sum += (x + d - 1) / d;          // ceil(x / d)
    return sum;
}

int smallestDivisor(vector<int>& nums, int threshold) {
    int low = 1, high = *max_element(nums.begin(), nums.end());
    int ans = high;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (sumByDivisor(nums, mid) <= threshold) {
            ans = mid;          // feasible; try smaller divisor
            high = mid - 1;
        } else {
            low = mid + 1;
        }
    }
    return ans;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N · log(max num)) |
| Space | O(1) |

> Same template as Koko bananas — only the feasibility function differs.
