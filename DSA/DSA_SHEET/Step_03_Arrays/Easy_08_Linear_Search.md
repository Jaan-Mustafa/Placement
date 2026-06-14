# Linear Search  🟢

**Reference:** [GeeksforGeeks — Linear Search](https://www.geeksforgeeks.org/linear-search/)

---

## Problem
Find the index of the first occurrence of `target` in an array. Return -1 if absent.

```
Input:  [3, 4, 1, 7, 5], target = 7    Output: 3
Input:  [3, 4, 1], target = 9          Output: -1
```

---

## Intuition
Scan left to right; return the index as soon as a match is found.

---

## Code (C++)
```cpp
int linearSearch(vector<int>& arr, int target) {
    for (int i = 0; i < arr.size(); i++)
        if (arr[i] == target) return i;     // first match
    return -1;                              // not found
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> Works on **unsorted** data. If the array is sorted, prefer Binary Search (O(log N)) — see Step 4.
