# Sort Characters by Frequency  🟡

**Reference:** [LeetCode 451 — Sort Characters By Frequency](https://leetcode.com/problems/sort-characters-by-frequency/)

---

## Problem
Sort the characters of a string in **decreasing order of frequency**. Characters with the same frequency may be in any order.

```
Input:  "tree"      Output: "eert" (or "eetr")
Input:  "Aabb"      Output: "bbAa" (or "bbaA")
```

---

## Intuition
Count frequencies, then order characters by count descending. A max-heap (priority_queue) of `{count, char}` pops the most frequent first; append each character `count` times.

---

## Code (C++)
```cpp
string frequencySort(string s) {
    unordered_map<char,int> freq;
    for (char c : s) freq[c]++;

    priority_queue<pair<int,char>> pq;       // max-heap by count
    for (auto& [ch, cnt] : freq) pq.push({cnt, ch});

    string res;
    while (!pq.empty()) {
        auto [cnt, ch] = pq.top(); pq.pop();
        res.append(cnt, ch);                 // ch repeated cnt times
    }
    return res;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N + K log K), K = distinct chars |
| Space | O(K) |

> Could also bucket-sort by frequency (buckets indexed 1..N) for O(N) when frequencies are bounded by string length.
