# Highest / Lowest Frequency Element  🟢

**Reference:** [GeeksforGeeks — Most & least frequent element](https://www.geeksforgeeks.org/frequent-element-array/)

---

## Problem
Given an array, find the element with the **maximum** frequency and the element with the **minimum** frequency.

```
Input:  [10, 5, 10, 15, 10, 5]
Output: Highest frequency: 10 (appears 3 times)
        Lowest  frequency: 15 (appears 1 time)
```

---

## Intuition
Build the frequency map in O(N), then do a single pass over the map to track the max-count and min-count elements.

---

## Code (C++)
```cpp
void highestLowestFreq(vector<int>& arr) {
    unordered_map<int,int> freq;
    for (int x : arr) freq[x]++;

    int maxEl = arr[0], minEl = arr[0];
    int maxCnt = 0, minCnt = INT_MAX;

    for (auto& [val, cnt] : freq) {
        if (cnt > maxCnt) { maxCnt = cnt; maxEl = val; }
        if (cnt < minCnt) { minCnt = cnt; minEl = val; }
    }

    cout << "Highest: " << maxEl << " (" << maxCnt << ")\n";
    cout << "Lowest:  " << minEl << " (" << minCnt << ")\n";
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N) — build map + iterate over distinct keys |
| Space | O(N) — frequency map |
