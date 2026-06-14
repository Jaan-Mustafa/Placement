# Check if Array is Sorted  🟢

**Reference:** [LeetCode 1752 — Check if Array Is Sorted and Rotated](https://leetcode.com/problems/check-if-array-is-sorted-and-rotated/) · [GeeksforGeeks](https://www.geeksforgeeks.org/program-check-array-sorted-not-iterative-recursive/)

---

## Problem
Check whether an array is sorted in non-decreasing order.

```
Input:  [1, 2, 3, 4, 5]    Output: true
Input:  [3, 1, 2]          Output: false
```

---

## Intuition
Walk the array once; if any element is strictly less than its predecessor, it's not sorted.

---

## Code (C++)
```cpp
bool isSorted(vector<int>& arr) {
    for (int i = 1; i < arr.size(); i++)
        if (arr[i] < arr[i - 1]) return false;   // out of order
    return true;
}
// or: is_sorted(arr.begin(), arr.end());
```

---

## Variant: sorted **and rotated** (LeetCode 1752)
A sorted-then-rotated array has **at most one** "drop" (place where `arr[i] > arr[i+1]`), counting the wrap-around.
```cpp
bool check(vector<int>& nums) {
    int n = nums.size(), drops = 0;
    for (int i = 0; i < n; i++)
        if (nums[i] > nums[(i + 1) % n]) drops++;
    return drops <= 1;
}
```

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |
