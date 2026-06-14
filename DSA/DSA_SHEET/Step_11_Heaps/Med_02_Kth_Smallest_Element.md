# Kth Smallest Element in an Array  🟡

**Reference:** [GeeksforGeeks — Kth Smallest Element](https://www.geeksforgeeks.org/kth-smallestlargest-element-unsorted-array/) · [LeetCode 215 (mirror)](https://leetcode.com/problems/kth-largest-element-in-an-array/)

---

## Problem
Return the `k`-th smallest element in an unsorted array.

```
Input:  arr = [7, 10, 4, 3, 20, 15], k = 3    Output: 7
Input:  arr = [7, 10, 4, 3, 20, 15], k = 4    Output: 10
```

---

## Intuition
Mirror of "k-th largest": maintain a **max-heap of size k**. Push each element; when the size exceeds `k`, pop the largest. The heap then keeps the `k` smallest seen, and its top is the largest of those — the `k`-th smallest. Max-heap is the STL default, so no comparator needed.

---

## Code (C++)
```cpp
int kthSmallest(vector<int>& arr, int k) {
    priority_queue<int> mx;        // max-heap (default): largest of k smallest on top
    for (int x : arr) {
        mx.push(x);
        if ((int)mx.size() > k) mx.pop();  // drop the largest, keep k smallest
    }
    return mx.top();               // kth smallest
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N log k) |
| Space | O(k) |

> ⚠️ Symmetry to remember: **k smallest → max-heap of size k**, **k largest → min-heap of size k**. The heap holds the candidate set; its top is the boundary element you want.
