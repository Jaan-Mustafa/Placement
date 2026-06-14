# Maximum XOR With an Element From Array  🔴

**Reference:** [LeetCode 1707 — Maximum XOR With an Element From Array](https://leetcode.com/problems/maximum-xor-with-an-element-from-array/)

---

## Problem
Given `nums` and queries `queries[k] = [x, m]`, for each query find the maximum `x ^ nums[j]` over all `nums[j] ≤ m`. If no element is `≤ m`, the answer is `-1`.

```
nums    = [0, 1, 2, 3, 4]
queries = [[3,1], [1,3], [5,6]]
Output  = [3, 3, 7]
// [3,1]: elements ≤1 are {0,1}; best 3^1 = 2? -> 3^0=3 -> 3
// [1,3]: elements ≤3 are {0,1,2,3}; 1^2 = 3
// [5,6]: elements ≤6 are all; 5^2 = 7
```

---

## Intuition
This is "Maximum XOR of two numbers" plus a **ceiling constraint** `nums[j] ≤ m`. Handle it with **offline processing**:
1. **Sort `nums`** ascending and **sort queries by `m`** ascending (remember original indices).
2. Sweep queries in increasing `m`. Maintain a pointer into `nums` and insert every `nums[j] ≤ m` into a binary trie **once**. Because both are sorted, each element is inserted exactly once across all queries.
3. For each query, if the trie is non-empty, do the standard greedy max-XOR query for `x`; otherwise the answer is `-1`.

---

## Code (C++)
```cpp
class Solution {
    static const int BITS = 30;                // nums, x ≤ 10^9 < 2^30
    struct Node { Node* child[2] = {nullptr, nullptr}; };
    Node* root;
    bool empty = true;

    void insert(int num) {
        Node* cur = root;
        for (int i = BITS; i >= 0; i--) {
            int b = (num >> i) & 1;
            if (!cur->child[b]) cur->child[b] = new Node();
            cur = cur->child[b];
        }
        empty = false;
    }

    int query(int num) {
        if (empty) return -1;                  // no eligible element
        Node* cur = root;
        int best = 0;
        for (int i = BITS; i >= 0; i--) {
            int b = (num >> i) & 1, want = 1 - b;
            if (cur->child[want]) { best |= (1 << i); cur = cur->child[want]; }
            else cur = cur->child[b];
        }
        return best;
    }

public:
    vector<int> maximizeXor(vector<int>& nums, vector<vector<int>>& queries) {
        root = new Node();
        sort(nums.begin(), nums.end());        // ascending values

        int q = queries.size();
        vector<int> order(q);
        for (int i = 0; i < q; i++) order[i] = i;
        // process queries in increasing m
        sort(order.begin(), order.end(),
             [&](int a, int b){ return queries[a][1] < queries[b][1]; });

        vector<int> ans(q);
        int p = 0, n = nums.size();
        for (int idx : order) {
            int x = queries[idx][0], m = queries[idx][1];
            while (p < n && nums[p] <= m) insert(nums[p++]);  // add eligible
            ans[idx] = query(x);
        }
        return ans;
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O((N + Q) × BITS + Q log Q) — each element inserted once |
| Space | O(N × BITS) trie nodes + O(Q) for ordering |

> ⚠️ Offline sorting is what makes the `nums[j] ≤ m` filter cheap: the insert pointer only moves forward, so total inserts are O(N), not O(N·Q). Don't forget to write answers back at the **original** query index (`ans[idx]`), since we processed them out of order.
