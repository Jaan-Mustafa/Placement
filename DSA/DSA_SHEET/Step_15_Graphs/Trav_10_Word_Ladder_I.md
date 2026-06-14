# Word Ladder I  🔴

**Reference:** [LeetCode 127 — Word Ladder](https://leetcode.com/problems/word-ladder/)

---

## Problem
Given `beginWord`, `endWord`, and a `wordList`, return the length of the shortest transformation sequence where each step changes exactly one letter and every intermediate word is in the list. Return `0` if no sequence exists.

```
Input: beginWord = "hit", endWord = "cog",
       wordList = ["hot","dot","dog","lot","log","cog"]
Output: 5     // hit -> hot -> dot -> dog -> cog
```

---

## Intuition
Treat each word as a graph node; an edge connects two words differing by exactly one letter. The shortest transformation is the shortest path in an unweighted graph, which is plain BFS. We do not pre-build the graph; instead, for each word we generate all one-letter neighbours on the fly and check membership in a hash set. Erasing a word from the set when first reached marks it visited and keeps the search O(1) per lookup.

---

## Code (C++)
```cpp
class Solution {
public:
    int ladderLength(string beginWord, string endWord, vector<string>& wordList) {
        unordered_set<string> dict(wordList.begin(), wordList.end());
        if (!dict.count(endWord)) return 0;     // target unreachable

        queue<pair<string,int>> q;              // {word, steps so far}
        q.push({beginWord, 1});
        dict.erase(beginWord);

        while (!q.empty()) {
            auto [word, steps] = q.front(); q.pop();
            if (word == endWord) return steps;

            // try changing each position to every letter a-z
            for (int i = 0; i < (int)word.size(); i++) {
                char original = word[i];
                for (char ch = 'a'; ch <= 'z'; ch++) {
                    word[i] = ch;
                    if (dict.count(word)) {     // valid unvisited neighbour
                        dict.erase(word);       // mark visited
                        q.push({word, steps + 1});
                    }
                }
                word[i] = original;             // restore for next position
            }
        }
        return 0;                               // no path found
    }
};
```

---

## Complexity
| | |
|---|---|
| Time | O(N * L * 26) |
| Space | O(N * L) |

> Erase a word from the set the moment you enqueue it — deferring the visited mark lets the same word enter the queue many times and blows up the search.
