# Recursive Insertion Sort  🟢

**Reference:** [GeeksforGeeks — Recursive Insertion Sort](https://www.geeksforgeeks.org/recursive-insertion-sort/)

---

## Problem
Implement **Insertion Sort recursively**.

---

## Intuition
- To sort the first `i` elements, first **recursively sort** the first `i-1` elements.
- Then **insert** `arr[i]` into its correct position within the sorted prefix.
- Base case: `i == n` (nothing left to insert) — or `i <= 1` for a single element.

---

## Code (C++)
```cpp
void insertionSort(vector<int>& arr, int i, int n) {
    if (i == n) return;                         // base case: done

    int key = arr[i];
    int j = i - 1;
    while (j >= 0 && arr[j] > key) {            // shift larger elements right
        arr[j + 1] = arr[j];
        j--;
    }
    arr[j + 1] = key;                           // insert key into the gap

    insertionSort(arr, i + 1, n);               // sort the rest
}

// call: insertionSort(arr, 0, arr.size());
```

---

## Complexity
| | |
|---|---|
| Time | O(N²) worst/avg, O(N) best (nearly sorted) |
| Space | O(N) recursion stack |
| Stable? | ✅ Yes |

> Same behavior as iterative insertion sort; the recursion replaces the outer `for` loop over `i`.
