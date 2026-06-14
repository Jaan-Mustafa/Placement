# Check if a Number is Odd or Even  🟢

**Reference:** [GeeksforGeeks — Check whether a number is odd or even using bitwise operator](https://www.geeksforgeeks.org/check-whether-given-number-even-odd/)

---

## Problem
Determine whether an integer `n` is odd or even **without** using the modulo (`%`) operator.

```
n = 10 → even
n = 7  → odd
```

---

## Intuition
The least-significant bit (bit 0) holds the value `2^0 = 1`. A number is **odd** exactly when this bit is set, because every higher bit contributes an even amount (`2, 4, 8, ...`). So `n & 1` equals the parity: `1` → odd, `0` → even.

---

## Code (C++)
```cpp
// true if n is odd, false if even.
bool isOdd(int n) {
    return (n & 1) == 1;   // inspect only the lowest bit
}
```

⚠️ This works for negative numbers too in two's complement: `-3 & 1 == 1` (odd), unlike `-3 % 2` which is `-1` in C++ and can trip up sign-based checks.

---

## Complexity
| | |
|---|---|
| Time | O(1) |
| Space | O(1) |
