# Convert Min-Heap to Max-Heap  🟡

**Reference:** [GeeksforGeeks — Convert min Heap to max Heap](https://www.geeksforgeeks.org/convert-min-heap-to-max-heap/)

---

## Problem
Given an array that is already a valid **min-heap**, rearrange it in place so it becomes a valid **max-heap**.

```
Input:  [3, 5, 9, 6, 8, 20, 10, 12, 18, 9]
Output: a valid max-heap, e.g. [20, 18, 10, 12, 9, 9, 3, 5, 6, 8]
```

---

## Intuition
Ignore the existing min-heap structure and **build a max-heap from scratch** using the standard `O(n)` build-heap. Start from the last internal node `n/2 - 1` and `heapifyDown` (sift down) toward the root. Processing bottom-up guarantees that when we heapify a node, both its subtrees are already valid max-heaps.

---

## Code (C++)
```cpp
void heapifyDown(vector<int>& a, int n, int i) {
    while (true) {
        int l = 2 * i + 1, r = 2 * i + 2, big = i;
        if (l < n && a[l] > a[big]) big = l;
        if (r < n && a[r] > a[big]) big = r;
        if (big == i) break;
        swap(a[i], a[big]);
        i = big;                       // follow the swapped child down
    }
}

void convertToMaxHeap(vector<int>& a) {
    int n = a.size();
    // build-heap: heapify every internal node, bottom-up → O(n)
    for (int i = n / 2 - 1; i >= 0; i--)
        heapifyDown(a, n, i);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(n) — bottom-up build-heap (not O(n log n)) |
| Space | O(1) in place |

> ⚠️ Build-heap is `O(n)`, not `O(n log n)`: most nodes are near the leaves and sift down only a short distance. Iterating top-down instead would be incorrect — children must be valid heaps before their parent is heapified.
