# Count Inversions  🔴

**Reference:** [GeeksforGeeks — Count Inversions](https://www.geeksforgeeks.org/counting-inversions/) · [Codeforces / SPOJ INVCNT]

---

## Problem
Count the number of pairs `(i, j)` with `i < j` but `arr[i] > arr[j]` — i.e., how "far from sorted" the array is.

```
Input:  [5, 3, 2, 4, 1]    Output: 8
```

---

## Intuition: modified Merge Sort
Brute force is O(N²). Instead, piggyback on **merge sort**: during the merge of two sorted halves, when `left[i] > right[j]`, then `left[i]` and **all elements after it** in the left half are greater than `right[j]` — so add `(mid - i + 1)` to the inversion count in one shot.

---

## Code (C++)
```cpp
long long merge(vector<int>& a, int low, int mid, int high) {
    vector<int> temp;
    int left = low, right = mid + 1;
    long long inv = 0;
    while (left <= mid && right <= high) {
        if (a[left] <= a[right]) temp.push_back(a[left++]);
        else {
            temp.push_back(a[right++]);
            inv += (mid - left + 1);        // all of left[left..mid] > a[right]
        }
    }
    while (left <= mid)  temp.push_back(a[left++]);
    while (right <= high) temp.push_back(a[right++]);
    for (int i = low; i <= high; i++) a[i] = temp[i - low];
    return inv;
}

long long mergeSort(vector<int>& a, int low, int high) {
    if (low >= high) return 0;
    int mid = (low + high) / 2;
    long long inv = mergeSort(a, low, mid);
    inv += mergeSort(a, mid + 1, high);
    inv += merge(a, low, mid, high);
    return inv;
}
// call: mergeSort(arr, 0, n-1);
```

---

## Complexity
| | |
|---|---|
| Time | O(N log N) |
| Space | O(N) for the temp buffer |

> The key insight: in a sorted left half, if `left[i] > right[j]`, everything from `i` to `mid` is also `> right[j]`, so we count them all at once instead of one by one.
