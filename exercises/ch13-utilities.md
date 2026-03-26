# Chapter 13: Utilities

---

## Quick Reference

- **Smart pointers**: `unique_ptr` (exclusive ownership), `shared_ptr` (shared ownership), `weak_ptr` (non-owning observer) (§13.2, p.121)
- `make_unique<T>(args)` and `make_shared<T>(args)` are safer and more efficient than raw `new` (§13.2.1, p.122)
- `shared_ptr` uses reference counting; the object is destroyed when the last `shared_ptr` to it is destroyed (§13.2.1, p.122)
- `weak_ptr` breaks cycles in `shared_ptr` graphs — call `lock()` to get a temporary `shared_ptr` (§13.2.1, p.123)
- `std::move` casts to rvalue; `std::forward` preserves value category (perfect forwarding) (§13.2.2, p.124)
- **`std::optional<T>`**: either holds a `T` or is empty (`std::nullopt`) — replaces sentinel values (§13.4.1, p.126)
- **`std::variant<T1,T2,...>`**: type-safe union — holds one of several types at a time (§13.4.2, p.127)
- **`std::any`**: holds any single value — type-safe alternative to `void*` (§13.4.3, p.128)
- Use `std::visit` with a visitor to process `variant` values (§13.4.2, p.127)
- `std::function<R(Args...)>` is a general-purpose polymorphic function wrapper (§13.5, p.129)
- **Chrono**: `<chrono>` provides `duration`, `time_point`, `system_clock`, `steady_clock` (§13.6, p.130)
- `std::pair` and `std::tuple` for ad-hoc grouping of values (§13.4, p.126)

---

## 1. Multiple Choice (8 questions)

**Q1.** When is `shared_ptr`'s reference count decremented? (§13.2.1, p.122)

a) When the pointee is modified
b) When a `shared_ptr` to the object is destroyed or reassigned
c) When `reset()` is called on any pointer in the program
d) At the end of every scope

<details><summary>Answer</summary>

**b)** The count drops when a `shared_ptr` is destroyed, goes out of scope, or is reassigned to point elsewhere. When the count reaches zero, the managed object is deleted.

</details>

---

**Q2.** What does `weak_ptr` solve? (§13.2.1, p.123)

a) Performance of shared_ptr
b) Circular reference cycles that prevent deallocation
c) Thread safety
d) Type erasure

<details><summary>Answer</summary>

**b)** If two `shared_ptr`s point to each other (cycle), neither's count reaches zero. Replacing one with `weak_ptr` breaks the cycle, allowing deallocation.

</details>

---

**Q3.** What does `std::optional<int> x;` contain by default? (§13.4.1, p.126)

a) `0`
b) `std::nullopt` (empty)
c) Undefined
d) Throws an exception

<details><summary>Answer</summary>

**b)** A default-constructed `optional` is empty (`std::nullopt`). Check with `x.has_value()` or `if (x)`.

</details>

---

**Q4.** How do you access the value in a `std::variant`? (§13.4.2, p.127)

a) Direct cast
b) `std::get<T>(v)` or `std::visit(visitor, v)`
c) `v.value()`
d) `v.get()`

<details><summary>Answer</summary>

**b)** `std::get<T>(v)` extracts the value (throws `bad_variant_access` if wrong type). `std::visit` applies a callable to the active alternative.

</details>

---

**Q5.** What does `std::forward` do? (§13.2.2, p.124)

a) Moves an object to a new location
b) Preserves the value category (lvalue/rvalue) of a forwarding reference
c) Forwards function calls to base classes
d) Creates a copy and forwards it

<details><summary>Answer</summary>

**b)** `std::forward<T>(arg)` preserves whether `arg` was originally an lvalue or rvalue. This is essential for perfect forwarding in template code.

</details>

---

**Q6.** What is `std::any`? (§13.4.3, p.128)

a) A variant that holds any type from a predefined list
b) A type-safe container for a single value of any type
c) A template for any container
d) An alias for `void*`

<details><summary>Answer</summary>

**b)** `std::any` can hold a value of any type. Access with `std::any_cast<T>(a)` which throws `bad_any_cast` if the type is wrong. Unlike `void*`, it's type-safe.

</details>

---

**Q7.** What is `std::function` used for? (§13.5, p.129)

a) Declaring functions
b) A polymorphic wrapper that can store any callable (function, lambda, functor) matching a signature
c) Creating coroutines
d) Function overloading

