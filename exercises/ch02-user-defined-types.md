# Chapter 2: User-Defined Types

---

## Quick Reference

- **Structures** (`struct`) group related data; members are public by default (§2.2, p.15)
- **Classes** (`class`) are like `struct` but members are private by default (§2.3, p.16)
- **Enumerations**: `enum class` (scoped, no implicit conversion to `int`); plain `enum` (unscoped, converts to `int`) (§2.5, p.21)
- **Unions** store one of several types in the same memory; only the last-written member is valid (§2.4, p.20)
- A `struct` with a constructor can enforce invariants at creation time (§2.2, p.15)
- Member functions defined inside the class body are implicitly `inline` (§2.3, p.17)
- Use `enum class` over plain `enum` to avoid name clashes and accidental implicit conversions (§2.5, p.21)
- Access specifiers: `public`, `private`, `protected` control member visibility (§2.3, p.16)
- The `class` interface (public part) should be separated from the implementation (private part) (§2.3, p.16)
- `union` is rarely used directly in modern C++; prefer `std::variant` (§2.4, p.20)
- The `struct` layout is guaranteed to have members in declaration order (§2.2, p.15)

---

## 1. Multiple Choice (8 questions)

**Q1.** What is the default access for members of a `struct`? (§2.2, p.15)

a) `private`
b) `protected`
c) `public`
d) Depends on the compiler

<details><summary>Answer</summary>

**c)** In C++, `struct` members are `public` by default, while `class` members are `private` by default. This is the only difference between `struct` and `class`.

</details>

---

**Q2.** What does this code output? (§2.5, p.21)

```cpp
enum class Color { Red, Green, Blue };
int x = Color::Red;
```

a) `0`
b) `1`
c) Compilation error
d) Undefined behavior

<details><summary>Answer</summary>

**c)** `enum class` does not implicitly convert to `int`. You would need `static_cast<int>(Color::Red)` for the conversion. This is a key advantage of scoped enumerations.

</details>

---

**Q3.** Which statement is true about `union`? (§2.4, p.20)

a) All members are stored separately in memory
b) Only the most recently written member can be safely read
c) A `union` can have virtual functions
d) Members can have different access specifiers

<details><summary>Answer</summary>

**b)** A `union` holds one value at a time; reading a member that wasn't the last one written is undefined behavior (type punning). All members share the same memory address.

</details>

---

**Q4.** What is the purpose of a constructor in a `struct`? (§2.2, p.15)

a) To allocate heap memory
b) To establish an invariant — ensuring the object is in a valid state
c) To make the struct copyable
d) To enable polymorphism

<details><summary>Answer</summary>

**b)** Constructors establish invariants by initializing data members to valid states. This is fundamental to Stroustrup's design philosophy — an object should be valid from the moment it's created.

</details>

---

**Q5.** What is the difference between `enum` and `enum class`? (§2.5, p.21)

a) `enum class` allows floating-point values
b) `enum` is scoped; `enum class` is unscoped
c) `enum class` is scoped and does not implicitly convert to `int`
d) There is no difference in modern C++

<details><summary>Answer</summary>

**c)** `enum class` (scoped enumeration) confines its enumerators to the enum's scope and forbids implicit conversions to integers. Plain `enum` leaks names into the enclosing scope and allows implicit conversion.

</details>

---

**Q6.** Given this struct, what does `sizeof(Point)` likely return on a 64-bit system? (§2.2, p.15)

```cpp
struct Point { double x; double y; };
```

a) 8
b) 16
c) 24
d) Depends entirely on the compiler

<details><summary>Answer</summary>

**b)** Two `double` values at 8 bytes each, tightly packed with no padding needed (both are naturally aligned), so `sizeof(Point)` is 16.

</details>

---

**Q7.** Which is the correct way to access `enum class` values? (§2.5, p.21)

a) `Color c = Red;`
b) `Color c = Color::Red;`
c) `Color c = Color.Red;`
d) `Color c = (Color)0;`

<details><summary>Answer</summary>

