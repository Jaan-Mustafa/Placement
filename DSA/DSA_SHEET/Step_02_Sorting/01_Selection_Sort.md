# Selection Sort  🟢

**Reference:** [GeeksforGeeks — Selection Sort](https://www.geeksforgeeks.org/selection-sort/)

---

## Problem
Given an array of N integers, sort it in ascending order using the **Selection Sort** algorithm.

**Example**
```
Input:  arr = [13, 46, 24, 52, 20, 9]
Output: [9, 13, 20, 24, 46, 52]
```

---

## Intuition
Selection sort works by repeatedly **selecting the minimum** element from the unsorted part and putting it at the beginning.

- Think of the array as two parts: a sorted prefix (initially empty) and an unsorted suffix.
- In each pass `i`, scan the unsorted suffix `[i..n-1]`, find the index of the smallest element, and swap it with `arr[i]`.
- After pass `i`, the first `i+1` elements are finalized.

---

## Code (C++)
```cpp
void selectionSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 0; i < n - 1; i++) {
        int minIndex = i;                       // assume current is smallest
        for (int j = i + 1; j < n; j++) {
            if (arr[j] < arr[minIndex])
                minIndex = j;                   // found a smaller element
        }
        swap(arr[i], arr[minIndex]);            // place smallest at position i
    }
}
```

---

## Dry run
`arr = [13, 46, 24, 52, 20, 9]`
- i=0: min in [13..9] is 9 (idx 5) → swap → `[9, 46, 24, 52, 20, 13]`
- i=1: min in [46..13] is 13 (idx 5) → swap → `[9, 13, 24, 52, 20, 46]`
- i=2: min is 20 (idx 4) → swap → `[9, 13, 20, 52, 24, 46]`
- i=3: min is 24 → swap → `[9, 13, 20, 24, 52, 46]`
- i=4: min is 46 → swap → `[9, 13, 20, 24, 46, 52]` ✅

---

## Complexity
| | |
|---|---|
| Time | O(N²) — two nested loops, always (best = worst = avg) |
| Space | O(1) — in-place |
| Stable? | ❌ No (swapping can change relative order of equal elements) |

> Note: selection sort does the **minimum number of swaps** (at most N-1), so it's useful when writes are expensive.
