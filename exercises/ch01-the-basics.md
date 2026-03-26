# Chapter 1: The Basics

---

## Quick Reference

- Every C++ program must have exactly one `main()` function; it returns `int` (§1.1, p.1)
- **Types** define the set of possible values and operations: `bool`, `char`, `int`, `double`, `unsigned` (§1.4, p.5)
- Use `auto` to let the compiler deduce the type from the initializer: `auto x = 3.14;` (§1.4.2, p.6)
- **Initialization** forms: `int x = 7;`, `int x{7};` (narrowing-checked), `int x(7);` (§1.4, p.5)
- Brace initialization `{}` prevents narrowing conversions — `int x{2.3};` is a compile error (§1.4, p.6)
- **References** (`T&`) are aliases; **pointers** (`T*`) hold addresses. A reference cannot be reseated (§1.7, p.11)
- The **scope resolution operator** `::` accesses namespace or class members: `std::cout` (§1.2.3, p.4)
- `nullptr` is the typed null pointer literal — prefer it over `0` or `NULL` (§1.7, p.11)
- **Range-for**: `for (auto& x : v)` iterates over every element of `v` (§1.7, p.10)
- `constexpr` means "evaluated at compile time"; `const` means "will not be modified" (§1.6, p.9)
- `if`, `switch`, and loops work as in C; C++17 adds init-statements in `if`: `if (auto n = v.size(); n > 0)` (§1.8, p.13)
- Separate compilation: declarations in headers (`.h`), definitions in source files (`.cpp`) (§1.2.2, p.3)

---

## 1. Multiple Choice (8 questions)

**Q1.** Which initialization syntax prevents narrowing conversions? (§1.4, p.6)

a) `double d = 7;`
b) `int i(3.9);`
c) `int i{3.9};`
d) `int i = 3.9;`

<details><summary>Answer</summary>

**c)** Brace initialization `{}` checks for narrowing conversions and produces a compile-time error when one would occur. Both `=` and `()` silently truncate 3.9 to 3.

</details>

---

**Q2.** What is the value of `b` after these lines? (§1.4, p.5)

```cpp
bool b = 42;
```

a) `42`
b) `true`
c) `false`
d) Undefined behavior

<details><summary>Answer</summary>

**b)** Any non-zero integer converts to `true`. This is an implicit narrowing conversion. With `bool b{42};` some compilers may warn.

</details>

---

**Q3.** Which statement about `auto` is correct? (§1.4.2, p.6)

a) `auto` can be used without an initializer
b) `auto` always deduces a reference type
c) `auto` deduces the type from the initializer expression
d) `auto` is only valid for local variables in loops

<details><summary>Answer</summary>

**c)** The compiler uses the initializer to determine the type. Without an initializer, `auto` cannot deduce anything and is ill-formed.

</details>

---

**Q4.** What is the difference between `const` and `constexpr`? (§1.6, p.9)

a) They are synonyms
b) `constexpr` is evaluated at compile time; `const` simply means "I promise not to modify"
c) `const` is evaluated at compile time; `constexpr` is a runtime promise
d) `constexpr` can only apply to integers

<details><summary>Answer</summary>

**b)** `constexpr` requires the value to be computable at compile time. `const` merely prevents modification after initialization and the value may be determined at runtime.

</details>

---

**Q5.** What does this print? (§1.7, p.11)

```cpp
int x = 10;
int* p = &x;
int& r = x;
r = 20;
std::cout << *p;
```

a) `10`
b) `20`
c) Address of `x`
d) Undefined behavior

<details><summary>Answer</summary>

**b)** `r` is an alias for `x`, so `r = 20` modifies `x`. Since `p` points to `x`, `*p` reads the updated value `20`.

</details>

---

**Q6.** Which of these is a valid use of `nullptr`? (§1.7, p.11)

a) `int n = nullptr;`
b) `int* p = nullptr;`
c) `double d = nullptr;`
d) `bool b = nullptr;`

<details><summary>Answer</summary>

**b)** `nullptr` is of type `std::nullptr_t` and converts to any pointer type but not to integral or floating-point types.

</details>

---

**Q7.** What is wrong with this range-for loop? (§1.7, p.10)

```cpp
std::vector<int> v = {1,2,3};
for (auto x : v) x *= 2;
// expect v to contain {2,4,6}
```

a) `auto` should be `int`
b) `x` is a copy, so `v` is unchanged
c) Range-for cannot be used with `std::vector`
d) `x *= 2` is undefined for `auto`

