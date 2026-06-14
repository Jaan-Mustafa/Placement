# Print 1 to N (Recursion)  🟢

**Reference:** [GeeksforGeeks — Print 1 to N using recursion](https://www.geeksforgeeks.org/print-1-to-n-without-using-loops/)

---

## Problem
Print numbers from 1 to N using recursion.

```
Input:  N = 5    Output: 1 2 3 4 5
```

---

## Intuition
Two ways:
1. **Forward (work then recurse):** print `i`, then recurse with `i+1`.
2. **Backtracking (recurse then work):** recurse down to 1 first, then print while unwinding — clever because it prints in increasing order even though calls go from N downward.

---

## Code (C++)
```cpp
// Approach 1: work on the way down
void print1toN(int i, int n) {
    if (i > n) return;
    cout << i << " ";
    print1toN(i + 1, n);
}

// Approach 2: backtracking — recurse first, print while unwinding
void print1toN_back(int n) {
    if (n < 1) return;
    print1toN_back(n - 1);     // go all the way to 1 first
    cout << n << " ";          // then print on the way back up
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(N) recursion stack |
