# Floor and Ceil in a Sorted Array  🟢

**Reference:** [GeeksforGeeks — Floor and ceil in sorted array](https://www.geeksforgeeks.org/ceiling-in-a-sorted-array/)

---

## Problem
- **Floor(x):** the largest element `≤ x`.
- **Ceil(x):** the smallest element `≥ x`.

Return -1 if undefined.

```
Input:  [10, 20, 30, 40, 50], x = 25
Output: floor = 20, ceil = 30
```

---

## Intuition
- **Ceil** is the classic **lower bound** value: first element `≥ x`.
- **Floor** is found by binary search: whenever `a[mid] <= x`, it's a floor candidate — record it and search right for a bigger valid one.

---

## Code (C++)
```cpp
int findFloor(vector<int>& a, int x) {
    int low = 0, high = a.size() - 1, ans = -1;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (a[mid] <= x) { ans = a[mid]; low = mid + 1; }  // candidate, go right
        else high = mid - 1;
    }
    return ans;
}

int findCeil(vector<int>& a, int x) {
    int low = 0, high = a.size() - 1, ans = -1;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (a[mid] >= x) { ans = a[mid]; high = mid - 1; }  // candidate, go left
        else low = mid + 1;
    }
    return ans;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(log N) each |
| Space | O(1) |

> Floor = "largest ≤ x" → move right on success. Ceil = "smallest ≥ x" → move left on success.
