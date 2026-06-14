# First and Last Occurrence  🟡

**Reference:** [LeetCode 34 — Find First and Last Position](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

---

## Problem
In a sorted array (with duplicates), find the first and last index of `target`. Return `[-1, -1]` if absent.

```
Input:  [5, 7, 7, 8, 8, 10], target = 8    Output: [3, 4]
Input:  [5, 7, 7, 8, 8, 10], target = 6    Output: [-1, -1]
```

---

## Intuition
Use bound functions:
- **First occurrence** = `lower_bound(target)` — first index with `a[i] >= target`. Valid only if that element equals target.
- **Last occurrence** = `upper_bound(target) - 1` — one before the first index with `a[i] > target`.

---

## Code (C++)
```cpp
int lowerBound(vector<int>& a, int x) {
    int lo = 0, hi = a.size() - 1, ans = a.size();
    while (lo <= hi) { int m = lo+(hi-lo)/2;
        if (a[m] >= x) { ans = m; hi = m-1; } else lo = m+1; }
    return ans;
}
int upperBound(vector<int>& a, int x) {
    int lo = 0, hi = a.size() - 1, ans = a.size();
    while (lo <= hi) { int m = lo+(hi-lo)/2;
        if (a[m] > x) { ans = m; hi = m-1; } else lo = m+1; }
    return ans;
}

vector<int> searchRange(vector<int>& a, int target) {
    int first = lowerBound(a, target);
    if (first == a.size() || a[first] != target) return {-1, -1};  // not present
    int last = upperBound(a, target) - 1;
    return {first, last};
}
```

---

## Complexity
| | |
|---|---|
| Time | O(log N) |
| Space | O(1) |
