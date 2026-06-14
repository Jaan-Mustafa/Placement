# Count Digits  🟢

**Reference:** [GeeksforGeeks — Count digits in a number](https://www.geeksforgeeks.org/program-count-digits-integer-3-different-methods/)

---

## Problem
Given an integer N, count the number of digits in it.

```
Input:  N = 12345     Output: 5
Input:  N = 7         Output: 1
```

---

## Intuition
Repeatedly divide the number by 10; each division removes one digit. Count how many divisions until the number becomes 0. (Or use the formula `log10(N) + 1`.)

---

## Code (C++)
```cpp
int countDigits(int n) {
    if (n == 0) return 1;
    int count = 0;
    n = abs(n);
    while (n > 0) {
        n /= 10;          // drop the last digit
        count++;
    }
    return count;
}

// O(1) formula version
int countDigitsLog(int n) {
    if (n == 0) return 1;
    return (int)log10(abs(n)) + 1;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(log₁₀ N) (loop) or O(1) (formula) |
| Space | O(1) |
