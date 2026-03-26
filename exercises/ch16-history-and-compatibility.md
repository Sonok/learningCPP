# Chapter 16: History and Compatibility

---

## Quick Reference

- C++ evolved from C with classes (1979), through standardization: C++98, C++03, C++11, C++14, C++17, C++20 (§16.1, p.153)
- **C++11** was the landmark modern C++ release: move semantics, lambdas, `auto`, range-for, smart pointers, threads (§16.1.3, p.155)
- **C++14** refined C++11: generic lambdas, `make_unique`, relaxed constexpr, variable templates (§16.1.3, p.156)
- **C++17** added: structured bindings, `if constexpr`, `optional`, `variant`, `any`, `string_view`, parallel algorithms (§16.1.3, p.156)
- **C++20** added: concepts, ranges, coroutines, modules, `<=>` (spaceship operator) (§16.1.3, p.157)
- C compatibility: C++ is "mostly" a superset of C, but with important differences (§16.2, p.158)
- Differences from C: `void*` doesn't implicitly convert to other pointer types; `struct` tags are types in C++ (§16.2, p.158)
- **ABI compatibility** — mixing C and C++ code requires `extern "C"` declarations (§16.2.1, p.159)
- Stroustrup's design philosophy: zero-overhead abstraction, type safety, direct hardware access (§16.3, p.160)
- "Leave no room for a lower-level language beneath C++ (except assembler)" (§16.3, p.160)
- Backward compatibility is a core value but shouldn't prevent progress (§16.1, p.153)
- The ISO C++ committee manages the standard's evolution (§16.1, p.153)

---

## 1. Multiple Choice (8 questions)

**Q1.** Which C++ standard introduced move semantics, lambdas, and `auto`? (§16.1.3, p.155)

a) C++03
b) C++11
c) C++14
d) C++17

<details><summary>Answer</summary>

**b)** C++11 was the transformative release that introduced move semantics, lambda expressions, `auto` type deduction, range-for loops, smart pointers, and threading.

</details>

---

**Q2.** What did C++17 add? (§16.1.3, p.156)

a) Concepts and ranges
b) Structured bindings, `if constexpr`, `optional`, `variant`
c) Lambda expressions
d) `constexpr` functions

<details><summary>Answer</summary>

**b)** C++17 added structured bindings, `if constexpr`, `std::optional`, `std::variant`, `std::any`, `std::string_view`, and parallel algorithms.

</details>

---

**Q3.** Why is `extern "C"` needed? (§16.2.1, p.159)

a) To make C++ code run faster
b) To tell the C++ compiler to use C linkage (no name mangling), enabling interop with C code
c) To import C libraries automatically
d) To disable exception handling

<details><summary>Answer</summary>

**b)** C++ mangles function names to support overloading. C does not. `extern "C"` disables mangling for specific declarations, allowing C and C++ code to link together.

</details>

---

**Q4.** Which is a key difference between C and C++ regarding `void*`? (§16.2, p.158)

a) C doesn't have `void*`
b) In C, `void*` implicitly converts to any pointer type; in C++, you need an explicit cast
c) They are handled identically
d) C++ `void*` is deprecated

<details><summary>Answer</summary>

**b)** C allows `int* p = malloc(sizeof(int));` (implicit `void*` to `int*`). C++ requires `int* p = static_cast<int*>(malloc(sizeof(int)));`. This is part of C++'s stricter type system.

</details>

---

**Q5.** What is the "zero-overhead principle"? (§16.3, p.160)

a) C++ programs use zero memory
b) You don't pay for what you don't use; what you use can't be coded more efficiently by hand
c) All C++ features are free
d) C++ has no runtime

<details><summary>Answer</summary>

**b)** The zero-overhead principle means: (1) you don't pay for features you don't use, and (2) when you do use a feature, the compiler generates code as efficient as what you'd write manually.

</details>

---

**Q6.** What did C++20 introduce for constraining templates? (§16.1.3, p.157)

a) SFINAE
b) `enable_if`
c) Concepts
d) Type traits

<details><summary>Answer</summary>

**c)** C++20 introduced concepts as a first-class language feature for constraining template parameters, replacing the need for SFINAE and `enable_if` tricks.

</details>

---

**Q7.** Which statement reflects Stroustrup's design philosophy? (§16.3, p.160)

a) Prefer runtime checks over compile-time checks
b) Make simple things simple and complex things possible; prefer compile-time checking
c) Always choose safety over performance
d) Avoid templates in favor of inheritance

