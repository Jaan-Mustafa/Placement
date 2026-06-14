# Print the Maximum Subarray  🟡

**Reference:** [LeetCode 53 — Maximum Subarray (follow-up)](https://leetcode.com/problems/maximum-subarray/) · [GeeksforGeeks](https://www.geeksforgeeks.org/largest-sum-contiguous-subarray/)

---

## Problem
Extend Kadane's algorithm to also return the **actual subarray** (its start and end indices) that produces the maximum sum.

```
Input:  [-2, 1, -3, 4, -1, 2, 1, -5, 4]
Output: sum = 6, subarray = [4, -1, 2, 1]  (indices 3..6)
```

---

## Intuition
Run Kadane's, but track a tentative `start` index. Whenever `cur` resets to 0, the next element becomes a potential new start. When `best` improves, lock in `[start, i]` as the answer range.

---

## Code (C++)
```cpp
pair<int,int> maxSubArrayIndices(vector<int>& nums) {
    long long cur = 0, best = LLONG_MIN;
    int start = 0, ansL = 0, ansR = 0;
    for (int i = 0; i < nums.size(); i++) {
        if (cur == 0) start = i;            // potential new beginning
        cur += nums[i];
        if (cur > best) {                   // new best window
            best = cur;
            ansL = start;
            ansR = i;
        }
        if (cur < 0) cur = 0;               // reset negative prefix
    }
    return {ansL, ansR};                    // print nums[ansL..ansR]
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> Set `start = i` **before** adding `nums[i]` whenever `cur == 0`, so the start aligns with where the winning window actually begins.
