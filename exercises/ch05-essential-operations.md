# Chapter 5: Essential Operations

---

## Quick Reference

- The **Rule of Five**: if you define any of destructor, copy constructor, copy assignment, move constructor, or move assignment — define or `= delete` them all (§5.1.1, p.49)
- **Copy** creates an independent duplicate; **move** transfers ownership, leaving the source in a valid-but-unspecified state (§5.2, p.51)
- `= default` tells the compiler to generate the default version; `= delete` suppresses it (§5.1.1, p.49)
- **RAII** (Resource Acquisition Is Initialization): acquire resources in constructors, release in destructors (§5.2, p.51)
- **Move semantics**: `std::move(x)` casts `x` to an rvalue reference, enabling the move constructor/assignment (§5.2.1, p.53)
- After a move, the source object must be in a state that can be destroyed and optionally assigned to (§5.2.1, p.53)
- **Copy elision** (RVO/NRVO): the compiler may skip copy/move entirely when returning local objects (§5.2.2, p.54)
- `explicit` on constructors prevents implicit conversions: `explicit Vector(int sz)` (§5.3, p.56)
- **Operator overloading** allows user-defined types to use `+`, `==`, `<<`, etc. naturally (§5.4, p.56)
- **Conversion operators**: `operator bool()` etc. — use `explicit` to prevent surprising implicit conversions (§5.4.1, p.57)
- **Conventional operations**: default constructors, comparisons (`==`, `<`), `swap()`, and hash should be provided when natural (§5.5, p.58)
- A destructor should never throw exceptions (§5.1.1, p.49)

---

## 1. Multiple Choice (8 questions)

**Q1.** What is the Rule of Five? (§5.1.1, p.49)

a) A class should have at most five member functions
b) If you define one of destructor/copy-ctor/copy-assign/move-ctor/move-assign, define or delete them all
c) Five is the maximum inheritance depth
d) Classes should have five or fewer data members

<details><summary>Answer</summary>

**b)** The Rule of Five states that if a class needs a custom destructor, copy constructor, copy assignment, move constructor, or move assignment operator, it likely needs all five. This ensures correct resource management.

</details>

---

**Q2.** What does `std::move(x)` actually do? (§5.2.1, p.53)

a) Moves the data out of `x`
b) Casts `x` to an rvalue reference, enabling move semantics
c) Deletes `x`
d) Copies `x` to a temporary

<details><summary>Answer</summary>

**b)** `std::move` is just a cast to `T&&`. It does not move anything — it signals that the caller is done with the value, allowing a move constructor or assignment to steal the resources.

</details>

---

**Q3.** After `auto y = std::move(x);`, what is the state of `x`? (§5.2.1, p.53)

a) Unchanged
b) Undefined behavior
c) Valid but unspecified — can be destroyed or assigned to
d) Null

<details><summary>Answer</summary>

**c)** A moved-from object must be in a "valid but unspecified" state. You can safely destroy it or assign a new value, but you should not rely on its contents.

</details>

---

**Q4.** What does `= delete` do when applied to a copy constructor? (§5.1.1, p.49)

a) Makes the copy constructor private
b) Prevents the class from being copied — any attempt is a compile error
c) Generates a throwing copy constructor
d) Deletes the object after copying

<details><summary>Answer</summary>

**b)** `= delete` explicitly removes the function. Any code that tries to copy will fail at compile time. This is useful for move-only types like `unique_ptr`.

</details>

---

**Q5.** Why should constructors that take a single argument be `explicit`? (§5.3, p.56)

a) For performance
b) To prevent accidental implicit conversions
c) It's required by the standard
d) To enable move semantics

<details><summary>Answer</summary>

**b)** Without `explicit`, `Vector v = 5;` would silently call `Vector(int)`. This is usually not the programmer's intent and can hide bugs. `explicit` requires `Vector v(5);` or `Vector v{5};`.

</details>

---

**Q6.** What is RAII? (§5.2, p.51)

a) A design pattern for inheritance hierarchies
b) Acquiring resources in constructors and releasing them in destructors
c) A way to avoid using pointers
d) Random Access Iterator Interface

<details><summary>Answer</summary>

**b)** RAII ties resource lifetime to object lifetime. The constructor acquires (opens a file, allocates memory), and the destructor releases. This guarantees cleanup even when exceptions occur.

