# Chapter 3: Modularity

---

## Quick Reference

- **Separate compilation** divides a program into translation units; headers declare, `.cpp` files define (§3.1, p.25)
- **Namespaces** group related declarations and prevent name clashes: `namespace Foo { ... }` (§3.3, p.28)
- `using namespace std;` brings all names from `std` into scope — use sparingly (§3.3, p.29)
- **Exceptions** separate error detection from error handling: `throw`, `try`/`catch` (§3.4, p.30)
- `std::runtime_error`, `std::out_of_range`, etc. are standard exception types (§3.4.1, p.31)
- An exception unwinds the stack, destroying local objects — RAII ensures cleanup (§3.4.2, p.32)
- **Invariants** are conditions that a class must maintain; constructors establish them, exceptions signal violations (§3.4.2, p.32)
- `static_assert(expr, msg)` checks conditions at compile time (§3.4.3, p.33)
- **Function arguments**: pass small/simple types by value, others by `const&`; use move for large objects you consume (§3.5, p.34)
- Return by value is efficient thanks to move semantics and copy elision (§3.5.1, p.34)
- **Structured bindings** (C++17): `auto [x, y] = pair;` unpacks aggregates (§3.5.3, p.36)
- Header files should contain declarations and inline definitions; avoid defining non-inline functions in headers to prevent ODR violations (§3.2, p.27)

---

## 1. Multiple Choice (8 questions)

**Q1.** What is the primary purpose of namespaces? (§3.3, p.28)

a) To speed up compilation
b) To prevent name collisions between independently developed libraries
c) To enforce access control like `private`
d) To enable dynamic dispatch

<details><summary>Answer</summary>

**b)** Namespaces organize code and prevent name collisions. Two libraries can both define `sort()` without conflict if each is in its own namespace.

</details>

---

**Q2.** What happens when an exception is thrown and not caught? (§3.4, p.30)

a) The program continues silently
b) `std::terminate()` is called
c) The exception is ignored
d) It causes a segmentation fault

<details><summary>Answer</summary>

**b)** An uncaught exception calls `std::terminate()`, which by default calls `std::abort()` to end the program.

</details>

---

**Q3.** Which is true about `static_assert`? (§3.4.3, p.33)

a) It is evaluated at runtime
b) It can test any boolean expression including runtime variables
c) It is evaluated at compile time and produces a compile error if false
d) It only works inside functions

<details><summary>Answer</summary>

**c)** `static_assert` checks a constant expression at compile time. If the condition is false, compilation fails with the provided message.

</details>

---

**Q4.** What does stack unwinding guarantee? (§3.4.2, p.32)

a) All heap memory is freed
b) Local objects with destructors are properly destroyed
c) Catch blocks are optional
d) Global state is rolled back

<details><summary>Answer</summary>

**b)** When an exception propagates, the runtime destroys all local objects in each stack frame. This is why RAII (resource acquisition is initialization) works — destructors release resources during unwinding.

</details>

---

**Q5.** Which is the recommended way to pass a large read-only object to a function? (§3.5, p.34)

a) By value
b) By pointer
c) By `const` reference
d) By `mutable` reference

<details><summary>Answer</summary>

**c)** `const T&` avoids copying while preventing modification. For large objects, this is more efficient than pass-by-value and safer than raw pointers.

</details>

---

**Q6.** What does this structured binding do? (§3.5.3, p.36)

```cpp
auto [name, age] = std::make_pair(std::string("Alice"), 30);
```

a) Creates a pair and copies it into two new variables
b) Creates references to the pair's members
c) Declares `name` and `age` as new variables initialized from the pair
d) Fails to compile

<details><summary>Answer</summary>

**c)** Structured bindings decompose an aggregate (pair, tuple, struct) into named variables. `name` is a `std::string` and `age` is an `int`.

</details>

---

**Q7.** Which exception type would you throw for an out-of-range index? (§3.4.1, p.31)

a) `std::runtime_error`
b) `std::logic_error`
c) `std::out_of_range`
d) `std::bad_alloc`

<details><summary>Answer</summary>

**c)** `std::out_of_range` is specifically designed for index/range violations. It inherits from `std::logic_error`.

</details>

---

**Q8.** Why should you avoid `using namespace std;` in header files? (§3.3, p.29)

a) It slows down compilation
b) It pollutes the namespace of every file that includes the header
c) It is not valid C++
d) It disables ADL (argument-dependent lookup)

<details><summary>Answer</summary>

