# Upper Bound  🟢

**Reference:** [GeeksforGeeks — Upper bound](https://www.geeksforgeeks.org/upper_bound-in-cpp/)

---

## Problem
Find the **upper bound**: the smallest index `i` such that `a[i] > x` (strictly greater). Return `n` if none.

```
Input:  [1, 2, 2, 3], x = 2    Output: 3   (first element > 2)
Input:  [1, 2, 2, 3], x = 3    Output: 4   (none, returns n)
```

---

## Intuition
Same as lower bound, but the condition is **strictly greater** (`>`) instead of `>=`. When `a[mid] > x`, record `mid` and search left for an earlier strictly-greater element.

---

## Code (C++)
```cpp
int upperBound(vector<int>& a, int x) {
    int low = 0, high = a.size() - 1, ans = a.size();
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (a[mid] > x) {
            ans = mid;          // candidate; look further left
            high = mid - 1;
        } else {
            low = mid + 1;
        }
    }
    return ans;
}
// STL: upper_bound(a.begin(), a.end(), x) - a.begin();
```

---

## Lower vs Upper bound
| | condition | finds |
|---|---|---|
| lower_bound | `a[i] >= x` | first element **≥** x |
| upper_bound | `a[i] > x` | first element **>** x |

`upper_bound(x) - lower_bound(x)` = count of elements equal to x.

## Complexity
| | |
|---|---|
| Time | O(log N) |
| Space | O(1) |