</details>

---

**Q7.** What does copy elision (NRVO) do? (§5.2.2, p.54)

a) Prevents all copies in a program
b) The compiler constructs the return value directly in the caller's space, skipping copy/move
c) It replaces copies with moves automatically
d) It only works with trivial types

<details><summary>Answer</summary>

**b)** Named Return Value Optimization constructs the returned local object directly in the caller's memory, avoiding both copy and move. This is guaranteed in some cases since C++17.

</details>

---

**Q8.** Which operator should typically be defined as a non-member function? (§5.4, p.56)

a) `operator=`
b) `operator()`
c) `operator==`
d) `operator[]`

<details><summary>Answer</summary>

**c)** Symmetric binary operators like `==`, `+`, `!=` are best as non-member (or friend) functions so both operands can undergo implicit conversions. `=`, `()`, `[]`, and `->` must be members.

</details>

---

## 2. Fill-in-the-blank Code (5 snippets)

**Snippet 1** — Rule of Five: resource-managing class (§5.1.1, p.49)

```cpp
class Buffer {
    int* data;
    int sz;
public:
    explicit Buffer(int n) : data{new int[n]}, sz{n} {}

    // Destructor
    ___() { delete[] data; }

    // Copy constructor — deep copy
    Buffer(const Buffer& other) : data{new int[other.sz]}, sz{other.sz} {
        std::copy(other.data, other.data + sz, data);
    }

    // Copy assignment
    Buffer& operator=(const Buffer& other) {
        if (this != &other) {
            ___[];           // free old memory
            sz = other.sz;
            data = new int[sz];
            std::copy(other.data, other.data + sz, data);
        }
        return ___;
    }

    // Move constructor
    Buffer(Buffer&& other) ___ : data{other.data}, sz{other.sz} {
        other.data = ___;
        other.sz = 0;
    }

    // Move assignment
    Buffer& operator=(Buffer&& other) noexcept;
};
```

<details><summary>Answer</summary>

```cpp
~Buffer() { delete[] data; }

delete[] data;   // free old in copy assignment

return *this;

Buffer(Buffer&& other) noexcept : data{other.data}, sz{other.sz} {
    other.data = nullptr;
    other.sz = 0;
}
```

</details>

---

**Snippet 2** — Move semantics (§5.2.1, p.53)

```cpp
#include <iostream>
#include <vector>
#include <utility>

int main() {
    std::vector<int> a = {1, 2, 3, 4, 5};
    std::vector<int> b = ___(a);           // move a into b
    std::cout << "a.size() = " << a.___() << '\n';  // moved-from
    std::cout << "b.size() = " << b.___() << '\n';
}
```

<details><summary>Answer</summary>

```cpp
std::vector<int> b = std::move(a);
std::cout << "a.size() = " << a.size() << '\n';  // likely 0
std::cout << "b.size() = " << b.size() << '\n';  // 5
```

</details>

---

**Snippet 3** — explicit constructor (§5.3, p.56)

```cpp
class Meter {
    double val;
public:
    ___ Meter(double v) : val{v} {}  // prevent implicit conversion
    double value() const { return val; }
};

void print_length(Meter m) {
    std::cout << m.value() << " meters\n";
}

int main() {
    // print_length(5.0);     // should this compile?
    print_length(Meter{5.0}); // explicit construction
}
```

<details><summary>Answer</summary>

```cpp
explicit Meter(double v) : val{v} {}
```

With `explicit`, `print_length(5.0)` is a compile error — you must write `Meter{5.0}`. Without `explicit`, `5.0` would silently convert.

</details>

---

**Snippet 4** — = default and = delete (§5.1.1, p.49)

```cpp
class Unique {
public:
    Unique() = ___;                             // compiler generates it
    Unique(const Unique&) = ___;                // no copying
    Unique& operator=(const Unique&) = ___;     // no copy assignment
    Unique(Unique&&) = ___;                     // compiler generates move
    Unique& operator=(Unique&&) = ___;          // compiler generates move assign
    ~Unique() = ___;
};
```

<details><summary>Answer</summary>