**b)** Scoped enumerations require the enum name as a qualifier: `Color::Red`. Unqualified `Red` is a compile error. While `(Color)0` works, it bypasses type safety.

</details>

---

**Q8.** What is the conventional C++ practice for separating interface from implementation in a class? (§2.3, p.16)

a) Put everything in one `.cpp` file
b) Declare public members first, private members after; definition in a separate `.cpp`
c) Use `friend` to expose all internals
d) Make all members `public`

<details><summary>Answer</summary>

**b)** The interface (public member declarations) goes in a header, and function definitions go in a `.cpp` file. Public members first makes the interface easy to read.

</details>

---

## 2. Fill-in-the-blank Code (5 snippets)

**Snippet 1** — Basic struct with constructor (§2.2, p.15)

```cpp
#include <iostream>

struct Vector {
    double* elem;
    int sz;

    ___(int s) : elem{new double[s]}, sz{s} {  // constructor
        for (int i = 0; i < s; ++i)
            elem[i] = 0;
    }
};

int main() {
    Vector v(5);
    std::cout << v.___ << '\n';  // print size
}
```

<details><summary>Answer</summary>

```cpp
Vector(int s) : elem{new double[s]}, sz{s} { ... }
std::cout << v.sz << '\n';
```

</details>

---

**Snippet 2** — Scoped enumeration (§2.5, p.21)

```cpp
#include <iostream>

___ ___ Traffic { Red, Yellow, Green };  // scoped enum

void act(Traffic t) {
    switch (t) {
        case Traffic___Red:    std::cout << "Stop\n";  break;
        case Traffic___Yellow: std::cout << "Slow\n";  break;
        case Traffic___Green:  std::cout << "Go\n";    break;
    }
}

int main() {
    act(Traffic::Green);
}
```

<details><summary>Answer</summary>

```cpp
enum class Traffic { Red, Yellow, Green };

case Traffic::Red:    ...
case Traffic::Yellow: ...
case Traffic::Green:  ...
```

</details>

---

**Snippet 3** — Class with private data (§2.3, p.16)

```cpp
#include <iostream>

class Counter {
___:                          // access specifier
    int count;
___:                          // access specifier
    Counter() : count{0} {}
    void increment() { ++count; }
    int get() ___ { return count; }  // should not modify state
};

int main() {
    Counter c;
    c.increment();
    std::cout << c.get() << '\n';
}
```

<details><summary>Answer</summary>

```cpp
private:
    int count;
public:
    Counter() : count{0} {}
    ...
    int get() const { return count; }
```

</details>

---

**Snippet 4** — Union basics (§2.4, p.20)

```cpp
#include <iostream>

___ Value {
    int i;
    double d;
    char c;
};

int main() {
    Value v;
    v.d = 3.14;
    std::cout << v.___ << '\n';  // access the active member
}
```

<details><summary>Answer</summary>

```cpp
union Value { int i; double d; char c; };
std::cout << v.d << '\n';   // d is the last-written member
```

Reading `v.i` or `v.c` after writing `v.d` would be undefined behavior.

</details>

---

**Snippet 5** — Struct with member functions (§2.2, p.15)

```cpp
#include <iostream>
#include <cmath>

struct Point {
    double x, y;

    double distance_to(___) ___ {  // const member, takes another Point
        double dx = x - other.x;
        double dy = y - other.y;
        return std::sqrt(dx*dx + dy*dy);
    }
};

int main() {
    Point a{0, 0}, b{3, 4};
    std::cout << a.distance_to(b) << '\n';  // 5
}
```

<details><summary>Answer</summary>

```cpp
double distance_to(const Point& other) const {
```

`const Point&` avoids copying; trailing `const` promises the function doesn't modify `*this`.

</details>

---

## 3. Code from Scratch — LeetCode Style (3 problems)

### Problem 1: Implement a Date struct (§2.2, p.15)

Create a `struct Date` with members `year`, `month`, `day`. Add a constructor that validates input (month 1–12, day 1–31) and a `to_string()` member that returns `"YYYY-MM-DD"`.

```
Signature:
  struct Date {
      Date(int y, int m, int d);
      std::string to_string() const;
  };
```

