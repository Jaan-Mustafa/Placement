# Rearrange Array Elements by Sign  🟡

**Reference:** [LeetCode 2149 — Rearrange Array Elements by Sign](https://leetcode.com/problems/rearrange-array-elements-by-sign/)

---

## Problem
Given an array with an **equal number** of positive and negative integers, rearrange so signs alternate, starting with a positive, while preserving the relative order within each sign group.

```
Input:  [3, 1, -2, -5, 2, -4]    Output: [3, -2, 1, -5, 2, -4]
```

---

## Intuition
Positives go to even indices `0, 2, 4, ...`; negatives go to odd indices `1, 3, 5, ...`. Use two write-pointers and place each element into the next slot of its sign group in one pass.

---

## Code (C++)
```cpp
vector<int> rearrangeBySign(vector<int>& nums) {
    int n = nums.size();
    vector<int> res(n);
    int pos = 0, neg = 1;                   // next even / odd slot
    for (int x : nums) {
        if (x > 0) { res[pos] = x; pos += 2; }
        else       { res[neg] = x; neg += 2; }
    }
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(N) for the result |

> **Variant (unequal counts):** when positives/negatives differ, place alternating pairs first, then append the leftovers — this needs separate positive/negative lists and a second loop.
