# Introduction to Priority Queue / Heaps  🟢

**Reference:** [GeeksforGeeks — Heap Data Structure](https://www.geeksforgeeks.org/heap-data-structure/) · [GeeksforGeeks — Priority Queue in C++ STL](https://www.geeksforgeeks.org/priority-queue-in-cpp-stl/)

---

## Problem
A **heap** is a complete binary tree stored in an array where every parent obeys the heap property. A **priority queue** is the abstract container that uses a heap to always serve the highest-priority element first. Learn the layout and the two core operations: `push` and `pop` of the top.

```
Max-heap array: [50, 30, 40, 10, 20]
        50
       /  \
      30   40
     / \
    10  20
top() == 50
```

---

## Intuition
Store the tree level-by-level in an array. For a node at index `i` (0-based):
- parent = `(i - 1) / 2`
- left child = `2*i + 1`, right child = `2*i + 2`

A **max-heap** keeps the largest on top (parent ≥ children); a **min-heap** keeps the smallest on top (parent ≤ children). The STL `priority_queue` is a **max-heap by default**; pass `greater<int>` as the comparator to get a min-heap. `push` bubbles a value up; `pop` removes the root and sifts the last element down.

---

## Code (C++)
```cpp
#include <queue>
using namespace std;

void demo() {
    // Max-heap (default): largest on top
    priority_queue<int> maxHeap;
    maxHeap.push(30);
    maxHeap.push(50);
    maxHeap.push(40);
    int largest = maxHeap.top();   // 50, O(1)
    maxHeap.pop();                 // removes 50, returns void

    // Min-heap: smallest on top — note greater<int>
    priority_queue<int, vector<int>, greater<int>> minHeap;
    minHeap.push(30);
    minHeap.push(50);
    minHeap.push(40);
    int smallest = minHeap.top();  // 30

    // Build a heap from a vector in O(n)
    vector<int> v = {3, 1, 4, 1, 5};
    priority_queue<int> pq(v.begin(), v.end());  // heapify, O(n)
}
```

---

## Complexity
| | |
|---|---|
| Time | push/pop O(log n), top O(1), build-heap O(n) |
| Space | O(n) for n elements |

> ⚠️ The comparator is "reversed" from intuition: `greater<int>` gives a **min**-heap. `pop()` returns void — always read `top()` first.