```cpp
Unique() = default;
Unique(const Unique&) = delete;
Unique& operator=(const Unique&) = delete;
Unique(Unique&&) = default;
Unique& operator=(Unique&&) = default;
~Unique() = default;
```

This creates a move-only type — similar to `unique_ptr`.

</details>

---

**Snippet 5** — Operator overloading (§5.4, p.56)

```cpp
#include <iostream>

class Vec2 {
public:
    double x, y;
    Vec2(double x, double y) : x{x}, y{y} {}

    Vec2 ___(const Vec2& rhs) const {    // operator+
        return {x + rhs.x, y + rhs.y};
    }

    bool ___(const Vec2& rhs) const {    // operator==
        return x == rhs.x && y == rhs.y;
    }
};

___ operator<<(std::ostream& os, const Vec2& v) {
    return os << "(" << v.x << ", " << v.y << ")";
}

int main() {
    Vec2 a{1, 2}, b{3, 4};
    std::cout << a + b << '\n';
}
```

<details><summary>Answer</summary>

```cpp
Vec2 operator+(const Vec2& rhs) const { ... }
bool operator==(const Vec2& rhs) const { ... }
std::ostream& operator<<(std::ostream& os, const Vec2& v) { ... }
```

</details>

---

## 3. Code from Scratch — LeetCode Style (3 problems)

### Problem 1: Resource-managing Buffer class (§5.1.1, p.49; §5.2.1, p.53)

Implement a `Buffer` class that owns a dynamically allocated `int` array. Implement the full Rule of Five: destructor, copy constructor, copy assignment, move constructor, move assignment.

```
Signature:
  class Buffer {
  public:
      explicit Buffer(int size);
      ~Buffer();
      Buffer(const Buffer&);
      Buffer& operator=(const Buffer&);
      Buffer(Buffer&&) noexcept;
      Buffer& operator=(Buffer&&) noexcept;
      int size() const;
      int& operator[](int i);
  };
```

**Examples:**

| Code | Expected |
|---|---|
| `Buffer a(5); a[0]=42;` | `a[0] == 42` |
| `Buffer b = a;` (copy) | `b[0] == 42`, `a[0] == 42` (independent) |
| `Buffer c = std::move(a);` | `c[0] == 42`, `a.size() == 0` |

**Constraints:** Must use RAII. No memory leaks. After move, source must be in a safe destructible state.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <algorithm>

class Buffer {
    int* data_;
    int sz_;
public:
    // Constructor: allocate and zero-initialize
    explicit Buffer(int size) : data_{new int[size]{}}, sz_{size} {}

    // Destructor: free the owned memory
    ~Buffer() { delete[] data_; }

    // Copy constructor: deep copy — allocate fresh memory and copy elements
    Buffer(const Buffer& other) : data_{new int[other.sz_]}, sz_{other.sz_} {
        std::copy(other.data_, other.data_ + sz_, data_);
    }

    // Copy assignment: handle self-assignment, free old, allocate new
    Buffer& operator=(const Buffer& other) {
        if (this != &other) {
            delete[] data_;
            sz_ = other.sz_;
            data_ = new int[sz_];
            std::copy(other.data_, other.data_ + sz_, data_);
        }
        return *this;
    }

    // Move constructor: steal resources, leave source empty
    Buffer(Buffer&& other) noexcept : data_{other.data_}, sz_{other.sz_} {
        other.data_ = nullptr;
        other.sz_ = 0;
    }

    // Move assignment: release own resources, steal from source
    Buffer& operator=(Buffer&& other) noexcept {
        if (this != &other) {
            delete[] data_;
            data_ = other.data_;
            sz_ = other.sz_;
            other.data_ = nullptr;
            other.sz_ = 0;
        }
        return *this;
    }

    int size() const { return sz_; }
    int& operator[](int i) { return data_[i]; }
};

