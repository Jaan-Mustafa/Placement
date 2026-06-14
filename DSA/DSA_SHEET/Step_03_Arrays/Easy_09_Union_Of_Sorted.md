# Union of Two Sorted Arrays  🟢

**Reference:** [GeeksforGeeks — Union of two sorted arrays](https://www.geeksforgeeks.org/union-and-intersection-of-two-sorted-arrays-2/)

---

## Problem
Given two sorted arrays, return their **union** (all distinct elements, sorted).

```
Input:  a = [1, 2, 3, 4, 5], b = [2, 3, 4, 4, 5, 6]
Output: [1, 2, 3, 4, 5, 6]
```

---

## Intuition
**Two pointers** walking both sorted arrays together. At each step push the smaller element (skipping anything equal to the last value pushed, to avoid duplicates), advancing that pointer. Then drain whatever remains.

---

## Code (C++)
```cpp
vector<int> unionSorted(vector<int>& a, vector<int>& b) {
    int i = 0, j = 0;
    vector<int> res;
    auto add = [&](int x){ if (res.empty() || res.back() != x) res.push_back(x); };

    while (i < a.size() && j < b.size()) {
        if (a[i] < b[j])       add(a[i++]);
        else if (a[i] > b[j])  add(b[j++]);
        else { add(a[i]); i++; j++; }      // equal: take once, advance both
    }
    while (i < a.size()) add(a[i++]);      // leftovers from a
    while (j < b.size()) add(b[j++]);      // leftovers from b
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N + M) |
| Space | O(N + M) for the output |

> **Intersection** uses the same two-pointer skeleton: push only when `a[i] == b[j]`.
