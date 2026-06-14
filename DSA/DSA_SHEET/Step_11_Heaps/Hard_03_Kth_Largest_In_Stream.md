# Kth Largest Element in a Stream  🔴

**Reference:** [LeetCode 703 — Kth Largest Element in a Stream](https://leetcode.com/problems/kth-largest-element-in-a-stream/)

---

## Problem
Design a class initialized with `k` and an array. Each `add(val)` inserts `val` into the stream and returns the **k-th largest** element seen so far.

```
KthLargest(3, [4,5,8,2])
add(3) -> 4   add(5) -> 5   add(10) -> 5   add(9) -> 8   add(4) -> 8
```

---

## Intuition
Keep a **min-heap capped at size k**. The heap always holds the `k` largest values seen, so its top is the running k-th largest. On each `add`, push and, if the size exceeds `k`, pop the smallest — `top()` is the answer in `O(log k)`.

---

## Code (C++)
```cpp
class KthLargest {
    int k;
    priority_queue<int, vector<int>, greater<int>> mn;   // min-heap of k largest
public:
    KthLargest(int k, vector<int>& nums) : k(k) {
        for (int x : nums) add(x);          // reuse add to build the heap
    }
    int add(int val) {
        mn.push(val);
        if ((int)mn.size() > k) mn.pop();   // keep only k largest
        return mn.top();                    // kth largest so far
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | add O(log k); constructor O(N log k) |
| Space | O(k) |

> ⚠️ The heap never grows beyond `k`, so this stays cheap even on long streams. Early on (fewer than k elements added) the top is the smallest seen — that is the correct k-th largest given the data so far.
