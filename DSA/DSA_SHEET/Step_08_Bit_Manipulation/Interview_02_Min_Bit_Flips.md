# Minimum Bit Flips to Convert A to B  🟡

**Reference:** [LeetCode 2220 — Minimum Bit Flips to Convert Number](https://leetcode.com/problems/minimum-bit-flips-to-convert-number/)

---

## Problem
Find the minimum number of single-bit flips needed to turn integer `start` into `goal`.

```
start = 10 = 1010
goal  =  7 = 0111
diff  = 1101 → 3 flips
```

---

## Intuition
A bit must be flipped exactly when it **differs** between `start` and `goal`. `start ^ goal` has a 1 in every differing position, so the answer is simply the **number of set bits** in the XOR.

---

## Code (C++)
```cpp
int minBitFlips(int start, int goal) {
    int diff = start ^ goal;   // 1s mark positions that differ
    int flips = 0;
    while (diff) {
        diff &= (diff - 1);    // Kernighan: clear lowest set bit
        flips++;
    }
    return flips;
}

// Equivalent using the builtin popcount:
int minBitFlipsBuiltin(int start, int goal) {
    return __builtin_popcount((unsigned)(start ^ goal));
}
```

---

## Complexity
| | |
|---|---|
| Time | O(set bits in diff) ≤ O(32) |
| Space | O(1) |

> Same XOR-then-popcount idea powers Hamming distance (LeetCode 461).
