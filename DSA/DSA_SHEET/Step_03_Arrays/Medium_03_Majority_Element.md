# Majority Element (> N/2)  🟡

**Reference:** [LeetCode 169 — Majority Element](https://leetcode.com/problems/majority-element/)

---

## Problem
Find the element that appears more than ⌊N/2⌋ times. It's guaranteed to exist.

```
Input:  [2, 2, 1, 1, 1, 2, 2]    Output: 2
```

---

## Intuition: Boyer-Moore Voting
A majority element occurs more than half the time, so if we pair off each occurrence of it against a different element, it still survives.
- Keep a `candidate` and a `count`.
- Same as candidate → `count++`; different → `count--`.
- When `count` hits 0, adopt the current element as the new candidate.

The true majority always ends as the candidate.

---

## Code (C++)
```cpp
int majorityElement(vector<int>& nums) {
    int candidate = nums[0], count = 0;
    for (int x : nums) {
        if (count == 0) candidate = x;     // pick new candidate
        count += (x == candidate) ? 1 : -1;
    }
    return candidate;                      // guaranteed majority exists
}
```

---

## Dry run
`[2,2,1,1,1,2,2]`: cand=2(c1),2(c2),1(c1),1(c0→),1(cand=1,c1),2(c0→),2(cand=2,c1) → **2**.

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> If existence isn't guaranteed, do a second pass to verify the candidate's count > N/2.
