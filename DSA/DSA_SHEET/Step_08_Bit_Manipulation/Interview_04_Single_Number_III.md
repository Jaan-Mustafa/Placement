# Single Number III (Two Elements Appear Once)  🟡

**Reference:** [LeetCode 260 — Single Number III](https://leetcode.com/problems/single-number-iii/)

---

## Problem
Exactly **two** elements appear once; every other element appears twice. Return the two unique numbers (any order) in O(N) time and O(1) space.

```
Input:  [1, 2, 1, 3, 2, 5]   Output: [3, 5]
```

---

## Intuition
XOR-ing the whole array cancels all pairs, leaving `xorAll = a ^ b` (the two uniques). Since `a != b`, `xorAll` has at least one set bit — a position where `a` and `b` **differ**.

Pick any such bit (the **lowest set bit**, `xorAll & -xorAll`) and use it to split every number into two groups: those with that bit set vs. not. Each unique lands in a different group, and each duplicate pair stays together, so XOR-ing within each group isolates `a` and `b` separately.

---

## Code (C++)
```cpp
vector<int> singleNumber(vector<int>& nums) {
    long long xorAll = 0;
    for (int v : nums) xorAll ^= v;        // = a ^ b

    // Lowest set bit where a and b differ.
    int diffBit = xorAll & (-xorAll);

    int a = 0, b = 0;
    for (int v : nums) {
        if (v & diffBit) a ^= v;           // group with the bit set
        else             b ^= v;           // group without it
    }
    return {a, b};
}
```

⚠️ Use `xorAll & -xorAll` to isolate the rightmost differing bit. Computing it on a `long long` avoids the `INT_MIN` negation overflow that `-xorAll` would hit on a 32-bit `int`.

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(1) |
