# Next Greater Element II  🟡

**Reference:** [LeetCode 503 — Next Greater Element II](https://leetcode.com/problems/next-greater-element-ii/)

---

## Problem
Same as Next Greater Element, but the array is **circular** — the search wraps around to the front.

```
Input:  [1, 2, 1]
Output: [2, -1, 2]
```

---

## Intuition
Simulate the circular array by iterating `2N` times and indexing with `i % n`. Keep a **monotonic decreasing stack of indices**. Going right to left over two passes guarantees every element sees its next greater even across the wrap-around. Only record answers during the first conceptual pass (when `i < n`), but let the second pass seed the stack.

---

## Code (C++)
```cpp
vector<int> nextGreaterElements(vector<int>& nums) {
    int n = nums.size();
    vector<int> res(n, -1);
    stack<int> st;                          // stack of indices, decreasing values
    for (int i = 2 * n - 1; i >= 0; i--) {
        int cur = nums[i % n];
        while (!st.empty() && nums[st.top()] <= cur)
            st.pop();
        if (i < n && !st.empty())           // record only on real pass
            res[i] = nums[st.top()];
        st.push(i % n);
    }
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — 2N iterations, each element pushed/popped once |
| Space | O(N) |

> The "iterate 2N, mod by N" pattern is the standard way to fake a circular array without physically duplicating it.
