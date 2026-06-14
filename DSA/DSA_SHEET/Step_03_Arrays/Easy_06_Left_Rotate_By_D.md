# Left Rotate Array by D Places  🟢

**Reference:** [LeetCode 189 — Rotate Array](https://leetcode.com/problems/rotate-array/) · [GeeksforGeeks](https://www.geeksforgeeks.org/array-rotation/)

---

## Problem
Rotate an array to the left by `D` positions (in place, O(1) extra space).

```
Input:  [1, 2, 3, 4, 5, 6, 7], D = 2
Output: [3, 4, 5, 6, 7, 1, 2]
```

---

## Intuition: the reversal algorithm
A rotation by D can be done with three reversals:
1. Reverse the first `D` elements.
2. Reverse the remaining `N-D` elements.
3. Reverse the **whole** array.

This swaps the two blocks into their rotated positions without extra space.

---

## Code (C++)
```cpp
void leftRotate(vector<int>& arr, int d) {
    int n = arr.size();
    d = d % n;                                   // handle d > n
    reverse(arr.begin(), arr.begin() + d);       // reverse first d
    reverse(arr.begin() + d, arr.end());         // reverse rest
    reverse(arr.begin(), arr.end());             // reverse whole
}
```
> For **right** rotation by d (LeetCode 189), either rotate left by `n-d`, or reverse whole → reverse first d → reverse rest.

---

## Why it works (D=2, [1..7])
- Reverse first 2: `[2,1,3,4,5,6,7]`
- Reverse rest:    `[2,1,7,6,5,4,3]`
- Reverse all:     `[3,4,5,6,7,1,2]` ✅

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> The naive approach (use a temp array of size D) is also O(N) time but O(D) space. The reversal trick is the in-place favorite.
