# Bit Prerequisites for Trie Problems  🟡

**Reference:** [GeeksforGeeks — Bitwise Trie / XOR using Trie](https://www.geeksforgeeks.org/maximum-xor-value-pair-from-given-array/) · [LeetCode 421 — Maximum XOR of Two Numbers in an Array](https://leetcode.com/problems/maximum-xor-of-two-numbers-in-an-array/)

---

## Problem
Not a judge problem — the bit foundations needed for **binary (XOR) tries**. Numbers are stored in a trie over their **binary representation** (fixed width, usually 32 bits), with two children per node: bit `0` and bit `1`. To maximize XOR against a query, we greedily walk toward the **opposite** bit at each level.

```
x = 5  -> binary (4 bits) = 0101
y = 6  -> binary (4 bits) = 0110
x ^ y  = 0011 = 3      // XOR is 1 exactly where the bits differ
```

---

## Intuition
Key bit facts that the XOR-trie problems rely on:
- **Bit i of n:** `(n >> i) & 1`.
- **XOR = "are they different?"** bit-by-bit: `1 ^ 1 = 0`, `0 ^ 0 = 0`, `1 ^ 0 = 1`. To make XOR large, you want differing bits, especially at **high** positions (a higher bit outweighs all lower bits combined).
- **Greedy from the MSB:** insert each number's bits most-significant-first. To maximize `x ^ y` for a fixed `x`, at every level prefer the child whose bit is `1 - currentBit` (the opposite), because that sets a `1` in the result at the highest available position.
- Use a **fixed bit width** (e.g. 31 or 32 bits) so all numbers align to the same depth in the trie.

---

## Code (C++)
```cpp
const int BITS = 31;   // enough for non-negative ints up to ~2^31

struct BitNode {
    BitNode* child[2] = {nullptr, nullptr};   // index 0 = bit 0, 1 = bit 1
};

// insert a number's bits from MSB (bit BITS) down to LSB (bit 0)
void insert(BitNode* root, int num) {
    BitNode* cur = root;
    for (int i = BITS; i >= 0; i--) {
        int bit = (num >> i) & 1;
        if (!cur->child[bit]) cur->child[bit] = new BitNode();
        cur = cur->child[bit];
    }
}

// best XOR of `num` against everything already inserted
int maxXorWith(BitNode* root, int num) {
    BitNode* cur = root;
    int best = 0;
    for (int i = BITS; i >= 0; i--) {
        int bit = (num >> i) & 1;
        int want = 1 - bit;                    // opposite bit gives a 1 in XOR
        if (cur->child[want]) {
            best |= (1 << i);                  // we got a differing bit here
            cur = cur->child[want];
        } else {
            cur = cur->child[bit];             // forced to take the same bit
        }
    }
    return best;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(BITS) per insert / query (constant, ~31) |
| Space | O(N × BITS) nodes for N numbers |

> ⚠️ Always iterate **MSB → LSB**; greedily flipping a high bit is worth more than any combination of lower bits. Pick `BITS` to cover the largest value in your input (use 31 for `int` ≤ 10^9).
