# Generate All Binary Strings (No Consecutive 1s)  🟡

**Reference:** [GeeksforGeeks — Generate binary strings without consecutive 1s](https://www.geeksforgeeks.org/generate-all-the-binary-strings-of-n-bits/)

---

## Problem
Generate all binary strings of length `n` that have **no two consecutive 1s**.

```
Input:  n = 3
Output: 000 001 010 100 101
```

---

## Intuition
Build the string position by position. At each step you can always place a `0`; you can place a `1` only if the previous character was not `1`. This "pick / don't-pick with a constraint" is the recursion-tree pattern.

---

## Code (C++)
```cpp
void generate(int n, string cur, char last, vector<string>& res) {
    if (cur.size() == n) { res.push_back(cur); return; }
    generate(n, cur + '0', '0', res);          // 0 is always allowed
    if (last != '1')
        generate(n, cur + '1', '1', res);      // 1 only after a 0
}

vector<string> generateBinaryStrings(int n) {
    vector<string> res;
    generate(n, "", '0', res);                 // pretend a leading 0
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(2ⁿ) worst (bounded by the constraint, ~Fibonacci count) |
| Space | O(n) recursion depth |
