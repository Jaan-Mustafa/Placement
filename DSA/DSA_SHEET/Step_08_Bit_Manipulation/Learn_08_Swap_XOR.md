# Swap Two Numbers Using XOR  🟢

**Reference:** [GeeksforGeeks — Swap two numbers without using a temporary variable](https://www.geeksforgeeks.org/swap-two-numbers-without-using-temporary-variable/)

---

## Problem
Swap the values of two integers `a` and `b` **without** using a temporary variable.

```
a = 5, b = 9   →   after swap:  a = 9, b = 5
```

---

## Intuition
XOR is its own inverse: `x ^ y ^ y = x`. Three XORs shuffle the values between the two slots:
```
a = a ^ b         // a now holds a^b
b = a ^ b         // = (a^b) ^ b = original a
a = a ^ b         // = (a^b) ^ (original a) = original b
```
No third storage location is needed because the combined `a^b` carries enough information to recover both originals.

---

## Code (C++)
```cpp
void swapXor(int &a, int &b) {
    if (&a == &b) return;   // guard: aliasing the same variable would zero it
    a = a ^ b;              // a = a^b
    b = a ^ b;              // b = (a^b)^b = a
    a = a ^ b;              // a = (a^b)^a = b
}
```

⚠️ If `a` and `b` are the **same** variable (`swapXor(x, x)`), the chain sets it to 0 — hence the aliasing guard. In real code, prefer `std::swap`; this trick is mainly a classic interview question.

---

## Complexity
| | |
|---|---|
| Time | O(1) |
| Space | O(1) — no temporary |
