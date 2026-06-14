# Kth Largest Element in an Array  🟡

**Reference:** [LeetCode 215 — Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/)

---

## Problem
Return the `k`-th largest element in an unsorted array (k-th in sorted order, **not** the k-th distinct).

```
Input:  nums = [3, 2, 1, 5, 6, 4], k = 2    Output: 5
Input:  nums = [3, 2, 3, 1, 2, 4, 5, 5, 6], k = 4    Output: 4
```

---

## Intuition
Maintain a **min-heap of size k**. Push every element; whenever the heap grows past `k`, pop the smallest. The heap then always holds the `k` largest seen so far, and its top is the smallest of those — exactly the `k`-th largest. A min-heap of size k is cheaper than sorting and avoids storing all of a stream.

---

## Code (C++)
```cpp
int findKthLargest(vector<int>& nums, int k) {
    // min-heap: smallest of the k largest sits on top
    priority_queue<int, vector<int>, greater<int>> mn;
    for (int x : nums) {
        mn.push(x);
        if ((int)mn.size() > k) mn.pop();  // drop the smallest, keep k largest
    }
    return mn.top();                       // kth largest
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N log k) |
| Space | O(k) |

> ⚠️ For **k-th largest** use a **min**-heap of size k; for **k-th smallest** use a **max**-heap of size k. A common slip is reaching for a max-heap here — that would force you to pop k-1 times from a full heap (O(N log N)).
