# Largest Element in Array  🟢

**Reference:** [GeeksforGeeks — Largest element in array](https://www.geeksforgeeks.org/c-program-find-largest-element-array/)

---

## Problem
Given an array, return the largest element.

```
Input:  [3, 3, 6, 1]    Output: 6
```

---

## Intuition
Single pass: track a running maximum, updating it whenever a larger element appears.

---

## Code (C++)
```cpp
int largestElement(vector<int>& arr) {
    int maxi = arr[0];
    for (int x : arr)
        maxi = max(maxi, x);
    return maxi;
}
// or simply: *max_element(arr.begin(), arr.end());
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |
