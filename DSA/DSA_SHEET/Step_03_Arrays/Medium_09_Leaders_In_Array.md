# Leaders in an Array  🟡

**Reference:** [GeeksforGeeks — Leaders in an array](https://www.geeksforgeeks.org/leaders-in-an-array/)

---

## Problem
An element is a **leader** if it is greater than (or equal to, per GfG) all elements to its right. The last element is always a leader. Return all leaders.

```
Input:  [10, 22, 12, 3, 0, 6]    Output: [22, 12, 6]
```

---

## Intuition
Scan from the **right**, tracking the maximum seen so far. An element is a leader iff it's ≥ that running max-to-its-right. This gives O(N) instead of the O(N²) brute force.

---

## Code (C++)
```cpp
vector<int> leaders(vector<int>& arr) {
    int n = arr.size();
    vector<int> res;
    int maxRight = INT_MIN;
    for (int i = n - 1; i >= 0; i--) {
        if (arr[i] >= maxRight) {           // greater than everything to the right
            res.push_back(arr[i]);
            maxRight = arr[i];
        }
    }
    reverse(res.begin(), res.end());        // restore left-to-right order
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) extra (output excluded) |
