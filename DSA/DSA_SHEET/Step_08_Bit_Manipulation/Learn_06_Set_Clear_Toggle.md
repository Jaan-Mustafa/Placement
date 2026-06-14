# Set / Clear / Toggle the i-th Bit  🟢

**Reference:** [GeeksforGeeks — Set, Clear and Toggle a given bit of a number](https://www.geeksforgeeks.org/modify-bit-given-position/)

---

## Problem
Given a number `n` and position `i`, produce a new number with the i-th bit **set** to 1, **cleared** to 0, or **toggled**.

```
n = 1010 (10), i = 0
set    → 1011 (11)
clear  → 1010 (10, already 0)
toggle → 1011 (11)
```

---

## Intuition
Build the single-bit mask `1 << i`, then:
- **Set:** OR with the mask → forces bit `i` to 1, leaves others unchanged.
- **Clear:** AND with the inverted mask `~(1 << i)` → forces bit `i` to 0.
- **Toggle:** XOR with the mask → flips bit `i` (1↔0), since `x ^ 1 = !x` and `x ^ 0 = x`.

---

## Code (C++)
```cpp
int setBit(int n, int i)    { return n | (1 << i); }    // force bit i to 1
int clearBit(int n, int i)  { return n & ~(1 << i); }   // force bit i to 0
int toggleBit(int n, int i) { return n ^ (1 << i); }    // flip bit i
```

⚠️ Use `1u << i` (or `1LL << i`) if `i` may reach the sign bit (31 for `int`); `1 << 31` is undefined behaviour on signed `int`.

---

## Complexity
| | |
|---|---|
| Time | O(1) |
| Space | O(1) |
