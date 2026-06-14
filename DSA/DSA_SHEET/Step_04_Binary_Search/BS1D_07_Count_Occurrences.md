# Count Occurrences in a Sorted Array  🟡

**Reference:** [GeeksforGeeks — Count occurrences in sorted array](https://www.geeksforgeeks.org/count-number-of-occurrences-or-frequency-in-a-sorted-array/)

---

## Problem
Count how many times `target` appears in a sorted array.

```
Input:  [1, 2, 2, 2, 3, 4], target = 2    Output: 3
```

---

## Intuition
Count = `upperBound(target) - lowerBound(target)`. The lower bound gives the first occurrence; the upper bound gives one past the last. Their difference is the frequency. O(log N), no linear scan.

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

int countOccurrences(vector<int>& a, int target) {
    return upperBound(a, target) - lowerBound(a, target);
}
// STL one-liner:
// upper_bound(a.begin(),a.end(),t) - lower_bound(a.begin(),a.end(),t);
```

---

## Complexity
| | |
|---|---|
| Time | O(log N) |
| Space | O(1) |
