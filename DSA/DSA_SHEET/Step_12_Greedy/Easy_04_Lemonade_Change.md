# Lemonade Change  🟢

**Reference:** [LeetCode 860 — Lemonade Change](https://leetcode.com/problems/lemonade-change/)

---

## Problem
Each lemonade costs `$5`. Customers pay with a `$5`, `$10`, or `$20` bill, in order. You start with no change. Return `true` if you can give every customer correct change.

```
Input:  bills = [5,5,5,10,20]   Output: true
Input:  bills = [5,5,10,10,20]  Output: false
```

---

## Intuition
Track counts of `$5` and `$10` bills on hand. The greedy rule appears when paying change for a `$20` (owe `$15`): **prefer giving one `$10` + one `$5` over three `$5`s**, because `$5` bills are more versatile (a `$10` payer can only be changed with a `$5`). Hoard the flexible small bills.

---

## Code (C++)
```cpp
bool lemonadeChange(vector<int>& bills) {
    int five = 0, ten = 0;             // $20 bills are never used as change
    for (int b : bills) {
        if (b == 5) {
            five++;                    // no change needed
        } else if (b == 10) {
            if (five == 0) return false;
            five--; ten++;             // give one $5 back
        } else {                       // b == 20, owe $15
            if (ten > 0 && five > 0) { // prefer $10 + $5
                ten--; five--;
            } else if (five >= 3) {    // fall back to three $5s
                five -= 3;
            } else {
                return false;          // can't make change
            }
        }
    }
    return true;
}
```

---

## Dry run
`[5,5,10,10,20]`: after two $5 → five=2; $10 → five=1,ten=1; $10 → five=0,ten=2; $20 owes $15 but no $5 and only two $10 → **false**.

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |

> ⚠️ For the `$20` case, always try `$10 + $5` first. Greedily spending three `$5`s when a `$10` was available can strand a future `$10` customer.