**b)** Any file that includes the header will have all `std` names injected into its global scope, causing potential name collisions the includer did not expect.

</details>

---

## 2. Fill-in-the-blank Code (5 snippets)

**Snippet 1** — Exception handling (§3.4, p.30)

```cpp
#include <iostream>
#include <stdexcept>

double divide(double a, double b) {
    if (b == 0)
        ___ std::___("division by zero");  // throw an exception
    return a / b;
}

int main() {
    ___ {
        std::cout << divide(10, 0) << '\n';
    } ___ (const std::exception& e) {
        std::cerr << "Error: " << e.___() << '\n';
    }
}
```

<details><summary>Answer</summary>

```cpp
throw std::runtime_error("division by zero");
try {
    ...
} catch (const std::exception& e) {
    std::cerr << "Error: " << e.what() << '\n';
}
```

</details>

---

**Snippet 2** — Namespace definition and use (§3.3, p.28)

```cpp
#include <iostream>

___ Math {
    constexpr double pi = 3.14159265;
    double circle_area(double r) { return pi * r * r; }
}

int main() {
    std::cout << ___::circle_area(5.0) << '\n';
}
```

<details><summary>Answer</summary>

```cpp
namespace Math { ... }
std::cout << Math::circle_area(5.0) << '\n';
```

</details>

---

**Snippet 3** — Structured bindings (§3.5.3, p.36)

```cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<std::string, int> ages = {{"Alice", 30}, {"Bob", 25}};
    for (___ [name, age] : ages) {
        std::cout << name << " is " << age << '\n';
    }
}
```

<details><summary>Answer</summary>

```cpp
for (const auto& [name, age] : ages) {
```

`const auto&` avoids copying each map entry while preventing modification.

</details>

---

**Snippet 4** — static_assert (§3.4.3, p.33)

```cpp
#include <type_traits>

template<typename T>
T safe_add(T a, T b) {
    ___(___, "safe_add requires an arithmetic type");
    return a + b;
}

int main() {
    auto x = safe_add(3, 4);       // OK
    // auto y = safe_add("a", "b"); // would fail static_assert
}
```

<details><summary>Answer</summary>

```cpp
static_assert(std::is_arithmetic_v<T>, "safe_add requires an arithmetic type");
```

</details>

---

**Snippet 5** — Returning by value with move semantics (§3.5.1, p.34)

```cpp
#include <vector>
#include <iostream>

std::vector<int> make_sequence(int n) {
    std::vector<int> v;
    for (int i = 0; i < n; ++i)
        v.___(___)    ;  // add i to the back
    return ___;            // return by value — move or copy elision
}

int main() {
    auto seq = make_sequence(5);
    for (auto x : seq)
        std::cout << x << " ";
}
```

<details><summary>Answer</summary>

```cpp
v.push_back(i);
return v;
```

Returning a local variable by value triggers move semantics or (more likely) copy elision (NRVO).

</details>

---

## 3. Code from Scratch — LeetCode Style (3 problems)

### Problem 1: Safe vector access with exceptions (§3.4, p.30)

Write a function `safe_get` that takes a `const std::vector<int>&` and an index, and returns the element. If the index is out of range, throw `std::out_of_range` with a descriptive message.

```
Signature: int safe_get(const std::vector<int>& v, int index);
```

**Examples:**

| Input | Output |
|---|---|
| `v={1,2,3}, index=1` | `2` |
| `v={1,2,3}, index=5` | throws `out_of_range` |
| `v={}, index=0` | throws `out_of_range` |

**Constraints:** Must throw `std::out_of_range`. Do not use `vector::at()` — implement the check yourself.

<details><summary>Solution</summary>

```cpp
#include <vector>
#include <stdexcept>
#include <iostream>

int safe_get(const std::vector<int>& v, int index) {
    // Manually check bounds instead of relying on at()
    if (index < 0 || static_cast<size_t>(index) >= v.size())
        throw std::out_of_range("index " + std::to_string(index) + " out of range");
    return v[index];  // operator[] is fast — no double-checking
}

int main() {
    std::vector<int> v = {10, 20, 30};
    try {
        std::cout << safe_get(v, 1) << '\n';  // 20
        std::cout << safe_get(v, 5) << '\n';  // throws
    } catch (const std::out_of_range& e) {
        std::cerr << "Caught: " << e.what() << '\n';
    }
}
```

</details>

---

### Problem 2: Namespace-based math library (§3.3, p.28)

Create a namespace `Geometry` containing functions `circle_area(double r)`, `rect_area(double w, double h)`, and a constant `pi`. Write a `main` that calls them using qualified names.