int main() {
    Buffer a(5);
    a[0] = 42;

    Buffer b = a;              // copy
    std::cout << b[0] << '\n'; // 42 — independent copy
    a[0] = 99;
    std::cout << b[0] << '\n'; // still 42

    Buffer c = std::move(a);   // move
    std::cout << c[0] << '\n'; // 99
    std::cout << a.size() << '\n'; // 0 — moved-from
}
```

</details>

---

### Problem 2: Move-only unique handle (§5.1.1, p.49; §5.2.1, p.53)

Implement a `UniqueHandle` class that wraps a raw `int*`. It is non-copyable but movable. Provide `get()`, `release()` (returns pointer and relinquishes ownership), and `reset(int*)`.

```
Signature:
  class UniqueHandle {
  public:
      explicit UniqueHandle(int* p = nullptr);
      ~UniqueHandle();
      UniqueHandle(const UniqueHandle&) = delete;
      UniqueHandle& operator=(const UniqueHandle&) = delete;
      UniqueHandle(UniqueHandle&&) noexcept;
      UniqueHandle& operator=(UniqueHandle&&) noexcept;
      int* get() const;
      int* release();
      void reset(int* p = nullptr);
  };
```

**Examples:**

| Code | Expected |
|---|---|
| `UniqueHandle h(new int(42)); *h.get()` | `42` |
| `UniqueHandle h2 = std::move(h); h.get()` | `nullptr` |
| `h2.release()` | returns pointer, `h2.get()` is `nullptr` |

**Constraints:** No copying allowed. Must not leak memory.

<details><summary>Solution</summary>

```cpp
#include <iostream>

class UniqueHandle {
    int* ptr_;
public:
    explicit UniqueHandle(int* p = nullptr) : ptr_{p} {}

    ~UniqueHandle() { delete ptr_; }

    // Delete copy operations — this is a move-only type
    UniqueHandle(const UniqueHandle&) = delete;
    UniqueHandle& operator=(const UniqueHandle&) = delete;

    // Move constructor: steal the pointer
    UniqueHandle(UniqueHandle&& other) noexcept : ptr_{other.ptr_} {
        other.ptr_ = nullptr;
    }

    // Move assignment: release own resource, steal other's
    UniqueHandle& operator=(UniqueHandle&& other) noexcept {
        if (this != &other) {
            delete ptr_;
            ptr_ = other.ptr_;
            other.ptr_ = nullptr;
        }
        return *this;
    }

    int* get() const { return ptr_; }

    // Release ownership: return pointer without deleting
    int* release() {
        int* tmp = ptr_;
        ptr_ = nullptr;
        return tmp;
    }

    // Reset: delete current, take ownership of new pointer
    void reset(int* p = nullptr) {
        delete ptr_;
        ptr_ = p;
    }
};

int main() {
    UniqueHandle h(new int(42));
    std::cout << *h.get() << '\n';       // 42

    UniqueHandle h2 = std::move(h);
    std::cout << (h.get() == nullptr) << '\n';  // 1 (true)
    std::cout << *h2.get() << '\n';      // 42

    int* raw = h2.release();
    std::cout << (h2.get() == nullptr) << '\n'; // 1
    delete raw;  // we own it now
}
```

</details>

---

### Problem 3: Copy-on-write string (§5.2, p.51; §5.2.1, p.53)

Implement a simple `COWString` that shares its internal buffer between copies but makes a deep copy when a non-const operation (like `set_char`) is called.

```
Signature:
  class COWString {
  public:
      COWString(const char* s);
      COWString(const COWString&);
      COWString& operator=(const COWString&);
      ~COWString();
      char get_char(int i) const;
      void set_char(int i, char c);  // triggers copy if shared
      int length() const;
  };
