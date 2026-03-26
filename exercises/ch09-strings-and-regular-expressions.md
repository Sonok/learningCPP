# Chapter 9: Strings and Regular Expressions

---

## Quick Reference

- `std::string` is a sequence of `char`; owns its memory via RAII (§9.2, p.85)
- String literals: `"hello"` is `const char[]`; `"hello"s` (with `using namespace std::literals`) is `std::string` (§9.2, p.86)
- Common operations: `+` (concatenation), `==`, `<`, `[]`, `substr()`, `find()`, `size()`, `c_str()` (§9.2, p.85)
- `std::string_view` is a non-owning reference to a character sequence — cheap to copy, no allocation (§9.3, p.88)
- **Regex**: `<regex>` provides `std::regex`, `std::smatch`, `std::regex_match`, `std::regex_search`, `std::regex_replace` (§9.4, p.89)
- `regex_match` checks if the **entire** string matches; `regex_search` looks for a match **anywhere** (§9.4.2, p.91)
- Regex syntax follows ECMAScript by default (§9.4, p.89)
- `std::smatch` holds sub-matches; `m[0]` is the full match, `m[1]`, `m[2]` are capture groups (§9.4.2, p.91)
- `std::regex_replace` substitutes matched patterns (§9.4.3, p.92)
- `string::npos` is returned by `find()` when no match is found (§9.2, p.85)
- String comparisons are lexicographic (§9.2, p.85)
- Short string optimization (SSO): short strings are stored in-line, avoiding heap allocation (§9.2, p.85)

---

## 1. Multiple Choice (8 questions)

**Q1.** What is the difference between `regex_match` and `regex_search`? (§9.4.2, p.91)

a) `regex_match` uses POSIX syntax; `regex_search` uses ECMAScript
b) `regex_match` requires the entire string to match; `regex_search` finds a match anywhere
c) They are identical
d) `regex_match` is for strings; `regex_search` is for files

<details><summary>Answer</summary>

**b)** `regex_match` succeeds only if the pattern matches the entire input string. `regex_search` succeeds if any substring matches.

</details>

---

**Q2.** What does `std::string_view` provide over `const std::string&`? (§9.3, p.88)

a) Ownership of the underlying data
b) It can refer to substrings, C-strings, and string literals without copying or allocating
c) Thread safety
d) Mutable access

<details><summary>Answer</summary>

**b)** `string_view` is a non-owning view: pointer + length. It can wrap any contiguous character sequence without allocation, making it lighter than `const string&` (which may force a temporary `string` construction from a `const char*`).

</details>

---

**Q3.** What does `s.find("abc")` return if `"abc"` is not found? (§9.2, p.85)

a) `-1`
b) `0`
c) `std::string::npos`
d) Throws an exception

<details><summary>Answer</summary>

**c)** `string::npos` is the sentinel value (typically the maximum value of `size_t`) returned when a search fails.

</details>

---

**Q4.** What is the type of `"hello"` vs `"hello"s`? (§9.2, p.86)

a) Both are `std::string`
b) `"hello"` is `const char*`; `"hello"s` is `std::string`
c) `"hello"` is `std::string`; `"hello"s` is `std::string_view`
d) Both are `const char*`

<details><summary>Answer</summary>

**b)** A raw string literal `"hello"` decays to `const char*`. The `s` suffix (from `std::literals`) creates a `std::string`.

</details>

---

**Q5.** What does `m[1]` contain after a successful `regex_search` with one capture group? (§9.4.2, p.91)

a) The entire string
b) The full match
c) The first capture group's match
d) The character at index 1

<details><summary>Answer</summary>

**c)** `m[0]` is the full match. `m[1]`, `m[2]`, etc. are the capture groups in order.

</details>

---

**Q6.** Which is a danger of `string_view`? (§9.3, p.88)

a) It's too slow for short strings
b) It can outlive the data it refers to, creating a dangling reference
c) It can't be constructed from a `std::string`
d) It forces a heap allocation

<details><summary>Answer</summary>

**b)** `string_view` doesn't own its data. If the underlying string is destroyed or reallocated, the view becomes dangling.

</details>

---

**Q7.** What does `std::regex_replace` do? (§9.4.3, p.92)

a) Modifies the regex pattern
b) Replaces all matches of a pattern in a string with a replacement
c) Removes invalid regex syntax
d) Replaces the string with a regex object

<details><summary>Answer</summary>

