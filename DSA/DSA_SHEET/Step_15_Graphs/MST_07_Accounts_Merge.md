# Accounts Merge  🔴

**Reference:** [LeetCode 721 — Accounts Merge](https://leetcode.com/problems/accounts-merge/)

---

## Problem
Each account is `[name, email1, email2, ...]`. Two accounts belong to the same person if they share any email. Merge such accounts; output each as the name followed by all its emails sorted lexicographically.

```
Input: [["John","a@x","b@x"],["John","b@x","c@x"],["Mary","m@x"]]
Output: [["John","a@x","b@x","c@x"],["Mary","m@x"]]
```

---

## Intuition
Model each **account index** as a DSU node. Map every email to the first account index that owns it; when an email is seen again, it ties the current account to that earlier one, so we `unite` them. After processing, every cluster of accounts sharing emails has a single DSU root. Group all emails by their account's root, dedupe and sort them, then prepend the name (any account in the cluster has the same name).

---

## Code (C++)
```cpp
#include <bits/stdc++.h>
using namespace std;

struct DSU {
    vector<int> parent, rank_;
    DSU(int n): parent(n), rank_(n, 0) { iota(parent.begin(), parent.end(), 0); }
    int find(int x) { return parent[x] == x ? x : parent[x] = find(parent[x]); }
    void unite(int a, int b) {
        a = find(a); b = find(b);
        if (a == b) return;
        if (rank_[a] < rank_[b]) swap(a, b);
        parent[b] = a;
        if (rank_[a] == rank_[b]) rank_[a]++;
    }
};

vector<vector<string>> accountsMerge(vector<vector<string>>& accounts) {
    int n = accounts.size();
    DSU dsu(n);
    unordered_map<string,int> emailOwner;            // email -> account index

    for (int i = 0; i < n; ++i)
        for (int j = 1; j < (int)accounts[i].size(); ++j) {
            const string& email = accounts[i][j];
            if (emailOwner.count(email))
                dsu.unite(i, emailOwner[email]);     // shared email links accounts
            else
                emailOwner[email] = i;
        }

    // collect emails under each account's root (set keeps them sorted & unique)
    unordered_map<int, set<string>> grouped;
    for (auto& [email, owner] : emailOwner)
        grouped[dsu.find(owner)].insert(email);

    vector<vector<string>> result;
    for (auto& [root, emails] : grouped) {
        vector<string> row;
        row.push_back(accounts[root][0]);            // the name
        row.insert(row.end(), emails.begin(), emails.end());
        result.push_back(move(row));
    }
    return result;
}
```

---

## Complexity
| | |
|---|---|
| Time | O(N·K·log(N·K)) |
| Space | O(N·K) |

> Union on account indices, not on emails — emails are just the glue that tells you which accounts to merge; sorting falls out for free by collecting into a `set`.
