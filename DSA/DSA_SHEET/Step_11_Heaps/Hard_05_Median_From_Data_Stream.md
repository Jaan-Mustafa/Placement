# Find Median from Data Stream  🔴

**Reference:** [LeetCode 295 — Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/)

---

## Problem
Support `addNum(x)` on a growing stream and `findMedian()` returning the median of all numbers seen so far.

```
addNum(1); addNum(2); findMedian() -> 1.5
addNum(3); findMedian() -> 2.0
```

---

## Intuition
Split the data into a lower half and an upper half. A **max-heap `lo`** holds the smaller half (its top is the largest of the small side); a **min-heap `hi`** holds the larger half (its top is the smallest of the large side). Keep them balanced (sizes differ by ≤ 1). The median is `lo.top()` when sizes differ, or the average of both tops when equal.

---

## Code (C++)
```cpp
class MedianFinder {
    priority_queue<int> lo;                              // max-heap: lower half
    priority_queue<int, vector<int>, greater<int>> hi;   // min-heap: upper half
public:
    void addNum(int num) {
        lo.push(num);                 // tentatively add to lower half
        hi.push(lo.top()); lo.pop();  // shift its max to upper half (sorts the two)
        if (hi.size() > lo.size()) {  // rebalance so lo >= hi in size
            lo.push(hi.top()); hi.pop();
        }
    }
    double findMedian() {
        if (lo.size() > hi.size()) return lo.top();      // odd count
        return (lo.top() + hi.top()) / 2.0;              // even count
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | addNum O(log n), findMedian O(1) |
| Space | O(n) |

> ⚠️ The push-into-lo then move-top-to-hi trick guarantees `max(lo) <= min(hi)` automatically. Invariant: `lo.size()` equals `hi.size()` or is exactly one larger — that is why `lo.top()` is the median for odd counts. Use `/ 2.0` to avoid integer division.
