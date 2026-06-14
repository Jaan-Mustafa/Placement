# GCD / HCF  🟢

**Reference:** [GeeksforGeeks — GCD of two numbers](https://www.geeksforgeeks.org/c-program-find-gcd-hcf-two-numbers/) · [LeetCode 1071 — GCD of Strings](https://leetcode.com/problems/greatest-common-divisor-of-strings/)

---

## Problem
Find the **Greatest Common Divisor** (a.k.a. Highest Common Factor) of two integers a and b — the largest number that divides both.

```
Input:  a = 12, b = 18    Output: 6
Input:  a = 20, b = 5     Output: 5
```

---

## Intuition
**Euclidean algorithm:** `gcd(a, b) = gcd(b, a % b)`, and `gcd(a, 0) = a`. Repeatedly replace the larger number with the remainder until one becomes 0. This is far faster than checking every number from min(a,b) down to 1.

---

## Code (C++)
```cpp
int gcd(int a, int b) {
    while (b != 0) {
        int rem = a % b;
        a = b;
        b = rem;
    }
    return a;
}

// recursive form
int gcdRec(int a, int b) {
    return b == 0 ? a : gcdRec(b, a % b);
}

// C++17 built-in: __gcd(a, b) or std::gcd(a, b)
// LCM:  lcm(a,b) = a / gcd(a,b) * b   (divide first to avoid overflow)
```

---

## Complexity
| | |
|---|---|
| Time | O(log(min(a, b))) |
| Space | O(1) iterative / O(log) recursive |
