# Jump Game  🟡

**Reference:** [LeetCode 55 — Jump Game](https://leetcode.com/problems/jump-game/)

---

## Problem
Given an array `nums` where `nums[i]` is the maximum jump length from index `i`, determine if you can reach the last index starting from index 0.

```
Input:  [2,3,1,1,4]   Output: true
Input:  [3,2,1,0,4]   Output: false  (stuck at index 3)
```

---

## Intuition
Track the **farthest reachable index** so far. Sweep left to right; at index `i`, if `i` is beyond the farthest reach, we're stuck → return false. Otherwise update `reach = max(reach, i + nums[i])`. Greedily maintaining the maximum reach is enough — we don't need to know how we got there, only whether index `i` is reachable.

---

## Code (C++)
```cpp
bool canJump(vector<int>& nums) {
    int reach = 0;                     // farthest index reachable so far
    for (int i = 0; i < (int)nums.size(); i++) {
        if (i > reach) return false;   // can't even arrive at i
        reach = max(reach, i + nums[i]);
        if (reach >= (int)nums.size() - 1) return true; // last index reachable
    }
    return true;
}
```

---

## Dry run
`[3,2,1,0,4]`: i=0 reach=3; i=1 reach=3; i=2 reach=3; i=3 reach=max(3,3)=3; i=4 but 4>3 → **false**.

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> ⚠️ The check `i > reach` must come before updating `reach`. A `0` at the current spot doesn't matter as long as an earlier index reaches past it.