<details><summary>Answer</summary>

**b)** Stroustrup emphasizes making common tasks easy while keeping expert-level power available. Compile-time checking is preferred because it catches errors early without runtime cost.

</details>

---

**Q8.** What is the purpose of C++20 modules? (§16.1.3, p.157)

a) To replace namespaces
b) To replace the #include/header model with a faster, more hygienic module system
c) To enable dynamic linking
d) To add runtime module loading

<details><summary>Answer</summary>

**b)** Modules replace `#include` with `import`, avoiding repeated parsing of headers, macro leakage, and include-order dependencies. They speed up compilation and improve encapsulation.

</details>

---

## 2. Fill-in-the-blank Code (5 snippets)

**Snippet 1** — extern "C" (§16.2.1, p.159)

```cpp
// In a header used by both C and C++ code:
#ifdef __cplusplus
___ "C" {
#endif

int compute(int x, int y);
void print_result(int r);

#ifdef __cplusplus
}
#endif
```

<details><summary>Answer</summary>

```cpp
extern "C" {
```

The `#ifdef __cplusplus` guard ensures the `extern "C"` only appears when compiled as C++.

</details>

---

**Snippet 2** — C++11 features (§16.1.3, p.155)

```cpp
#include <vector>
#include <memory>
#include <iostream>

int main() {
    ___ v = std::vector<int>{1, 2, 3, 4, 5};  // type deduction
    ___ p = std::make_unique<int>(42);           // smart pointer

    for (___ x : v)  // range-based for
        std::cout << x << " ";

    auto square = [](int x) { return x * x; };  // ___
    std::cout << '\n' << square(5);
}
```

<details><summary>Answer</summary>

```cpp
auto v = std::vector<int>{1, 2, 3, 4, 5};
auto p = std::make_unique<int>(42);
for (auto x : v)
// lambda
```

</details>

---

**Snippet 3** — C++17 features (§16.1.3, p.156)

```cpp
#include <optional>
#include <variant>
#include <iostream>

int main() {
    std::___<int> maybe = 42;              // optional
    std::___<int, double, std::string> v;  // variant
    v = 3.14;

    if (maybe)
        std::cout << *maybe << '\n';

    ___ (auto val = std::get<double>(v); val > 0) {  // C++17 if-init
        std::cout << "positive: " << val << '\n';
    }
}
```

<details><summary>Answer</summary>

```cpp
std::optional<int> maybe = 42;
std::variant<int, double, std::string> v;
if (auto val = std::get<double>(v); val > 0) { ... }
```

</details>

---

**Snippet 4** — C++14 generic lambda (§16.1.3, p.156)

```cpp
#include <iostream>
#include <string>

int main() {
    auto print = [](___ x) {  // generic lambda (C++14)
        std::cout << x << '\n';
    };
    print(42);
    print(3.14);
    print(std::string("hello"));
}
```

<details><summary>Answer</summary>

```cpp
auto print = [](auto x) { ... };
```

`auto` in lambda parameters was introduced in C++14, making lambdas into abbreviated function templates.

</details>

---

**Snippet 5** — C++20 concepts (§16.1.3, p.157)

```cpp
#include <concepts>
#include <iostream>

___ <typename T>
___ Numeric = std::is_arithmetic_v<T>;

void print(Numeric ___ val) {
    std::cout << val << '\n';
}

int main() {
    print(42);
    print(3.14);
    // print("hello");  // error: const char* is not Numeric
}
```

<details><summary>Answer</summary>

```cpp
template<typename T>
concept Numeric = std::is_arithmetic_v<T>;

void print(Numeric auto val) { ... }
```

</details>

---

## 3. Code from Scratch — LeetCode Style (3 problems)

### Problem 1: Feature-detection macro usage (§16.1, p.153)

Write a function that uses `if constexpr` (C++17) to provide different behavior based on type, with a C++11 fallback using template specialization.

```
Signature: template<typename T> std::string type_name();
```

**Examples:**

| Type | Output |
|---|---|
| `int` | `"integer"` |
| `double` | `"floating"` |
| `std::string` | `"string"` |

**Constraints:** Primary implementation with `if constexpr`. Show what the C++11 equivalent would look like as comments.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <string>
#include <type_traits>

// C++17: single function with compile-time branching
template<typename T>
std::string type_name() {
    if constexpr (std::is_integral_v<T>)
        return "integer";
    else if constexpr (std::is_floating_point_v<T>)
        return "floating";
    else
        return "other";

    // C++11 equivalent would need specialization:
    // template<> std::string type_name<int>() { return "integer"; }
    // template<> std::string type_name<double>() { return "floating"; }
    // Or SFINAE with enable_if
}

