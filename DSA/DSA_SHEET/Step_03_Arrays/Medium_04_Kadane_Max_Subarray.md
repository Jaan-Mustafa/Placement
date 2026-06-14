# Maximum Subarray Sum (Kadane's Algorithm)  🟡

**Reference:** [LeetCode 53 — Maximum Subarray](https://leetcode.com/problems/maximum-subarray/)

---

## Problem
Find the contiguous subarray with the largest sum and return that sum.

```
Input:  [-2, 1, -3, 4, -1, 2, 1, -5, 4]    Output: 6   ([4,-1,2,1])
```

---

## Intuition: Kadane's
At each index, decide: extend the current subarray, or start fresh from here. The running sum `cur` should be reset to 0 whenever it goes negative — a negative prefix can only hurt any subarray that follows.

`cur = max(arr[i], cur + arr[i])`, and track the global best.

---

## Code (C++)
```cpp
int maxSubArray(vector<int>& nums) {
    long long cur = 0, best = LLONG_MIN;
    for (int x : nums) {
        cur += x;
        best = max(best, cur);
        if (cur < 0) cur = 0;          // drop negative prefix
    }
    return (int)best;
}
```

---

## Dry run
`[-2,1,-3,4,-1,2,1,-5,4]`: cur resets after negatives; best peaks at the window `[4,-1,2,1]` = **6**.

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> `best` starts at `LLONG_MIN` so all-negative arrays return the largest (least negative) element correctly. To also **print** the subarray, record start/end indices when `best` updates.