**Examples:**

| Input | Output |
|---|---|
| `Date(2024, 3, 15)` | `"2024-03-15"` |
| `Date(2024, 13, 1)` | throws or asserts |

**Constraints:** Must validate month and day ranges. Use `struct`, not `class`.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <string>
#include <stdexcept>

struct Date {
    int year, month, day;

    // Constructor enforces the invariant: valid date ranges
    Date(int y, int m, int d) : year{y}, month{m}, day{d} {
        if (m < 1 || m > 12)
            throw std::out_of_range("month must be 1–12");
        if (d < 1 || d > 31)
            throw std::out_of_range("day must be 1–31");
    }

    // Format as YYYY-MM-DD with zero-padding
    std::string to_string() const {
        auto pad = [](int v, int w) {
            std::string s = std::to_string(v);
            while (static_cast<int>(s.size()) < w) s = "0" + s;
            return s;
        };
        return pad(year, 4) + "-" + pad(month, 2) + "-" + pad(day, 2);
    }
};

int main() {
    Date d(2024, 3, 15);
    std::cout << d.to_string() << '\n';  // 2024-03-15
}
```

</details>

---

### Problem 2: Implement a scoped enum + switch dispatcher (§2.5, p.21)

Create an `enum class Op { Add, Sub, Mul, Div }` and a function `double apply(Op op, double a, double b)` that dispatches to the correct arithmetic operation using `switch`.

```
Signature: double apply(Op op, double a, double b);
```

**Examples:**

| Input | Output |
|---|---|
| `apply(Op::Add, 3, 4)` | `7.0` |
| `apply(Op::Div, 10, 3)` | `3.333...` |

**Constraints:** Use `enum class` (not plain `enum`). Handle division by zero by throwing.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <stdexcept>

enum class Op { Add, Sub, Mul, Div };

double apply(Op op, double a, double b) {
    switch (op) {
        case Op::Add: return a + b;
        case Op::Sub: return a - b;
        case Op::Mul: return a * b;
        case Op::Div:
            if (b == 0.0) throw std::domain_error("division by zero");
            return a / b;
    }
    throw std::logic_error("unknown op");  // unreachable if all cases covered
}

int main() {
    std::cout << apply(Op::Add, 3, 4) << '\n';    // 7
    std::cout << apply(Op::Div, 10, 3) << '\n';   // 3.33333
}
```

Key decisions: `enum class` prevents implicit int conversion and requires `Op::` qualification. The `switch` has no `default` so the compiler warns if a case is missed.

</details>

---

### Problem 3: Tagged union with type safety (§2.4, p.20)

Implement a `TaggedValue` that can hold either an `int` or a `double`. Use a `struct` with a `union` inside and an `enum class Tag { Int, Double }` to track which member is active. Provide `get_int()` and `get_double()` accessors that throw if the wrong type is accessed.

```
Signature:
  struct TaggedValue {
      TaggedValue(int i);
      TaggedValue(double d);
      int get_int() const;
      double get_double() const;
  };
```

**Examples:**

| Construction | get_int() | get_double() |
|---|---|---|
| `TaggedValue(42)` | `42` | throws |
| `TaggedValue(3.14)` | throws | `3.14` |

**Constraints:** Use a `union` and an `enum class` tag. No `std::variant`.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <stdexcept>

struct TaggedValue {
    enum class Tag { Int, Double };

    Tag tag;
    union {                     // anonymous union shares memory
        int i;
        double d;
    };

    TaggedValue(int val)    : tag{Tag::Int},    i{val} {}   // set tag + active member
    TaggedValue(double val) : tag{Tag::Double}, d{val} {}

    int get_int() const {
        if (tag != Tag::Int) throw std::runtime_error("not an int");
        return i;
    }

    double get_double() const {
        if (tag != Tag::Double) throw std::runtime_error("not a double");
        return d;
    }
};

