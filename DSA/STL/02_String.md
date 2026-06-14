# string — Text Sequence

`#include <string>`

`std::string` is essentially a `vector<char>` with text-specific helpers. Mutable, contiguous, random access in O(1). Supports almost every vector operation plus searching, substrings, and number conversions.

---

## Construction

```cpp
string s;                       // empty ""
string s = "hello";             // literal
string s(5, 'a');               // "aaaaa"  (count, char)
string s2(s);                   // copy
string s3(s, 1, 3);             // from s, start=1, len=3 -> "ell"
string s4(s.begin(), s.end());  // from a range
string s5 = string(3, 'x') + "!";   // "xxx!"
```

---

## Full function list

### Access & capacity
| Function | Description | Complexity |
|---|---|---|
| `s[i]` / `s.at(i)` | Char access (`at` bounds-checked) | O(1) |
| `s.front()` / `s.back()` | First / last char | O(1) |
| `s.size()` / `s.length()` | Number of chars (identical) | O(1) |
| `s.empty()` | True if "" | O(1) |
| `s.capacity()` / `s.reserve(n)` | Like vector | O(1) / O(n) |
| `s.c_str()` / `s.data()` | C-style `const char*` | O(1) |

### Modify
| Function | Description | Complexity |
|---|---|---|
| `s += "x"` / `s.append("x")` | Concatenate | O(len added) |
| `s.push_back('c')` | Append one char | Amortized O(1) |
| `s.pop_back()` | Remove last char | O(1) |
| `s.insert(pos, "str")` | Insert at index | O(n) |
| `s.erase(pos, len)` | Erase `len` chars from index | O(n) |
| `s.replace(pos, len, "str")` | Replace range with str | O(n) |
| `s.clear()` | Empty the string | O(n) |
| `s.resize(n)` / `s.resize(n,'x')` | Change length | O(n) |
| `s.substr(pos, len)` | **Returns a new string** (copy) | O(len) |
| `s.swap(s2)` | Swap two strings | O(1) |

### Search
| Function | Description | Returns |
|---|---|---|
| `s.find("str")` | First occurrence (from start) | index or `string::npos` |
| `s.find("str", pos)` | First occurrence at/after pos | index or npos |
| `s.rfind("str")` | **Last** occurrence | index or npos |
| `s.find_first_of("abc")` | First index of ANY of these chars | index or npos |
| `s.find_last_of("abc")` | Last index of any of these chars | index or npos |
| `s.find_first_not_of("abc")` | First char NOT in set | index or npos |
| `s.compare(s2)` | <0, 0, >0 lexicographic | int |
| `s.starts_with("pre")` / `s.ends_with("fix")` | C++20 | bool |
| `s.contains("x")` | C++23 | bool |

---

## Examples

### Building & slicing
```cpp
string s = "hello";
s += " world";                  // "hello world"
s.push_back('!');               // "hello world!"
cout << s.substr(0, 5);         // "hello"  (start, len)
cout << s.substr(6);            // "world!" (start -> end)

s.insert(5, "XYZ");             // "helloXYZ world!"
s.erase(5, 3);                  // back to "hello world!"
s.replace(0, 5, "HI");          // "HI world!"
```

### Searching
```cpp
string s = "abcabc";
cout << s.find("bc");           // 1  (first)
cout << s.rfind("bc");          // 4  (last)
cout << s.find("z");            // 18446744073709551615 (string::npos)

if (s.find("xyz") == string::npos) cout << "not found";

// loop over all occurrences of "bc"
size_t pos = s.find("bc");
while (pos != string::npos) {
    cout << pos << " ";
    pos = s.find("bc", pos + 1);
}
```

### Number ↔ string conversions
```cpp
int n      = stoi("42");        // string -> int
long l     = stol("42");        // -> long
long long ll = stoll("12345678901");   // -> long long
double d   = stod("3.14");      // -> double
int hex    = stoi("ff", 0, 16); // parse base-16 -> 255

string a = to_string(123);      // int -> "123"
string b = to_string(3.14);     // double -> "3.140000"
```

### Char utilities (`#include <cctype>`)
```cpp
isalpha('a');   // letter?
isdigit('5');   // digit?
isalnum('a');   // letter or digit?
isspace(' ');   // whitespace?
isupper('A');   islower('a');
toupper('a');   // 'A'
tolower('A');   // 'a'

// char digit <-> int
int d = '7' - '0';        // 7
char c = 7 + '0';         // '7'
// letter -> 0..25
int idx = 'c' - 'a';      // 2
```

### Common transforms
```cpp
string s = "Hello";

reverse(s.begin(), s.end());                 // "olleH"
sort(s.begin(), s.end());                    // sort chars
transform(s.begin(), s.end(), s.begin(), ::tolower);   // lowercase whole string

int vowels = count_if(s.begin(), s.end(),
    [](char c){ return string("aeiou").find(tolower(c)) != string::npos; });
```

### Splitting a string by delimiter
```cpp
#include <sstream>
string s = "a,b,c,d";
stringstream ss(s);
string token;
vector<string> parts;
while (getline(ss, token, ','))   // split on ','
    parts.push_back(token);       // {"a","b","c","d"}

// split on whitespace
stringstream ss2("the quick fox");
string w;
while (ss2 >> w) cout << w << "\n";   // the / quick / fox
```

### Frequency / anagram pattern
```cpp
// count char frequencies (lowercase letters)
int freq[26] = {0};
for (char c : s) freq[c - 'a']++;

// check anagram
bool isAnagram(string a, string b) {
    if (a.size() != b.size()) return false;
    sort(a.begin(), a.end());
    sort(b.begin(), b.end());
    return a == b;
}
```

---

## Gotchas
- ⚠️ `substr` **returns a copy** — it's O(len), not free. Don't call it in a tight loop if you can use indices instead.
- ⚠️ `find` returns `string::npos` (a huge unsigned value), NOT -1. Always compare `== string::npos`.
- ⚠️ Comparing with `==` works (`s == "abc"`), but `s.compare()` returns -1/0/1, not a bool.
- ⚠️ `s[i]` on an out-of-range index is UB; `s.at(i)` throws. The null terminator `s[s.size()]` is valid and reads `'\0'` (C++11+).
- Strings are passed by value = full copy. Use `const string&` for read-only params.

## DSA use cases
- String matching, palindromes, anagrams.
- Sliding window on characters (longest substring without repeats).
- Trie / hashing keys.
- Building output efficiently (prefer `+=` / `reserve` over repeated concatenation in loops).