<details><summary>Answer</summary>

**b)** `std::function<int(int)>` can hold any callable that takes an `int` and returns `int` — functions, lambdas, function objects, or bind expressions.

</details>

---

**Q8.** Which clock should you use for measuring elapsed time? (§13.6, p.130)

a) `system_clock` — it gives wall time
b) `steady_clock` — it's monotonic and not affected by clock adjustments
c) `high_resolution_clock` — it's always the most precise
d) `time_t` from C

<details><summary>Answer</summary>

**b)** `steady_clock` is guaranteed to be monotonic (never goes backward). `system_clock` can be adjusted (NTP, DST). `high_resolution_clock` may alias either one.

</details>

---

## 2. Fill-in-the-blank Code (5 snippets)

**Snippet 1** — shared_ptr and weak_ptr (§13.2.1, p.122–123)

```cpp
#include <memory>
#include <iostream>

int main() {
    auto sp = std::___<int>(42);
    std::___<int> wp = sp;           // non-owning observer
    std::cout << sp.___() << '\n';    // reference count
    if (auto locked = wp.___()) {     // try to get shared_ptr
        std::cout << *locked << '\n';
    }
}
```

<details><summary>Answer</summary>

```cpp
auto sp = std::make_shared<int>(42);
std::weak_ptr<int> wp = sp;
std::cout << sp.use_count() << '\n';   // 1
if (auto locked = wp.lock()) { ... }
```

</details>

---

**Snippet 2** — std::optional (§13.4.1, p.126)

```cpp
#include <optional>
#include <string>
#include <iostream>

std::optional<std::string> find_user(int id) {
    if (id == 1) return "Alice";
    return ___;  // empty optional
}

int main() {
    auto user = find_user(1);
    if (___) {
        std::cout << *user << '\n';
    }
    auto missing = find_user(99);
    std::cout << missing.___("unknown") << '\n';
}
```

<details><summary>Answer</summary>

```cpp
return std::nullopt;
if (user) { ... }        // or user.has_value()
missing.value_or("unknown")
```

</details>

---

**Snippet 3** — std::variant with visit (§13.4.2, p.127)

```cpp
#include <variant>
#include <string>
#include <iostream>

int main() {
    std::variant<int, double, std::string> v = "hello";
    std::___([](___ const& val) {
        std::cout << val << '\n';
    }, v);
}
```

<details><summary>Answer</summary>

```cpp
std::visit([](auto const& val) {
    std::cout << val << '\n';
}, v);
```

The `auto` lambda accepts any type held by the variant.

</details>

---

**Snippet 4** — std::function (§13.5, p.129)

```cpp
#include <functional>
#include <iostream>

int add(int a, int b) { return a + b; }

int main() {
    std::___<int(int,int)> op = add;
    std::cout << op(3, 4) << '\n';   // 7

    op = [](int a, int b) { return a * b; };
    std::cout << op(3, 4) << '\n';   // 12
}
```

<details><summary>Answer</summary>

```cpp
std::function<int(int,int)> op = add;
```

</details>

---

**Snippet 5** — chrono timing (§13.6, p.130)

```cpp
#include <chrono>
#include <iostream>
#include <thread>

int main() {
    auto start = std::chrono::___::now();
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
    auto end = std::chrono::___::now();
    auto elapsed = std::chrono::duration_cast<std::chrono::___>(end - start);
    std::cout << elapsed.count() << " ms\n";
}
```

<details><summary>Answer</summary>

```cpp
auto start = std::chrono::steady_clock::now();
auto end = std::chrono::steady_clock::now();
auto elapsed = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);
```

</details>

---

## 3. Code from Scratch — LeetCode Style (3 problems)

### Problem 1: Safe division with optional (§13.4.1, p.126)

Write a function that divides two doubles. If the divisor is zero, return `std::nullopt`.

```
Signature: std::optional<double> safe_divide(double a, double b);
```

**Examples:**

| Input | Output |
|---|---|
| `10, 3` | `3.333...` |
| `5, 0` | `std::nullopt` |

**Constraints:** Use `std::optional`. No exceptions.

<details><summary>Solution</summary>