int main() {
    TaggedValue a(42);
    std::cout << a.get_int() << '\n';     // 42

    TaggedValue b(3.14);
    std::cout << b.get_double() << '\n';  // 3.14
}
```

Key decisions: The `Tag` enum tracks which union member is active — this is the "discriminated union" pattern. Accessors check the tag before reading, preventing UB from reading the wrong union member.

</details>

---

## 4. Output Prediction (4 snippets)

**Snippet 1** (§2.5, p.21)

```cpp
#include <iostream>
enum Color { Red, Green, Blue };
int main() {
    int x = Green;
    std::cout << x << '\n';
}
```

<details><summary>Answer</summary>

Output: `1`. Plain (unscoped) `enum` values implicitly convert to `int`. `Green` is the second enumerator, so its value is 1 (Red=0, Green=1, Blue=2).

</details>

---

**Snippet 2** (§2.5, p.21)

```cpp
#include <iostream>
enum class Fruit { Apple = 5, Banana, Cherry = 3 };
int main() {
    std::cout << static_cast<int>(Fruit::Banana) << '\n';
}
```

<details><summary>Answer</summary>

Output: `6`. `Banana` follows `Apple = 5`, so it gets 5 + 1 = 6. `Cherry = 3` doesn't affect `Banana` because it comes after.

</details>

---

**Snippet 3** (§2.2, p.15)

```cpp
#include <iostream>
struct S { int a; int b; int c; };
int main() {
    S s = {1, 2};
    std::cout << s.a << s.b << s.c << '\n';
}
```

<details><summary>Answer</summary>

Output: `120`. Aggregate initialization with fewer values than members value-initializes the rest. `s.c` is initialized to 0. Output is `1`, `2`, `0` concatenated.

</details>

---

**Snippet 4** (§2.4, p.20)

```cpp
#include <iostream>
union U { int i; float f; };
int main() {
    U u;
    u.i = 42;
    u.f = 3.14f;
    std::cout << u.i << '\n';
}
```

<details><summary>Answer</summary>

This is **undefined behavior**. After writing `u.f`, reading `u.i` accesses an inactive member. In practice, it prints the bit-reinterpretation of `3.14f` as an `int` (likely `1078523331` on IEEE 754 systems), but the behavior is not guaranteed.

</details>

---

## 5. Debug This (3 snippets)

**Bug 1** — Accessing enum class without scope (§2.5, p.21)

This code is supposed to assign a direction value:

```cpp
enum class Direction { North, South, East, West };
int main() {
    Direction d = North;
}
```

<details><summary>Answer</summary>

**Bug:** `North` is not in scope — `enum class` requires the qualified name.

**Fix:** `Direction d = Direction::North;`

</details>

---

**Bug 2** — Missing const on accessor (§2.3, p.16)

This code is supposed to print the size of a const Vector:

```cpp
class Vector {
    int sz;
public:
    Vector(int s) : sz{s} {}
    int size() { return sz; }
};

void print_size(const Vector& v) {
    std::cout << v.size() << '\n';
}
```

<details><summary>Answer</summary>

**Bug:** `size()` is not declared `const`, so it cannot be called on a `const Vector&` reference. The compiler will reject the call.

**Fix:** `int size() const { return sz; }`

</details>

---

**Bug 3** — Struct member initialization order (§2.2, p.15)

This code is supposed to create a properly initialized struct:

```cpp
#include <iostream>
struct Pair {
    int first;
    int second;
    Pair(int b, int a) : second{b}, first{a} {}
};

