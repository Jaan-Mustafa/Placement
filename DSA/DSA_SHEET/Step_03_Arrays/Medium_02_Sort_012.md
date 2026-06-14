# Sort Array of 0s, 1s, 2s (Dutch National Flag)  🟡

**Reference:** [LeetCode 75 — Sort Colors](https://leetcode.com/problems/sort-colors/)

---

## Problem
Sort an array containing only 0s, 1s, and 2s in a single pass, in place (no counting/sorting library).

```
Input:  [2, 0, 2, 1, 1, 0]    Output: [0, 0, 1, 1, 2, 2]
```

---

## Intuition: Dutch National Flag algorithm
Three pointers partition the array into three regions:
- `[0..low-1]` → all 0s
- `[low..mid-1]` → all 1s
- `[high+1..n-1]` → all 2s
- `[mid..high]` → unprocessed

Process `arr[mid]`:
- `0` → swap with `low`, advance both `low` and `mid`.
- `1` → already in place, advance `mid`.
- `2` → swap with `high`, decrement `high` (don't advance mid — the swapped-in value is unexamined).

---

## Code (C++)
```cpp
void sortColors(vector<int>& arr) {
    int low = 0, mid = 0, high = arr.size() - 1;
    while (mid <= high) {
        if (arr[mid] == 0)      swap(arr[low++], arr[mid++]);
        else if (arr[mid] == 1) mid++;
        else                    swap(arr[mid], arr[high--]);  // arr[mid]==2
    }
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) single pass |
| Space | O(1) |

> ⚠️ On the `2` case, do **not** increment `mid` — the element swapped from `high` hasn't been checked yet.
