# Reverse a Number  🟢

**Reference:** [LeetCode 7 — Reverse Integer](https://leetcode.com/problems/reverse-integer/) · [GeeksforGeeks](https://www.geeksforgeeks.org/write-a-program-to-reverse-digits-of-a-number/)

---

## Problem
Given an integer N, return its digits reversed. (LeetCode variant: return 0 if the reversed value overflows a 32-bit signed integer.)

```
Input:  N = 12345     Output: 54321
Input:  N = 120       Output: 21
```

---

## Intuition
Peel off the last digit with `% 10`, append it to the result by doing `rev = rev * 10 + digit`, then drop it from N with `/= 10`. Repeat until N is 0. Guard against 32-bit overflow before each multiply.

---

## Code (C++)
```cpp
int reverse(int x) {
    int rev = 0;
    while (x != 0) {
        int digit = x % 10;            // last digit (sign preserved in C++)
        // overflow check for 32-bit signed int
        if (rev > INT_MAX / 10 || rev < INT_MIN / 10) return 0;
        rev = rev * 10 + digit;
        x /= 10;
    }
    return rev;
}
```

---

## Notes
- In C++, `-123 % 10 == -3`, so negatives are handled automatically.
- Without the overflow guard, large inputs (e.g. 1534236469) produce undefined behavior.

## Complexity
| | |
|---|---|
| Time | O(log₁₀ N) |
| Space | O(1) |
