# Longest Subarray with Sum K (Positives)  🟡

**Reference:** [GeeksforGeeks — Longest sub-array with sum K](https://www.geeksforgeeks.org/longest-sub-array-sum-k/)

---

## Problem
Given an array of **positive** integers and a target K, find the length of the longest contiguous subarray whose sum equals K.

```
Input:  [1, 2, 3, 1, 1, 1, 1], K = 3    Output: 3   ([1,1,1] at the end)
Input:  [2, 3, 5], K = 5                Output: 2   ([2,3])
```

---

## Intuition
**Sliding window** works because all numbers are positive: growing the window only increases the sum, shrinking only decreases it.
- Expand `right`, adding to `sum`.
- While `sum > K`, shrink from `left`.
- When `sum == K`, update the best length.

---

## Code (C++)
```cpp
int longestSubarrayWithSumK(vector<int>& a, long long k) {
    int left = 0, n = a.size(), best = 0;
    long long sum = 0;
    for (int right = 0; right < n; right++) {
        sum += a[right];                    // expand window
        while (sum > k && left <= right)    // shrink while too big
            sum -= a[left++];
        if (sum == k)
            best = max(best, right - left + 1);
    }
    return best;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — each index enters/leaves the window once |
| Space | O(1) |

> ⚠️ This sliding window is valid **only for positive numbers**. With negatives, use the prefix-sum + hashmap method (next problem).
