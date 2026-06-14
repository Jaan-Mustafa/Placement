# K Most Frequent Elements  🔴

**Reference:** [LeetCode 347 — Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/)

---

## Problem
Given an array, return the `k` elements that appear **most frequently** (any order).

```
Input:  nums = [1,1,1,2,2,3], k = 2    Output: [1, 2]
Input:  nums = [1], k = 1              Output: [1]
```

---

## Intuition
Count frequencies in a hash map, then select the `k` highest. Maintain a **min-heap of size k keyed on frequency** (`{freq, value}` pairs). Push each `{freq, value}`; when the heap exceeds `k`, pop the lowest-frequency entry. What survives is the top-k frequent — the k-of-size-min-heap trick applied to frequency.

---

## Code (C++)
```cpp
vector<int> topKFrequent(vector<int>& nums, int k) {
    unordered_map<int,int> freq;
    for (int x : nums) freq[x]++;

    // min-heap on frequency: {freq, value}, smallest freq on top
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>> mn;
    for (auto& [val, f] : freq) {
        mn.push({f, val});
        if ((int)mn.size() > k) mn.pop();   // drop least frequent, keep k most
    }

    vector<int> res;
    while (!mn.empty()) { res.push_back(mn.top().second); mn.pop(); }
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N log k) |
| Space | O(N) for the frequency map |

> ⚠️ Put **freq first** in the pair so the heap orders by frequency, not value. `greater<pair<int,int>>` makes a min-heap (smallest freq on top) — exactly what lets us evict the least frequent and retain the k most frequent.
