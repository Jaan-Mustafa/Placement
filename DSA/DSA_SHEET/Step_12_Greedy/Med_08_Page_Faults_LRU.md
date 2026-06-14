# Page Faults in LRU  🟡

**Reference:** [GeeksforGeeks — Program for Page Replacement Algorithms | LRU](https://www.geeksforgeeks.org/program-for-least-recently-used-lru-page-replacement-algorithm/)

---

## Problem
Given a reference string of page requests and a fixed number of frames `C`, count the number of **page faults** using the Least Recently Used (LRU) replacement policy.

```
Input:  pages = [7,0,1,2,0,3,0,4], C = 4
Output: 6
```

---

## Intuition
A page fault occurs when a requested page is not currently in a frame. When all frames are full and a new page faults, **evict the page that was used least recently** — the one whose most recent access is furthest in the past. Track each page's last-use "time" and, on eviction, remove the page with the minimum last-use time.

---

## Code (C++)
```cpp
int pageFaults(vector<int>& pages, int C) {
    unordered_map<int,int> lastUsed;   // page -> most recent access time
    int faults = 0;
    for (int t = 0; t < (int)pages.size(); t++) {
        int p = pages[t];
        if (lastUsed.count(p)) {       // hit: page already in a frame
            lastUsed[p] = t;           // refresh its recency
            continue;
        }
        faults++;                      // miss / page fault
        if ((int)lastUsed.size() == C) {
            // evict least recently used page (smallest last-use time)
            int lruPage = -1, oldest = INT_MAX;
            for (auto& [pg, tm] : lastUsed)
                if (tm < oldest) { oldest = tm; lruPage = pg; }
            lastUsed.erase(lruPage);
        }
        lastUsed[p] = t;               // bring new page in
    }
    return faults;
}
```

---

## Dry run
`[7,0,1,2,0,3,0,4]`, C=4: faults on 7,0,1,2 (fill, 4 faults), 0 hit, 3 fault (evict 7) = 5, 0 hit, 4 fault (evict 1) = **6**.

## Complexity
| | |
|---|---|
| Time | O(N·C) with map scan (O(N) using a hashmap + doubly linked list) |
| Space | O(C) |

> ⚠️ The eviction scan makes this O(N·C). For optimal O(N), pair an `unordered_map` with a doubly linked list (the standard LRU-cache structure) so eviction is O(1).
