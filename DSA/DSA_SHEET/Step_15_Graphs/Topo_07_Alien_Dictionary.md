# Alien Dictionary  🔴

**Reference:** [GeeksforGeeks — Given a sorted dictionary, find precedence of characters](https://www.geeksforgeeks.org/given-sorted-dictionary-find-precedence-characters/)

---

## Problem
Given a sorted list of `N` words from an alien language that uses the first `K` letters of the lowercase English alphabet, determine the order of these letters. Return any valid ordering as a string (empty string if no valid order exists).

```
Input:  K = 4, words = ["baa","abcd","abca","cab","cad"]
Output: "bdac"
```

---

## Intuition
The words are sorted, so comparing two adjacent words tells us the first position where they differ: that earlier word's character must come before the later word's character in the alien alphabet. Each such comparison yields a directed edge `c1 -> c2`. Collect all these precedence edges and run a topological sort (Kahn's) over the `K` letters — the resulting order is the alphabet. Two failure cases: (1) a cycle in the edges means contradictory ordering, and (2) an invalid dictionary where a longer word appears before its own prefix (e.g. `"abc"` before `"ab"`), which can never happen in a truly sorted list. Both yield the empty string.

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    string findOrder(vector<string>& words, int K) {
        vector<vector<int>> adj(K);
        vector<int> indegree(K, 0);

        // Build edges from each adjacent pair of words.
        for (int i = 0; i + 1 < (int)words.size(); i++) {
            const string& a = words[i];
            const string& b = words[i + 1];
            int len = min(a.size(), b.size());
            int j = 0;
            while (j < len && a[j] == b[j]) j++;     // first differing position

            if (j == len) {
                // No difference in the common prefix. If the earlier word is
                // longer, the dictionary is invalid ("abc" before "ab").
                if (a.size() > b.size()) return "";
            } else {
                int u = a[j] - 'a', v = b[j] - 'a';  // u must come before v
                adj[u].push_back(v);
                indegree[v]++;
            }
        }

        // Kahn's topological sort over the K letters.
        queue<int> q;
        for (int i = 0; i < K; i++)
            if (indegree[i] == 0)
                q.push(i);

        string order;
        while (!q.empty()) {
            int node = q.front(); q.pop();
            order += char('a' + node);
            for (int next : adj[node])
                if (--indegree[next] == 0)
                    q.push(next);
        }

        if ((int)order.size() != K) return "";       // cycle => contradiction
        return order;
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(N * L + K + E) |
| Space | O(K + E) |

> Only compare the *first* differing character of adjacent words — once you find a mismatch, stop. And do not forget the invalid-prefix case (`a.size() > b.size()` with no difference) returns `""`.
