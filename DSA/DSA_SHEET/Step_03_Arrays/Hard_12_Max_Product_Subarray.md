# Maximum Product Subarray  🔴

**Reference:** [LeetCode 152 — Maximum Product Subarray](https://leetcode.com/problems/maximum-product-subarray/)

---

## Problem
Find the contiguous subarray with the largest **product** and return that product.

```
Input:  [2, 3, -2, 4]     Output: 6    ([2,3])
Input:  [-2, 0, -1]       Output: 0
```

---

## Intuition
Unlike sum, a negative number flips the sign — so the **minimum** product can become the maximum when multiplied by a negative. Track both the running max and min product ending at the current index; swap them when the current number is negative.

Alternatively (and very clean): a prefix product and a suffix product traversal — the answer is the max value either reaches. Zeros reset both to 1.

---

## Code (C++) — prefix/suffix method
```cpp
int maxProduct(vector<int>& nums) {
    int n = nums.size();
    long long pre = 1, suf = 1, best = LLONG_MIN;
    for (int i = 0; i < n; i++) {
        if (pre == 0) pre = 1;             // reset after a zero
        if (suf == 0) suf = 1;
        pre *= nums[i];                    // prefix product
        suf *= nums[n - 1 - i];            // suffix product
        best = max({best, pre, suf});
    }
    return (int)best;
}
```

### Alternative: track max & min
```cpp
int maxProduct(vector<int>& nums) {
    long long mx = nums[0], mn = nums[0], best = nums[0];
    for (int i = 1; i < nums.size(); i++) {
        if (nums[i] < 0) swap(mx, mn);     // negative flips roles
        mx = max((long long)nums[i], mx * nums[i]);
        mn = min((long long)nums[i], mn * nums[i]);
        best = max(best, mx);
    }
    return (int)best;
}
```

---

## Why prefix AND suffix?
The max-product subarray ends at an array boundary except when split by zeros. Scanning products from both ends guarantees we capture it: an even count of negatives is handled by the prefix; an odd count is fixed by dropping a leading or trailing negative — which the opposite-direction scan covers.

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |
