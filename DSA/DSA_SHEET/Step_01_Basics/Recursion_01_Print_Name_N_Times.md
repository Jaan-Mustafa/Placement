# Print Name N Times (Recursion)  🟢

**Reference:** [takeUforward — Recursion basics](https://takeuforward.org/recursion/introduction-to-recursion-print-something-n-times/)

---

## Problem
Print a given string `N` times using recursion (no loops).

```
Input:  name = "Coder", N = 3
Output: Coder
        Coder
        Coder
```

---

## Intuition
A recursive function calls itself with a parameter moving toward a **base case**. Here we print once, then recurse with `i+1` until `i` reaches `N`.

---

## Code (C++)
```cpp
void printName(int i, int n, string name) {
    if (i > n) return;             // base case: stop
    cout << name << "\n";          // work
    printName(i + 1, n, name);     // recursive call toward base case
}

// call: printName(1, n, "Coder");
```

---

## Recursion tree (N=3)
```
printName(1) -> print -> printName(2) -> print -> printName(3) -> print -> printName(4) -> return
```

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(N) recursion stack |

> Every recursion needs a **base case** (to stop) and a **recursive case** that moves toward it — without the base case you get a stack overflow.
