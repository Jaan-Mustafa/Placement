# Task Scheduler  🟡

**Reference:** [LeetCode 621 — Task Scheduler](https://leetcode.com/problems/task-scheduler/)

---

## Problem
Given tasks (letters) and a cooldown `n`, the same task must be separated by at least `n` intervals. Each interval runs one task or is idle. Return the **minimum total intervals** to finish all tasks.

```
Input:  tasks = ["A","A","A","B","B","B"], n = 2   Output: 8
        A B idle A B idle A B
Input:  tasks = ["A","A","A","B","B","B"], n = 0   Output: 6
```

---

## Intuition
Greedily run the **most frequent available task** first to spread heavy tasks out. Use a **max-heap of remaining counts**. Each cooldown cycle of length `n+1`: pop up to `n+1` distinct tasks, decrement them, and stash those still > 0. Add `n+1` to the time for a full cycle, or just the count run if the heap empties last cycle (no trailing idles needed).

---

## Code (C++)
```cpp
int leastInterval(vector<char>& tasks, int n) {
    int freq[26] = {0};
    for (char c : tasks) freq[c - 'A']++;

    priority_queue<int> mx;                 // max-heap of counts
    for (int f : freq) if (f) mx.push(f);

    int time = 0;
    while (!mx.empty()) {
        vector<int> tmp;                    // tasks run this cycle
        int slots = n + 1;                  // one cycle = n+1 slots
        int done = 0;
        while (slots-- && !mx.empty()) {
            int c = mx.top(); mx.pop();
            if (--c > 0) tmp.push_back(c);  // still has copies left
            done++;
        }
        for (int c : tmp) mx.push(c);       // re-add for next cycle
        // if heap now empty, no trailing idles: add only what we ran
        time += mx.empty() ? done : n + 1;
    }
    return time;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(T log 26) ≈ O(T), T = number of tasks |
| Space | O(26) = O(1) |

> ⚠️ The last cycle must add `done` (actual tasks run), not `n+1`, otherwise you over-count idle slots at the very end. Heap size is bounded by 26 distinct letters.