```cpp
#include <optional>
#include <iostream>

std::optional<double> safe_divide(double a, double b) {
    if (b == 0.0)
        return std::nullopt;   // signal failure without exceptions
    return a / b;
}

int main() {
    auto r1 = safe_divide(10, 3);
    if (r1)
        std::cout << *r1 << '\n';  // 3.33333

    auto r2 = safe_divide(5, 0);
    std::cout << r2.value_or(-1) << '\n';  // -1 (default for empty)
}
```

</details>

---

### Problem 2: Type-safe config with variant (§13.4.2, p.127)

Implement a `Config` class that stores settings as `map<string, variant<int, double, string, bool>>`. Provide `set()` and a type-safe `get<T>()`.

```
Signature:
  class Config {
  public:
      using Value = std::variant<int, double, std::string, bool>;
      void set(const std::string& key, Value val);
      template<typename T> T get(const std::string& key) const;
  };
```

**Examples:**

| Code | Result |
|---|---|
| `cfg.set("port", 8080); cfg.get<int>("port")` | `8080` |
| `cfg.get<string>("port")` | throws `bad_variant_access` |

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <map>
#include <string>
#include <variant>
#include <stdexcept>

class Config {
public:
    using Value = std::variant<int, double, std::string, bool>;
private:
    std::map<std::string, Value> data_;
public:
    void set(const std::string& key, Value val) {
        data_[key] = std::move(val);
    }

    template<typename T>
    T get(const std::string& key) const {
        auto it = data_.find(key);
        if (it == data_.end())
            throw std::runtime_error("key not found: " + key);
        return std::get<T>(it->second);  // throws bad_variant_access if wrong type
    }
};

int main() {
    Config cfg;
    cfg.set("port", 8080);
    cfg.set("host", std::string("localhost"));
    cfg.set("debug", true);

    std::cout << cfg.get<int>("port") << '\n';           // 8080
    std::cout << cfg.get<std::string>("host") << '\n';   // localhost
    std::cout << std::boolalpha << cfg.get<bool>("debug") << '\n'; // true
}
```

</details>

---

### Problem 3: Callback registry with std::function (§13.5, p.129)

Implement an `EventBus` that maps event names to lists of callbacks. Provide `subscribe(name, callback)` and `emit(name, data)`.

```
Signature:
  class EventBus {
  public:
      void subscribe(const std::string& event, std::function<void(int)> cb);
      void emit(const std::string& event, int data);
  };
```

**Examples:**

| Code | Output |
|---|---|
| subscribe "click" with two handlers, emit "click" 42 | Both handlers called with 42 |

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <map>
#include <functional>

class EventBus {
    // Each event name maps to a list of callbacks
    std::map<std::string, std::vector<std::function<void(int)>>> handlers_;
public:
    void subscribe(const std::string& event, std::function<void(int)> cb) {
        handlers_[event].push_back(std::move(cb));
    }

    void emit(const std::string& event, int data) {
        auto it = handlers_.find(event);
        if (it != handlers_.end()) {
            for (auto& cb : it->second)
                cb(data);   // call each registered handler
        }
    }
};

int main() {
    EventBus bus;
    bus.subscribe("click", [](int x) { std::cout << "Handler A: " << x << '\n'; });
    bus.subscribe("click", [](int x) { std::cout << "Handler B: " << x << '\n'; });
    bus.emit("click", 42);
    // Handler A: 42
    // Handler B: 42
}
```

</details>

---

## 4. Output Prediction (4 snippets)

**Snippet 1** — shared_ptr reference counting (§13.2.1, p.122)

```cpp
#include <memory>
#include <iostream>
int main() {
    auto a = std::make_shared<int>(10);
    auto b = a;
    auto c = a;
    std::cout << a.use_count() << '\n';
    b.reset();
    std::cout << a.use_count() << '\n';
}
```

<details><summary>Answer</summary>

Output:
```
3
2
```

Three `shared_ptr`s share the object, so count is 3. After `b.reset()`, count drops to 2.

</details>

---

**Snippet 2** — optional value_or (§13.4.1, p.126)

```cpp
#include <optional>
#include <iostream>
int main() {
    std::optional<int> a = 42;
    std::optional<int> b;
    std::cout << a.value_or(0) << '\n';
    std::cout << b.value_or(-1) << '\n';
}
```

<details><summary>Answer</summary>

Output:
```
42
-1
```

`a` has a value (42), so `value_or` returns it. `b` is empty, so `value_or` returns the default (-1).

</details>

---

