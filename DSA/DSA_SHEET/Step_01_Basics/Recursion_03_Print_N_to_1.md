# Print N to 1 (Recursion)  🟢

**Reference:** [GeeksforGeeks — Print N to 1 using recursion](https://www.geeksforgeeks.org/print-numbers-1-n-using-indirect-recursion/)

---

## Problem
Print numbers from N down to 1 using recursion.

```
Input:  N = 5    Output: 5 4 3 2 1
```

---

## Intuition
- **Forward:** print `i`, then recurse with `i-1`.
- **Backtracking:** recurse from 1 up to N first, then print while unwinding — prints in decreasing order.

---

## Code (C++)
```cpp
// Approach 1: work on the way down
void printNto1(int i) {
    if (i < 1) return;
    cout << i << " ";
    printNto1(i - 1);
}

// Approach 2: backtracking — print while unwinding
void printNto1_back(int i, int n) {
    if (i > n) return;
    printNto1_back(i + 1, n);   // go up to n first
    cout << i << " ";           // print on the way back -> n..1
}

// call approach 2: printNto1_back(1, n);
```

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(N) recursion stack |
