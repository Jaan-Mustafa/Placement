# Bubble Sort  🟢

**Reference:** [GeeksforGeeks — Bubble Sort](https://www.geeksforgeeks.org/bubble-sort/)

---

## Problem
Sort an array of N integers in ascending order using **Bubble Sort**.

**Example**
```
Input:  arr = [13, 46, 24, 52, 20, 9]
Output: [9, 13, 20, 24, 46, 52]
```

---

## Intuition
Bubble sort repeatedly steps through the list, **compares adjacent elements** and swaps them if they are in the wrong order. After each full pass, the largest remaining element "bubbles up" to its correct position at the end.

- Pass `i` pushes the largest of the unsorted region to index `n-1-i`.
- An **early-exit optimization**: if a whole pass makes no swaps, the array is already sorted → stop. This makes the best case O(N).

---

## Code (C++)
```cpp
void bubbleSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = n - 1; i >= 1; i--) {
        bool swapped = false;
        for (int j = 0; j < i; j++) {
            if (arr[j] > arr[j + 1]) {          // adjacent out of order
                swap(arr[j], arr[j + 1]);
                swapped = true;
            }
        }
        if (!swapped) break;                    // already sorted → stop early
    }
}
```

---

## Dry run
`arr = [13, 46, 24, 52, 20, 9]`
- Pass 1: largest (52) bubbles to end → `[13, 24, 46, 20, 9, 52]`
- Pass 2: 46 bubbles → `[13, 24, 20, 9, 46, 52]`
- Pass 3: `[13, 20, 9, 24, 46, 52]`
- Pass 4: `[13, 9, 20, 24, 46, 52]`
- Pass 5: `[9, 13, 20, 24, 46, 52]` ✅

---

## Complexity
| | |
|---|---|
| Time | O(N²) worst/avg, **O(N) best** (already sorted, with early-exit) |
| Space | O(1) — in-place |
| Stable? | ✅ Yes (only swaps strictly greater adjacent elements) |
