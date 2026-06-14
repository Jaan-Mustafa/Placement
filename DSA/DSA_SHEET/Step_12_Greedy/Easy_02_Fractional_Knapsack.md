# Fractional Knapsack  🟡

**Reference:** [GeeksforGeeks — Fractional Knapsack](https://www.geeksforgeeks.org/fractional-knapsack-problem/)

---

## Problem
Given `N` items, each with a `value` and `weight`, and a knapsack of capacity `W`, maximize total value. Unlike 0/1 knapsack, you **may take fractions** of an item.

```
Input:  W = 50, items = [(60,10),(100,20),(120,30)]   // (value,weight)
Output: 240.0   // take items 1,2 fully (160, wt 30) + 2/3 of item 3 (80)
```

---

## Intuition
**Sort items by value-to-weight ratio (descending).** Greedily take whole items with the best ratio first; when an item doesn't fully fit, take the fraction that fills the remaining capacity. Because fractions are allowed, the exchange argument is exact: replacing any part of the load with a higher-ratio unit of weight never decreases value.

---

## Code (C++)
```cpp
struct Item { int value, weight; };

double fractionalKnapsack(int W, vector<Item>& items) {
    // sort by value/weight descending (use cross-multiplication to avoid float error)
    sort(items.begin(), items.end(), [](const Item& a, const Item& b) {
        return (long long)a.value * b.weight > (long long)b.value * a.weight;
    });

    double total = 0.0;
    for (auto& it : items) {
        if (W >= it.weight) {          // whole item fits
            W -= it.weight;
            total += it.value;
        } else {                       // take fraction to fill remaining capacity
            total += it.value * ((double)W / it.weight);
            break;                     // knapsack is now full
        }
    }
    return total;
}
```

---

## Dry run
Ratios: 6, 5, 4. Take (60,10) → W=40; take (100,20) → W=20; item (120,30) doesn't fit → take 20/30 = 80. Total **240.0**.

## Complexity
| | |
|---|---|
| Time | O(N log N) (sorting dominates) |
| Space | O(1) extra |

> ⚠️ The greedy ratio strategy works ONLY because fractions are allowed. For 0/1 knapsack (no fractions) you need DP — greedy fails there.
