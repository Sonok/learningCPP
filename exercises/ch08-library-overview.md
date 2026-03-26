# Chapter 8: Library Overview

---

## Quick Reference

- The C++ standard library provides components for I/O, strings, containers, algorithms, numerics, concurrency, and more (§8.1, p.79)
- Library components are defined in **namespace `std`** and accessed via `#include` headers (§8.2, p.80)
- Prefer standard library facilities over hand-written code — they're tested, optimized, and portable (§8.1, p.79)
- Key header categories: `<string>`, `<vector>`, `<map>`, `<algorithm>`, `<iostream>`, `<memory>`, `<thread>`, `<regex>` (§8.2, p.80)
- The library uses **RAII** pervasively: `vector`, `string`, `unique_ptr` all manage resources (§8.3, p.81)
- Containers and algorithms are connected by **iterators** (§8.3, p.81)
- `std::pair` and `std::tuple` hold fixed collections of typed values (§8.4, p.82)
- Smart pointers (`unique_ptr`, `shared_ptr`) replace raw `new`/`delete` (§8.4, p.82)
- The library design emphasizes generic programming — algorithms work on any container via iterators (§8.3, p.81)
- Standard headers do not have `.h` extensions: `<cstdio>` not `<stdio.h>` (§8.2, p.80)
- `using namespace std;` is acceptable in small programs but not in headers (§8.2, p.80)

---

## 1. Multiple Choice (8 questions)

**Q1.** What is the primary organizational mechanism for the standard library? (§8.2, p.80)

a) Macros
b) The `std` namespace and header files
c) Global functions
d) Template specializations

<details><summary>Answer</summary>

**b)** All standard library names are in namespace `std`, organized into headers like `<vector>`, `<algorithm>`, etc.

</details>

---

**Q2.** What design pattern does the standard library rely on for resource management? (§8.3, p.81)

a) Singleton
b) RAII — resource acquisition is initialization
c) Observer
d) Factory

<details><summary>Answer</summary>

**b)** RAII is fundamental to the library. Containers acquire memory in constructors and release in destructors. Smart pointers do the same for heap objects.

</details>

---

**Q3.** What connects containers and algorithms in the standard library? (§8.3, p.81)

a) Inheritance
b) Function pointers
c) Iterators
d) Virtual dispatch

<details><summary>Answer</summary>

**c)** Iterators abstract the traversal of elements. Algorithms like `std::sort` work with iterator pairs, making them independent of the underlying container.

</details>

---

**Q4.** Which header provides `std::unique_ptr`? (§8.4, p.82)

a) `<utility>`
b) `<memory>`
c) `<pointer>`
d) `<smart_ptr>`

<details><summary>Answer</summary>

**b)** `<memory>` provides `unique_ptr`, `shared_ptr`, `weak_ptr`, and related utilities.

</details>

---

**Q5.** Why does Stroustrup recommend using standard library facilities over hand-written equivalents? (§8.1, p.79)

a) They are always faster
b) They are well-tested, maintained, and often optimized beyond what individual programmers achieve
c) The standard requires their use
d) Custom code is not allowed in modern C++

<details><summary>Answer</summary>

**b)** Standard library components are the product of expert design, extensive testing, and continuous optimization. Using them reduces bugs and improves portability.

</details>

---

**Q6.** What does `<cstdio>` provide compared to `<stdio.h>`? (§8.2, p.80)

a) They are identical
b) `<cstdio>` places names in `std::`, while `<stdio.h>` places them globally
c) `<cstdio>` is slower
d) `<stdio.h>` is C++ only

<details><summary>Answer</summary>

**b)** The `<cXXX>` headers are the C++ versions of C headers. They place names in `namespace std`, avoiding global namespace pollution.

</details>

---

**Q7.** Which is NOT a standard library container? (§8.3, p.81)

a) `std::vector`
b) `std::map`
c) `std::hashtable`
d) `std::unordered_map`

<details><summary>Answer</summary>

**c)** There is no `std::hashtable`. The hash-based containers are `std::unordered_map`, `std::unordered_set`, etc.

</details>

---

**Q8.** What is `std::pair` used for? (§8.4, p.82)

a) Pairing threads for parallel execution
b) Holding exactly two values of potentially different types
c) Creating key-value stores
d) Connecting two containers

<details><summary>Answer</summary>