```
Signatures:
  namespace Geometry {
      constexpr double pi;
      double circle_area(double r);
      double rect_area(double w, double h);
  }
```

**Examples:**

| Call | Output |
|---|---|
| `Geometry::circle_area(1.0)` | `~3.14159` |
| `Geometry::rect_area(3, 4)` | `12.0` |

**Constraints:** Use `constexpr` for `pi`. Do not use `using namespace Geometry`.

<details><summary>Solution</summary>

```cpp
#include <iostream>

namespace Geometry {
    constexpr double pi = 3.14159265358979;

    double circle_area(double r) {
        return pi * r * r;  // pi*r^2
    }

    double rect_area(double w, double h) {
        return w * h;
    }
}

int main() {
    // Qualified names — clear and unambiguous
    std::cout << Geometry::circle_area(1.0) << '\n';  // 3.14159...
    std::cout << Geometry::rect_area(3, 4) << '\n';   // 12
}
```

</details>

---

### Problem 3: Structured binding unpacker (§3.5.3, p.36)

Write a function that takes a `std::vector<std::pair<std::string, int>>` representing student grades and returns the name of the student with the highest grade. Use structured bindings in the implementation.

```
Signature: std::string top_student(const std::vector<std::pair<std::string, int>>& grades);
```

**Examples:**

| Input | Output |
|---|---|
| `{{"Alice",90},{"Bob",95},{"Carol",88}}` | `"Bob"` |
| `{{"Solo",100}}` | `"Solo"` |

**Constraints:** Use structured bindings (`auto [name, grade]`). Throw if the vector is empty.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <stdexcept>

std::string top_student(const std::vector<std::pair<std::string, int>>& grades) {
    if (grades.empty())
        throw std::invalid_argument("no students");

    std::string best_name;
    int best_grade = -1;

    // Structured binding decomposes each pair
    for (const auto& [name, grade] : grades) {
        if (grade > best_grade) {
            best_grade = grade;
            best_name = name;
        }
    }
    return best_name;
}

int main() {
    std::vector<std::pair<std::string, int>> grades = {
        {"Alice", 90}, {"Bob", 95}, {"Carol", 88}
    };
    std::cout << top_student(grades) << '\n';  // Bob
}
```

</details>

---

## 4. Output Prediction (4 snippets)

**Snippet 1** (§3.4, p.30)

```cpp
#include <iostream>
#include <stdexcept>
int main() {
    try {
        throw std::runtime_error("oops");
        std::cout << "after throw\n";
    } catch (const std::exception& e) {
        std::cout << e.what() << '\n';
    }
    std::cout << "done\n";
}
```

<details><summary>Answer</summary>

Output:
```
oops
done
```

`throw` immediately transfers control to the `catch` block; "after throw" is never reached. Execution continues normally after the `try`/`catch`.

</details>

---

**Snippet 2** (§3.3, p.28)

```cpp
#include <iostream>
namespace A { int x = 1; }
namespace B { int x = 2; }
int main() {
    using namespace A;
    using namespace B;
    std::cout << x << '\n';
}
```

<details><summary>Answer</summary>

**Compilation error.** Both `A::x` and `B::x` are brought into scope, making `x` ambiguous. This demonstrates why `using namespace` can cause problems.

</details>

---

**Snippet 3** (§3.5.3, p.36)

```cpp
#include <iostream>
#include <tuple>
int main() {
    auto [a, b, c] = std::make_tuple(1, 2.5, 'z');
    std::cout << a << " " << b << " " << c << '\n';
}
```

<details><summary>Answer</summary>

Output: `1 2.5 z`. Structured bindings decompose the tuple: `a` is `int`, `b` is `double`, `c` is `char`.

</details>

---

**Snippet 4** (§3.4.2, p.32)

```cpp
#include <iostream>
#include <stdexcept>

struct Loud {
    std::string name;
    Loud(std::string n) : name{n} { std::cout << "+" << name << '\n'; }
    ~Loud() { std::cout << "-" << name << '\n'; }
};

void f() {
    Loud a("a");
    Loud b("b");
    throw std::runtime_error("bang");
}

int main() {
    try { f(); }
    catch (...) { std::cout << "caught\n"; }
}
```

<details><summary>Answer</summary>

Output:
```
+a
+b
-b
-a
caught
```

Stack unwinding destroys `b` then `a` (reverse order of construction) before the `catch` block runs. This demonstrates RAII-based cleanup during exception propagation.

</details>

---

## 5. Debug This (3 snippets)

**Bug 1** — Catching exception by value causes slicing (§3.4, p.30)

This code is supposed to catch a derived exception:

```cpp
#include <iostream>
#include <stdexcept>

