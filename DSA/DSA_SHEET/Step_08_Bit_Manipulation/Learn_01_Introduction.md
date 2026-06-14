# Introduction to Bit Manipulation  🟢

**Reference:** [GeeksforGeeks — Bitwise Operators in C/C++](https://www.geeksforgeeks.org/bitwise-operators-in-c-cpp/)

---

## Problem
Understand the core bitwise operators and how integers are stored in binary (two's complement). These are the building blocks for every bit-manipulation problem.

```
5  = 0101 (binary)
3  = 0011

5 & 3 = 0001 = 1     (AND)
5 | 3 = 0111 = 7     (OR)
5 ^ 3 = 0110 = 6     (XOR)
~5    = ...1010 = -6 (NOT, two's complement)
5 << 1 = 1010 = 10   (left shift  → multiply by 2)
5 >> 1 = 0010 = 2    (right shift → divide by 2)
```

---

## Intuition
Every integer is a fixed-width sequence of bits. Bitwise operators act on each bit position independently:
- `&` keeps a bit only if **both** are 1 → masking.
- `|` sets a bit if **either** is 1 → turning bits on.
- `^` sets a bit if the bits **differ** → toggling / pairing.
- `~` flips every bit.
- `<<k` / `>>k` multiply / divide by `2^k`.

Negative numbers use **two's complement**: the most significant bit is the sign bit, and `~x == -x - 1`.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int a = 5, b = 3;          // 0101 and 0011
    cout << (a & b) << "\n";   // 1  → common bits
    cout << (a | b) << "\n";   // 7  → union of bits
    cout << (a ^ b) << "\n";   // 6  → differing bits
    cout << (~a)    << "\n";   // -6 → two's complement
    cout << (a << 1)<< "\n";   // 10 → a * 2
    cout << (a >> 1)<< "\n";   // 2  → a / 2
    return 0;
}
```

⚠️ Shifting a signed negative number right is implementation-defined; prefer `unsigned`/`long long` when bit-twiddling large or signed-sensitive values. Also `1 << 31` overflows a 32-bit `int` — use `1u << 31` or `1LL << 31`.

---

## Complexity
| | |
|---|---|
| Time | O(1) per operation |
| Space | O(1) |

> All other bit tricks (set bit, power of two, etc.) are just combinations of these six operators.
