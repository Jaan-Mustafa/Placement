# Sum of First N Numbers (Recursion)  🟢

**Reference:** [GeeksforGeeks — Sum of natural numbers using recursion](https://www.geeksforgeeks.org/sum-of-natural-numbers-using-recursion/)

---

## Problem
Find the sum 1 + 2 + ... + N using recursion.

```
Input:  N = 5    Output: 15
```

---

## Intuition
- **Parameterized way:** carry a running `sum` accumulator down the calls.
- **Functional way:** `sum(N) = N + sum(N-1)`, with `sum(0) = 0`.

---

## Code (C++)
```cpp
// Functional: return value built up while unwinding
int sumN(int n) {
    if (n == 0) return 0;          // base case
    return n + sumN(n - 1);        // N + sum of rest
}

// Parameterized: accumulator passed down
void sumN_param(int n, int sum) {
    if (n == 0) { cout << sum; return; }
    sumN_param(n - 1, sum + n);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(N) recursion stack |

> Closed-form formula (no recursion): `N * (N + 1) / 2` in O(1).
