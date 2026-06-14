# Number of NGEs to the Right  🟡

**Reference:** [GeeksforGeeks — Number of NGEs to the right](https://www.geeksforgeeks.org/number-nges-right/)

---

## Problem
Given an array and a list of query indices, for each query index return the **count** of elements to its right that are strictly greater than `arr[index]`.

```
arr = [3, 4, 2, 7, 5, 8, 10, 6], queries = [0, 5]
index 0 (=3): greater to right -> 4,7,5,8,10,6  => 6
index 5 (=8): greater to right -> 10            => 1
```

---

## Intuition
This is a counting variant, not a "nearest" variant, so a monotonic stack alone won't give counts directly. The simplest correct approach is, per query, scan to the right and count greater elements — O(N) per query. (A merge-sort / BIT approach can pre-compute all counts in O(N log N), but the direct scan is what the sheet expects.)

---

## Code (C++)
```cpp
// Returns, for each query index, how many elements to its right are greater.
vector<int> countNGEs(vector<int>& arr, vector<int>& queries) {
    int n = arr.size();
    vector<int> ans;
    for (int q : queries) {
        int cnt = 0;
        for (int j = q + 1; j < n; j++)
            if (arr[j] > arr[q]) cnt++;     // strictly greater on the right
        ans.push_back(cnt);
    }
    return ans;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(Q · N) for Q queries |
| Space | O(1) extra (besides output) |

> ⚠️ "Number of" greater elements is a *count*, distinct from "next greater" (nearest). Monotonic stacks find the nearest in O(N); counting all of them needs a scan or an order-statistics structure (BIT/merge sort) for O(N log N).
