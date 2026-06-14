# Divide Two Integers Without * / %  🟡

**Reference:** [LeetCode 29 — Divide Two Integers](https://leetcode.com/problems/divide-two-integers/)

---

## Problem
Divide `dividend` by `divisor` returning the **truncated** quotient, without using multiplication, division, or modulo. Result must fit in a 32-bit signed integer.

```
dividend = 10, divisor = 3 → 3
dividend = 7,  divisor = -3 → -2  (truncated toward zero)
```

---

## Intuition
Division is repeated subtraction, but subtracting one divisor at a time is too slow. Instead use **bit shifting (doubling)**: find the largest `divisor << k` that still fits in the remaining dividend, subtract it, and add `1 << k` to the quotient. This is binary long division — each step handles one bit of the answer.

Work in `long long` (unsigned magnitudes) so `abs(INT_MIN)` doesn't overflow, and fix the sign at the end.

---

## Code (C++)
```cpp
int divide(int dividend, int divisor) {
    // Only overflow case: INT_MIN / -1 = INT_MAX+1 → clamp.
    if (dividend == INT_MIN && divisor == -1) return INT_MAX;

    // Result is negative iff signs differ.
    bool negative = (dividend < 0) ^ (divisor < 0);

    // Use 64-bit unsigned magnitudes to avoid abs(INT_MIN) overflow.
    long long a = llabs((long long)dividend);
    long long b = llabs((long long)divisor);

    long long quotient = 0;
    for (int k = 31; k >= 0; --k) {
        // If (b << k) fits into the remaining a, subtract it.
        if ((b << k) <= a) {
            a -= (b << k);
            quotient += (1LL << k);
        }
    }
    return negative ? (int)(-quotient) : (int)quotient;
}
```

⚠️ The single overflow case `INT_MIN / -1` must be special-cased; everything else fits once magnitudes are computed in `long long`.

---

## Complexity
| | |
|---|---|
| Time | O(log²(dividend)) — 32 shift levels, each a constant-bounded compare |
| Space | O(1) |
