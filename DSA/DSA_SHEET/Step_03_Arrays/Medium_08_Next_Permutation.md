# Next Permutation  🟡

**Reference:** [LeetCode 31 — Next Permutation](https://leetcode.com/problems/next-permutation/)

---

## Problem
Rearrange the array into the **next lexicographically greater** permutation, in place. If it's the largest (descending), wrap to the smallest (ascending).

```
Input:  [1, 2, 3]    Output: [1, 3, 2]
Input:  [3, 2, 1]    Output: [1, 2, 3]
Input:  [1, 1, 5]    Output: [1, 5, 1]
```

---

## Intuition (3 steps)
1. **Find the pivot:** scan from the right for the first index `i` where `arr[i] < arr[i+1]`. The suffix after `i` is non-increasing (already the largest arrangement).
2. **Find swap target:** if a pivot exists, scan from the right for the first element greater than `arr[i]`, and swap them.
3. **Reverse the suffix** after `i` to make it ascending (smallest). If no pivot existed, just reverse the whole array.

---

## Code (C++)
```cpp
void nextPermutation(vector<int>& a) {
    int n = a.size(), i = n - 2;
    while (i >= 0 && a[i] >= a[i + 1]) i--;     // 1. find pivot

    if (i >= 0) {                               // 2. find & swap target
        int j = n - 1;
        while (a[j] <= a[i]) j--;
        swap(a[i], a[j]);
    }
    reverse(a.begin() + i + 1, a.end());        // 3. reverse suffix
}
```

---

## Dry run
`[1,3,5,4,2]`: pivot i=1 (3<5); target j=3 (4>3) → swap → `[1,4,5,3,2]`; reverse suffix → `[1,4,2,3,5]`.

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> The STL one-liner `next_permutation(a.begin(), a.end())` does exactly this — interviews usually want the manual version.
