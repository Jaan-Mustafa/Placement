# Reverse Words in a String  🟢

**Reference:** [LeetCode 151 — Reverse Words in a String](https://leetcode.com/problems/reverse-words-in-a-string/)

---

## Problem
Reverse the order of words. Trim leading/trailing spaces and collapse multiple spaces between words to one.

```
Input:  "the sky is blue"          Output: "blue is sky the"
Input:  "  hello   world  "        Output: "world hello"
```

---

## Intuition
Extract words (a `stringstream` naturally skips runs of whitespace), then output them in reverse order. This handles trimming and collapsing for free.

---

## Code (C++)
```cpp
string reverseWords(string s) {
    stringstream ss(s);
    string word, res;
    vector<string> words;
    while (ss >> word) words.push_back(word);     // splits on whitespace

    for (int i = words.size() - 1; i >= 0; i--) {
        res += words[i];
        if (i != 0) res += " ";
    }
    return res;
}
```

### In-place O(1) extra space variant
Reverse the whole string, then reverse each word back, while compacting spaces with two pointers.

---

## Complexity
| | |
|---|---|
| Time | O(N) |
| Space | O(N) for the word list |
