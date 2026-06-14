# Count Number of Nice Subarrays  🟡

**Reference:** [LeetCode 1248](https://leetcode.com/problems/count-number-of-nice-subarrays/)

---

## Problem
A subarray is **nice** if it contains exactly `k` odd numbers. Given `nums` and `k`, count the nice subarrays.

```
Input:  nums = [1,1,2,1,1], k = 3   Output: 2   ([1,1,2,1], [1,2,1,1])
Input:  nums = [2,4,6], k = 1       Output: 0
Input:  nums = [2,2,2,1,2,2,1,2,2,2], k = 2  Output: 16
```

---

## Intuition: map odds → 0/1, reuse "exactly k"
Replace each number by its parity: odd → `1`, even → `0`. Then "exactly `k` odd numbers" becomes **"subarrays with sum exactly `k`"** on a binary array — the same problem as Binary Subarrays With Sum.

`exactly(k) = atMost(k) − atMost(k−1)`, where `atMost(g)` counts subarrays with at most `g` odd numbers.

---

## Code (C++)
```cpp
int atMost(vector<int>& nums, int g) {
    if (g < 0) return 0;
    int left = 0, odd = 0, res = 0;
    for (int right = 0; right < (int)nums.size(); right++) {
        odd += nums[right] & 1;          // expand: count odds
        while (odd > g) {                // shrink while too many odds
            odd -= nums[left] & 1;
            left++;
        }
        res += right - left + 1;
    }
    return res;
}

int numberOfSubarrays(vector<int>& nums, int k) {
    return atMost(nums, k) - atMost(nums, k - 1);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> The parity trick (`nums[i] & 1`) turns this into the identical template as LeetCode 930.