int main() {
    std::cout << type_name<int>() << '\n';        // integer
    std::cout << type_name<double>() << '\n';     // floating
    std::cout << type_name<std::string>() << '\n'; // other
}
```

</details>

---

### Problem 2: C/C++ interop wrapper (§16.2.1, p.159)

Write a C-compatible API for a C++ string stack. Provide `extern "C"` functions: `stack_create`, `stack_push`, `stack_pop`, `stack_destroy`.

```
extern "C" {
    void* stack_create();
    void stack_push(void* handle, const char* str);
    const char* stack_pop(void* handle);
    void stack_destroy(void* handle);
}
```

**Constraints:** Implementation uses C++ (`std::stack<std::string>`). Interface is pure C.

<details><summary>Solution</summary>

```cpp
#include <stack>
#include <string>
#include <cstring>

// Internal C++ implementation
struct StringStack {
    std::stack<std::string> data;
    std::string last_pop;  // buffer for pop result
};

// C-compatible interface — no name mangling
extern "C" {

void* stack_create() {
    return new StringStack();  // opaque handle
}

void stack_push(void* handle, const char* str) {
    auto* s = static_cast<StringStack*>(handle);
    s->data.push(str);
}

const char* stack_pop(void* handle) {
    auto* s = static_cast<StringStack*>(handle);
    if (s->data.empty()) return nullptr;
    s->last_pop = s->data.top();  // store in buffer
    s->data.pop();
    return s->last_pop.c_str();   // return C string
}

void stack_destroy(void* handle) {
    delete static_cast<StringStack*>(handle);
}

}  // extern "C"

// C++ test
#include <iostream>
int main() {
    auto* s = stack_create();
    stack_push(s, "hello");
    stack_push(s, "world");
    std::cout << stack_pop(s) << '\n';  // world
    std::cout << stack_pop(s) << '\n';  // hello
    stack_destroy(s);
}
```

Key decisions: `void*` is the opaque handle pattern used in C APIs. `last_pop` buffers the popped string so its `c_str()` stays valid. `extern "C"` prevents name mangling.

</details>

---

### Problem 3: Compare old vs modern C++ (§16.1.3, p.155)

Rewrite this C++98-style code using modern C++ (C++17). The code reads numbers into a vector, sorts them, and prints unique values.

Original:
```cpp
// C++98 style
std::vector<int> v;
for (int i = 0; i < n; ++i) { int x; std::cin >> x; v.push_back(x); }
std::sort(v.begin(), v.end());
std::vector<int>::iterator it = std::unique(v.begin(), v.end());
v.erase(it, v.end());
for (std::vector<int>::iterator i = v.begin(); i != v.end(); ++i)
    std::cout << *i << " ";
```

**Constraints:** Use `auto`, range-for, `std::erase`/`std::unique`, structured bindings where appropriate.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    int n;
    std::cin >> n;

    // C++11: auto, range initialization
    std::vector<int> v;
    v.reserve(n);
    for (int i = 0; i < n; ++i) {
        int x;
        std::cin >> x;
        v.push_back(x);
    }

    // C++11: auto for iterators
    std::sort(v.begin(), v.end());
    auto it = std::unique(v.begin(), v.end());
    v.erase(it, v.end());

    // C++11: range-for
    for (auto x : v)
        std::cout << x << " ";
    std::cout << '\n';
}
```

Improvements: `auto` replaces verbose iterator types. Range-for replaces manual iterator loops. `reserve` prevents reallocation. Each change reduces verbosity while maintaining clarity.

</details>

---

## 4. Output Prediction (4 snippets)

**Snippet 1** — void* in C vs C++ (§16.2, p.158)

```cpp
// This is C++ code
#include <cstdlib>
int main() {
    int* p = malloc(sizeof(int));
    *p = 42;
    free(p);
}
```

<details><summary>Answer</summary>

**Compilation error in C++.** `malloc` returns `void*`, which does not implicitly convert to `int*` in C++. In C, this would compile. Fix: `int* p = static_cast<int*>(malloc(sizeof(int)));` (or better: use `new`/`delete`).

</details>

---

**Snippet 2** — auto deduction (§16.1.3, p.155)

