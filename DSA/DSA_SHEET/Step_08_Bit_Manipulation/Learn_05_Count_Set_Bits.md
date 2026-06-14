# Count the Number of Set Bits  🟢

**Reference:** [LeetCode 191 — Number of 1 Bits](https://leetcode.com/problems/number-of-1-bits/) · [GeeksforGeeks](https://www.geeksforgeeks.org/count-set-bits-in-an-integer/)

---

## Problem
Count how many bits are set (`1`) in the binary representation of `n` — its **population count**.

```
n = 13 = 1101 → 3 set bits
n = 7  = 0111 → 3 set bits
```

---

## Intuition
**Brian Kernighan's trick:** `n & (n - 1)` removes the **lowest set bit** of `n` in one step. Repeating until `n` becomes 0 runs exactly once per set bit, so it is O(number of set bits) instead of O(total bits).

---

## Code (C++)
```cpp
int countSetBits(int n) {
    unsigned u = n;          // treat as unsigned to handle negatives safely
    int count = 0;
    while (u) {
        u &= (u - 1);        // drop the lowest set bit
        count++;
    }
    return count;
}

// Idiomatic one-liner using the compiler builtin:
int countSetBitsBuiltin(int n) {
    return __builtin_popcount((unsigned)n);
}
```

⚠️ Use `unsigned` (or `__builtin_popcountll` for 64-bit). Looping on a signed negative with arithmetic right shift can spin forever as sign bits keep filling in.

---

## Complexity
| | |
|---|---|
| Time | O(set bits) ≤ O(32) |
| Space | O(1) |
