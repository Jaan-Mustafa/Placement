# Evaluate Boolean Expression to True (Boolean Parenthesization)  🔴

**Reference:** https://www.geeksforgeeks.org/boolean-parenthesization-problem-dp-37/

---

## Problem

Given a boolean expression string made of operands `T` (true) and `F` (false), and operators `&` (AND), `|` (OR), `^` (XOR), count the number of ways to **parenthesize** it so that the whole expression evaluates to `true`. Answer is taken modulo `1000000007`.

```
expr = "T|T&F"
Parenthesizations: (T|(T&F)) = T, ((T|T)&F) = F
Ways to be true = 1
```

---

## Intuition

This is **partition DP**: we split on each operator and combine the truth counts of the two halves. For a sub-expression spanning operand indices `[i, j]`, pick each operator position `k` (operators sit at odd indices `i+1, i+3, ...`) as the **last operator evaluated**. We need both *true counts* and *false counts* of the halves to combine across `&`, `|`, `^`.

Define `f(i, j, isTrue)` = number of ways the sub-expression `[i, j]` evaluates to `isTrue`.

For each operator at position `k`, let `lT,lF` be left half true/false counts and `rT,rF` the right half. Then:
- `&` : true += `lT*rT`;  false += `lT*rF + lF*rT + lF*rF`
- `|` : true += `lT*rT + lT*rF + lF*rT`;  false += `lF*rF`
- `^` : true += `lT*rF + lF*rT`;  false += `lT*rT + lF*rF`

Base case (`i == j`): single operand, count 1 for matching `T`/`F`. Answer = `f(0, n-1, true)`.

All arithmetic is done **mod 1e9+7**.

---

## Code (C++)

```cpp
#include <string>
#include <vector>
using namespace std;

const long long MOD = 1000000007LL;

// returns ways for s[i..j] to be true (t) when isTrue=1, false otherwise
long long solve(int i, int j, bool isTrue, const string& s,
                vector<vector<vector<long long>>>& dp) {
    if (i > j) return 0;
    if (i == j) {                       // single operand
        if (isTrue) return s[i] == 'T';
        else        return s[i] == 'F';
    }
    if (dp[i][j][isTrue] != -1) return dp[i][j][isTrue];

    long long ways = 0;
    for (int k = i + 1; k < j; k += 2) {        // k = operator position
        long long lT = solve(i, k - 1, true,  s, dp);
        long long lF = solve(i, k - 1, false, s, dp);
        long long rT = solve(k + 1, j, true,  s, dp);
        long long rF = solve(k + 1, j, false, s, dp);

        if (s[k] == '&') {
            if (isTrue) ways += lT * rT;
            else        ways += lT * rF + lF * rT + lF * rF;
        } else if (s[k] == '|') {
            if (isTrue) ways += lT * rT + lT * rF + lF * rT;
            else        ways += lF * rF;
        } else { // '^'
            if (isTrue) ways += lT * rF + lF * rT;
            else        ways += lT * rT + lF * rF;
        }
        ways %= MOD;
    }
    return dp[i][j][isTrue] = ways % MOD;
}

int countWays(const string& s) {
    int n = s.size();
    // dp[i][j][b]; -1 = uncomputed
    vector<vector<vector<long long>>> dp(
        n, vector<vector<long long>>(n, vector<long long>(2, -1)));
    return (int)solve(0, n - 1, true, s, dp);
}
```

> Use `long long` for products before taking the modulo — `lT*rT` can overflow 32-bit int. A bottom-up version fills intervals by increasing length over the same recurrence.

---

## Complexity

| | |
|---|---|
| Time | O(n³) |
| Space | O(n²) (× 2 states) + recursion |

> ⚠️ Operators live at odd offsets, so the partition loop steps `k += 2`. Always reduce mod 1e9+7 after each multiply/add, and store both true and false counts — you cannot recover false from true in general.
