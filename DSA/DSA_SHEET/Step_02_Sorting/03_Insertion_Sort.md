# Insertion Sort  🟢

**Reference:** [GeeksforGeeks — Insertion Sort](https://www.geeksforgeeks.org/insertion-sort/)

---

## Problem
Sort an array of N integers in ascending order using **Insertion Sort**.

**Example**
```
Input:  arr = [13, 46, 24, 52, 20, 9]
Output: [9, 13, 20, 24, 46, 52]
```

---

## Intuition
Like sorting playing cards in your hand: take one element at a time and **insert it into its correct position** among the already-sorted elements on its left.

- Maintain a sorted prefix `[0..i-1]`.
- Take `arr[i]` and shift all larger elements in the prefix one step right, then drop `arr[i]` into the gap.

---

## Code (C++)
```cpp
void insertionSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 1; i < n; i++) {
        int key = arr[i];                       // element to insert
        int j = i - 1;
        while (j >= 0 && arr[j] > key) {        // shift larger elements right
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key;                       // place key in the gap
    }
}
```

---

## Dry run
`arr = [13, 46, 24, 52, 20, 9]`
- i=1: 46 already > 13 → `[13, 46, ...]`
- i=2: insert 24 → `[13, 24, 46, 52, 20, 9]`
- i=3: 52 stays → no change
- i=4: insert 20 → `[13, 20, 24, 46, 52, 9]`
- i=5: insert 9 → `[9, 13, 20, 24, 46, 52]` ✅

---

## Complexity
| | |
|---|---|
| Time | O(N²) worst/avg, **O(N) best** (nearly sorted) |
| Space | O(1) — in-place |
| Stable? | ✅ Yes |

> Best choice among the O(N²) sorts for **small or nearly-sorted** arrays. Many hybrid sorts (introsort/timsort) fall back to insertion sort for tiny subarrays.