<details><summary>Answer</summary>

**b)** `auto x` deduces by value, so each `x` is a copy of the element. To modify the vector, use `auto& x`.

</details>

---

**Q8.** In C++, what does separate compilation give you? (§1.2.2, p.3)

a) Faster runtime performance
b) The ability to compile parts of a program independently and link them later
c) Automatic optimization across translation units
d) Dynamic linking at runtime

<details><summary>Answer</summary>

**b)** Separate compilation allows different `.cpp` files to be compiled independently. The linker combines the resulting object files. This reduces rebuild times and supports modular development.

</details>

---

## 2. Fill-in-the-blank Code (5 snippets)

**Snippet 1** — Brace initialization and type deduction (§1.4, p.5–6)

```cpp
#include <iostream>

int main() {
    ___ pi = 3.14159;          // deduce type from initializer
    int narrow ___ 2.7 ___;    // brace-init: should this compile?
    std::cout << pi << '\n';
}
```

<details><summary>Answer</summary>

```cpp
auto pi = 3.14159;
int narrow{2.7};    // ERROR: narrowing conversion from double to int
```

The brace initialization `{2.7}` prevents silent truncation and produces a compile error.

</details>

---

**Snippet 2** — Pointers and references (§1.7, p.11)

```cpp
#include <iostream>

void increment(int___ val) {   // pass by reference
    val += 1;
}

int main() {
    int x = 5;
    int___ p = ___x;           // pointer to x
    increment(x);
    std::cout << x << " " << ___p << '\n';  // dereference
}
```

<details><summary>Answer</summary>

```cpp
void increment(int& val) { val += 1; }

int main() {
    int x = 5;
    int* p = &x;
    increment(x);
    std::cout << x << " " << *p << '\n';  // prints 6 6
}
```

</details>

---

**Snippet 3** — Range-for with modification (§1.7, p.10)

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    for (___ elem : v)  // must modify elements in place
        elem *= 10;
    for (___ elem : v)  // read-only traversal
        std::cout << elem << " ";
}
```

<details><summary>Answer</summary>

```cpp
for (auto& elem : v)       // reference to modify in place
    elem *= 10;
for (const auto& elem : v) // const reference for read-only
    std::cout << elem << " ";
```

</details>

---

**Snippet 4** — constexpr function (§1.6, p.9)

```cpp
#include <iostream>

___ int square(int x) {  // evaluated at compile time
    return x * x;
}

int main() {
    ___ int val = square(5);  // must be compile-time constant
    int arr[val];              // valid: val is constexpr
    std::cout << sizeof(arr) / sizeof(int) << '\n';
}
```

<details><summary>Answer</summary>

```cpp
constexpr int square(int x) { return x * x; }
constexpr int val = square(5);
```

`constexpr` on the function allows compile-time evaluation; `constexpr` on `val` guarantees the result is a constant expression usable as an array size.

</details>

---

**Snippet 5** — if with initializer (C++17) (§1.8, p.13)

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {10, 20, 30};
    if (___ n = v.size(); ___) {  // declare n, then test
        std::cout << "vector has " << n << " elements\n";
    }
}
```

<details><summary>Answer</summary>

```cpp
if (auto n = v.size(); n > 0) {
    std::cout << "vector has " << n << " elements\n";
}
```

C++17 allows an init-statement inside `if` to limit the scope of `n` to the if-block.

</details>

---

## 3. Code from Scratch — LeetCode Style (3 problems)

### Problem 1: Swap without std::swap (§1.7, p.11)

Write a function `my_swap` that takes two `int`s **by reference** and swaps their values without using `std::swap` or a temporary third variable (use arithmetic or XOR).

```
Signature: void my_swap(int& a, int& b);
```

**Examples:**

| Input (a, b) | Output (a, b) |
|---|---|
| 3, 7 | 7, 3 |
| -1, 0 | 0, -1 |
| 5, 5 | 5, 5 |

**Constraints:** Do not use `std::swap`. Do not use a temporary variable. Use only references.

<details><summary>Solution</summary>

```cpp
#include <iostream>

// XOR swap — works because XOR is its own inverse
void my_swap(int& a, int& b) {
    if (&a == &b) return;  // guard against same-address case
    a ^= b;   // a now holds a XOR b
    b ^= a;   // b = b XOR (a XOR b) = original a
    a ^= b;   // a = (a XOR b) XOR original a = original b
}

int main() {
    int x = 3, y = 7;
    my_swap(x, y);
    std::cout << x << " " << y << '\n';  // 7 3
}
```

