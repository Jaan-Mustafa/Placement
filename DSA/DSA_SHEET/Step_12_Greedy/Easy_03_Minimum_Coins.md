# Minimum Number of Coins  🟢

**Reference:** [GeeksforGeeks — Find minimum number of coins](https://www.geeksforgeeks.org/find-minimum-number-of-coins-that-make-a-change/)

---

## Problem
Given an **canonical** currency system (e.g. Indian `[1,2,5,10,20,50,100,200,500,1000]`), find the minimum number of coins/notes that sum to a value `V`.

```
Input:  V = 70
Output: 2   (50 + 20)
```

---

## Intuition
For a canonical denomination system, **always pick the largest coin that does not exceed the remaining amount.** Take as many of it as possible, subtract, and repeat. The greedy choice is optimal for standard currency because each larger denomination is a multiple/combination that dominates smaller ones.

---

## Code (C++)
```cpp
int minCoins(int V) {
    vector<int> coins = {1,2,5,10,20,50,100,200,500,1000};
    int count = 0;
    // iterate from largest to smallest denomination
    for (int i = coins.size() - 1; i >= 0; i--) {
        while (V >= coins[i]) {        // take as many of this coin as fit
            V -= coins[i];
            count++;
        }
    }
    return count;                      // V is now 0
}
```

---

## Dry run
`V=70`: largest ≤70 is 50 → V=20, count=1; next is 20 → V=0, count=2. Answer **2**.

## Complexity
| | |
|---|---|
| Time | O(V) worst case (or O(D + answer)) |
| Space | O(1) |

> ⚠️ Greedy is correct only for **canonical** systems. For arbitrary coins like `[1,3,4]` and `V=6`, greedy gives `4+1+1=3` but optimal is `3+3=2` — use DP instead.
