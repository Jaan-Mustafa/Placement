# Armstrong Number  🟢

**Reference:** [GeeksforGeeks — Armstrong Numbers](https://www.geeksforgeeks.org/program-for-armstrong-numbers/)

---

## Problem
A number is an **Armstrong number** if the sum of each digit raised to the power of the number of digits equals the number itself. (For 3-digit numbers, sum of cubes of digits.)

```
153 = 1³ + 5³ + 3³ = 1 + 125 + 27 = 153  ->  true
371 = 3³ + 7³ + 1³ = 27 + 343 + 1 = 371  ->  true
123 = 1 + 8 + 27 = 36 != 123             ->  false
```

---

## Intuition
Count the digits `d`, then sum each digit raised to the power `d`, and compare with the original number.

---

## Code (C++)
```cpp
bool isArmstrong(int n) {
    int original = n;
    int digits = (int)log10(n) + 1;        // number of digits
    int sum = 0, temp = n;
    while (temp > 0) {
        int d = temp % 10;
        sum += pow(d, digits);             // digit ^ (#digits)
        temp /= 10;
    }
    return sum == original;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(log₁₀ N) |
| Space | O(1) |

> Be careful with `pow()` returning a `double` — for large numbers, compute the power with an integer loop to avoid floating-point rounding.
