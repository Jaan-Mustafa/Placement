# Check if the i-th Bit is Set  🟢

**Reference:** [GeeksforGeeks — Check whether the K-th bit is set or not](https://www.geeksforgeeks.org/check-whether-k-th-bit-set-not/)

---

## Problem
Given a number `n` and a bit position `i` (0-indexed from the right), determine whether the i-th bit is `1` (set) or `0`.

```
n = 13 = 1101
i = 0 → set (1)
i = 1 → not set (0)
i = 2 → set (1)
```

---

## Intuition
Two equivalent tricks:
- **Left-shift a mask:** `(n & (1 << i))` is non-zero exactly when bit `i` is set, because the mask `1 << i` has a single 1 at position `i`.
- **Right-shift the number:** `((n >> i) & 1)` slides bit `i` into the lowest position, then masks it.

---

## Code (C++)
```cpp
// Returns true if the i-th bit of n is set.
bool isBitSet(int n, int i) {
    return (n & (1 << i)) != 0;   // mask out everything but bit i
}

// Alternative: bring bit i down to position 0, then read it.
bool isBitSetAlt(int n, int i) {
    return ((n >> i) & 1) == 1;
}
```

⚠️ If `i` can be 31 on a 32-bit `int`, `1 << 31` overflows. Use `1u << i` or operate on `long long` with `1LL << i`.

---

## Complexity
| | |
|---|---|
| Time | O(1) |
| Space | O(1) |