**Snippet 3** — variant and visit (§13.4.2, p.127)

```cpp
#include <variant>
#include <iostream>
int main() {
    std::variant<int, std::string> v = 42;
    std::cout << std::get<int>(v) << '\n';
    v = "hello";
    std::cout << std::get<std::string>(v) << '\n';
    std::cout << v.index() << '\n';
}
```

<details><summary>Answer</summary>

Output:
```
42
hello
1
```

Initially holds `int` (index 0). After assigning `"hello"`, holds `string` (index 1).

</details>

---

**Snippet 4** — any_cast failure (§13.4.3, p.128)

```cpp
#include <any>
#include <iostream>
int main() {
    std::any a = 42;
    try {
        std::cout << std::any_cast<double>(a) << '\n';
    } catch (const std::bad_any_cast& e) {
        std::cout << e.what() << '\n';
    }
}
```

<details><summary>Answer</summary>

Output: the `what()` message of `bad_any_cast` (implementation-defined, e.g., "bad any_cast"). The `any` holds an `int`, but we tried to cast to `double` — type mismatch throws.

</details>

---

## 5. Debug This (3 snippets)

**Bug 1** — shared_ptr cycle (§13.2.1, p.123)

```cpp
#include <memory>
#include <iostream>

struct Node {
    std::shared_ptr<Node> next;
    ~Node() { std::cout << "~Node\n"; }
};

int main() {
    auto a = std::make_shared<Node>();
    auto b = std::make_shared<Node>();
    a->next = b;
    b->next = a;  // cycle!
}
// expect ~Node ~Node, but nothing prints
```

<details><summary>Answer</summary>

**Bug:** Circular reference. `a` holds `b` and `b` holds `a`. Neither ref count reaches zero — memory leak, destructors never called.

**Fix:** Make one link a `weak_ptr`: `std::weak_ptr<Node> next;`

</details>

---

**Bug 2** — Accessing empty optional (§13.4.1, p.126)

```cpp
#include <optional>
#include <iostream>

int main() {
    std::optional<int> x;
    std::cout << *x << '\n';
}
```

<details><summary>Answer</summary>

**Bug:** Dereferencing an empty `optional` is undefined behavior. There's no value to read.

**Fix:** Check `if (x)` first, or use `x.value()` which throws `std::bad_optional_access` if empty.

</details>

---

**Bug 3** — std::function overhead for simple case (§13.5, p.129)

```cpp
#include <functional>
#include <vector>
#include <iostream>

int main() {
    std::vector<std::function<int()>> funcs;
    for (int i = 0; i < 1000000; ++i)
        funcs.push_back([i]() { return i; });
    // This works but is slower than it needs to be
}
```

<details><summary>Answer</summary>

**Bug:** `std::function` has overhead (type erasure, possible heap allocation). For simple cases where the callable type is known at compile time, a template parameter or `auto` lambda is more efficient.

**Fix:** If all callbacks have the same type, use `std::vector<decltype(lambda)>` or a template approach. If polymorphism is truly needed, `std::function` is correct but comes with a cost.

</details>

---

## 6. System Design / Open-ended (2 questions)

**Q1.** Design a JSON-like value type using `std::variant`. It should represent: null, bool, int, double, string, array (vector of values), and object (map of string to value). What challenges arise from the recursive definition? How would you use `std::visit` to implement a `to_string()` function? (§13.4.2, p.127)

---

**Q2.** You're building a plugin system where plugins register callbacks for different events. Compare using `std::function` (type-erased, flexible) vs. templates (compile-time, fast). What if plugins are loaded dynamically at runtime from shared libraries — which approach must you use? (§13.5, p.129)

---

## 7. Rapid Fire (10 true/false)

1. `unique_ptr` can be copied. (§13.2.1, p.122)

<details><summary>Answer</summary>False. `unique_ptr` is move-only. Copy is deleted.</details>

2. `make_shared` performs one allocation for both the object and the control block. (§13.2.1, p.122)

<details><summary>Answer</summary>True. This is more efficient than `shared_ptr(new T)` which does two allocations.</details>

3. `weak_ptr` increments the reference count. (§13.2.1, p.123)

<details><summary>Answer</summary>False. `weak_ptr` does not affect the reference count. It only observes.</details>

4. `std::optional` can hold a reference. (§13.4.1, p.126)