```

**Examples:**

| Code | Expected |
|---|---|
| `COWString a("hello"); COWString b = a;` | both share buffer |
| `b.set_char(0, 'H');` | b detaches: `b="Hello"`, `a="hello"` |

**Constraints:** Use reference counting with `int* ref_count`. Must not leak. Must not double-free.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <cstring>

class COWString {
    char* data_;
    int len_;
    int* ref_count_;  // shared reference count

    // Detach: if shared, make a private copy
    void detach() {
        if (*ref_count_ > 1) {
            --(*ref_count_);
            ref_count_ = new int(1);
            char* new_data = new char[len_ + 1];
            std::memcpy(new_data, data_, len_ + 1);
            data_ = new_data;
        }
    }

public:
    COWString(const char* s)
        : len_{static_cast<int>(std::strlen(s))},
          ref_count_{new int(1)} {
        data_ = new char[len_ + 1];
        std::memcpy(data_, s, len_ + 1);
    }

    // Copy constructor: share the buffer, increment ref count
    COWString(const COWString& other)
        : data_{other.data_}, len_{other.len_}, ref_count_{other.ref_count_} {
        ++(*ref_count_);
    }

    // Copy assignment
    COWString& operator=(const COWString& other) {
        if (this != &other) {
            // Release current
            if (--(*ref_count_) == 0) { delete[] data_; delete ref_count_; }
            // Share with other
            data_ = other.data_;
            len_ = other.len_;
            ref_count_ = other.ref_count_;
            ++(*ref_count_);
        }
        return *this;
    }

    ~COWString() {
        if (--(*ref_count_) == 0) {
            delete[] data_;
            delete ref_count_;
        }
    }

    char get_char(int i) const { return data_[i]; }

    void set_char(int i, char c) {
        detach();        // copy if shared — this is the "write" trigger
        data_[i] = c;
    }

    int length() const { return len_; }
};

int main() {
    COWString a("hello");
    COWString b = a;                    // shared
    std::cout << b.get_char(0) << '\n'; // 'h'
    b.set_char(0, 'H');                 // detaches b
    std::cout << a.get_char(0) << '\n'; // 'h' (unchanged)
    std::cout << b.get_char(0) << '\n'; // 'H'
}
```

</details>

---

## 4. Output Prediction (4 snippets)

**Snippet 1** — Move semantics (§5.2.1, p.53)

```cpp
#include <iostream>
#include <vector>
int main() {
    std::vector<int> v = {1, 2, 3};
    std::vector<int> w = std::move(v);
    std::cout << v.size() << " " << w.size() << '\n';
}
```

<details><summary>Answer</summary>

Output: `0 3`. After the move, `v` is in a valid but empty state (the standard guarantees `vector`'s moved-from state is empty). `w` now owns the data.

</details>

---

**Snippet 2** — Copy vs. move (§5.2, p.51)

```cpp
#include <iostream>
struct S {
    S() { std::cout << "ctor "; }
    S(const S&) { std::cout << "copy "; }
    S(S&&) { std::cout << "move "; }
    ~S() { std::cout << "dtor "; }
};
int main() {
    S a;
    S b = a;
    S c = std::move(a);
}
```

<details><summary>Answer</summary>

Output: `ctor copy move dtor dtor dtor `

`a` is constructed, `b` is copy-constructed from `a`, `c` is move-constructed from `a`. All three are destructed in reverse order at end of `main`.

</details>

---

**Snippet 3** — Deleted copy (§5.1.1, p.49)

```cpp
#include <iostream>
struct NoCopy {
    NoCopy() = default;
    NoCopy(const NoCopy&) = delete;
};
int main() {
    NoCopy a;
    NoCopy b = a;
}
```

<details><summary>Answer</summary>

**Compilation error.** The copy constructor is deleted, so `NoCopy b = a;` is ill-formed.

</details>

---

**Snippet 4** — explicit constructor (§5.3, p.56)

```cpp
#include <iostream>
struct Wrapper {
    int val;
    explicit Wrapper(int v) : val{v} {}
};
void print(Wrapper w) { std::cout << w.val << '\n'; }
int main() {
    print(42);
}
```

<details><summary>Answer</summary>

**Compilation error.** `Wrapper(int)` is `explicit`, so `42` cannot implicitly convert to `Wrapper`. You need `print(Wrapper{42})`.

</details>

---

## 5. Debug This (3 snippets)

**Bug 1** — Double-free due to missing copy constructor (§5.1.1, p.49)

This code is supposed to safely copy a resource-holding class:

```cpp
class Holder {
    int* data;
public:
    Holder(int v) : data{new int(v)} {}
    ~Holder() { delete data; }
};

int main() {
    Holder a(42);
    Holder b = a;
}
```

<details><summary>Answer</summary>

**Bug:** No user-defined copy constructor. The compiler generates a shallow copy: `b.data` and `a.data` point to the same `int`. When both destruct, the same memory is `delete`d twice — **double-free**.

**Fix:** Implement the Rule of Five. At minimum, add a copy constructor that performs a deep copy: `Holder(const Holder& other) : data{new int(*other.data)} {}`.

</details>

---

**Bug 2** — Moved-from object used (§5.2.1, p.53)

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v = {1, 2, 3};
    std::vector<int> w = std::move(v);
    std::cout << v[0] << '\n';  // use moved-from vector
}
```

<details><summary>Answer</summary>

**Bug:** After `std::move(v)`, `v` is in a valid-but-unspecified state (empty for `vector`). Accessing `v[0]` is undefined behavior because the vector is empty.

**Fix:** Either don't use `v` after moving, or check `v.empty()` first, or assign a new value to `v` before using it.

</details>

---

**Bug 3** — Missing `noexcept` on move operations (§5.2.1, p.53)

```cpp
#include <vector>