**b)** `std::pair<T1, T2>` holds two values. It's used by `std::map` for key-value pairs and as a general-purpose utility.

</details>

---

## 2. Fill-in-the-blank Code (5 snippets)

**Snippet 1** — Including standard headers (§8.2, p.80)

```cpp
#include <___>    // for std::vector
#include <___>    // for std::sort
#include <___>    // for std::cout

int main() {
    std::vector<int> v = {3, 1, 4, 1, 5};
    std::sort(v.begin(), v.end());
    for (auto x : v)
        std::cout << x << " ";
}
```

<details><summary>Answer</summary>

```cpp
#include <vector>
#include <algorithm>
#include <iostream>
```

</details>

---

**Snippet 2** — Using std::pair (§8.4, p.82)

```cpp
#include <iostream>
#include <utility>

int main() {
    auto p = std::___<std::string, int>("Alice", 30);
    std::cout << p.___ << " is " << p.___ << '\n';
}
```

<details><summary>Answer</summary>

```cpp
auto p = std::make_pair<std::string, int>("Alice", 30);
// or simply: auto p = std::pair("Alice"s, 30); with CTAD
std::cout << p.first << " is " << p.second << '\n';
```

</details>

---

**Snippet 3** — unique_ptr basic usage (§8.4, p.82)

```cpp
#include <memory>
#include <iostream>

int main() {
    auto p = std::___<int>(42);  // create on heap
    std::cout << *p << '\n';     // dereference
    // no delete needed — automatic cleanup
}
```

<details><summary>Answer</summary>

```cpp
auto p = std::make_unique<int>(42);
```

</details>

---

**Snippet 4** — Iterator-based algorithm (§8.3, p.81)

```cpp
#include <vector>
#include <algorithm>
#include <iostream>

int main() {
    std::vector<int> v = {5, 2, 8, 1, 9};
    auto it = std::___(v.begin(), v.end());  // find largest
    std::cout << "Max: " << *it << '\n';
}
```

<details><summary>Answer</summary>

```cpp
auto it = std::max_element(v.begin(), v.end());
```

</details>

---

**Snippet 5** — tuple and structured bindings (§8.4, p.82)

```cpp
#include <tuple>
#include <iostream>

auto get_record() {
    return std::___("Alice", 30, 3.9);  // name, age, gpa
}

int main() {
    auto [name, age, gpa] = get_record();
    std::cout << name << " " << age << " " << gpa << '\n';
}
```

<details><summary>Answer</summary>

```cpp
return std::make_tuple("Alice", 30, 3.9);
```

</details>

---

## 3. Code from Scratch — LeetCode Style (3 problems)

### Problem 1: Top-K elements using standard library (§8.3, p.81)

Given a `vector<int>`, return the `k` largest elements in descending order. Use standard library algorithms only.

```
Signature: std::vector<int> top_k(std::vector<int> nums, int k);
```

**Examples:**

| Input | Output |
|---|---|
| `{3,1,5,2,4}, k=3` | `{5,4,3}` |
| `{7,7,7}, k=2` | `{7,7}` |

**Constraints:** Use `std::sort` or `std::partial_sort`. Return a new vector.

<details><summary>Solution</summary>

```cpp
#include <vector>
#include <algorithm>
#include <iostream>

std::vector<int> top_k(std::vector<int> nums, int k) {
    // partial_sort puts the k largest at the front when using greater<>
    std::partial_sort(nums.begin(), nums.begin() + k, nums.end(),
                      std::greater<>{});
    // Extract just the first k elements
    return std::vector<int>(nums.begin(), nums.begin() + k);
}

int main() {
    auto result = top_k({3, 1, 5, 2, 4}, 3);
    for (auto x : result)
        std::cout << x << " ";  // 5 4 3
}
```

</details>

---

### Problem 2: Build a lookup table with pair (§8.4, p.82)

Create a function that takes a vector of `pair<string, int>` (name, score) and returns the pair with the highest score.

```
Signature: std::pair<std::string, int> highest_score(
               const std::vector<std::pair<std::string, int>>& records);
```

**Examples:**

| Input | Output |
|---|---|
| `{{"A",90},{"B",95},{"C",80}}` | `{"B",95}` |

**Constraints:** Use `std::max_element` with a custom comparator.

<details><summary>Solution</summary>