<details><summary>Answer</summary>False. `optional<T&>` is not supported in C++17. Use a pointer or `std::reference_wrapper`.</details>

5. `std::variant` can be in a "valueless by exception" state. (§13.4.2, p.127)

<details><summary>Answer</summary>True. If assignment throws during type change, the variant may enter this state.</details>

6. `std::any_cast` with the wrong type returns `nullptr`. (§13.4.3, p.128)

<details><summary>Answer</summary>Partially true. Pointer overload returns `nullptr`; value overload throws `bad_any_cast`.</details>

7. `std::function` has zero overhead compared to a raw function pointer. (§13.5, p.129)

<details><summary>Answer</summary>False. `std::function` involves type erasure and may allocate heap memory.</details>

8. `steady_clock` can be adjusted by the system (NTP, user, etc.). (§13.6, p.130)

<details><summary>Answer</summary>False. `steady_clock` is monotonic — it never adjusts backward.</details>

9. `std::move` transfers ownership of a `unique_ptr`. (§13.2.2, p.124)

<details><summary>Answer</summary>True. `auto p2 = std::move(p1)` transfers ownership; `p1` becomes null.</details>

10. `std::visit` requires the visitor to handle all alternatives of the variant. (§13.4.2, p.127)

<details><summary>Answer</summary>True. The visitor must be callable for every type in the variant; otherwise it's a compile error.</details>

---

## 8. Capstone Implementation Challenge

### JSON-Like Document Model (§13.2–13.5, p.121–129)

**Motivation:** You're building a lightweight document model (like JSON) where a `Value` can be null, boolean, integer, double, string, an array of values, or an object (map of string to value). This exercises `variant`, `optional`, `shared_ptr` (for recursive types), and `visit`.

**Signatures:**

```cpp
class Value {
public:
    using Null = std::monostate;
    using Array = std::vector<Value>;
    using Object = std::map<std::string, Value>;
    using Storage = std::variant<Null, bool, int, double, std::string,
                                  std::shared_ptr<Array>, std::shared_ptr<Object>>;

    Value();                          // null
    Value(bool b);
    Value(int i);
    Value(double d);
    Value(const std::string& s);
    Value(const char* s);

    // Array operations
    static Value array(std::initializer_list<Value> items);
    void push_back(Value v);
    const Value& operator[](int index) const;

    // Object operations
    static Value object(std::initializer_list<std::pair<std::string, Value>> items);
    void set(const std::string& key, Value v);
    std::optional<Value> get(const std::string& key) const;

    // Type queries
    bool is_null() const;
    bool is_number() const;
    bool is_string() const;
    bool is_array() const;
    bool is_object() const;

    // Convert to string (JSON-like)
    std::string to_string() const;
};
```

**Test Scenarios:**

1. Create nested structure: `{"name": "Alice", "age": 30, "scores": [95, 87, 92]}`. `to_string()` outputs valid JSON-like text.
2. `value.get("name")` returns `optional<Value>` with `"Alice"`. `value.get("missing")` returns `nullopt`.
3. Null values, booleans, nested objects all serialize correctly.

**Constraints:**
- Use `std::variant` with `std::monostate` for null
- Use `std::shared_ptr` for Array and Object to handle the recursive type
- Use `std::visit` for `to_string()` and type queries
- Use `std::optional` for safe key lookup
- Use `std::function` or lambdas with `visit`

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <variant>
#include <vector>
#include <map>
#include <string>
#include <memory>
#include <optional>
#include <sstream>

class Value {
public:
    using Null = std::monostate;
    using Array = std::vector<Value>;
    using Object = std::map<std::string, Value>;
    // shared_ptr enables recursive type (Value contains vector<Value>)
    using Storage = std::variant<Null, bool, int, double, std::string,
                                  std::shared_ptr<Array>, std::shared_ptr<Object>>;
private:
    Storage data_;

public:
    Value() : data_{Null{}} {}
    Value(bool b) : data_{b} {}
    Value(int i) : data_{i} {}
    Value(double d) : data_{d} {}
    Value(const std::string& s) : data_{s} {}
    Value(const char* s) : data_{std::string(s)} {}

    // Factory for arrays
    static Value array(std::initializer_list<Value> items) {
        Value v;
        v.data_ = std::make_shared<Array>(items);
        return v;
    }

    void push_back(Value v) {
        if (auto* arr = std::get_if<std::shared_ptr<Array>>(&data_))
            (*arr)->push_back(std::move(v));
    }

