# XOR of Numbers in Range [L, R]  🟡

**Reference:** [LeetCode 1486 — XOR Operation in an Array](https://leetcode.com/problems/xor-operation-in-an-array/) · [GeeksforGeeks — XOR of numbers from L to R](https://www.geeksforgeeks.org/find-xor-of-numbers-from-the-range-l-r/)

---

## Problem
Compute `L ^ (L+1) ^ ... ^ R` efficiently (no loop).

```
L = 4, R = 8 → 4^5^6^7^8 = 8
```

---

## Intuition
Define `f(n) = 0 ^ 1 ^ ... ^ n` (prefix XOR). Then by the cancellation property:
```
XOR(L..R) = f(R) ^ f(L-1)
```
`f(n)` has a beautiful closed form based on `n % 4`, because XOR of any four consecutive integers starting at a multiple of 4 is 0:
```
n % 4 == 0 → f(n) = n
n % 4 == 1 → f(n) = 1
n % 4 == 2 → f(n) = n + 1
n % 4 == 3 → f(n) = 0
```

---

## Code (C++)
```cpp
// Prefix XOR 0..n in O(1).
int prefixXor(int n) {
    switch (n & 3) {        // n % 4
        case 0: return n;
        case 1: return 1;
        case 2: return n + 1;
        default: return 0;  // case 3
    }
}

// XOR of all integers in [L, R].
int xorRange(int L, int R) {
    return prefixXor(R) ^ prefixXor(L - 1);
}
```

⚠️ Watch the `L - 1` boundary when `L = 0`: `prefixXor(-1)` is meaningless, but for non-negative ranges callers should keep `L >= 1`, or special-case `L == 0` to just return `prefixXor(R)`.

---

## Complexity
| | |
|---|---|
| Time | O(1) |
| Space | O(1) |
