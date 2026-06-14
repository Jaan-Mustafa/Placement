# Remove the Last (Lowest) Set Bit  🟢

**Reference:** [GeeksforGeeks — Turn off the rightmost set bit](https://www.geeksforgeeks.org/turn-off-the-rightmost-set-bit/)

---

## Problem
Given `n`, return the number with its **lowest set bit** cleared to 0 (all other bits unchanged).

```
n = 12 = 1100 → 1000 = 8   (lowest set bit was at position 2)
n = 7  = 0111 → 0110 = 6
```

---

## Intuition
Subtracting 1 flips the lowest set bit to 0 and turns every bit **below** it into 1:
```
n     = 1100
n - 1 = 1011
n & (n-1) = 1000   → lowest set bit removed
```
ANDing keeps only the bits that stayed 1 in both, which are exactly the bits **above** the lowest set bit. This is the same primitive used by Kernighan's set-bit count.

---

## Code (C++)
```cpp
int removeLastSetBit(int n) {
    return n & (n - 1);   // clears the rightmost 1 bit
}

// Related trick — isolate (keep only) the lowest set bit:
int lowestSetBit(int n) {
    return n & (-n);      // two's complement isolates the rightmost 1
}
```

⚠️ For `n = 0`, `n & (n-1)` is still 0 (no set bit to remove) — safe.

---

## Complexity
| | |
|---|---|
| Time | O(1) |
| Space | O(1) |
