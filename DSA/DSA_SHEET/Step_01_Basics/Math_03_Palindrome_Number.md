# Palindrome Number  🟢

**Reference:** [LeetCode 9 — Palindrome Number](https://leetcode.com/problems/palindrome-number/)

---

## Problem
Given an integer N, return `true` if it reads the same forwards and backwards. Negative numbers are **not** palindromes (because of the `-` sign).

```
Input:  121    Output: true
Input:  -121   Output: false
Input:  10     Output: false
```

---

## Intuition
Reverse the number and check if it equals the original. (Optimization: reverse only the second half and compare to the first half — avoids overflow entirely.)

---

## Code (C++)
```cpp
bool isPalindrome(int x) {
    if (x < 0) return false;               // negatives never palindromes

    long long rev = 0, original = x;
    while (x > 0) {
        rev = rev * 10 + x % 10;
        x /= 10;
    }
    return rev == original;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(log₁₀ N) |
| Space | O(1) |

> Using `long long` for `rev` sidesteps overflow. The half-reversal trick (`while (x > rev) { rev = rev*10 + x%10; x /= 10; }` then compare `x == rev || x == rev/10`) keeps everything in `int`.
