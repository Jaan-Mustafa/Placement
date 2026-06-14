# Kth Missing Positive Number  🟡

**Reference:** [LeetCode 1539 — Kth Missing Positive Number](https://leetcode.com/problems/kth-missing-positive-number/)

---

## Problem
Given a sorted array of distinct positive integers, find the `k`-th positive integer that is **missing**.

```
Input:  arr = [2, 3, 4, 7, 11], k = 5    Output: 9
Input:  arr = [1, 2, 3, 4], k = 2        Output: 6
```

---

## Intuition: binary search on the index
At index `i`, the count of missing numbers before `arr[i]` is `arr[i] - (i + 1)` (how far the actual value is from the "ideal" value `i+1`). This count is **monotonic**, so binary search for the first index where missing-count ≥ k.

After the search, the answer is `low + k` (equivalently `high + 1 + k`).

---

## Code (C++)
```cpp
int findKthPositive(vector<int>& arr, int k) {
    int low = 0, high = arr.size() - 1;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        int missing = arr[mid] - (mid + 1);     // missing count up to mid
        if (missing < k) low = mid + 1;
        else high = mid - 1;
    }
    // missing numbers before index `low` is < k; answer extends past arr[high]
    return low + k;
}
```

---

## Why `low + k`?
After the loop, `high` is the last index where missing-count < k. The answer = `arr[high] + (k - missingBefore)` which simplifies to `high + 1 + k = low + k`.

## Complexity
| | |
|---|---|
| Time | O(log N) |
| Space | O(1) |

> A simpler O(N) scan also works (walk and count gaps); the BS version is the optimal follow-up.