Key decisions: References let us modify the caller's variables directly. The `&a == &b` guard prevents the XOR-swap from zeroing a value when both references alias the same object.

</details>

---

### Problem 2: Count digits using a pointer walk (§1.7, p.11)

Write a function that takes a C-style string (`const char*`) and returns the count of digit characters (`'0'`–`'9'`).

```
Signature: int count_digits(const char* s);
```

**Examples:**

| Input | Output |
|---|---|
| `"abc123"` | `3` |
| `"no digits"` | `0` |
| `""` | `0` |

**Constraints:** Use pointer arithmetic to walk the string. Do not use `strlen` or any standard library algorithm.

<details><summary>Solution</summary>

```cpp
#include <iostream>

int count_digits(const char* s) {
    int count = 0;
    // walk the pointer until we hit the null terminator
    for (const char* p = s; *p != '\0'; ++p) {
        if (*p >= '0' && *p <= '9')  // check ASCII range for digits
            ++count;
    }
    return count;
}

int main() {
    std::cout << count_digits("abc123") << '\n';     // 3
    std::cout << count_digits("no digits") << '\n';  // 0
    std::cout << count_digits("") << '\n';            // 0
}
```

Key decisions: `const char*` because we only read the string. Pointer walk via `++p` is idiomatic C++. Null-terminator check `*p != '\0'` is the standard way to detect end-of-string.

</details>

---

### Problem 3: Compile-time factorial (§1.6, p.9)

Write a `constexpr` function that computes the factorial of a non-negative integer. Use it to initialize a compile-time constant.

```
Signature: constexpr long long factorial(int n);
```

**Examples:**

| Input | Output |
|---|---|
| `0` | `1` |
| `5` | `120` |
| `10` | `3628800` |

**Constraints:** Must be `constexpr`. Must handle `n == 0`. Do not use any library functions.

<details><summary>Solution</summary>

```cpp
#include <iostream>

constexpr long long factorial(int n) {
    // base case: 0! = 1
    long long result = 1;
    for (int i = 2; i <= n; ++i)  // C++14 allows loops in constexpr
        result *= i;
    return result;
}

int main() {
    constexpr long long f10 = factorial(10);  // computed at compile time
    static_assert(f10 == 3628800, "factorial(10) must be 3628800");
    std::cout << factorial(5) << '\n';   // 120
}
```

Key decisions: `constexpr` guarantees compile-time evaluation when called with constant arguments. `long long` avoids overflow for moderate inputs. `static_assert` verifies the compile-time result.

</details>

---

## 4. Output Prediction (4 snippets)

**Snippet 1** (§1.4, p.5)

```cpp
#include <iostream>
int main() {
    char c = 256;
    std::cout << static_cast<int>(c) << '\n';
}
```

What does this print, or does it fail to compile?

<details><summary>Answer</summary>

Implementation-defined, but on most systems where `char` is signed and 8-bit, `256` overflows to `0`. Output: `0`. This is a narrowing conversion that `{}` initialization would have caught.

</details>

---

**Snippet 2** (§1.7, p.11)

```cpp
#include <iostream>
int main() {
    int a = 1;
    int& r = a;
    int b = 2;
    r = b;
    b = 99;
    std::cout << a << " " << r << " " << b << '\n';
}
```

<details><summary>Answer</summary>

Output: `2 2 99`. `r` is bound to `a` permanently. `r = b` copies the value of `b` (2) into `a`; it does not rebind `r` to `b`. Changing `b` to 99 does not affect `a` or `r`.

</details>

---

**Snippet 3** (§1.4, p.6)

```cpp
#include <iostream>
int main() {
    auto x = 5;
    auto y = 2;
    auto z = x / y;
    std::cout << z << '\n';
}
```

<details><summary>Answer</summary>

Output: `2`. Both `x` and `y` are deduced as `int`, so `x / y` is integer division, truncating 2.5 to 2.

</details>

---

**Snippet 4** (§1.6, p.9)

```cpp
#include <iostream>
constexpr int f(int n) { return n <= 1 ? 1 : n * f(n - 1); }
int main() {
    int x;
    std::cin >> x;          // user types 5
    constexpr int a = f(5); // line A
    int b = f(x);           // line B
    std::cout << a << " " << b << '\n';
}
```

<details><summary>Answer</summary>

Output: `120 120` (assuming user types 5). Line A is evaluated at compile time because `f(5)` is a constant expression. Line B calls `f` at runtime with a non-constexpr argument — this is allowed because `constexpr` functions can also run at runtime.

