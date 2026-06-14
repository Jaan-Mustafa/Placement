# Word Ladder II  🔴

**Reference:** [LeetCode 126 — Word Ladder II](https://leetcode.com/problems/word-ladder-ii/)

---

## Problem
Given `beginWord`, `endWord`, and a `wordList`, return **all** the shortest transformation sequences from `beginWord` to `endWord`, each step changing one letter to a word in the list.

```
Input: beginWord = "hit", endWord = "cog",
       wordList = ["hot","dot","dog","lot","log","cog"]
Output: [["hit","hot","dot","dog","cog"],
         ["hit","hot","lot","log","cog"]]
```

---

## Intuition
We need every shortest path, not just its length. Do a level-by-level BFS that records, for each word, the set of predecessors that first reached it at the shortest distance. Words discovered in the current BFS level are only removed from the dictionary **after** the level finishes, so multiple parents at the same depth are all captured. Once `endWord` is reached, DFS-backtrack through the parent map from `endWord` to `beginWord`, reversing each collected path.

---

## Code (C++)
```cpp
class Solution {
public:
    vector<vector<string>> findLadders(string beginWord, string endWord,
                                        vector<string>& wordList) {
        unordered_set<string> dict(wordList.begin(), wordList.end());
        unordered_map<string, vector<string>> parents;   // word -> its predecessors
        vector<vector<string>> ans;
        if (!dict.count(endWord)) return ans;

        unordered_set<string> currentLevel = {beginWord};
        dict.erase(beginWord);
        bool found = false;

        while (!currentLevel.empty() && !found) {
            unordered_set<string> nextLevel;             // words found this level
            for (const string& w : currentLevel) {
                string word = w;
                for (int i = 0; i < (int)word.size(); i++) {
                    char original = word[i];
                    for (char ch = 'a'; ch <= 'z'; ch++) {
                        word[i] = ch;
                        if (dict.count(word)) {
                            nextLevel.insert(word);
                            parents[word].push_back(w);  // record predecessor
                            if (word == endWord) found = true;
                        }
                    }
                    word[i] = original;
                }
            }
            // erase this level's words only after fully processing it
            for (const string& w : nextLevel) dict.erase(w);
            currentLevel = nextLevel;
        }

        if (!found) return ans;

        // backtrack from endWord to beginWord via parents
        vector<string> path = {endWord};
        backtrack(endWord, beginWord, parents, path, ans);
        return ans;
    }

    void backtrack(const string& word, const string& beginWord,
                   unordered_map<string, vector<string>>& parents,
                   vector<string>& path, vector<vector<string>>& ans) {
        if (word == beginWord) {
            vector<string> seq(path.rbegin(), path.rend());  // reverse to forward
            ans.push_back(seq);
            return;
        }
        for (const string& par : parents[word]) {
            path.push_back(par);
            backtrack(par, beginWord, parents, path, ans);
            path.pop_back();
        }
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(N * L * 26 + paths) |
| Space | O(N * L) |

> Erase words **per level, not per word**, otherwise you'd drop valid same-depth parents and miss some shortest paths entirely.