```cpp
#include <iostream>
int main() {
    auto x = 42;
    auto y = 42.0;
    auto z = 42u;
    std::cout << sizeof(x) << " " << sizeof(y) << " " << sizeof(z) << '\n';
}
```

<details><summary>Answer</summary>

Output (typical): `4 8 4`. `x` is `int` (4 bytes), `y` is `double` (8 bytes), `z` is `unsigned int` (4 bytes). `auto` deduces the type from the literal suffix.

</details>

---

**Snippet 3** — C++17 if with init (§16.1.3, p.156)

```cpp
#include <map>
#include <string>
#include <iostream>
int main() {
    std::map<std::string, int> m = {{"a", 1}, {"b", 2}};
    if (auto it = m.find("b"); it != m.end())
        std::cout << it->second << '\n';
    else
        std::cout << "not found\n";
}
```

<details><summary>Answer</summary>

Output: `2`. The C++17 if-with-initializer declares `it` scoped to the if/else block. `"b"` is found with value 2.

</details>

---

**Snippet 4** — Structured bindings (§16.1.3, p.156)

```cpp
#include <map>
#include <iostream>
int main() {
    std::map<std::string, int> m = {{"x", 10}, {"y", 20}};
    for (const auto& [key, val] : m)
        std::cout << key << "=" << val << " ";
}
```

<details><summary>Answer</summary>

Output: `x=10 y=20 ` (map iterates in sorted key order). Structured bindings decompose each `pair<const string, int>`.

</details>

---

## 5. Debug This (3 snippets)

**Bug 1** — Using C-style cast in C++ (§16.2, p.158)

```cpp
#include <iostream>

class Base { public: virtual ~Base() {} };
class Derived : public Base { public: int x = 42; };

int main() {
    Base* b = new Derived();
    Derived* d = (Derived*)b;  // C-style cast
    std::cout << d->x << '\n';
    delete b;
}
```

<details><summary>Answer</summary>

**Bug:** The C-style cast `(Derived*)b` works here but is dangerous — it bypasses safety checks. If `b` didn't actually point to a `Derived`, it would silently succeed and cause UB.

**Fix:** Use `dynamic_cast<Derived*>(b)` which returns `nullptr` on failure, or `static_cast` if you're certain of the type.

</details>

---

**Bug 2** — C++11 feature used in C++03 mode (§16.1.3, p.155)

```cpp
// Compiled with -std=c++03
#include <vector>
int main() {
    auto v = std::vector<int>{1, 2, 3};
    for (auto x : v)
        std::cout << x << '\n';
}
```

<details><summary>Answer</summary>

**Bug:** `auto` type deduction, brace initialization, and range-for loops are C++11 features. Compiling with `-std=c++03` will fail.

**Fix:** Compile with `-std=c++11` or later.

</details>

---

**Bug 3** — Name mangling prevents C linkage (§16.2.1, p.159)

```cpp
// lib.cpp (compiled as C++)
int add(int a, int b) { return a + b; }

// main.c (compiled as C)
extern int add(int, int);
int main() { return add(1, 2); }
```

<details><summary>Answer</summary>

**Bug:** The C++ compiler mangles `add` to something like `_Z3addii`. The C linker looks for `add`. Linking fails: "undefined reference to `add`".

**Fix:** Declare with `extern "C"` in the C++ source:
```cpp
extern "C" int add(int a, int b) { return a + b; }
```

</details>

---

## 6. System Design / Open-ended (2 questions)

**Q1.** You're maintaining a large codebase that's still on C++11 and considering a migration to C++17. Which features would give you the most immediate benefit (developer productivity, safety, performance)? What are the risks of upgrading? How would you approach it incrementally? (§16.1.3, p.155–157)

---

**Q2.** Stroustrup's zero-overhead principle states "what you don't use, you don't pay for." Give three examples of C++ features that follow this principle and one that arguably violates it (at least in practice). How does this principle influence language design decisions? (§16.3, p.160)

---

## 7. Rapid Fire (10 true/false)

1. C++11 was the first standard to include move semantics. (§16.1.3, p.155)

<details><summary>Answer</summary>True. Move semantics (rvalue references, move constructors) were introduced in C++11.</details>

2. C++ is a strict superset of C. (§16.2, p.158)

<details><summary>Answer</summary>False. Most C code compiles as C++, but there are differences (e.g., `void*` conversion, variable-length arrays, designated initializers before C++20).</details>

3. `extern "C"` disables name mangling for the declared functions. (§16.2.1, p.159)

<details><summary>Answer</summary>True. This allows C++ functions to be called from C code.</details>