int main() {
    Pair p(10, 20);
    std::cout << p.first << " " << p.second << '\n';
}
```

<details><summary>Answer</summary>

**Bug:** Members are initialized in **declaration order** (`first` then `second`), not in the order listed in the initializer list. Here it works by coincidence, but the misleading order triggers compiler warnings and can cause bugs if one member depends on another. `first` is initialized with `a` (20), `second` with `b` (10). Output: `20 10`.

**Fix:** Match initializer list order to declaration order: `: first{a}, second{b}`.

</details>

---

## 6. System Design / Open-ended (2 questions)

**Q1.** Design a simple shape representation system using only `struct`, `enum class`, and `union`. You have shapes: Circle (radius), Rectangle (width, height), and Triangle (base, height). How would you store these efficiently? What are the risks compared to using a class hierarchy with virtual functions? (§2.2, p.15; §2.4, p.20; §2.5, p.21)

---

**Q2.** Stroustrup says the purpose of a constructor is to "establish the invariant" for a class. Design a `BankAccount` struct where the invariant is: balance must never be negative. What interface would you expose? Where would you enforce the invariant — constructor only, or every mutating operation? What happens to the invariant if members are public? (§2.2, p.15; §2.3, p.16)

---

## 7. Rapid Fire (10 true/false)

1. `struct` and `class` are identical except for default access. (§2.3, p.16)

<details><summary>Answer</summary>True. The only difference is `struct` defaults to `public` and `class` to `private`.</details>

2. `enum class` values implicitly convert to `int`. (§2.5, p.21)

<details><summary>Answer</summary>False. `enum class` requires explicit `static_cast` for conversion to `int`.</details>

3. A `union` can hold all its members simultaneously. (§2.4, p.20)

<details><summary>Answer</summary>False. A union stores exactly one member at a time; all members share the same memory.</details>

4. Members in a struct initializer list are initialized in the order they appear in the list. (§2.2, p.15)

<details><summary>Answer</summary>False. They are initialized in declaration order in the struct, regardless of the initializer list order.</details>

5. `struct S { int x = 5; };` is valid C++. (§2.2, p.15)

<details><summary>Answer</summary>True. Default member initializers are valid since C++11.</details>

6. A `const` member function can modify `mutable` members. (§2.3, p.16)

<details><summary>Answer</summary>True. `mutable` members are exempt from const restrictions.</details>

7. `enum class Color { R, G, B }; Color c = 1;` compiles. (§2.5, p.21)

<details><summary>Answer</summary>False. `enum class` does not allow implicit conversion from `int`. You need `static_cast<Color>(1)`.</details>

8. You can forward-declare a scoped enum: `enum class Status;` (§2.5, p.21)

<details><summary>Answer</summary>True. Scoped enumerations have a fixed underlying type (`int` by default) and can be forward-declared.</details>

9. Reading a `union` member that was not the last one written is well-defined. (§2.4, p.20)

<details><summary>Answer</summary>False. It is undefined behavior (except for common initial sequences of standard-layout structs).</details>

10. A `struct` cannot have private members. (§2.2, p.15)

<details><summary>Answer</summary>False. A `struct` can use `private:` just like a `class`; only the default differs.</details>

---

## 8. Capstone Implementation Challenge

### INI-Style Configuration Store (§2.2–2.5, p.15–22)

**Motivation:** You're building a configuration system that stores typed settings. Each setting has a name, a type (string, integer, or boolean), and a value. The store supports adding settings, querying by name, and printing all settings grouped by type.

**Signatures:**

```cpp
enum class ValueType { String, Int, Bool };

struct ConfigValue {
    ValueType type;
    union {
        int int_val;
        bool bool_val;
    };
    char str_val[64];  // used when type == String

    // Constructors for each type
    static ConfigValue make_string(const char* s);
    static ConfigValue make_int(int v);
    static ConfigValue make_bool(bool v);

    void print() const;
};

