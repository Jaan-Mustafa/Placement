# Painter's Partition Problem  🔴

**Reference:** [GeeksforGeeks — Painter's Partition](https://www.geeksforgeeks.org/painters-partition-problem/) · [Coding Ninjas / InterviewBit]

---

## Problem
`k` painters paint `n` boards (lengths given). A painter paints **contiguous** boards, each board painted by one painter, and painting 1 unit takes a fixed time. Minimize the time to paint all boards (= minimize the maximum length any single painter paints).

```
Input:  boards = [10, 20, 30, 40], k = 2    Output: 60
        (painter1: [10,20,30]=60, painter2: [40]=40 → max 60)
```

---

## Intuition: binary search on the time limit
Identical to **Book Allocation** and **Split Array Largest Sum**. The answer (max length per painter) ranges from `max(boards)` to `sum(boards)`. Larger limit → fewer painters needed (monotonic). Binary search for the smallest limit doable with ≤ `k` painters.

---

## Code (C++)
```cpp
int paintersNeeded(vector<int>& boards, long long limit) {
    int painters = 1;
    long long cur = 0;
    for (int b : boards) {
        if (cur + b > limit) { painters++; cur = 0; }
        cur += b;
    }
    return painters;
}

long long painterPartition(vector<int>& boards, int k) {
    long long low = *max_element(boards.begin(), boards.end());
    long long high = accumulate(boards.begin(), boards.end(), 0LL);
    long long ans = high;
    while (low <= high) {
        long long mid = low + (high - low) / 2;
        if (paintersNeeded(boards, mid) <= k) { ans = mid; high = mid - 1; }
        else low = mid + 1;
    }
    return ans;
}
```

---

## The "split into k groups" family
| Problem | Story | Same code? |
|---|---|---|
| Book Allocation | books → students, min max pages | ✅ |
| Split Array Largest Sum | array → k parts, min max sum | ✅ |
| Painter's Partition | boards → painters, min max length | ✅ |

## Complexity
| | |
|---|---|
| Time | O(N · log(sum − max)) |
| Space | O(1) |
