# Chapter 6: Templates

---

## Quick Reference

- **Function templates** define a family of functions parameterized by types: `template<typename T> T max(T a, T b)` (§6.2, p.61)
- **Class templates** define a family of classes: `template<typename T> class Vector { ... }` (§6.2, p.62)
- Template arguments can be **deduced** from function arguments: `max(3, 7)` deduces `T = int` (§6.2, p.61)
- **Non-type template parameters**: `template<typename T, int N> class Array { T elems[N]; }` (§6.2, p.63)
- Templates are instantiated at compile time — each unique set of arguments generates a new specialization (§6.2, p.61)
- **Template argument deduction** works for function templates; class templates got CTAD in C++17 (§6.2.3, p.64)
- **Variadic templates** use `template<typename... Ts>` for parameter packs (§6.2.4, p.65)
- **Fold expressions** (C++17) reduce parameter packs: `(args + ...)` (§6.2.4, p.65)
- **Template aliases**: `template<typename T> using Vec = std::vector<T>;` (§6.3, p.66)
- Function objects (functors) and lambdas work well with templates as policies (§6.3.2, p.67)
- Templates enable **generic programming** — writing code that works with any type meeting requirements (§6.1, p.61)
- `if constexpr` enables compile-time branching in templates (§6.2.5, p.66)

---

## 1. Multiple Choice (8 questions)

**Q1.** When is a function template instantiated? (§6.2, p.61)

a) At program startup
b) When the template is defined
c) When it is called with specific type arguments (at compile time)
d) At link time

<details><summary>Answer</summary>

**c)** Templates are instantiated at compile time when used with specific type arguments. Each unique combination of arguments produces a separate function.

</details>

---

**Q2.** What does CTAD stand for and when was it introduced? (§6.2.3, p.64)

a) Compile-Time Argument Deduction, C++11
b) Class Template Argument Deduction, C++17
c) Constexpr Template Auto Declaration, C++14
d) Class Type Auto Deduction, C++20

<details><summary>Answer</summary>

**b)** Class Template Argument Deduction allows `std::pair p(1, 2.0);` instead of `std::pair<int, double> p(1, 2.0);`. It was introduced in C++17.

</details>

---

**Q3.** What is the result of `(args + ...)` in a fold expression? (§6.2.4, p.65)

a) Concatenates all arguments as strings
b) Right fold: `arg1 + (arg2 + (arg3 + ... ))`
c) Always returns zero
d) Left fold: `(... + (arg1 + arg2)) + arg3`

<details><summary>Answer</summary>

**b)** `(args + ...)` is a right fold. It expands as `arg1 + (arg2 + (arg3))`. A left fold would be `(... + args)`.

</details>

---

**Q4.** What is a non-type template parameter? (§6.2, p.63)

a) A template parameter that is not used
b) A compile-time constant value (like `int N`) rather than a type
c) A parameter that accepts any type
d) A runtime function argument

<details><summary>Answer</summary>

**b)** Non-type parameters are compile-time constants: `template<int N>` lets you parameterize templates on values like array sizes. The value must be known at compile time.

</details>

---

**Q5.** Which of these is a valid template alias? (§6.3, p.66)

a) `typedef template<typename T> Vec = vector<T>;`
b) `template<typename T> using Vec = std::vector<T>;`
c) `#define Vec(T) std::vector<T>`
d) `template alias Vec = std::vector;`

<details><summary>Answer</summary>

**b)** `using` with `template` creates an alias template. This is the modern C++ way to create type aliases that depend on template parameters.

</details>

---

**Q6.** What does `if constexpr` do? (§6.2.5, p.66)

a) Same as regular `if` but faster
b) Evaluates the condition at compile time; the false branch is discarded (not compiled)
c) Only works inside `constexpr` functions
d) Replaces `#ifdef` preprocessor directives

<details><summary>Answer</summary>

**b)** `if constexpr` evaluates the condition at compile time. The branch not taken is discarded entirely, which is essential for template code where the discarded branch might not even be valid for the given type.

</details>

---

**Q7.** What happens if you call `max(3, 4.0)` on a template `template<typename T> T max(T a, T b)`? (§6.2, p.61)

a) `T` is deduced as `double`
b) `T` is deduced as `int`
c) Compilation error — ambiguous deduction
d) Runtime error

<details><summary>Answer</summary>

**c)** The compiler can't decide if `T` is `int` or `double` — deduction fails because both arguments must produce the same `T`. Fix: `max<double>(3, 4.0)` or use two template parameters.

</details>

---

**Q8.** Why are templates defined in header files rather than `.cpp` files? (§6.2, p.61)

