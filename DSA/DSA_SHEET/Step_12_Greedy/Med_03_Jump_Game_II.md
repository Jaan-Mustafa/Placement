# Jump Game II  🟡

**Reference:** [LeetCode 45 — Jump Game II](https://leetcode.com/problems/jump-game-ii/)

---

## Problem
Given `nums` where `nums[i]` is the max jump from index `i` (assume the last index is always reachable), return the **minimum number of jumps** to reach the last index.

```
Input:  [2,3,1,1,4]   Output: 2   (0->1->4)
```

---

## Intuition
A **BFS-by-levels** greedy. Think of indices reachable in `k` jumps as "level k". Maintain the current level's right boundary `curEnd`; while scanning, track the `farthest` index reachable from anything in the current level. When `i` reaches `curEnd`, we must spend a jump and the new boundary becomes `farthest`. The number of boundary crossings is the minimum jumps.

---

## Code (C++)
```cpp
int jump(vector<int>& nums) {
    int jumps = 0, curEnd = 0, farthest = 0;
    for (int i = 0; i < (int)nums.size() - 1; i++) {  // no jump needed at last index
        farthest = max(farthest, i + nums[i]);        // best reach within this level
        if (i == curEnd) {                            // exhausted current jump's range
            jumps++;                                  // must jump again
            curEnd = farthest;                        // extend boundary to new level
        }
    }
    return jumps;
}
```

---

## Dry run
`[2,3,1,1,4]`: i=0 farthest=2, i==curEnd(0) → jumps=1,curEnd=2; i=1 farthest=4; i=2 i==curEnd(2) → jumps=2,curEnd=4. Loop ends. Answer **2**.

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> ⚠️ Loop to `n-2`, not `n-1`. If `i` reaches the last index, incrementing `jumps` again would over-count one extra jump.
