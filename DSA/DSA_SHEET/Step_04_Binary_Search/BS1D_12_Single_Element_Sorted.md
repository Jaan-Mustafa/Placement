# Single Element in a Sorted Array  🟡

**Reference:** [LeetCode 540 — Single Element in a Sorted Array](https://leetcode.com/problems/single-element-in-a-sorted-array/)

---

## Problem
Every element appears exactly twice except one, which appears once. The array is sorted. Find that single element in O(log N).

```
Input:  [1, 1, 2, 3, 3, 4, 4, 8, 8]    Output: 2
```

---

## Intuition
Before the single element, pairs start at **even** indices: `(0,1), (2,3), ...` so `a[even] == a[even+1]`. After the single element, this pattern shifts. Binary search on the index parity:
- At `mid` (make it even), if `a[mid] == a[mid+1]`, the single element is to the **right** → `low = mid + 2`.
- Otherwise it's at `mid` or to the **left** → `high = mid`.

---

## Code (C++)
```cpp
int singleNonDuplicate(vector<int>& a) {
    int low = 0, high = a.size() - 1;
    while (low < high) {
        int mid = low + (high - low) / 2;
        if (mid % 2 == 1) mid--;            // force mid to be even
        if (a[mid] == a[mid + 1])
            low = mid + 2;                  // single element is to the right
        else
            high = mid;                     // single element at mid or left
    }
    return a[low];
}
```

---

## Dry run
`[1,1,2,3,3,4,4,8,8]`: pairs align at even indices until index 2 where `a[2]=2 != a[3]=3` → answer narrows to **2**.

## Complexity
| | |
|---|---|
| Time | O(log N) |
| Space | O(1) |

> A linear XOR of all elements also finds it in O(N) — but the problem specifically wants O(log N), so exploit the sorted structure.
