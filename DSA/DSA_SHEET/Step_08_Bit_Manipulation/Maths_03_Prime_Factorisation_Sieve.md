# Prime Factorisation Using a Sieve (SPF)  🟡

**Reference:** [GeeksforGeeks — Prime factorization using Smallest Prime Factor](https://www.geeksforgeeks.org/prime-factorization-using-sieve-olog-n-multiple-queries/)

---

## Problem
Answer **many** queries of the form "give the prime factorisation of `x`" quickly, for `x` up to some limit `N`.

```
x = 60 → 2 × 2 × 3 × 5
x = 84 → 2 × 2 × 3 × 7
```

---

## Intuition
Precompute the **Smallest Prime Factor (SPF)** of every number up to `N` with a modified sieve. Then factoring any `x` is just: divide out `spf[x]` repeatedly, jumping to `x / spf[x]` each time — each step peels off one prime in O(1), so a query costs O(log x) instead of O(√x).

---

## Code (C++)
```cpp
const int N = 1000000;
int spf[N + 1];                       // spf[i] = smallest prime factor of i

void buildSPF() {
    for (int i = 2; i <= N; ++i) spf[i] = i;     // init: each is its own SPF
    for (long long i = 2; i * i <= N; ++i) {
        if (spf[i] == i) {                        // i is prime
            for (long long j = i * i; j <= N; j += i)
                if (spf[j] == j) spf[j] = (int)i; // set SPF if not already set
        }
    }
}

vector<int> factorize(int x) {
    vector<int> factors;
    while (x > 1) {
        factors.push_back(spf[x]);    // smallest prime dividing x
        x /= spf[x];                  // remove it, repeat
    }
    return factors;                   // primes in non-decreasing order
}
```

⚠️ Call `buildSPF()` once before any query. `spf` is built so that composites point at their smallest prime; primes keep `spf[p] == p`, which terminates the loop correctly.

---

## Complexity
| | |
|---|---|
| Time | Build O(N log log N); per query O(log x) |
| Space | O(N) for the SPF table |
