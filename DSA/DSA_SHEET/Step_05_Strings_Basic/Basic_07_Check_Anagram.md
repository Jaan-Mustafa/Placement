# Check Anagram  🟢

**Reference:** [LeetCode 242 — Valid Anagram](https://leetcode.com/problems/valid-anagram/)

---

## Problem
Two strings are anagrams if one is a rearrangement of the other (same characters, same counts).

```
Input:  s = "anagram", t = "nagaram"    Output: true
Input:  s = "rat", t = "car"            Output: false
```

---

## Intuition
Anagrams have identical character frequencies. Count each character in `s` (increment) and `t` (decrement) in one array; if all counts return to zero, they match. Different lengths → instantly false.

---

## Code (C++)
```cpp
bool isAnagram(string s, string t) {
    if (s.size() != t.size()) return false;
    int freq[26] = {0};
    for (char c : s) freq[c - 'a']++;
    for (char c : t) freq[c - 'a']--;
    for (int x : freq) if (x != 0) return false;
    return true;
}
```

### Sorting alternative
```cpp
sort(s.begin(), s.end());
sort(t.begin(), t.end());
return s == t;          // O(N log N)
```

---

## Complexity
| | |
|---|---|
| Time | O(N) (counting) vs O(N log N) (sorting) |
| Space | O(1) — fixed 26-size array |

> For Unicode/arbitrary characters, use an `unordered_map<char,int>` instead of the 26-size array.
