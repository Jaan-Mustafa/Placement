# Binary Subarrays With Sum  🟡

**Reference:** [LeetCode 930](https://leetcode.com/problems/binary-subarrays-with-sum/)

---

## Problem
Given a binary array `nums` and integer `goal`, count the number of **non-empty subarrays** whose sum equals `goal`.

```
Input:  nums = [1,0,1,0,1], goal = 2   Output: 4
Input:  nums = [0,0,0,0,0], goal = 0   Output: 15
```

---

## Intuition: exactly(goal) = atMost(goal) − atMost(goal−1)
A direct sliding window for "exactly goal" is awkward because of leading/trailing zeros. But **"at most `g`"** is a clean window: expand `right`, and while the window sum exceeds `g`, shrink from `left`. Each `right` then contributes `right - left + 1` subarrays.

`exactly(goal) = atMost(goal) − atMost(goal − 1)`

---

## Code (C++)
```cpp
int atMost(vector<int>& nums, int g) {
    if (g < 0) return 0;                 // no subarray has negative sum
    int left = 0, sum = 0, res = 0;
    for (int right = 0; right < (int)nums.size(); right++) {
        sum += nums[right];              // expand
        while (sum > g) {                // shrink while over the cap
            sum -= nums[left];
            left++;
        }
        res += right - left + 1;         // subarrays ending at right
    }
    return res;
}

int numSubarraysWithSum(vector<int>& nums, int goal) {
    return atMost(nums, goal) - atMost(nums, goal - 1);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — two linear passes |
| Space | O(1) |

> ⚠️ The `if (g < 0) return 0;` guard matters: when `goal = 0`, `atMost(goal-1) = atMost(-1)` must be 0.
