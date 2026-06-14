# Reverse an Array (Recursion)  🟢

**Reference:** [GeeksforGeeks — Reverse an array using recursion](https://www.geeksforgeeks.org/write-a-program-to-reverse-an-array-or-string/)

---

## Problem
Reverse an array in place using recursion.

```
Input:  [1, 2, 3, 4, 5]    Output: [5, 4, 3, 2, 1]
```

---

## Intuition
Use **two pointers**, `left` and `right`. Swap them, then recurse moving inward (`left+1`, `right-1`). Stop when the pointers cross.

---

## Code (C++)
```cpp
void reverseArray(vector<int>& arr, int left, int right) {
    if (left >= right) return;          // pointers crossed → done
    swap(arr[left], arr[right]);        // swap the ends
    reverseArray(arr, left + 1, right - 1);  // move inward
}

// call: reverseArray(arr, 0, arr.size() - 1);

// single-pointer variant:
void reverse(vector<int>& arr, int i, int n) {
    if (i >= n / 2) return;
    swap(arr[i], arr[n - i - 1]);
    reverse(arr, i + 1, n);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(N) recursion stack (O(1) if done iteratively) |