```cpp
#include <vector>
#include <string>
#include <algorithm>
#include <iostream>

std::pair<std::string, int> highest_score(
    const std::vector<std::pair<std::string, int>>& records)
{
    auto it = std::max_element(records.begin(), records.end(),
        // Compare by second element (score)
        [](const auto& a, const auto& b) { return a.second < b.second; }
    );
    return *it;
}

int main() {
    std::vector<std::pair<std::string, int>> records = {
        {"Alice", 90}, {"Bob", 95}, {"Carol", 80}
    };
    auto [name, score] = highest_score(records);
    std::cout << name << ": " << score << '\n';  // Bob: 95
}
```

</details>

---

### Problem 3: Smart pointer resource manager (§8.4, p.82)

Write a factory function that creates objects of different types (Circle, Rectangle) and returns them as `unique_ptr<Shape>`. Demonstrate that cleanup happens automatically.

```
Signature: std::unique_ptr<Shape> create_shape(const std::string& type, double a, double b);
```

**Examples:**

| Call | Result |
|---|---|
| `create_shape("circle", 5, 0)` | unique_ptr to Circle(5) |
| `create_shape("rect", 3, 4)` | unique_ptr to Rectangle(3,4) |

**Constraints:** Use `make_unique`. No raw `new`/`delete` in user code.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <memory>
#include <string>

class Shape {
public:
    virtual double area() const = 0;
    virtual ~Shape() { std::cout << "~Shape\n"; }
};

class Circle : public Shape {
    double r;
public:
    Circle(double radius) : r{radius} {}
    double area() const override { return 3.14159 * r * r; }
    ~Circle() { std::cout << "~Circle\n"; }
};

class Rectangle : public Shape {
    double w, h;
public:
    Rectangle(double width, double height) : w{width}, h{height} {}
    double area() const override { return w * h; }
    ~Rectangle() { std::cout << "~Rectangle\n"; }
};

// Factory: creates shapes without exposing raw new to the caller
std::unique_ptr<Shape> create_shape(const std::string& type, double a, double b) {
    if (type == "circle")
        return std::make_unique<Circle>(a);
    if (type == "rect")
        return std::make_unique<Rectangle>(a, b);
    return nullptr;
}

int main() {
    auto s1 = create_shape("circle", 5, 0);
    auto s2 = create_shape("rect", 3, 4);
    std::cout << s1->area() << '\n';  // ~78.54
    std::cout << s2->area() << '\n';  // 12
    // destructors called automatically when s1, s2 go out of scope
}
```

</details>

---

## 4. Output Prediction (4 snippets)

**Snippet 1** (§8.3, p.81)

```cpp
#include <vector>
#include <algorithm>
#include <iostream>
int main() {
    std::vector<int> v = {3, 1, 4, 1, 5, 9};
    auto n = std::count(v.begin(), v.end(), 1);
    std::cout << n << '\n';
}
```

<details><summary>Answer</summary>

Output: `2`. `std::count` counts occurrences of the value `1` in the vector. There are two 1s.

</details>

---

**Snippet 2** (§8.4, p.82)

```cpp
#include <memory>
#include <iostream>
int main() {
    auto p = std::make_unique<int>(42);
    auto q = std::move(p);
    std::cout << (p == nullptr) << " " << *q << '\n';
}
```

<details><summary>Answer</summary>

Output: `1 42`. After moving `p` to `q`, `p` is null. `q` owns the int with value 42.

</details>

---

**Snippet 3** (§8.4, p.82)

```cpp
#include <tuple>
#include <iostream>
int main() {
    auto t = std::make_tuple(1, "hello", 3.14);
    std::cout << std::get<0>(t) << " "
              << std::get<1>(t) << " "
              << std::get<2>(t) << '\n';
}
```

<details><summary>Answer</summary>

Output: `1 hello 3.14`. `std::get<N>` accesses the Nth element of the tuple.

</details>

---

**Snippet 4** (§8.3, p.81)

```cpp
#include <vector>
#include <algorithm>
#include <iostream>
int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    std::reverse(v.begin(), v.end());
    std::cout << v.front() << " " << v.back() << '\n';
}
```

<details><summary>Answer</summary>

Output: `5 1`. After reversing, the first element is 5 and the last is 1.

</details>

---

## 5. Debug This (3 snippets)

**Bug 1** — Missing header (§8.2, p.80)

```cpp
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3};
    std::cout << v.size() << '\n';
}
```

<details><summary>Answer</summary>

**Bug:** `<vector>` is not included. `std::vector` is undefined.

**Fix:** Add `#include <vector>`.

