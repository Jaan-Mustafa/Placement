# Print All Divisors  🟢

**Reference:** [GeeksforGeeks — Find all divisors of a natural number](https://www.geeksforgeeks.org/find-all-divisors-of-a-natural-number-set-1/)

---

## Problem
Print all divisors of a positive integer `n` in sorted order.

```
n = 36 → 1 2 3 4 6 9 12 18 36
```

---

## Intuition
Divisors come in **pairs**: if `i` divides `n`, then so does `n / i`. One of each pair is `≤ sqrt(n)`, so we only need to scan up to `sqrt(n)`. For each divisor `i` found, record both `i` and `n / i`. Collect, then sort (avoid double-counting the perfect-square root `i == n/i`).

---

## Code (C++)
```cpp
vector<int> allDivisors(int n) {
    vector<int> divs;
    // Scan only up to sqrt(n); i*i avoids floating-point sqrt.
    for (long long i = 1; i * i <= n; ++i) {
        if (n % i == 0) {
            divs.push_back((int)i);
            if (i != n / i)            // skip duplicate for perfect squares
                divs.push_back((int)(n / i));
        }
    }
    sort(divs.begin(), divs.end());    // pairs come out unsorted
    return divs;
}
```

⚠️ Use `i * i <= n` with `long long i` rather than `i <= sqrt(n)` to dodge floating-point precision errors at boundaries.

---

## Complexity
| | |
|---|---|
| Time | O(√n) to find + O(d log d) to sort (d = number of divisors) |
| Space | O(d) |