</details>

---

## 5. Debug This (3 snippets)

**Bug 1** — Dangling pointer (§1.7, p.11)

This code is supposed to return a pointer to a computed value:

```cpp
#include <iostream>

int* compute() {
    int result = 42;
    return &result;
}

int main() {
    int* p = compute();
    std::cout << *p << '\n';
}
```

<details><summary>Answer</summary>

**Bug:** `result` is a local variable. Returning its address creates a **dangling pointer** — `result` is destroyed when `compute()` returns. Dereferencing `p` is undefined behavior.

**Fix:** Return by value (`int compute() { return 42; }`) or allocate with `new` (and remember to `delete`). Prefer returning by value.

</details>

---

**Bug 2** — Uninitialized variable (§1.4, p.5)

This code is supposed to sum the elements of an array:

```cpp
#include <iostream>

int main() {
    int arr[] = {1, 2, 3, 4, 5};
    int sum;
    for (auto x : arr)
        sum += x;
    std::cout << "Sum: " << sum << '\n';
}
```

<details><summary>Answer</summary>

**Bug:** `sum` is uninitialized. In C++, local variables of built-in type are not zero-initialized. Reading `sum` before assigning it is undefined behavior.

**Fix:** `int sum = 0;` or `int sum{0};`.

</details>

---

**Bug 3** — Using `=` instead of `==` in condition (§1.8, p.13)

This code is supposed to check if `x` equals 10:

```cpp
#include <iostream>

int main() {
    int x = 5;
    if (x = 10) {
        std::cout << "x is ten\n";
    } else {
        std::cout << "x is not ten\n";
    }
}
```

<details><summary>Answer</summary>

**Bug:** `x = 10` is assignment, not comparison. It assigns 10 to `x`, then the expression evaluates to 10, which is truthy — so the `if` branch always executes.

**Fix:** Use `if (x == 10)`. Modern compilers warn about assignment in conditions; use `-Wall`.

</details>

---

## 6. System Design / Open-ended (2 questions)

**Q1.** You need to write a small calculator that reads an expression like `3 + 4` from `std::cin`, evaluates it, and prints the result. Using only what Chapter 1 covers (basic types, `if`/`switch`, I/O, functions), sketch the design. What types would you use? How would you dispatch on the operator? What happens if the user enters an invalid operator? (§1.8, p.13)

---

**Q2.** Stroustrup emphasizes that C++ supports multiple programming paradigms. Using only the features from Chapter 1, compare how you would structure a "unit converter" (e.g., Celsius to Fahrenheit) as: (a) a set of free functions, (b) using `constexpr` for compile-time conversion factors. What are the tradeoffs? When would you choose one over the other? (§1.1, p.1; §1.6, p.9)

---

## 7. Rapid Fire (10 true/false)

1. `int x{};` initializes `x` to `0`. (§1.4, p.5)

<details><summary>Answer</summary>True. Value initialization with `{}` for built-in types yields zero.</details>

2. A reference can be rebound to a different variable after initialization. (§1.7, p.11)

<details><summary>Answer</summary>False. A reference is permanently bound to its initial target.</details>

3. `nullptr` can be assigned to an `int` variable. (§1.7, p.11)

<details><summary>Answer</summary>False. `nullptr` is of type `std::nullptr_t` and does not implicitly convert to integers.</details>

4. `constexpr` functions can only be called at compile time. (§1.6, p.9)

<details><summary>Answer</summary>False. They can also be called at runtime with non-constant arguments.</details>

5. `auto` deduces references: `int& r = x; auto a = r;` makes `a` an `int&`. (§1.4.2, p.6)

<details><summary>Answer</summary>False. `auto` drops references by default; `a` is a plain `int` copy. Use `auto&` to get a reference.</details>

6. `for (auto x : vec)` modifies elements of `vec` in place. (§1.7, p.10)

<details><summary>Answer</summary>False. `auto x` creates a copy. Use `auto& x` to modify in place.</details>

7. `switch` statements in C++ require a `break` to prevent fall-through. (§1.8, p.13)

<details><summary>Answer</summary>True. Without `break`, execution falls through to the next `case`.</details>

8. `int arr[constexpr_val];` is valid when `constexpr_val` is a `constexpr int`. (§1.6, p.9)

<details><summary>Answer</summary>True. `constexpr` values are compile-time constants and valid as array sizes.</details>

