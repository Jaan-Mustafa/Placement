# Recursive Bubble Sort  🟢

**Reference:** [GeeksforGeeks — Recursive Bubble Sort](https://www.geeksforgeeks.org/recursive-bubble-sort/)

---

## Problem
Implement **Bubble Sort recursively** (no explicit outer loop).

---

## Intuition
The outer loop of bubble sort can be replaced by recursion:
- One recursive call performs **one full pass**, bubbling the largest element of the current range to the end.
- Then recurse on the array with size reduced by one (`n-1`).
- Base case: when the range size is 1, the array is sorted.

---

## Code (C++)
```cpp
void bubbleSort(vector<int>& arr, int n) {
    if (n == 1) return;                         // base case: single element

    bool swapped = false;
    for (int j = 0; j < n - 1; j++) {           // one pass
        if (arr[j] > arr[j + 1]) {
            swap(arr[j], arr[j + 1]);
            swapped = true;
        }
    }
    if (!swapped) return;                       // already sorted → stop early

    bubbleSort(arr, n - 1);                     // recurse on smaller range
}

// call: bubbleSort(arr, arr.size());
```

---

## Complexity
| | |
|---|---|
| Time | O(N²) worst/avg, O(N) best (early-exit) |
| Space | O(N) recursion stack (vs O(1) for the iterative version) |
| Stable? | ✅ Yes |

> Purely an exercise in converting iteration → recursion. The iterative version is preferred in practice (no stack overhead).