class Widget {
    int* data;
public:
    Widget() : data{new int(0)} {}
    Widget(Widget&& other) : data{other.data} { other.data = nullptr; }
    Widget(const Widget& other) : data{new int(*other.data)} {}
    Widget& operator=(const Widget&) = default;
    ~Widget() { delete data; }
};

int main() {
    std::vector<Widget> v;
    for (int i = 0; i < 100; ++i)
        v.push_back(Widget());
}
```

<details><summary>Answer</summary>

**Bug:** The move constructor is not `noexcept`. `std::vector` only uses move during reallocation if the move constructor is `noexcept`. Without it, `vector` falls back to **copying** for exception safety, making the code much slower.

**Fix:** `Widget(Widget&& other) noexcept : data{other.data} { other.data = nullptr; }`

</details>

---

## 6. System Design / Open-ended (2 questions)

**Q1.** Explain how the Rule of Five protects against double-free bugs. Draw the ownership model for a class that owns a heap-allocated buffer. Trace what happens when: (a) you copy the object, (b) you move the object, (c) both original and copy are destroyed. What would go wrong without the custom operations? (§5.1.1, p.49; §5.2, p.51)

---

**Q2.** You're designing a `FileHandle` class that wraps an OS file descriptor. It should be movable but not copyable (you can't duplicate an OS handle trivially). Sketch the class interface. Why is `noexcept` on the move constructor important for containers like `std::vector`? What happens if the destructor throws when closing the file? (§5.2.1, p.53; §5.1.1, p.49)

---

## 7. Rapid Fire (10 true/false)

1. `std::move(x)` physically moves data out of `x`. (§5.2.1, p.53)

<details><summary>Answer</summary>False. `std::move` is just a cast to rvalue reference. The move constructor/assignment does the actual transfer.</details>

2. The compiler-generated copy constructor performs a shallow (member-wise) copy. (§5.1.1, p.49)

<details><summary>Answer</summary>True. It copies each member, which is shallow for raw pointers.</details>

3. A moved-from object is in an undefined state and must not be touched. (§5.2.1, p.53)

<details><summary>Answer</summary>False. It's in a valid-but-unspecified state. You can destroy it or assign to it.</details>

4. `= delete` makes a function inaccessible — not just private, but non-existent for overload resolution. (§5.1.1, p.49)

<details><summary>Answer</summary>True. A deleted function participates in overload resolution but causes a compile error if selected.</details>

5. RAII ensures resources are released even if an exception is thrown. (§5.2, p.51)

<details><summary>Answer</summary>True. The destructor runs during stack unwinding, releasing resources.</details>

6. `noexcept` on a move constructor is optional and has no practical impact. (§5.2.1, p.53)

<details><summary>Answer</summary>False. Without `noexcept`, containers like `vector` fall back to copying during reallocation.</details>

7. `explicit` prevents both implicit construction and implicit conversion. (§5.3, p.56)

<details><summary>Answer</summary>True. It prevents the compiler from using the constructor for implicit conversions.</details>

8. The Rule of Zero says: if you don't manage resources directly, don't define any special operations. (§5.1.1, p.49)

<details><summary>Answer</summary>True. Use smart pointers and standard containers, and the compiler-generated operations do the right thing.</details>

9. Copy elision is an optimization the compiler must always perform. (§5.2.2, p.54)

<details><summary>Answer</summary>False. Mandatory copy elision was introduced in C++17 for certain cases (returning prvalues), but NRVO remains optional.</details>

10. A destructor should be marked `noexcept` explicitly. (§5.1.1, p.49)

<details><summary>Answer</summary>False (technically). Destructors are implicitly `noexcept` since C++11. You don't need to mark them explicitly.</details>