int main() {
    try {
        throw std::out_of_range("bad index");
    } catch (std::exception e) {
        std::cout << e.what() << '\n';
    }
}
```

<details><summary>Answer</summary>

**Bug:** Catching by value (`std::exception e`) slices the `out_of_range` object, losing the derived class information. On some compilers, `what()` may still work, but this is fragile and invokes the copy constructor needlessly.

**Fix:** Catch by const reference: `catch (const std::exception& e)`.

</details>

---

**Bug 2** — Wrong catch order (§3.4.1, p.31)

```cpp
#include <iostream>
#include <stdexcept>

int main() {
    try {
        throw std::out_of_range("oops");
    } catch (const std::exception& e) {
        std::cout << "generic\n";
    } catch (const std::out_of_range& e) {
        std::cout << "out of range\n";
    }
}
```

<details><summary>Answer</summary>

**Bug:** `std::exception` is a base class of `std::out_of_range`. The first catch matches, so the more specific handler is unreachable. Most compilers warn about this.

**Fix:** Put the most derived catch first:
```cpp
catch (const std::out_of_range& e) { ... }
catch (const std::exception& e) { ... }
```

</details>

---

**Bug 3** — Forgetting to include `<stdexcept>` (§3.4.1, p.31)

```cpp
#include <iostream>

int main() {
    throw std::runtime_error("something went wrong");
}
```

<details><summary>Answer</summary>

**Bug:** `std::runtime_error` is defined in `<stdexcept>`, which is not included. This causes a compile error.

**Fix:** Add `#include <stdexcept>`.

</details>

---

## 6. System Design / Open-ended (2 questions)

**Q1.** You're building a library with three modules: `Parser`, `Evaluator`, and `Formatter`. Each module has its own types and functions. How would you use namespaces to organize them? Would you use nested namespaces (`Lib::Parser`)? What if two modules need to share a common type — where does it live? (§3.3, p.28)

---

**Q2.** Stroustrup argues that exceptions and RAII are complementary. Explain how returning error codes (the C way) can lead to resource leaks, and how throwing exceptions combined with destructors solves this. What happens if a destructor itself throws during stack unwinding? (§3.4.2, p.32)

---

## 7. Rapid Fire (10 true/false)

1. `using namespace std;` is safe to use in header files. (§3.3, p.29)

<details><summary>Answer</summary>False. It pollutes the namespace of every translation unit that includes the header.</details>

2. `static_assert` can use variables computed at runtime. (§3.4.3, p.33)

<details><summary>Answer</summary>False. `static_assert` requires a constant expression evaluable at compile time.</details>

3. Stack unwinding calls destructors in reverse order of construction. (§3.4.2, p.32)

<details><summary>Answer</summary>True. Destructors are called in reverse construction order as the stack unwinds.</details>

4. `throw;` inside a catch block re-throws the current exception. (§3.4, p.30)

<details><summary>Answer</summary>True. `throw;` with no operand re-throws the currently handled exception.</details>

5. Returning a local `std::vector` by value always copies it. (§3.5.1, p.34)

<details><summary>Answer</summary>False. Move semantics and copy elision (NRVO) typically eliminate the copy entirely.</details>

6. `auto [a, b] = std::pair(1, 2);` is valid C++17. (§3.5.3, p.36)

<details><summary>Answer</summary>True. Structured bindings can decompose pairs (and tuples, structs, arrays).</details>

7. Namespaces can be nested: `namespace A::B { }` is valid since C++17. (§3.3, p.28)

<details><summary>Answer</summary>True. C++17 allows nested namespace definitions with `::` syntax.</details>

8. `catch (...)` catches all exceptions, including non-class types like `int`. (§3.4, p.30)

<details><summary>Answer</summary>True. The ellipsis catch handler catches any thrown exception, regardless of type.</details>

9. If no exception is thrown, `try`/`catch` has zero runtime cost. (§3.4, p.30)

<details><summary>Answer</summary>True (on most modern implementations). The "zero-cost" exception model has no overhead on the normal path; cost is incurred only when an exception is actually thrown.</details>

10. A structured binding `auto [x, y] = s;` requires `s` to be a `std::pair`. (§3.5.3, p.36)

<details><summary>Answer</summary>False. Structured bindings work with any aggregate: pairs, tuples, structs, and arrays.</details>

