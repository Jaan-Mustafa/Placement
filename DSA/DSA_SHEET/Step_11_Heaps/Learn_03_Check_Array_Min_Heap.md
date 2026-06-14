# Check if an Array Represents a Min-Heap  🟡

**Reference:** [GeeksforGeeks — How to check if a given array represents a Binary Heap](https://www.geeksforgeeks.org/how-to-check-if-a-given-array-represents-a-binary-heap/)

---

## Problem
Given an array, decide whether it is a valid **min-heap** when read as a complete binary tree (level-order). Every parent must be ≤ both its children.

```
Input:  [10, 20, 30, 21, 23]   Output: true
Input:  [10, 20, 30, 25, 5]    Output: false   (25's child 5 < 25)
```

---

## Intuition
No heap needed — just verify the property directly on the array. For a node at index `i`, children sit at `2*i+1` and `2*i+2`. It suffices to check every **internal** node (indices `0 .. n/2 - 1`); leaves have no children. If any parent exceeds a child, it is not a min-heap.

---

## Code (C++)
```cpp
bool isMinHeap(vector<int>& a) {
    int n = a.size();
    // only internal nodes have children: indices 0 .. n/2 - 1
    for (int i = 0; i <= (n - 2) / 2; i++) {
        int l = 2 * i + 1, r = 2 * i + 2;
        if (l < n && a[i] > a[l]) return false;   // parent > left child
        if (r < n && a[i] > a[r]) return false;   // parent > right child
    }
    return true;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(n) |
| Space | O(1) |

> ⚠️ Loop bound: the last internal node is `(n-2)/2`. Always guard `l < n` and `r < n` before indexing — the last parent may have only a left child. For a **max-heap** check, flip the comparisons to `a[i] < a[l]`.
