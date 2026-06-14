# Reverse Pairs  🔴

**Reference:** [LeetCode 493 — Reverse Pairs](https://leetcode.com/problems/reverse-pairs/)

---

## Problem
Count pairs `(i, j)` with `i < j` and `arr[i] > 2 * arr[j]`.

```
Input:  [1, 3, 2, 3, 1]    Output: 2
Input:  [2, 4, 3, 5, 1]    Output: 3
```

---

## Intuition: modified Merge Sort
Like counting inversions, but the condition (`arr[i] > 2*arr[j]`) is asymmetric, so we **count first, then merge** separately. During merge, for each element in the left half, advance a pointer over the right half counting how many satisfy `left[i] > 2*right[j]`. Both halves are sorted, so this counting is linear per level.

---

## Code (C++)
```cpp
void countPairs(vector<int>& a, int low, int mid, int high, long long& cnt) {
    int right = mid + 1;
    for (int i = low; i <= mid; i++) {
        while (right <= high && a[i] > 2LL * a[right]) right++;
        cnt += (right - (mid + 1));
    }
}

void merge(vector<int>& a, int low, int mid, int high) {
    vector<int> temp;
    int l = low, r = mid + 1;
    while (l <= mid && r <= high)
        temp.push_back(a[l] <= a[r] ? a[l++] : a[r++]);
    while (l <= mid)  temp.push_back(a[l++]);
    while (r <= high) temp.push_back(a[r++]);
    for (int i = low; i <= high; i++) a[i] = temp[i - low];
}

void mergeSort(vector<int>& a, int low, int high, long long& cnt) {
    if (low >= high) return;
    int mid = (low + high) / 2;
    mergeSort(a, low, mid, cnt);
    mergeSort(a, mid + 1, high, cnt);
    countPairs(a, low, mid, high, cnt);    // count BEFORE merging
    merge(a, low, mid, high);
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N log N) — the counting pointer never resets within a merge level |
| Space | O(N) |

> Use `2LL * a[right]` to avoid 32-bit overflow when elements are near INT_MAX.