</details>

---

**Bug 2** — Using raw pointer instead of smart pointer (§8.4, p.82)

```cpp
#include <iostream>

struct Widget { int x; };

Widget* create() { return new Widget{42}; }

int main() {
    auto w = create();
    std::cout << w->x << '\n';
    // forgot delete w;
}
```

<details><summary>Answer</summary>

**Bug:** Memory leak — `w` is a raw pointer and is never deleted.

**Fix:** Return `std::unique_ptr<Widget>`:
```cpp
std::unique_ptr<Widget> create() { return std::make_unique<Widget>(Widget{42}); }
```

</details>

---

**Bug 3** — Wrong header for C function (§8.2, p.80)

```cpp
#include <string.h>  // C header

int main() {
    char s[] = "hello";
    std::size_t len = std::strlen(s);
}
```

<details><summary>Answer</summary>

**Bug:** `<string.h>` is the C header; it doesn't guarantee `strlen` is in `namespace std`. Using `std::strlen` may fail.

**Fix:** Use `<cstring>`: `#include <cstring>` which properly places `strlen` in `std::`.

</details>

---

## 6. System Design / Open-ended (2 questions)

**Q1.** The standard library separates containers, iterators, and algorithms. Explain the benefit of this three-part design. If you had to add a new container (e.g., a ring buffer), what would you need to implement to make standard algorithms work with it? (§8.3, p.81)

---

**Q2.** Compare `unique_ptr` vs `shared_ptr` for a tree data structure where nodes own their children. When would you use each? What happens if you accidentally create a cycle with `shared_ptr`? (§8.4, p.82)

---

## 7. Rapid Fire (10 true/false)

1. Standard library headers use `.h` extensions (e.g., `<vector.h>`). (§8.2, p.80)

<details><summary>Answer</summary>False. C++ standard headers have no extension: `<vector>`, `<string>`, etc.</details>

2. `std::pair` can hold two elements of different types. (§8.4, p.82)

<details><summary>Answer</summary>True. `pair<int, string>` holds an int and a string.</details>

3. `unique_ptr` can be copied. (§8.4, p.82)

<details><summary>Answer</summary>False. `unique_ptr` is move-only; copying would violate exclusive ownership.</details>

4. Iterators are the glue between containers and algorithms. (§8.3, p.81)

<details><summary>Answer</summary>True. Algorithms operate on iterator ranges, not directly on containers.</details>

5. `<cstdio>` and `<stdio.h>` provide identical functionality. (§8.2, p.80)

<details><summary>Answer</summary>True in terms of functionality, but `<cstdio>` places names in `std::` while `<stdio.h>` uses global scope.</details>

6. `make_shared<T>()` performs two separate allocations: one for the object, one for the control block. (§8.4, p.82)

<details><summary>Answer</summary>False. `make_shared` typically combines both into a single allocation for efficiency.</details>

7. `std::tuple` can only hold up to 3 elements. (§8.4, p.82)

<details><summary>Answer</summary>False. Tuples can hold any number of elements.</details>

8. The standard library is purely header-based with no compiled components. (§8.2, p.80)

<details><summary>Answer</summary>False. Some components (like I/O streams and threading) have compiled library code that gets linked.</details>

9. `std::sort` requires random-access iterators. (§8.3, p.81)

