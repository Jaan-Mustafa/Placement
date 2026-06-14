# Book Allocation Problem  🔴

**Reference:** [GeeksforGeeks — Allocate minimum number of pages](https://www.geeksforgeeks.org/allocate-minimum-number-pages/) · [LeetCode 410 (same family)](https://leetcode.com/problems/split-array-largest-sum/)

---

## Problem
`n` books with page counts must be allocated to `m` students. Each student gets a **contiguous** block of books, every book assigned to exactly one student. Minimize the **maximum** number of pages assigned to any student.

```
Input:  pages = [12, 34, 67, 90], m = 2    Output: 113
        (split: [12,34,67] = 113, [90] = 90 → max 113)
```

---

## Intuition: binary search on the page limit
If `m > n`, it's impossible (return -1). The answer (a max-pages limit) ranges from `max(pages)` (no student gets less than the biggest book) to `sum(pages)` (one student gets all). A larger limit needs fewer students — monotonic. Binary search for the smallest limit allocatable to ≤ `m` students.

Feasibility (greedy): walk books, accumulate pages; when adding a book exceeds the limit, give it to a new student.

---

## Code (C++)
```cpp
int studentsRequired(vector<int>& pages, int limit) {
    int students = 1, cur = 0;
    for (int p : pages) {
        if (cur + p > limit) { students++; cur = 0; }   // new student
        cur += p;
    }
    return students;
}

int findPages(vector<int>& pages, int m) {
    if (m > (int)pages.size()) return -1;
    int low = *max_element(pages.begin(), pages.end());
    int high = accumulate(pages.begin(), pages.end(), 0);
    int ans = high;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (studentsRequired(pages, mid) <= m) { ans = mid; high = mid - 1; }
        else low = mid + 1;
    }
    return ans;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N · log(sum − max)) |
| Space | O(1) |

> Identical structure to **Split Array Largest Sum** and **Painter's Partition** — same code, different story.
