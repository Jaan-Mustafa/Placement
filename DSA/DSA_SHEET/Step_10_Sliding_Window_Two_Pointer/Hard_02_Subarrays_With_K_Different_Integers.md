# Subarrays with K Different Integers  🔴

**Reference:** [LeetCode 992](https://leetcode.com/problems/subarrays-with-k-different-integers/)

---

## Problem
Given an array `nums` and integer `k`, count the **subarrays with exactly `k` distinct integers** (a "good" subarray).

```
Input:  nums = [1,2,1,2,3], k = 2   Output: 7
Input:  nums = [1,2,1,3,4], k = 3   Output: 3
```

---

## Intuition: exactly(k) = atMost(k) − atMost(k−1)
Counting "exactly `k` distinct" directly is hard, but **"at most `k` distinct"** is a clean sliding window. Each `right` in the at-most window contributes `right - left + 1` subarrays.

`exactly(k) = atMost(k) − atMost(k − 1)`

---

## Code (C++)
```cpp
int atMostK(vector<int>& nums, int k) {
    if (k == 0) return 0;
    unordered_map<int,int> cnt;
    int left = 0, res = 0;
    for (int right = 0; right < (int)nums.size(); right++) {
        cnt[nums[right]]++;              // expand
        while ((int)cnt.size() > k) {    // shrink while too many distinct
            if (--cnt[nums[left]] == 0)
                cnt.erase(nums[left]);
            left++;
        }
        res += right - left + 1;         // subarrays ending at right
    }
    return res;
}

int subarraysWithKDistinct(vector<int>& nums, int k) {
    return atMostK(nums, k) - atMostK(nums, k - 1);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — two linear passes |
| Space | O(K) |

> ⚠️ `atMostK(.., k-1)` with `k = 1` calls `atMostK(.., 0)`, which must return 0 — hence the `k == 0` guard.
