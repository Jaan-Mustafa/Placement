# Number of Times Array is Rotated  🟡

**Reference:** [GeeksforGeeks — Find rotation count](https://www.geeksforgeeks.org/find-rotation-count-rotated-sorted-array/)

---

## Problem
A sorted array was rotated some number of times. Find how many times — equivalently, the **index of the minimum** element.

```
Input:  [4, 5, 6, 7, 0, 1, 2]    Output: 4   (min 0 is at index 4)
Input:  [3, 4, 5, 1, 2]          Output: 3
```

---

## Intuition
The number of rotations equals the **index of the minimum element**. Use the same logic as "find minimum in rotated array", but track the **index** of the smallest value rather than the value.

---

## Code (C++)
```cpp
int findRotationCount(vector<int>& a) {
    int low = 0, high = a.size() - 1;
    int ans = INT_MAX, idx = 0;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (a[low] <= a[mid]) {                 // left half sorted
            if (a[low] < ans) { ans = a[low]; idx = low; }
            low = mid + 1;
        } else {                                // right half sorted
            if (a[mid] < ans) { ans = a[mid]; idx = mid; }
            high = mid - 1;
        }
    }
    return idx;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(log N) |
| Space | O(1) |

> A non-rotated (already sorted) array returns index 0 — zero rotations.
