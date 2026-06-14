# Split Array - Largest Sum  🔴

**Reference:** [LeetCode 410 — Split Array Largest Sum](https://leetcode.com/problems/split-array-largest-sum/)

---

## Problem
Split the array into `k` non-empty **contiguous** subarrays to minimize the **largest** subarray sum.

```
Input:  nums = [7, 2, 5, 10, 8], k = 2    Output: 18
        (split [7,2,5] = 14 and [10,8] = 18 → max 18)
```

---

## Intuition: binary search on the largest-sum limit
This is the **Book Allocation** problem reworded. The answer ranges from `max(nums)` to `sum(nums)`. A larger allowed sum needs fewer splits — monotonic. Binary search for the smallest "largest sum" achievable with ≤ `k` subarrays.

---

## Code (C++)
```cpp
int subarraysNeeded(vector<int>& nums, int limit) {
    int parts = 1, cur = 0;
    for (int x : nums) {
        if (cur + x > limit) { parts++; cur = 0; }
        cur += x;
    }
    return parts;
}

int splitArray(vector<int>& nums, int k) {
    int low = *max_element(nums.begin(), nums.end());
    int high = accumulate(nums.begin(), nums.end(), 0);
    int ans = high;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (subarraysNeeded(nums, mid) <= k) { ans = mid; high = mid - 1; }
        else low = mid + 1;
    }
    return ans;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N · log(sum − max)) |
| Space | O(1) |

> "Minimize the maximum" / "maximize the minimum" phrasing is the tell-tale sign of binary-search-on-answer.
