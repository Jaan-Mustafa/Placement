# Left Rotate Array by One Place  🟢

**Reference:** [GeeksforGeeks — Rotate array by one](https://www.geeksforgeeks.org/c-program-to-cyclically-rotate-an-array-by-one/)

---

## Problem
Rotate an array to the **left** by one position.

```
Input:  [1, 2, 3, 4, 5]    Output: [2, 3, 4, 5, 1]
```

---

## Intuition
Save the first element, shift every other element one step left, then place the saved element at the end.

---

## Code (C++)
```cpp
void leftRotateByOne(vector<int>& arr) {
    int n = arr.size();
    int first = arr[0];                 // save the element leaving the front
    for (int i = 1; i < n; i++)
        arr[i - 1] = arr[i];            // shift left
    arr[n - 1] = first;                 // wrap it to the end
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |
