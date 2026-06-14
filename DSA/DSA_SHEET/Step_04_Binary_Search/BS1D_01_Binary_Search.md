# Binary Search to Find X  🟢

**Reference:** [LeetCode 704 — Binary Search](https://leetcode.com/problems/binary-search/)

---

## Problem
Given a **sorted** array, find the index of `target`. Return -1 if absent.

```
Input:  [-1, 0, 3, 5, 9, 12], target = 9    Output: 4
```

---

## Intuition
Repeatedly halve the search space. Compare `target` with the middle element: if equal → found; if target is smaller → search the left half; if larger → search the right half. Each step discards half, giving O(log N).

---

## Code (C++)
```cpp
int search(vector<int>& a, int target) {
    int low = 0, high = a.size() - 1;
    while (low <= high) {
        int mid = low + (high - low) / 2;     // avoids overflow vs (low+high)/2
        if (a[mid] == target) return mid;
        else if (a[mid] < target) low = mid + 1;   // go right
        else high = mid - 1;                        // go left
    }
    return -1;
}
```

### Recursive form
```cpp
int bs(vector<int>& a, int low, int high, int t) {
    if (low > high) return -1;
    int mid = low + (high - low) / 2;
    if (a[mid] == t) return mid;
    return a[mid] < t ? bs(a, mid + 1, high, t) : bs(a, low, mid - 1, t);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(log N) |
| Space | O(1) iterative / O(log N) recursive |

> ⚠️ Use `mid = low + (high - low) / 2` to avoid integer overflow when `low + high` exceeds INT_MAX.
