# Sum of Subarray Minimums  🟡

**Reference:** [LeetCode 907 — Sum of Subarray Minimums](https://leetcode.com/problems/sum-of-subarray-minimums/)

---

## Problem
Return the sum of `min(b)` over every contiguous subarray `b`, modulo 1e9+7.

```
Input:  [3, 1, 2, 4]
Output: 17   (mins of all subarrays summed)
```

---

## Intuition
Instead of enumerating subarrays, count each element's **contribution**: how many subarrays have `arr[i]` as their minimum. That equals `left * right`, where `left` = number of consecutive elements to the left strictly greater than `arr[i]` (distance to previous-less-or-equal), and `right` = distance to next strictly-less element. Two **monotonic stacks** find these spans in O(N). Use strict on one side and non-strict on the other to avoid double-counting equal values.

---

## Code (C++)
```cpp
int sumSubarrayMins(vector<int>& arr) {
    const int MOD = 1e9 + 7;
    int n = arr.size();
    vector<int> left(n), right(n);          // span counts
    stack<int> st;

    // left[i]: # subarrays ending at i where arr[i] is min
    // previous strictly smaller element
    for (int i = 0; i < n; i++) {
        while (!st.empty() && arr[st.top()] > arr[i]) st.pop();
        left[i] = st.empty() ? i + 1 : i - st.top();
        st.push(i);
    }
    while (!st.empty()) st.pop();

    // right[i]: previous-or-equal smaller element to the right
    for (int i = n - 1; i >= 0; i--) {
        while (!st.empty() && arr[st.top()] >= arr[i]) st.pop();
        right[i] = st.empty() ? n - i : st.top() - i;
        st.push(i);
    }

    long long ans = 0;
    for (int i = 0; i < n; i++)
        ans = (ans + (long long)arr[i] * left[i] % MOD * right[i]) % MOD;
    return ans;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(N) |

> ⚠️ To handle duplicate values exactly once, use strict `>` on the left scan and non-strict `>=` on the right (or vice-versa) — symmetric strictness would double-count.
