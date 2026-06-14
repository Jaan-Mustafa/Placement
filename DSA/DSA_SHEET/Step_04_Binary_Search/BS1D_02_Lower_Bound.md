# Lower Bound  🟢

**Reference:** [GeeksforGeeks — Lower bound](https://www.geeksforgeeks.org/lower_bound-in-cpp/)

---

## Problem
Find the **lower bound**: the smallest index `i` such that `a[i] >= x`. If no such element exists, return `n` (past the end).

```
Input:  [1, 2, 2, 3], x = 2    Output: 1   (first element >= 2)
Input:  [1, 2, 2, 3], x = 4    Output: 4   (none, returns n)
```

---

## Intuition
Binary search for the **leftmost** position not less than `x`. When `a[mid] >= x`, `mid` is a candidate answer — record it and search further left for an even earlier one. Otherwise discard the left half.

---

## Code (C++)
```cpp
int lowerBound(vector<int>& a, int x) {
    int low = 0, high = a.size() - 1, ans = a.size();
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (a[mid] >= x) {
            ans = mid;          // candidate; look for an earlier one
            high = mid - 1;
        } else {
            low = mid + 1;
        }
    }
    return ans;
}
// STL: lower_bound(a.begin(), a.end(), x) - a.begin();
```

---

## Complexity
| | |
|---|---|
| Time | O(log N) |
| Space | O(1) |

> Lower bound is the building block for many BS problems: count of elements, floor/ceil, insert position, search range.