**b)** `regex_replace(str, pattern, replacement)` returns a new string with all matches substituted.

</details>

---

**Q8.** Which regex flag changes to case-insensitive matching? (§9.4, p.89)

a) `std::regex::multiline`
b) `std::regex::icase`
c) `std::regex::nosubs`
d) `std::regex::nocase`

<details><summary>Answer</summary>

**b)** `std::regex::icase` makes the pattern case-insensitive.

</details>

---

## 2. Fill-in-the-blank Code (5 snippets)

**Snippet 1** — String operations (§9.2, p.85)

```cpp
#include <string>
#include <iostream>

int main() {
    std::string s = "Hello, World!";
    auto pos = s.___(", ");          // find the comma-space
    if (pos != std::string::___)
        std::cout << s.___(pos + 2) << '\n';  // substring from after ", "
}
```

<details><summary>Answer</summary>

```cpp
auto pos = s.find(", ");
if (pos != std::string::npos)
    std::cout << s.substr(pos + 2) << '\n';  // "World!"
```

</details>

---

**Snippet 2** — string_view (§9.3, p.88)

```cpp
#include <string>
#include <string_view>
#include <iostream>

void greet(std::___ name) {  // non-owning view
    std::cout << "Hello, " << name << "!\n";
}

int main() {
    greet("Alice");                    // from const char*
    std::string s = "Bob";
    greet(s);                          // from std::string
    greet(std::string_view(s).___( ));  // first 2 characters
}
```

<details><summary>Answer</summary>

```cpp
void greet(std::string_view name) { ... }
greet(std::string_view(s).substr(0, 2));  // "Bo"
```

</details>

---

**Snippet 3** — regex_match (§9.4.2, p.91)

```cpp
#include <regex>
#include <iostream>

int main() {
    std::string date = "2024-03-15";
    std::regex pattern(R"((\d{4})-(\d{2})-(\d{2}))");
    std::___ m;

    if (std::___(date, m, pattern)) {
        std::cout << "Year: "  << m[___] << '\n';
        std::cout << "Month: " << m[___] << '\n';
        std::cout << "Day: "   << m[___] << '\n';
    }
}
```

<details><summary>Answer</summary>

```cpp
std::smatch m;
if (std::regex_match(date, m, pattern)) {
    std::cout << "Year: "  << m[1] << '\n';
    std::cout << "Month: " << m[2] << '\n';
    std::cout << "Day: "   << m[3] << '\n';
}
```

</details>

---

**Snippet 4** — regex_replace (§9.4.3, p.92)

```cpp
#include <regex>
#include <iostream>

int main() {
    std::string text = "apples 5, bananas 3, oranges 8";
    std::regex nums(R"(\d+)");
    std::string result = std::___(text, nums, "##");
    std::cout << result << '\n';
}
```

<details><summary>Answer</summary>

```cpp
std::string result = std::regex_replace(text, nums, "##");
// Output: "apples ##, bananas ##, oranges ##"
```

</details>

---

**Snippet 5** — String concatenation and comparison (§9.2, p.85)

```cpp
#include <string>
#include <iostream>

int main() {
    std::string a = "Hello";
    std::string b = " World";
    std::string c = a ___ b;           // concatenate
    bool same = (a ___ "Hello");       // compare with C-string
    std::cout << c << " " << std::boolalpha << same << '\n';
}
```

<details><summary>Answer</summary>

```cpp
std::string c = a + b;     // "Hello World"
bool same = (a == "Hello"); // true
```

</details>

---

## 3. Code from Scratch — LeetCode Style (3 problems)

### Problem 1: Extract emails from text using regex (§9.4.2, p.91)

Write a function that extracts all email addresses from a string using `regex_search` in a loop.

```
Signature: std::vector<std::string> extract_emails(const std::string& text);
```

**Examples:**

| Input | Output |
|---|---|
| `"Contact alice@example.com or bob@test.org"` | `{"alice@example.com", "bob@test.org"}` |
| `"no emails here"` | `{}` |

**Constraints:** Use `<regex>`. Pattern: `\w+@\w+\.\w+` (simplified).

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <regex>

std::vector<std::string> extract_emails(const std::string& text) {
    std::vector<std::string> result;
    std::regex email_re(R"(\w+@\w+\.\w+)");
    // sregex_iterator walks through all matches
    auto begin = std::sregex_iterator(text.begin(), text.end(), email_re);
    auto end = std::sregex_iterator();
    for (auto it = begin; it != end; ++it) {
        result.push_back((*it)[0].str());  // [0] is the full match
    }
    return result;
}

