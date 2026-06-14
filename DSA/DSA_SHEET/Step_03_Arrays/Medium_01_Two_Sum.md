# Two Sum  🟡

**Reference:** [LeetCode 1 — Two Sum](https://leetcode.com/problems/two-sum/)

---

## Problem
Given an array and a target, return the indices of two numbers that add up to the target. Exactly one solution exists.

```
Input:  nums = [2, 7, 11, 15], target = 9    Output: [0, 1]   (2 + 7)
```

---

## Intuition
For each element `x`, we need `target - x` (its complement). A **hash map** of `value → index` lets us check in O(1) whether the complement was already seen. One pass suffices.

---

## Code (C++)
```cpp
vector<int> twoSum(vector<int>& nums, int target) {
    unordered_map<int,int> seen;            // value -> index
    for (int i = 0; i < nums.size(); i++) {
        int need = target - nums[i];
        if (seen.count(need))               // complement already present
            return {seen[need], i};
        seen[nums[i]] = i;                  // record current
    }
    return {-1, -1};
}
```

---

## Variant: "does a pair exist?" (sorted, two pointers)
If you only need yes/no and may sort, use two pointers from both ends — O(N log N) time, O(1) space:
```cpp
sort(...); int l = 0, r = n-1;
while (l < r) { int s = a[l]+a[r]; if (s==target) return true; s<target ? l++ : r--; }
```

## Complexity
| | |
|---|---|
| Time | O(N) (hash map) |
| Space | O(N) |