a) Convention only — either works
b) The compiler needs the full template definition at each point of instantiation
c) Templates don't generate machine code
d) `.cpp` files cannot contain template code

<details><summary>Answer</summary>

**b)** Template instantiation happens at compile time in each translation unit that uses the template. Without the full definition visible, the compiler cannot generate the specialized code.

</details>

---

## 2. Fill-in-the-blank Code (5 snippets)

**Snippet 1** — Function template (§6.2, p.61)

```cpp
#include <iostream>

___<___ T>
T maximum(T a, T b) {
    return (a > b) ? a : b;
}

int main() {
    std::cout << maximum(3, 7) << '\n';       // deduced: int
    std::cout << maximum(3.14, 2.72) << '\n'; // deduced: double
}
```

<details><summary>Answer</summary>

```cpp
template<typename T>
T maximum(T a, T b) { ... }
```

</details>

---

**Snippet 2** — Class template with non-type parameter (§6.2, p.63)

```cpp
#include <iostream>

template<typename T, ___ N>  // N is a compile-time constant
class StaticArray {
    T data[N];
public:
    T& operator[](int i) { return data[i]; }
    constexpr int size() const { return ___; }
};

int main() {
    StaticArray<int, 5> arr;
    arr[0] = 42;
    std::cout << arr.size() << '\n';  // 5
}
```

<details><summary>Answer</summary>

```cpp
template<typename T, int N>
...
constexpr int size() const { return N; }
```

</details>

---

**Snippet 3** — Variadic template with fold expression (§6.2.4, p.65)

```cpp
#include <iostream>

template<typename... Ts>
auto sum(Ts... args) {
    return (___);  // fold expression to sum all args
}

int main() {
    std::cout << sum(1, 2, 3, 4, 5) << '\n';   // 15
    std::cout << sum(1.5, 2.5) << '\n';          // 4.0
}
```

<details><summary>Answer</summary>

```cpp
return (args + ...);
```

This is a right fold: expands to `1 + (2 + (3 + (4 + 5)))`.

</details>

---

**Snippet 4** — if constexpr (§6.2.5, p.66)

```cpp
#include <iostream>
#include <type_traits>

template<typename T>
void describe(T val) {
    ___ (std::is_integral_v<T>) {
        std::cout << val << " is an integer\n";
    } else {
        std::cout << val << " is not an integer\n";
    }
}

int main() {
    describe(42);      // integer
    describe(3.14);    // not an integer
}
```

<details><summary>Answer</summary>

```cpp
if constexpr (std::is_integral_v<T>) { ... }
```

</details>

---

**Snippet 5** — Template alias (§6.3, p.66)

```cpp
#include <vector>
#include <string>
#include <iostream>

template<typename T>
___ Vec = std::vector<T>;  // template alias

int main() {
    Vec<std::string> words = {"hello", "world"};
    for (const auto& w : words)
        std::cout << w << " ";
}
```

<details><summary>Answer</summary>

```cpp
template<typename T>
using Vec = std::vector<T>;
```

</details>

---

## 3. Code from Scratch — LeetCode Style (3 problems)

### Problem 1: Generic stack (§6.2, p.62)

Implement a class template `Stack<T>` with `push()`, `pop()`, `top()`, and `empty()` operations backed by a `std::vector<T>`.

```
Signature:
  template<typename T>
  class Stack {
  public:
      void push(const T& val);
      void pop();
      const T& top() const;
      bool empty() const;
  };
```

**Examples:**

| Operation | Result |
|---|---|
| `push(1), push(2), top()` | `2` |
| `push(1), pop(), empty()` | `true` |

**Constraints:** Use `std::vector` internally. Throw `std::runtime_error` on `top()`/`pop()` when empty.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <vector>
#include <stdexcept>

template<typename T>
class Stack {
    std::vector<T> elems_;  // vector handles memory management
public:
    void push(const T& val) {
        elems_.push_back(val);
    }

    void pop() {
        if (elems_.empty())
            throw std::runtime_error("pop on empty stack");
        elems_.pop_back();
    }

    const T& top() const {
        if (elems_.empty())
            throw std::runtime_error("top on empty stack");
        return elems_.back();
    }

    bool empty() const {
        return elems_.empty();
    }
};

int main() {
    Stack<int> s;
    s.push(1);
    s.push(2);
    std::cout << s.top() << '\n';  // 2
    s.pop();
    std::cout << s.top() << '\n';  // 1
    s.pop();
    std::cout << std::boolalpha << s.empty() << '\n';  // true
}
```

</details>

---

### Problem 2: Variadic print function (§6.2.4, p.65)

Write a variadic function template `print_all` that prints each argument separated by commas.

```
Signature: template<typename... Ts> void print_all(Ts... args);
```

**Examples:**

| Call | Output |
|---|---|
| `print_all(1, 2.5, "hi")` | `1, 2.5, hi` |
| `print_all(42)` | `42` |

**Constraints:** Use fold expressions. No trailing comma.

<details><summary>Solution</summary>

```cpp
#include <iostream>