<details><summary>Answer</summary>True. `sort` needs random access for efficient partitioning (you can't sort a `std::list` with `std::sort`).</details>

10. Stroustrup recommends writing your own containers when the standard ones don't fit perfectly. (§8.1, p.79)

<details><summary>Answer</summary>False. He recommends using and adapting standard facilities first, only writing custom ones when genuinely necessary.</details>

---

## 8. Capstone Implementation Challenge

### Resource Pool Manager (§8.2–8.4, p.80–82)

**Motivation:** You're building a connection pool for a database client. The pool pre-allocates a set of `Connection` objects and hands them out on request. When a connection is released, it goes back to the pool. The pool uses smart pointers with custom deleters to auto-return connections when they go out of scope.

**Signatures:**

```cpp
struct Connection {
    int id;
    bool active;
    Connection(int i) : id{i}, active{true} {}
    void query(const std::string& sql) const;
    void reset();
};

class ConnectionPool {
public:
    explicit ConnectionPool(int size);

    // Acquires a connection; returns a unique_ptr with custom deleter
    // that returns the connection to the pool instead of destroying it
    using PooledConnection = std::unique_ptr<Connection, std::function<void(Connection*)>>;
    PooledConnection acquire();

    int available() const;
    int total() const;
};
```

**Test Scenarios:**

1. Create pool of 3 connections. `available()` is 3. `acquire()` reduces it to 2.
2. When the `PooledConnection` goes out of scope, the connection is returned to the pool — `available()` goes back up.
3. Acquiring when all connections are in use returns `nullptr` (or throws).

**Constraints:**
- Use `std::unique_ptr` with a custom deleter (not `shared_ptr`)
- Use standard containers (`vector`, `queue`) from the standard library
- Use `std::function` for the deleter
- RAII: connections are auto-returned, never leaked
- No raw `new`/`delete` in user code

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <memory>
#include <vector>
#include <queue>
#include <string>
#include <functional>
#include <stdexcept>

struct Connection {
    int id;
    bool active;

    Connection(int i) : id{i}, active{true} {}

    void query(const std::string& sql) const {
        std::cout << "[conn " << id << "] " << sql << '\n';
    }

    // Reset state between uses
    void reset() {
        active = true;
    }
};

class ConnectionPool {
    // All connections owned by the pool
    std::vector<std::unique_ptr<Connection>> all_;
    // Available connections (raw pointers — pool owns the objects)
    std::queue<Connection*> free_;

public:
    explicit ConnectionPool(int size) {
        for (int i = 0; i < size; ++i) {
            all_.push_back(std::make_unique<Connection>(i));
            free_.push(all_.back().get());
        }
    }

    // Custom deleter type: returns the connection to the pool
    using PooledConnection = std::unique_ptr<Connection, std::function<void(Connection*)>>;

    // Acquire: take a connection from the free queue
    PooledConnection acquire() {
        if (free_.empty())
            throw std::runtime_error("no connections available");

        Connection* conn = free_.front();
        free_.pop();
        conn->reset();

        // Custom deleter captures `this` to return the connection
        return PooledConnection(conn, [this](Connection* c) {
            c->active = false;
            free_.push(c);  // return to pool instead of deleting
        });
    }

    int available() const { return static_cast<int>(free_.size()); }
    int total() const { return static_cast<int>(all_.size()); }
};

int main() {
    ConnectionPool pool(3);
    std::cout << "Available: " << pool.available() << '\n';  // 3

    {
        // Scenario 1: acquire a connection
        auto conn = pool.acquire();
        conn->query("SELECT * FROM users");
        std::cout << "Available after acquire: " << pool.available() << '\n';  // 2

        // Scenario 2: acquire another
        auto conn2 = pool.acquire();
        conn2->query("INSERT INTO logs ...");
        std::cout << "Available: " << pool.available() << '\n';  // 1

    }  // conn and conn2 go out of scope — auto-returned to pool

    // Scenario 2 continued: connections returned
    std::cout << "Available after scope exit: " << pool.available() << '\n';  // 3

    // Scenario 3: exhaust the pool
    auto c1 = pool.acquire();
    auto c2 = pool.acquire();
    auto c3 = pool.acquire();
    std::cout << "Available: " << pool.available() << '\n';  // 0

    try {
        auto c4 = pool.acquire();  // throws!
    } catch (const std::runtime_error& e) {
        std::cout << "Error: " << e.what() << '\n';
    }
}
```

Key decisions:
- **`unique_ptr` with custom deleter** is the core pattern — the deleter returns the connection to the pool instead of deleting it (§8.4). This is RAII in action.
- **`std::function<void(Connection*)>`** as the deleter type enables the lambda capture of `this` (§8.4).
- **`vector<unique_ptr<Connection>>`** owns all connection objects — they live for the pool's lifetime (§8.3).
- **`queue<Connection*>`** tracks available connections — raw pointers are safe here because the pool owns the objects (§8.3).
- **RAII guarantee**: even if an exception occurs between acquire and release, the `PooledConnection`'s destructor returns it to the pool.

</details>
