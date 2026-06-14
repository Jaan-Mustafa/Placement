# Next Smaller Element  🟡

**Reference:** [GeeksforGeeks — Next Smaller Element](https://www.geeksforgeeks.org/next-smaller-element/) · [InterviewBit](https://www.interviewbit.com/problems/nearest-smaller-element/)

---

## Problem
For each element, find the first element to its **right** that is strictly smaller. If none exists, output `-1`.

```
Input:  [4, 8, 5, 2, 25]
Output: [2, 5, 2, -1, -1]
```

---

## Intuition
Mirror of Next Greater Element — just flip the comparison. Use a **monotonic increasing stack** (scanning right to left): pop everything ≥ the current value, since those can't be the next smaller for anyone to the left. The stack top is the answer.

---

## Code (C++)
```cpp
vector<int> nextSmallerElements(vector<int>& nums) {
    int n = nums.size();
    vector<int> res(n, -1);
    stack<int> st;                          // increasing stack of values
    for (int i = n - 1; i >= 0; i--) {
        while (!st.empty() && st.top() >= nums[i])
            st.pop();                       // larger/equal can't be NSE
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
| Time | O(N) |
| Space | O(N) |

> Greater vs smaller is just `<=` vs `>=` in the pop condition. Keep a mental table: decreasing stack → next greater, increasing stack → next smaller.