int main() {
    auto emails = extract_emails("Contact alice@example.com or bob@test.org");
    for (const auto& e : emails)
        std::cout << e << '\n';
}
```

</details>

---

### Problem 2: Word frequency counter (§9.2, p.85)

Given a string of space-separated words, return a `vector<pair<string,int>>` of words sorted by frequency (descending).

```
Signature: std::vector<std::pair<std::string,int>> word_freq(const std::string& text);
```

**Examples:**

| Input | Output |
|---|---|
| `"the cat sat on the mat the"` | `{{"the",3},{"cat",1},{"mat",1},{"on",1},{"sat",1}}` |

**Constraints:** Use `std::string` operations or `istringstream` to split. Use `std::map` for counting.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <string>
#include <sstream>
#include <map>
#include <vector>
#include <algorithm>

std::vector<std::pair<std::string,int>> word_freq(const std::string& text) {
    std::map<std::string, int> counts;
    std::istringstream iss(text);
    std::string word;
    // Extract words one by one
    while (iss >> word)
        ++counts[word];

    // Move into a vector for sorting
    std::vector<std::pair<std::string,int>> result(counts.begin(), counts.end());
    // Sort by frequency descending, then alphabetically for ties
    std::sort(result.begin(), result.end(),
        [](const auto& a, const auto& b) {
            return a.second != b.second ? a.second > b.second : a.first < b.first;
        });
    return result;
}

int main() {
    auto freq = word_freq("the cat sat on the mat the");
    for (const auto& [word, count] : freq)
        std::cout << word << ": " << count << '\n';
}
```

</details>

---

### Problem 3: Safe substring with string_view (§9.3, p.88)

Write a function that returns the first `n` characters of a string as a `string_view`. If `n` exceeds the string length, return the whole string. Never allocate.

```
Signature: std::string_view first_n(std::string_view sv, size_t n);
```

**Examples:**

| Input | Output |
|---|---|
| `"hello", 3` | `"hel"` |
| `"hi", 10` | `"hi"` |
| `"", 5` | `""` |

**Constraints:** Use `string_view::substr`. No allocation.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <string_view>
#include <algorithm>

std::string_view first_n(std::string_view sv, size_t n) {
    // substr on string_view is cheap — no allocation, just adjusts the view
    return sv.substr(0, std::min(n, sv.size()));
}

int main() {
    std::cout << first_n("hello", 3) << '\n';  // hel
    std::cout << first_n("hi", 10) << '\n';    // hi
    std::cout << first_n("", 5) << '\n';       // (empty)
}
```

</details>

---

## 4. Output Prediction (4 snippets)

**Snippet 1** (§9.2, p.85)

```cpp
#include <string>
#include <iostream>
int main() {
    std::string s = "Hello";
    s += " World";
    s[0] = 'h';
    std::cout << s << '\n';
    std::cout << s.size() << '\n';
}
```

<details><summary>Answer</summary>

Output:
```
hello World
11
```

`+=` appends. `s[0] = 'h'` changes 'H' to 'h'. Size is 11 characters.

</details>

---

**Snippet 2** (§9.3, p.88)

```cpp
#include <string>
#include <string_view>
#include <iostream>
std::string_view bad() {
    std::string s = "temporary";
    return s;
}
int main() {
    std::cout << bad() << '\n';
}
```

<details><summary>Answer</summary>

**Undefined behavior.** The `string_view` refers to a local `string` that is destroyed when `bad()` returns. The view dangles. May print garbage, crash, or appear to work.

</details>

---

**Snippet 3** (§9.4.2, p.91)

```cpp
#include <regex>
#include <iostream>
int main() {
    std::string s = "abc123def456";
    std::regex r(R"(\d+)");
    std::smatch m;
    std::regex_search(s, m, r);
    std::cout << m[0] << '\n';
}
```

<details><summary>Answer</summary>

Output: `123`. `regex_search` finds the first match of one or more digits. `m[0]` is the full match.

</details>

---

**Snippet 4** (§9.2, p.85)

```cpp
#include <string>
#include <iostream>
int main() {
    std::string a = "apple";
    std::string b = "banana";
    std::cout << std::boolalpha << (a < b) << '\n';
    std::cout << (a == "apple") << '\n';
}
```

<details><summary>Answer</summary>

Output:
```
true
true
```

String comparison is lexicographic. `"apple" < "banana"` because `'a' < 'b'`.

</details>

---

## 5. Debug This (3 snippets)

**Bug 1** — Dangling string_view (§9.3, p.88)

```cpp
#include <string>
#include <string_view>
#include <iostream>

