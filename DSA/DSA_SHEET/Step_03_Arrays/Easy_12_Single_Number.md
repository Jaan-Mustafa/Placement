# Number Appearing Once (Rest Twice)  🟢

**Reference:** [LeetCode 136 — Single Number](https://leetcode.com/problems/single-number/)

---

## Problem
Every element appears **twice** except one, which appears once. Find that single element in O(N) time and O(1) space.

```
Input:  [4, 1, 2, 1, 2]    Output: 4
```

---

## Intuition
**XOR** has two key properties:
- `x ^ x = 0` (a number XOR itself cancels)
- `x ^ 0 = x`

XOR-ing every element together cancels all the pairs, leaving only the unique number.

---

## Code (C++)
```cpp
int singleNumber(vector<int>& nums) {
    int x = 0;
    for (int v : nums) x ^= v;     // pairs cancel out
    return x;
}
```

---

## Why it works
`4 ^ 1 ^ 2 ^ 1 ^ 2` = `4 ^ (1^1) ^ (2^2)` = `4 ^ 0 ^ 0` = `4`. Order doesn't matter (XOR is commutative & associative).

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> Related: "appears thrice except one" (LeetCode 137) and "two appear once" (260) — Step 8 Bit Manipulation.