---

## 8. Capstone Implementation Challenge

### Mini Test Framework (§3.3–3.5, p.28–36)

**Motivation:** You're building a lightweight test runner for a C++ project. It should let users register test cases (name + function), run all tests, catch exceptions thrown by failing tests, and report pass/fail with structured results.

**Signatures:**

```cpp
namespace testing {

struct TestResult {
    std::string name;
    bool passed;
    std::string error_message;  // empty if passed
};

class TestRunner {
public:
    using TestFunc = std::function<void()>;

    void add_test(const std::string& name, TestFunc func);
    std::vector<TestResult> run_all();
    void print_report(const std::vector<TestResult>& results) const;
};

// Assertion helpers
void assert_true(bool condition, const std::string& msg = "assertion failed");
void assert_equal(int expected, int actual);

}  // namespace testing
```

**Test Scenarios:**

1. Register 3 tests: one passes, one throws `std::runtime_error`, one fails an assertion. `run_all()` returns 3 results: 1 pass, 2 fail.
2. `print_report` outputs formatted results with pass/fail counts.
3. An exception thrown inside a test is caught and recorded — it doesn't crash the runner.

**Constraints:**
- Use a namespace (`testing::`) for all components
- Use `try`/`catch` to isolate test failures
- Use structured bindings to unpack `TestResult`
- Throw standard exceptions (`std::runtime_error`)
- Functions take arguments by `const&` where appropriate

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <functional>
#include <stdexcept>

namespace testing {

struct TestResult {
    std::string name;
    bool passed;
    std::string error_message;
};

// Assertion helpers — throw on failure
void assert_true(bool condition, const std::string& msg = "assertion failed") {
    if (!condition)
        throw std::runtime_error(msg);
}

void assert_equal(int expected, int actual) {
    if (expected != actual)
        throw std::runtime_error(
            "expected " + std::to_string(expected) +
            " but got " + std::to_string(actual));
}

class TestRunner {
public:
    using TestFunc = std::function<void()>;

private:
    // Each registered test is a name + callable
    struct TestCase {
        std::string name;
        TestFunc func;
    };
    std::vector<TestCase> tests_;

public:
    void add_test(const std::string& name, TestFunc func) {
        tests_.push_back({name, std::move(func)});
    }

    // Run all tests, catching exceptions to isolate failures
    std::vector<TestResult> run_all() {
        std::vector<TestResult> results;
        for (const auto& test : tests_) {
            try {
                test.func();  // run the test
                results.push_back({test.name, true, ""});
            } catch (const std::exception& e) {
                // Test failed — record the error without crashing
                results.push_back({test.name, false, e.what()});
            } catch (...) {
                results.push_back({test.name, false, "unknown exception"});
            }
        }
        return results;
    }

    // Print a formatted report using structured bindings
    void print_report(const std::vector<TestResult>& results) const {
        int pass_count = 0, fail_count = 0;

        for (const auto& [name, passed, error] : results) {
            if (passed) {
                std::cout << "  PASS: " << name << '\n';
                ++pass_count;
            } else {
                std::cout << "  FAIL: " << name << " — " << error << '\n';
                ++fail_count;
            }
        }

        std::cout << "\nResults: " << pass_count << " passed, "
                  << fail_count << " failed, "
                  << results.size() << " total\n";
    }
};

}  // namespace testing

// ---- Usage ----
int main() {
    testing::TestRunner runner;

    // Test 1: passes
    runner.add_test("addition works", []() {
        testing::assert_equal(4, 2 + 2);
    });

    // Test 2: fails with runtime_error
    runner.add_test("always fails", []() {
        throw std::runtime_error("intentional failure");
    });

    // Test 3: assertion failure
    runner.add_test("wrong result", []() {
        testing::assert_equal(5, 2 + 2);  // 4 != 5
    });

    auto results = runner.run_all();
    runner.print_report(results);
}
```

Expected output:
```
  PASS: addition works
  FAIL: always fails — intentional failure
  FAIL: wrong result — expected 5 but got 4

Results: 1 passed, 2 failed, 3 total
```

Key decisions:
- **Namespace `testing`** encapsulates the entire framework (§3.3).
- **`try`/`catch`** in `run_all()` isolates each test — one failure doesn't abort others (§3.4).
- **Structured bindings** in `print_report` make the loop readable (§3.5.3).
- **`const&` parameters** avoid copying strings and vectors (§3.5).
- **Returning `vector<TestResult>` by value** is efficient thanks to move semantics (§3.5.1).

</details>