// Helper: print with separator
template<typename T>
void print_one(const T& val, bool is_first) {
    if (!is_first) std::cout << ", ";
    std::cout << val;
}

// Variadic: expand pack with index trick using fold
template<typename First, typename... Rest>
void print_all(First first, Rest... rest) {
    std::cout << first;                          // print first without comma
    ((std::cout << ", " << rest), ...);          // fold: print each remaining with comma
    std::cout << '\n';
}

// Base case: single argument
template<typename T>
void print_all(T val) {
    std::cout << val << '\n';
}

int main() {
    print_all(1, 2.5, "hi");  // 1, 2.5, hi
    print_all(42);             // 42
}
```

Key decisions: The comma fold `((std::cout << ", " << rest), ...)` prints a comma before each remaining argument. Separating the first argument handles the "no leading comma" requirement.

</details>

---

### Problem 3: Compile-time array with non-type parameter (§6.2, p.63)

Implement `FixedArray<T, N>` — a stack-allocated array of `N` elements of type `T`. Provide `operator[]`, `size()`, `fill(T val)`, and a `begin()`/`end()` for range-for support.

```
Signature:
  template<typename T, int N>
  class FixedArray {
  public:
      T& operator[](int i);
      constexpr int size() const;
      void fill(const T& val);
      T* begin();
      T* end();
  };
```

**Examples:**

| Code | Output |
|---|---|
| `FixedArray<int,3> a; a.fill(7); a[1]` | `7` |
| Range-for prints all 3 elements | `7 7 7` |

**Constraints:** No heap allocation. Use a plain C array internally. `N` must be a compile-time constant.

<details><summary>Solution</summary>

```cpp
#include <iostream>

template<typename T, int N>
class FixedArray {
    T data_[N];  // stack-allocated — size fixed at compile time
public:
    T& operator[](int i) { return data_[i]; }
    const T& operator[](int i) const { return data_[i]; }

    constexpr int size() const { return N; }

    void fill(const T& val) {
        for (int i = 0; i < N; ++i)
            data_[i] = val;
    }

    // begin/end enable range-for
    T* begin() { return data_; }
    T* end() { return data_ + N; }
    const T* begin() const { return data_; }
    const T* end() const { return data_ + N; }
};

int main() {
    FixedArray<int, 3> a;
    a.fill(7);
    for (auto x : a)
        std::cout << x << " ";  // 7 7 7
    std::cout << '\n';
    std::cout << "size: " << a.size() << '\n';  // 3
}
```

</details>

---

## 4. Output Prediction (4 snippets)

**Snippet 1** — Template argument deduction (§6.2, p.61)

```cpp
#include <iostream>
template<typename T> T add(T a, T b) { return a + b; }
int main() {
    std::cout << add(1, 2) << '\n';
    std::cout << add(1.5, 2.5) << '\n';
}
```

<details><summary>Answer</summary>

Output:
```
3
4
```

First call deduces `T=int`, second `T=double`. `1.5 + 2.5 = 4.0`, printed as `4`.

</details>

---

**Snippet 2** — Non-type template parameter (§6.2, p.63)

```cpp
#include <iostream>
template<int N> int multiply(int x) { return x * N; }
int main() {
    std::cout << multiply<3>(7) << '\n';
    std::cout << multiply<0>(100) << '\n';
}
```

<details><summary>Answer</summary>

Output:
```
21
0
```

`multiply<3>` generates a function that multiplies by 3. `multiply<0>` always returns 0.

</details>

---

**Snippet 3** — Fold expression (§6.2.4, p.65)

```cpp
#include <iostream>
template<typename... Ts>
bool all_true(Ts... args) { return (args && ...); }
int main() {
    std::cout << std::boolalpha;
    std::cout << all_true(true, true, true) << '\n';
    std::cout << all_true(true, false, true) << '\n';
}
```

<details><summary>Answer</summary>

Output:
```
true
false
```

`(args && ...)` is a right fold: `true && (true && true)` = `true`; `true && (false && true)` = `false`.

</details>

---

**Snippet 4** — if constexpr with wrong branch (§6.2.5, p.66)

```cpp
#include <iostream>
#include <type_traits>
#include <string>

template<typename T>
std::string stringify(T val) {
    if constexpr (std::is_arithmetic_v<T>)
        return std::to_string(val);
    else
        return val;
}

