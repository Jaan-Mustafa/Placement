# Print All Divisors  🟢

**Reference:** [GeeksforGeeks — Find all divisors of a natural number](https://www.geeksforgeeks.org/find-divisors-natural-number-set-1/)

---

## Problem
Given an integer N, print all its divisors (numbers that divide N exactly).

```
Input:  N = 36
Output: 1 2 3 4 6 9 12 18 36
```

---

## Intuition
A naive loop from 1 to N is O(N). Better: iterate only up to **√N**. Whenever `i` divides N, both `i` **and** `N/i` are divisors. Collect both, then sort. This cuts the work to O(√N).

---

## Code (C++)
```cpp
vector<int> printDivisors(int n) {
    vector<int> divisors;
    for (int i = 1; i * i <= n; i++) {     // i up to sqrt(n)
        if (n % i == 0) {
            divisors.push_back(i);
            if (i != n / i)                // avoid duplicating the sqrt
                divisors.push_back(n / i);
        }
    }
    sort(divisors.begin(), divisors.end());
    return divisors;
}
```

---

## Why `i != n/i`?
For a perfect square like 36, when `i = 6`, `n/i = 6` too — without the check we'd add 6 twice.

## Complexity
| | |
|---|---|
| Time | O(√N) to collect + O(D log D) to sort (D = #divisors) |
| Space | O(D) |