std::string_view get_greeting() {
    std::string s = "Hello, World!";
    return std::string_view(s);
}

int main() {
    auto sv = get_greeting();
    std::cout << sv << '\n';
}
```

<details><summary>Answer</summary>

**Bug:** `string_view` does not own the data. The local `string s` is destroyed at the end of `get_greeting()`, leaving the view dangling.

**Fix:** Return `std::string` instead, or ensure the data outlives the view (e.g., use a string literal or static string).

</details>

---

**Bug 2** — Using regex_match when regex_search is needed (§9.4.2, p.91)

```cpp
#include <regex>
#include <iostream>

int main() {
    std::string text = "My phone is 555-1234";
    std::regex phone(R"(\d{3}-\d{4})");
    if (std::regex_match(text, phone))
        std::cout << "Found phone!\n";
    else
        std::cout << "Not found\n";
}
```

<details><summary>Answer</summary>

**Bug:** `regex_match` requires the **entire** string to match `\d{3}-\d{4}`. The full string is "My phone is 555-1234", which doesn't match.

**Fix:** Use `std::regex_search` to find the pattern anywhere in the string.

</details>

---

**Bug 3** — Comparing find() result with -1 (§9.2, p.85)

```cpp
#include <string>
#include <iostream>

int main() {
    std::string s = "hello";
    if (s.find("xyz") == -1)
        std::cout << "not found\n";
}
```

<details><summary>Answer</summary>

**Bug:** `find()` returns `std::string::npos` (a large `size_t` value), not `-1`. Comparing with `-1` happens to work on many systems due to implicit conversion, but it's technically comparing `size_t` with a signed value — fragile and produces warnings.

**Fix:** `if (s.find("xyz") == std::string::npos)`

</details>

---

## 6. System Design / Open-ended (2 questions)

**Q1.** You're building a log parser that needs to extract timestamps, log levels, and messages from lines like `[2024-03-15 10:30:00] ERROR: Something failed`. Design the regex and the parsing function. Should you precompile the regex or create it each call? What's the performance implication? (§9.4, p.89; §9.4.2, p.91)

---

**Q2.** Compare `std::string`, `std::string_view`, and `const char*` as function parameter types. When is each the right choice? What are the lifetime and ownership implications? How does `string_view` enable "zero-copy" parsing? (§9.2, p.85; §9.3, p.88)

---

## 7. Rapid Fire (10 true/false)

1. `std::string` manages its own memory via RAII. (§9.2, p.85)

<details><summary>Answer</summary>True. The destructor frees the allocated buffer.</details>

2. `"hello"s` creates a `std::string_view`. (§9.2, p.86)

<details><summary>Answer</summary>False. The `s` suffix creates a `std::string`. `"hello"sv` creates a `string_view`.</details>

3. `string_view` owns its data. (§9.3, p.88)

<details><summary>Answer</summary>False. It's a non-owning reference — just a pointer and a length.</details>

4. `regex_search` finds the first match anywhere in the string. (§9.4.2, p.91)

<details><summary>Answer</summary>True. Unlike `regex_match`, which requires the full string to match.</details>

5. `std::smatch` stores sub-matches indexed from 0. (§9.4.2, p.91)

<details><summary>Answer</summary>True. `m[0]` is the full match, `m[1]` onward are capture groups.</details>

6. `string::find()` throws if the substring is not found. (§9.2, p.85)

<details><summary>Answer</summary>False. It returns `string::npos`.</details>

7. Short strings may avoid heap allocation due to SSO. (§9.2, p.85)

<details><summary>Answer</summary>True. Small String Optimization stores short strings in the object itself.</details>

8. `std::regex` uses POSIX regex syntax by default. (§9.4, p.89)

<details><summary>Answer</summary>False. The default is ECMAScript syntax.</details>

9. `string_view::substr` allocates a new string. (§9.3, p.88)

<details><summary>Answer</summary>False. `string_view::substr` returns another `string_view` — no allocation.</details>

10. `R"(...)"` is a raw string literal that disables escape processing. (§9.4, p.89)

<details><summary>Answer</summary>True. Raw string literals let you write regex patterns without double-escaping backslashes.</details>
