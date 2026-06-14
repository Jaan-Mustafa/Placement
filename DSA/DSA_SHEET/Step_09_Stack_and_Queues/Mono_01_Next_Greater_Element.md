# Next Greater Element  🟡

**Reference:** [LeetCode 496 — Next Greater Element I](https://leetcode.com/problems/next-greater-element-i/) · [GeeksforGeeks](https://www.geeksforgeeks.org/next-greater-element/)

---

## Problem
For each element, find the first element to its **right** that is strictly greater. If none exists, output `-1`.

```
Input:  [4, 5, 2, 25]
Output: [5, 25, 25, -1]
```

---

## Intuition
Use a **monotonic decreasing stack** of values, scanning right to left. The stack holds candidates for "next greater". Before deciding an element's answer, pop everything ≤ it — those can never be the next greater for anything to the left. The stack top (if any) is the answer; then push the current value.

---

## Code (C++)
```cpp
vector<int> nextGreaterElements(vector<int>& nums) {
    int n = nums.size();
    vector<int> res(n, -1);
    stack<int> st;                         // decreasing stack of values
    for (int i = n - 1; i >= 0; i--) {
        while (!st.empty() && st.top() <= nums[i])
            st.pop();                      // smaller/equal can't be NGE
        if (!st.empty()) res[i] = st.top();
        st.push(nums[i]);
    }
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — each element pushed/popped once |
| Space | O(N) |

> The monotonic-stack invariant: the stack stays decreasing from bottom to top, so its top is always the nearest greater element seen so far on the right.
