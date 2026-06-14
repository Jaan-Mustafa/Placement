# Check if a Number is a Power of 2  🟢

**Reference:** [LeetCode 231 — Power of Two](https://leetcode.com/problems/power-of-two/) · [GeeksforGeeks](https://www.geeksforgeeks.org/program-to-find-whether-a-no-is-power-of-two/)

---

## Problem
Given an integer `n`, return `true` if it is a power of two (`1, 2, 4, 8, 16, ...`), else `false`.

```
n = 16 → true  (2^4)
n = 18 → false
n = 1  → true  (2^0)
```

---

## Intuition
A power of two has **exactly one** set bit:
```
8 = 1000
7 = 0111   (8 - 1 flips that bit and sets all lower bits)
8 & 7 = 0000
```
Subtracting 1 turns the lone set bit off and all lower zeros on, so `n & (n - 1) == 0` iff there is at most one set bit. Guard `n > 0` to exclude 0 and negatives.

---

## Code (C++)
```cpp
bool isPowerOfTwo(int n) {
    // n > 0 ensures 0 and negatives are rejected.
    // n & (n-1) clears the lowest set bit; result 0 ⇒ only one set bit.
    return n > 0 && (n & (n - 1)) == 0;
}
```

⚠️ For `n = INT_MIN` (only the sign bit set), `n & (n-1)` is 0, but `n > 0` correctly rejects it.

---

## Complexity
| | |
|---|---|
| Time | O(1) |
| Space | O(1) |