class ConfigStore {
public:
    void set(const char* name, ConfigValue val);
    const ConfigValue* get(const char* name) const;
    void print_all() const;
    int size() const;
private:
    struct Entry { char name[32]; ConfigValue value; };
    Entry entries_[100];
    int count_ = 0;
};
```

**Test Scenarios:**

1. Set `"app_name"` to string `"MyApp"`, `"port"` to int `8080`, `"debug"` to bool `true`. `size()` returns 3.
2. `get("port")` returns a `ConfigValue` with `type == Int` and `int_val == 8080`.
3. `get("missing")` returns `nullptr`.
4. `print_all()` outputs all settings with their types.

**Constraints:**
- Use `struct`, `enum class`, `union` — all three user-defined type mechanisms from this chapter
- No `std::string`, `std::vector`, or `std::map` — use C-style arrays and `<cstring>`
- The `class` must enforce invariants (private data, public interface)
- Use the `class` vs `struct` distinction meaningfully

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <cstring>

enum class ValueType { String, Int, Bool };

struct ConfigValue {
    ValueType type;
    union {
        int int_val;
        bool bool_val;
    };
    char str_val[64];  // only valid when type == String

    // Factory functions establish the correct tag + value pairing
    static ConfigValue make_string(const char* s) {
        ConfigValue v;
        v.type = ValueType::String;
        std::strncpy(v.str_val, s, sizeof(v.str_val) - 1);
        v.str_val[sizeof(v.str_val) - 1] = '\0';
        return v;
    }

    static ConfigValue make_int(int val) {
        ConfigValue v;
        v.type = ValueType::Int;
        v.int_val = val;
        return v;
    }

    static ConfigValue make_bool(bool val) {
        ConfigValue v;
        v.type = ValueType::Bool;
        v.bool_val = val;
        return v;
    }

    // Print based on active type — switch on enum class
    void print() const {
        switch (type) {
            case ValueType::String:
                std::cout << "(string) \"" << str_val << "\"";
                break;
            case ValueType::Int:
                std::cout << "(int) " << int_val;
                break;
            case ValueType::Bool:
                std::cout << "(bool) " << (bool_val ? "true" : "false");
                break;
        }
    }
};

class ConfigStore {
    // Entry pairs a name with a value — struct for simple grouping
    struct Entry {
        char name[32];
        ConfigValue value;
    };

    Entry entries_[100];  // fixed capacity — no dynamic allocation
    int count_ = 0;

    // Internal helper: find index by name, or -1
    int find_index(const char* name) const {
        for (int i = 0; i < count_; ++i) {
            if (std::strcmp(entries_[i].name, name) == 0)
                return i;
        }
        return -1;
    }

public:
    // Set: update if exists, append if new
    void set(const char* name, ConfigValue val) {
        int idx = find_index(name);
        if (idx >= 0) {
            entries_[idx].value = val;  // update existing
        } else if (count_ < 100) {
            std::strncpy(entries_[count_].name, name, 31);
            entries_[count_].name[31] = '\0';
            entries_[count_].value = val;
            ++count_;
        }
    }

    // Get: return pointer to value, or nullptr if not found
    const ConfigValue* get(const char* name) const {
        int idx = find_index(name);
        if (idx >= 0)
            return &entries_[idx].value;
        return nullptr;
    }

    // Print all entries
    void print_all() const {
        for (int i = 0; i < count_; ++i) {
            std::cout << "  " << entries_[i].name << " = ";
            entries_[i].value.print();
            std::cout << '\n';
        }
    }

    int size() const { return count_; }
};

int main() {
    ConfigStore cfg;

    // Scenario 1: add three settings of different types
    cfg.set("app_name", ConfigValue::make_string("MyApp"));
    cfg.set("port",     ConfigValue::make_int(8080));
    cfg.set("debug",    ConfigValue::make_bool(true));
    std::cout << "Size: " << cfg.size() << '\n';  // 3

    // Scenario 2: query by name
    if (const auto* v = cfg.get("port")) {
        std::cout << "port = ";
        v->print();  // (int) 8080
        std::cout << '\n';
    }

    // Scenario 3: missing key
    if (cfg.get("missing") == nullptr)
        std::cout << "\"missing\" not found\n";

    // Scenario 4: print all
    std::cout << "All settings:\n";
    cfg.print_all();
}
```

Key decisions:
- **`enum class ValueType`** is the discriminator tag — scoped to avoid name clashes (§2.5).
- **`union`** stores either `int_val` or `bool_val` in the same memory (§2.4). `str_val` is separate since it's an array.
- **Factory functions** (`make_string`, `make_int`, `make_bool`) establish the tag+value invariant — the caller can't set a mismatched type/value.
- **`class ConfigStore`** uses `private` data and a `public` interface, enforcing the invariant that `count_` matches actual entries (§2.3).
- **`struct Entry`** is a simple data grouping — no invariant to enforce, so `struct` is appropriate (§2.2).

</details>
