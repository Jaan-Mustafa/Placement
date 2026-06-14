# Sieve of Eratosthenes  🟡

**Reference:** [LeetCode 204 — Count Primes](https://leetcode.com/problems/count-primes/) · [GeeksforGeeks — Sieve of Eratosthenes](https://www.geeksforgeeks.org/sieve-of-eratosthenes/)

---

## Problem
Find all prime numbers up to `n` (or count primes `< n`).

```
n = 10 → primes: 2 3 5 7   (count = 4)
```

---

## Intuition
Start assuming every number is prime. Walk from the first prime `2` upward; whenever you hit a number still marked prime, **strike out all its multiples** — they cannot be prime. Begin striking at `p * p` (smaller multiples were already removed by smaller primes), and only sieve `p` up to `sqrt(n)`.

---

## Code (C++)
```cpp
vector<int> sieve(int n) {
    vector<char> isPrime(n + 1, true);     // char keeps the table compact
    isPrime[0] = isPrime[1] = false;

    for (long long p = 2; p * p <= n; ++p) {
        if (isPrime[p]) {
            // Start at p*p; all smaller multiples already marked.
            for (long long m = p * p; m <= n; m += p)
                isPrime[m] = false;
        }
    }

    vector<int> primes;
    for (int i = 2; i <= n; ++i)
        if (isPrime[i]) primes.push_back(i);
    return primes;
}
```

⚠️ Use `long long` for `p * p` and the multiple index `m` so they don't overflow when `n` is near `INT_MAX`.

---

## Complexity
| | |
|---|---|
| Time | O(n log log n) |
| Space | O(n) |