9. Stroustrup recommends using `0` or `NULL` rather than `nullptr`. (§1.7, p.11)

<details><summary>Answer</summary>False. `nullptr` is the recommended null pointer literal in modern C++.</details>

10. In C++17, `if (auto n = size(); n > 0)` limits the scope of `n` to the if/else block. (§1.8, p.13)

<details><summary>Answer</summary>True. The init-statement in `if` scopes the variable to the entire if/else construct.</details>

---

## 8. Capstone Implementation Challenge

### Simple Order Book (§1.4–1.8, p.5–13)

**Motivation:** You're building a simplified order book for a trading system. Each order has an ID, price, quantity, and side (buy/sell). The book must support adding orders, canceling by ID, and querying the best bid (highest buy) and best ask (lowest sell).

**Signatures:**

```cpp
struct Order {
    int id;
    double price;
    int qty;
    bool is_buy;
};

class OrderBook {
public:
    void add_order(Order o);
    bool cancel_order(int id);
    const Order* best_bid() const;   // highest buy price, or nullptr
    const Order* best_ask() const;   // lowest sell price, or nullptr
    int count() const;
};
```

**Test Scenarios:**

1. Add buy orders at 100, 102, 101. `best_bid()` returns the order at 102.
2. Add sell orders at 105, 103, 104. `best_ask()` returns the order at 103.
3. Cancel the best bid. `best_bid()` now returns the order at 101.

**Constraints:**
- Use only basic types, `struct`, arrays or `std::vector`, pointers, references
- Use `const` and `constexpr` where appropriate
- Use range-for loops (not index-based) where natural
- No `<algorithm>` — implement your own linear search
- All functions use pass-by-reference or pass-by-const-reference

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <vector>

struct Order {
    int id;
    double price;
    int qty;
    bool is_buy;
};

class OrderBook {
    std::vector<Order> orders_;  // simple storage — no sorting needed
public:
    // Add an order to the book
    void add_order(Order o) {
        orders_.push_back(o);
    }

    // Cancel by ID — returns true if found and removed
    bool cancel_order(int id) {
        for (auto it = orders_.begin(); it != orders_.end(); ++it) {
            if (it->id == id) {
                // Swap with last and pop — O(1) removal, order doesn't matter
                *it = orders_.back();
                orders_.pop_back();
                return true;
            }
        }
        return false;
    }

    // Best bid: highest-priced buy order
    const Order* best_bid() const {
        const Order* best = nullptr;
        for (const auto& o : orders_) {
            if (o.is_buy) {
                if (!best || o.price > best->price)
                    best = &o;
            }
        }
        return best;  // nullptr if no buys
    }

    // Best ask: lowest-priced sell order
    const Order* best_ask() const {
        const Order* best = nullptr;
        for (const auto& o : orders_) {
            if (!o.is_buy) {
                if (!best || o.price < best->price)
                    best = &o;
            }
        }
        return best;  // nullptr if no sells
    }

    int count() const { return static_cast<int>(orders_.size()); }
};

int main() {
    OrderBook book;

    // Scenario 1: buy orders
    book.add_order({1, 100.0, 10, true});
    book.add_order({2, 102.0, 5,  true});
    book.add_order({3, 101.0, 8,  true});

    if (auto* bid = book.best_bid())
        std::cout << "Best bid: $" << bid->price
                  << " (id=" << bid->id << ")\n";  // $102, id=2

    // Scenario 2: sell orders
    book.add_order({4, 105.0, 3, false});
    book.add_order({5, 103.0, 7, false});
    book.add_order({6, 104.0, 2, false});

    if (auto* ask = book.best_ask())
        std::cout << "Best ask: $" << ask->price
                  << " (id=" << ask->id << ")\n";  // $103, id=5

    // Scenario 3: cancel best bid
    book.cancel_order(2);
    if (auto* bid = book.best_bid())
        std::cout << "New best bid: $" << bid->price
                  << " (id=" << bid->id << ")\n";  // $101, id=3

    std::cout << "Total orders: " << book.count() << '\n';  // 5
}
```

Key decisions:
- **`const Order*` return** for queries — `nullptr` signals "no orders on that side," avoiding exceptions for a normal condition. Pointers are covered in §1.7.
- **Swap-and-pop removal** avoids shifting elements — uses only basic pointer/reference operations.
- **Range-for with `const auto&`** in queries avoids copies and respects const-correctness (§1.7, p.10).
- **`const` member functions** on queries promise not to modify state (§1.6, p.9).

</details>
