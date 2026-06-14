# Quick Sort  🟡

**Reference:** [GeeksforGeeks — Quick Sort](https://www.geeksforgeeks.org/quick-sort/) · [LeetCode 912 — Sort an Array](https://leetcode.com/problems/sort-an-array/)

---

## Problem
Sort an array of N integers using **Quick Sort** (divide and conquer, in-place).

**Example**
```
Input:  arr = [4, 6, 2, 5, 7, 9, 1, 3]
Output: [1, 2, 3, 4, 5, 6, 7, 9]
```

---

## Intuition
Pick a **pivot**, then **partition** the array so that everything ≤ pivot is on its left and everything > pivot is on its right. The pivot is now in its final sorted position. Recurse on the left and right partitions.

Unlike merge sort, quick sort needs **no extra array** — partitioning happens in place.

---

## Code (C++)
```cpp
int partition(vector<int>& arr, int low, int high) {
    int pivot = arr[low];                   // choose first element as pivot
    int i = low, j = high;
    while (i < j) {
        while (i <= high - 1 && arr[i] <= pivot) i++;   // find element > pivot
        while (j >= low + 1  && arr[j] >  pivot) j--;   // find element <= pivot
        if (i < j) swap(arr[i], arr[j]);
    }
    swap(arr[low], arr[j]);                 // put pivot in its correct place
    return j;                               // pivot's final index
}

void quickSort(vector<int>& arr, int low, int high) {
    if (low < high) {
        int pIndex = partition(arr, low, high);
        quickSort(arr, low, pIndex - 1);    // sort left of pivot
        quickSort(arr, pIndex + 1, high);   // sort right of pivot
    }
}

// call: quickSort(arr, 0, arr.size() - 1);
```

---

## Complexity
| | |
|---|---|
| Time | O(N log N) average/best; **O(N²) worst** (already sorted + bad pivot) |
| Space | O(log N) average recursion stack (in-place, no extra array) |
| Stable? | ❌ No |

> **Avoiding the worst case:** pick a random pivot or median-of-three. In practice quick sort is often the fastest comparison sort due to good cache behavior and low constant factors — `std::sort` uses an introsort (quicksort + heapsort fallback).
