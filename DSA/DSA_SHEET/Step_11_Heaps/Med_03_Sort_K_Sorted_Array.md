# Sort a K-Sorted (Nearly Sorted) Array  🟡

**Reference:** [GeeksforGeeks — Sort a nearly sorted (or K sorted) array](https://www.geeksforgeeks.org/nearly-sorted-algorithm/)

---

## Problem
Each element is at most `k` positions away from its final sorted position. Sort the array efficiently using this guarantee.

```
Input:  arr = [6, 5, 3, 2, 8, 10, 9], k = 3
Output: [2, 3, 5, 6, 8, 9, 10]
```

---

## Intuition
Because every element is within `k` of its sorted slot, the **smallest unplaced element** is always among the next `k+1` elements. Keep a **min-heap of size k+1**: push the first `k+1`, then for each remaining element pop the min (it is now in final order) and push the new element. Finally drain the heap. This beats `O(N log N)` full sort when `k` is small.

---

## Code (C++)
```cpp
void sortKSorted(vector<int>& arr, int k) {
    priority_queue<int, vector<int>, greater<int>> mn;  // min-heap
    int n = arr.size(), idx = 0;
    // prime the heap with the first k+1 elements
    for (int i = 0; i <= k && i < n; i++) mn.push(arr[i]);
    // for each later element: extract min into place, then add it
    for (int i = k + 1; i < n; i++) {
        arr[idx++] = mn.top(); mn.pop();
        mn.push(arr[i]);
    }
    while (!mn.empty()) {            // drain remaining sorted values
        arr[idx++] = mn.top(); mn.pop();
    }
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N log k) |
| Space | O(k) |

> ⚠️ The heap must hold `k+1` elements, not `k` — an element `k` slots out of place needs `k+1` candidates in view to guarantee the true minimum is present. Guard the priming loop with `i < n` for small arrays.
