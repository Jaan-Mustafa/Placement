# Maximum XOR of Two Numbers in an Array  🔴

**Reference:** [LeetCode 421 — Maximum XOR of Two Numbers in an Array](https://leetcode.com/problems/maximum-xor-of-two-numbers-in-an-array/)

---

## Problem
Given an array `nums`, return the maximum value of `nums[i] ^ nums[j]` over all pairs `i, j`.

```
Input:  [3, 10, 5, 25, 2, 8]
Output: 28          // 5 ^ 25 = 28
```

---

## Intuition
Brute force is O(N²). Instead, build a **binary trie** of every number's bits (MSB → LSB, fixed 31-bit width). For each number `x`, query the trie greedily: at each bit, try to follow the **opposite** bit so the XOR has a `1` at the highest possible position; if that branch is missing, follow the same bit. The query returns the best XOR achievable between `x` and some previously inserted number. Insert all numbers first, then query each — the overall max is the answer.

---

## Code (C++)
```cpp
class Solution {
    static const int BITS = 31;
    struct Node { Node* child[2] = {nullptr, nullptr}; };
    Node* root;

    void insert(int num) {
        Node* cur = root;
        for (int i = BITS; i >= 0; i--) {
            int b = (num >> i) & 1;
            if (!cur->child[b]) cur->child[b] = new Node();
            cur = cur->child[b];
        }
    }

    int query(int num) {                       // best XOR vs inserted numbers
        Node* cur = root;
        int best = 0;
        for (int i = BITS; i >= 0; i--) {
            int b = (num >> i) & 1;
            int want = 1 - b;                  // prefer the opposite bit
            if (cur->child[want]) {
                best |= (1 << i);              // got a differing high bit
                cur = cur->child[want];
            } else {
                cur = cur->child[b];           // forced same bit
            }
        }
        return best;
    }

public:
    int findMaximumXOR(vector<int>& nums) {
        root = new Node();
        for (int x : nums) insert(x);          // build the trie
        int ans = 0;
        for (int x : nums) ans = max(ans, query(x));
        return ans;
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(N × BITS) ≈ O(31N) for insert + query |
| Space | O(N × BITS) trie nodes |

> ⚠️ Inserting every number before querying is fine here because XOR is symmetric — querying `x` against the full set includes the best partner regardless of insertion order. Use 31 bits for `nums[i] ≤ 2·10^9`; for strictly ≤ 10^9, 30 bits also suffice.