int main() {
    std::cout << stringify(42) << '\n';
    std::cout << stringify(std::string("hello")) << '\n';
}
```

<details><summary>Answer</summary>

Output:
```
42
hello
```

For `int`, the first branch is taken; `to_string(42)` = `"42"`. For `string`, the else branch is taken; `val` is returned directly. Without `if constexpr`, this would fail to compile because `to_string(string)` is invalid.

</details>

---

## 5. Debug This (3 snippets)

**Bug 1** — Template definition in .cpp file (§6.2, p.61)

This code split across two files doesn't link:

```cpp
// max.h
template<typename T>
T my_max(T a, T b);

// max.cpp
template<typename T>
T my_max(T a, T b) { return (a > b) ? a : b; }

// main.cpp
#include "max.h"
int main() { return my_max(1, 2); }
```

<details><summary>Answer</summary>

**Bug:** The template definition is in `max.cpp`, but `main.cpp` only sees the declaration in the header. The compiler cannot instantiate `my_max<int>` without the definition.

**Fix:** Move the template definition into the header file `max.h`.

</details>

---

**Bug 2** — Ambiguous template deduction (§6.2, p.61)

```cpp
template<typename T>
T add(T a, T b) { return a + b; }

int main() {
    auto result = add(1, 2.5);
}
```

<details><summary>Answer</summary>

**Bug:** `T` is deduced as `int` from the first argument and `double` from the second — ambiguous. Template argument deduction requires consistent deduction for each parameter.

**Fix:** Either specify explicitly `add<double>(1, 2.5)` or use two template parameters: `template<typename T, typename U>`.

</details>

---

**Bug 3** — Forgetting `typename` for dependent type (§6.2, p.62)

```cpp
template<typename Container>
void print_first(const Container& c) {
    Container::iterator it = c.begin();
    std::cout << *it << '\n';
}
```

<details><summary>Answer</summary>

**Bug:** `Container::iterator` is a **dependent name** — the compiler doesn't know it's a type. Without `typename`, it's parsed as a value.

**Fix:** `typename Container::iterator it = c.begin();` (or use `auto`).

</details>

---

## 6. System Design / Open-ended (2 questions)

**Q1.** You want to write a generic `serialize` function that converts any type to a string. For arithmetic types, use `std::to_string`. For string-like types, return as-is. For containers, iterate and concatenate. How would you use `if constexpr` and type traits to dispatch? What happens when someone passes a type you haven't handled? (§6.2.5, p.66)

---

**Q2.** Compare templates vs. virtual functions for achieving polymorphism. Templates give you compile-time polymorphism with no vtable overhead. Virtual functions give runtime polymorphism at the cost of indirection. When would you choose one over the other? Give a concrete example where templates are superior and one where virtual functions are. (§6.1, p.61; §6.3.2, p.67)

---

## 7. Rapid Fire (10 true/false)

1. Template code is compiled once and reused for all types. (§6.2, p.61)

<details><summary>Answer</summary>False. Each unique instantiation generates separate code — `max<int>` and `max<double>` are different functions.</details>

2. `template<typename T>` and `template<class T>` are identical. (§6.2, p.61)

<details><summary>Answer</summary>True. `typename` and `class` are interchangeable in template parameter declarations.</details>

3. Non-type template parameters must be compile-time constants. (§6.2, p.63)

<details><summary>Answer</summary>True. Values like `int N` must be known at compile time.</details>

4. CTAD allows `std::vector v = {1, 2, 3};` without specifying `<int>`. (§6.2.3, p.64)

<details><summary>Answer</summary>True. C++17 class template argument deduction infers the type from the initializer.</details>

5. Variadic templates can accept zero arguments. (§6.2.4, p.65)

<details><summary>Answer</summary>True. A parameter pack `Ts...` can be empty.</details>

6. `if constexpr` evaluates both branches at compile time. (§6.2.5, p.66)

<details><summary>Answer</summary>False. Only the taken branch is instantiated; the other is discarded.</details>

7. Template definitions must be in header files. (§6.2, p.61)

<details><summary>Answer</summary>True (in practice). The compiler needs the full definition at each point of instantiation.</details>

8. `(args * ...)` is a valid fold expression for multiplication. (§6.2.4, p.65)

<details><summary>Answer</summary>True. Any binary operator can be used in a fold expression.</details>

9. Template aliases created with `using` can be specialized. (§6.3, p.66)

<details><summary>Answer</summary>False. Alias templates cannot be partially or explicitly specialized.</details>

10. A lambda can be passed as a template argument. (§6.3.2, p.67)

<details><summary>Answer</summary>True. Lambdas are objects with a unique type and can be passed to function templates.</details>
