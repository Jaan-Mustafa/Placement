# Single Number II (Each Element Thrice Except One)  🟡

**Reference:** [LeetCode 137 — Single Number II](https://leetcode.com/problems/single-number-ii/)

---

## Problem
Every element appears **three** times except one, which appears once. Find it in O(N) time and O(1) space.

```
Input:  [2, 2, 3, 2]      Output: 3
Input:  [0, 1, 0, 1, 0, 1, 99]  Output: 99
```

---

## Intuition
**Bit-counting approach:** for each of the 32 bit positions, sum that bit across all numbers. Every thrice-appearing element contributes a multiple of 3 to each column, so `sum % 3` leaves only the bits belonging to the unique number.

A slicker **two-state machine** uses `ones` and `twos` to track bits seen once and twice, resetting both when a bit reaches a count of three.

---

## Code (C++)
```cpp
// Approach 1 — count bits per position (clear and general).
int singleNumber(vector<int>& nums) {
    int result = 0;
    for (int bit = 0; bit < 32; ++bit) {
        int count = 0;
        for (int v : nums)
            count += (v >> bit) & 1;     // tally bit `bit`
        if (count % 3)                    // leftover bit belongs to the answer
            result |= (1 << bit);
    }
    return result;
}

// Approach 2 — O(1) bitwise state machine.
int singleNumberFast(vector<int>& nums) {
    int ones = 0, twos = 0;
    for (int v : nums) {
        ones = (ones ^ v) & ~twos;   // add to "seen once", drop if in twos
        twos = (twos ^ v) & ~ones;   // add to "seen twice", drop if in ones
    }
    return ones;                     // bits seen exactly once
}
```

⚠️ In Approach 1, setting `result |= (1 << 31)` reconstructs negative numbers correctly under two's complement, so it works for negative inputs as well.

---

## Complexity
| | |
|---|---|
| Time | O(32·N) ≈ O(N) (Approach 1), O(N) (Approach 2) |
| Space | O(1) |
