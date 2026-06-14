# Counting Frequencies of Elements  🟢

**Reference:** [GeeksforGeeks — Counting frequencies of array elements](https://www.geeksforgeeks.org/counting-frequencies-of-array-elements/)

---

## Problem
Given an array, count how many times each element appears.

```
Input:  [10, 5, 10, 15, 10, 5]
Output: 10 -> 3, 5 -> 2, 15 -> 1
```

---

## Intuition
**Hashing** trades space for speed. Instead of scanning the array for every query (O(N) each), we build a frequency table once in O(N):
- For small, bounded integer keys → use a fixed array `hash[N]` (fastest).
- For large or arbitrary keys (or strings) → use `unordered_map`.

---

## Code (C++)
```cpp
// Using a hash map (general)
void countFreq(vector<int>& arr) {
    unordered_map<int,int> freq;
    for (int x : arr) freq[x]++;        // build table in O(N)

    for (auto& [val, cnt] : freq)
        cout << val << " -> " << cnt << "\n";
}

// Using an array (when values are small, e.g. 0..1e6)
int hashArr[1000001] = {0};
void countFreqArray(vector<int>& arr) {
    for (int x : arr) hashArr[x]++;
    // query any element in O(1): hashArr[q]
}
```

---

## Why precompute?
Naively, answering Q queries of "how many times does X appear?" is O(N·Q). With a precomputed hash, it's O(N) build + O(1) per query.

## Complexity
| | |
|---|---|
| Time | O(N) build, O(1) per query |
| Space | O(N) (or O(range) for the array version) |

> See `../../STL/11_Unordered_Map.md` for the full `unordered_map` reference.