4. C++14 introduced concepts. (§16.1.3, p.156)

<details><summary>Answer</summary>False. Concepts were introduced in C++20.</details>

5. `std::optional` was introduced in C++17. (§16.1.3, p.156)

<details><summary>Answer</summary>True. Along with `variant`, `any`, and `string_view`.</details>

6. The zero-overhead principle means all C++ features are free to use. (§16.3, p.160)

<details><summary>Answer</summary>False. It means you don't pay for features you don't use. Features you do use are as efficient as hand-written alternatives.</details>

7. C++20 modules replace the `#include` preprocessor mechanism. (§16.1.3, p.157)

<details><summary>Answer</summary>True. Modules provide a modern alternative that's faster and avoids macro leakage.</details>

8. Generic lambdas (`[](auto x)`) were introduced in C++11. (§16.1.3, p.155)

<details><summary>Answer</summary>False. Generic lambdas were introduced in C++14.</details>

9. The `<=>` (spaceship) operator was added in C++20. (§16.1.3, p.157)

<details><summary>Answer</summary>True. It enables compiler-generated comparison operators.</details>

10. Stroustrup designed C++ at Bell Labs starting in 1979. (§16.1, p.153)

<details><summary>Answer</summary>True. Originally called "C with Classes," it was renamed C++ in 1983.</details>

---

## 8. Capstone Implementation Challenge

### C++ Standards Time Machine (§16.1–16.3, p.153–160)

**Motivation:** You're creating a teaching tool that demonstrates how the same problem is solved differently across C++ standards. Implement a "word frequency counter" four times — once in each of C++98, C++11, C++14, and C++17 style — all in a single C++17 file. This demonstrates the evolution of the language and the practical impact of each standard.

**The Task:**

Given a string of space-separated words, count the frequency of each word and return the top 3 most frequent words with their counts.

**Signatures:**

```cpp
namespace cpp98_style {
    std::vector<std::pair<std::string, int>> top3(const std::string& text);
}

namespace cpp11_style {
    std::vector<std::pair<std::string, int>> top3(const std::string& text);
}

namespace cpp14_style {
    std::vector<std::pair<std::string, int>> top3(const std::string& text);
}

namespace cpp17_style {
    std::vector<std::pair<std::string, int>> top3(const std::string& text);
}
```

**Test Scenario:**

All four versions produce the same output for input `"the cat sat on the mat the cat"`:
`{{"the", 3}, {"cat", 2}, {"mat", 1}}` (or any single-occurrence word for 3rd place).

**Constraints:**
- C++98 version: no `auto`, no range-for, no lambdas, explicit iterator types, `typedef`
- C++11 version: use `auto`, range-for, lambdas, `unordered_map`
- C++14 version: use generic lambdas, `make_unique`, return type deduction
- C++17 version: use structured bindings, `if` with init, `string_view` where possible
- Use `extern "C"` compatibility wrapper for the C++17 version's entry point
- Each version must be in its own namespace
- Comment each version with which features are specific to that standard

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <string>
#include <sstream>
#include <vector>
#include <map>
#include <unordered_map>
#include <algorithm>
#include <utility>
#include <string_view>

// ============================================================
// C++98 STYLE: verbose, explicit types, no auto/range-for/lambda
// ============================================================
namespace cpp98_style {

typedef std::pair<std::string, int> WordCount;  // typedef, not using
typedef std::vector<WordCount> WordCountVec;

// Comparison function (no lambdas in C++98)
bool compare_by_count(const WordCount& a, const WordCount& b) {
    return a.second > b.second;  // descending
}

WordCountVec top3(const std::string& text) {
    // Parse words
    std::istringstream iss(text);
    std::string word;
    std::map<std::string, int> freq;  // no unordered_map in C++98

    while (iss >> word) {
        // No emplace, no [], so:
        std::map<std::string, int>::iterator it = freq.find(word);
        if (it != freq.end())
            it->second++;
        else
            freq.insert(std::make_pair(word, 1));
    }

    // Copy to vector for sorting
    WordCountVec vec;
    for (std::map<std::string, int>::const_iterator it = freq.begin();
         it != freq.end(); ++it) {
        vec.push_back(std::make_pair(it->first, it->second));
    }

    // Sort with function pointer (no lambda)
    std::sort(vec.begin(), vec.end(), compare_by_count);

    // Truncate to 3
    if (vec.size() > 3)
        vec.resize(3);

    return vec;
}

}  // namespace cpp98_style

