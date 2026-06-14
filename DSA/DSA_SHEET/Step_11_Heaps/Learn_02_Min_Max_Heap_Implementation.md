# Min Heap and Max Heap Implementation  🟡

**Reference:** [GeeksforGeeks — Binary Heap](https://www.geeksforgeeks.org/binary-heap/) · [GeeksforGeeks — Min Heap in C++](https://www.geeksforgeeks.org/min-heap-in-cpp/)

---

## Problem
Implement a heap **from scratch** (array-backed) supporting `insert`, `getTop`, and `extractTop`, using the two core helpers `heapifyUp` (sift up after insert) and `heapifyDown` (sift down after removing the root).

```
insert: 30, 50, 40, 10  →  max-heap top() = 50
extractTop() → 50, new top() = 40
```

---

## Intuition
Keep the tree in a `vector`. After inserting at the end, **bubble up**: swap with the parent while it violates the heap property. To extract the root, move the last element to index 0 and **sift down**: repeatedly swap with the larger child (max-heap) / smaller child (min-heap) until the property holds. A min-heap is identical with the comparison flipped.

---

## Code (C++)
```cpp
class MaxHeap {
    vector<int> h;
    void up(int i) {                       // heapifyUp
        while (i > 0 && h[(i - 1) / 2] < h[i]) {
            swap(h[i], h[(i - 1) / 2]);
            i = (i - 1) / 2;
        }
    }
    void down(int i) {                     // heapifyDown
        int n = h.size();
        while (true) {
            int l = 2 * i + 1, r = 2 * i + 2, big = i;
            if (l < n && h[l] > h[big]) big = l;
            if (r < n && h[r] > h[big]) big = r;
            if (big == i) break;           // property restored
            swap(h[i], h[big]);
            i = big;
        }
    }
public:
    void insert(int x) { h.push_back(x); up(h.size() - 1); }
    int  getTop()      { return h[0]; }
    int  extractTop() {
        int top = h[0];
        h[0] = h.back(); h.pop_back();     // move last to root
        if (!h.empty()) down(0);
        return top;
    }
    bool empty() { return h.empty(); }
};
// For a MIN-heap, flip the comparisons: '<' becomes '>' in up(),
// and pick the SMALLER child in down().
```

---

## Complexity
| | |
|---|---|
| Time | insert O(log n), extractTop O(log n), getTop O(1) |
| Space | O(n) |

> ⚠️ In `extractTop`, guard `down(0)` with `!h.empty()` — popping the only element leaves the heap empty. Integer parent index `(i-1)/2` is correct only for `i > 0`.