    const Value& operator[](int index) const {
        return (*std::get<std::shared_ptr<Array>>(data_))[index];
    }

    // Factory for objects
    static Value object(std::initializer_list<std::pair<std::string, Value>> items) {
        Value v;
        auto obj = std::make_shared<Object>();
        for (auto& [key, val] : items)
            (*obj)[key] = val;
        v.data_ = obj;
        return v;
    }

    void set(const std::string& key, Value v) {
        if (auto* obj = std::get_if<std::shared_ptr<Object>>(&data_))
            (**obj)[key] = std::move(v);
    }

    // Safe lookup: returns nullopt if key missing or not an object
    std::optional<Value> get(const std::string& key) const {
        if (auto* obj = std::get_if<std::shared_ptr<Object>>(&data_)) {
            auto it = (*obj)->find(key);
            if (it != (*obj)->end())
                return it->second;
        }
        return std::nullopt;
    }

    // Type queries using visit
    bool is_null() const   { return std::holds_alternative<Null>(data_); }
    bool is_number() const { return std::holds_alternative<int>(data_) ||
                                    std::holds_alternative<double>(data_); }
    bool is_string() const { return std::holds_alternative<std::string>(data_); }
    bool is_array() const  { return std::holds_alternative<std::shared_ptr<Array>>(data_); }
    bool is_object() const { return std::holds_alternative<std::shared_ptr<Object>>(data_); }

    // Serialize to JSON-like string using std::visit
    std::string to_string() const {
        return std::visit([](const auto& val) -> std::string {
            using T = std::decay_t<decltype(val)>;

            if constexpr (std::is_same_v<T, Null>)
                return "null";
            else if constexpr (std::is_same_v<T, bool>)
                return val ? "true" : "false";
            else if constexpr (std::is_same_v<T, int>)
                return std::to_string(val);
            else if constexpr (std::is_same_v<T, double>) {
                std::ostringstream oss;
                oss << val;
                return oss.str();
            }
            else if constexpr (std::is_same_v<T, std::string>)
                return "\"" + val + "\"";
            else if constexpr (std::is_same_v<T, std::shared_ptr<Array>>) {
                std::string s = "[";
                for (size_t i = 0; i < val->size(); ++i) {
                    if (i > 0) s += ", ";
                    s += (*val)[i].to_string();
                }
                return s + "]";
            }
            else if constexpr (std::is_same_v<T, std::shared_ptr<Object>>) {
                std::string s = "{";
                bool first = true;
                for (const auto& [k, v] : *val) {
                    if (!first) s += ", ";
                    s += "\"" + k + "\": " + v.to_string();
                    first = false;
                }
                return s + "}";
            }
        }, data_);
    }
};

int main() {
    // Scenario 1: build a nested document
    auto doc = Value::object({
        {"name", Value("Alice")},
        {"age", Value(30)},
        {"active", Value(true)},
        {"scores", Value::array({Value(95), Value(87), Value(92)})},
        {"address", Value::object({
            {"city", Value("Portland")},
            {"zip", Value(97201)}
        })}
    });

    std::cout << doc.to_string() << "\n\n";

    // Scenario 2: safe lookup
    if (auto name = doc.get("name"))
        std::cout << "Name: " << name->to_string() << '\n';

    if (auto missing = doc.get("phone"))
        std::cout << "Found phone\n";
    else
        std::cout << "Phone: not found\n";

    // Scenario 3: type checks
    auto scores = doc.get("scores");
    std::cout << "scores is array: " << std::boolalpha << scores->is_array() << '\n';

    // Scenario 4: null value
    Value null_val;
    std::cout << "null: " << null_val.to_string() << '\n';
}
```

Key decisions:
- **`std::variant`** with `std::monostate` for null — type-safe union of all JSON types (§13.4.2).
- **`std::shared_ptr<Array>`/`shared_ptr<Object>`** solves the recursive type problem — `Value` can't directly contain `vector<Value>` in a variant (incomplete type) (§13.2.1).
- **`std::visit`** with generic lambda + `if constexpr` dispatches serialization per-type (§13.4.2).
- **`std::optional`** for `.get()` — absent keys return `nullopt` instead of throwing (§13.4.1).
- **`std::holds_alternative`** for type queries — cleaner than `get_if` when you only need a bool (§13.4.2).

</details>