// ============================================================
// C++11 STYLE: auto, range-for, lambdas, unordered_map, move
// ============================================================
namespace cpp11_style {

std::vector<std::pair<std::string, int>> top3(const std::string& text) {
    std::istringstream iss(text);
    std::string word;
    std::unordered_map<std::string, int> freq;  // C++11: hash map

    while (iss >> word)
        ++freq[word];  // operator[] with increment

    // Move to vector
    std::vector<std::pair<std::string, int>> vec(freq.begin(), freq.end());

    // Lambda for comparison (C++11)
    std::sort(vec.begin(), vec.end(),
        [](const std::pair<std::string, int>& a,
           const std::pair<std::string, int>& b) {
            return a.second > b.second;
        });

    if (vec.size() > 3) vec.resize(3);
    return vec;  // move semantics: efficient return
}

}  // namespace cpp11_style

// ============================================================
// C++14 STYLE: generic lambdas, auto return type deduction
// ============================================================
namespace cpp14_style {

auto top3(const std::string& text) {  // C++14: return type deduced
    std::istringstream iss(text);
    std::string word;
    std::unordered_map<std::string, int> freq;

    while (iss >> word) ++freq[word];

    std::vector<std::pair<std::string, int>> vec(freq.begin(), freq.end());

    // C++14: generic lambda with auto parameters
    std::sort(vec.begin(), vec.end(),
        [](const auto& a, const auto& b) { return a.second > b.second; });

    if (vec.size() > 3) vec.resize(3);
    return vec;
}

}  // namespace cpp14_style

// ============================================================
// C++17 STYLE: structured bindings, if-init, string_view
// ============================================================
namespace cpp17_style {

std::vector<std::pair<std::string, int>> top3(const std::string& text) {
    std::istringstream iss(text);
    std::string word;
    std::unordered_map<std::string, int> freq;

    while (iss >> word) ++freq[word];

    std::vector<std::pair<std::string, int>> vec(freq.begin(), freq.end());

    // C++17: structured bindings in lambda
    std::sort(vec.begin(), vec.end(),
        [](const auto& a, const auto& b) { return a.second > b.second; });

    if (auto n = vec.size(); n > 3)  // C++17: if with initializer
        vec.resize(3);

    return vec;
}

// C++17: print using structured bindings
void print_results(const std::vector<std::pair<std::string, int>>& results) {
    for (const auto& [word, count] : results)  // C++17: structured binding
        std::cout << "  " << word << ": " << count << '\n';
}

}  // namespace cpp17_style

// extern "C" wrapper for C interop (§16.2.1)
extern "C" {
    void run_top3(const char* text) {
        auto results = cpp17_style::top3(text);
        cpp17_style::print_results(results);
    }
}

int main() {
    const std::string text = "the cat sat on the mat the cat";

    std::cout << "=== C++98 style ===\n";
    auto r98 = cpp98_style::top3(text);
    for (size_t i = 0; i < r98.size(); ++i)
        std::cout << "  " << r98[i].first << ": " << r98[i].second << '\n';

    std::cout << "\n=== C++11 style ===\n";
    for (const auto& p : cpp11_style::top3(text))
        std::cout << "  " << p.first << ": " << p.second << '\n';

    std::cout << "\n=== C++14 style ===\n";
    for (const auto& p : cpp14_style::top3(text))
        std::cout << "  " << p.first << ": " << p.second << '\n';

    std::cout << "\n=== C++17 style ===\n";
    cpp17_style::print_results(cpp17_style::top3(text));

    std::cout << "\n=== via extern C ===\n";
    run_top3(text.c_str());
}
```

Key decisions:
- **Namespaces** isolate each version, demonstrating the Chapter 3 concept of modularity (§16.1).
- **C++98**: no `auto`, explicit iterator types, `typedef`, function pointer comparator — shows the verbosity that motivated C++11 (§16.1.3).
- **C++11**: `auto`, lambdas, range-for, `unordered_map`, move semantics — each feature labeled (§16.1.3).
- **C++14**: generic lambdas (`auto` params), return type deduction — incremental refinement (§16.1.3).
- **C++17**: structured bindings, if-with-initializer — syntactic sugar that improves readability (§16.1.3).
- **`extern "C"`** wrapper demonstrates C/C++ interop — the function takes `const char*` (C-compatible) and uses C++ internally (§16.2.1).
- Each version is progressively shorter and more readable — demonstrating the language's evolution toward expressiveness.

</details>
