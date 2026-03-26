# Chapter 4: Classes

---

## Quick Reference

- **Concrete types** behave "just like built-in types" — they can be placed on the stack, copied, and moved (§4.2, p.39)
- **Abstract types** define interfaces via pure virtual functions (`= 0`); you cannot instantiate them (§4.3, p.43)
- **Virtual functions** enable runtime polymorphism; the actual function called depends on the object's dynamic type (§4.3, p.43)
- A class with a virtual function should have a **virtual destructor** (§4.3, p.44)
- Use `override` to explicitly mark functions that override a base class virtual (§4.3, p.44)
- **Class hierarchies** use inheritance: `class Derived : public Base { ... }` (§4.4, p.45)
- `dynamic_cast<T*>(p)` checks at runtime whether `p` points to a `T` — returns `nullptr` if not (§4.4.2, p.47)
- Avoid **object slicing**: passing a derived object by value to a base parameter loses derived data (§4.4, p.46)
- Access base class functions with `Base::func()` inside a derived class (§4.4, p.45)
- Prefer abstract interfaces over deep hierarchies (§4.3, p.43)
- `unique_ptr` and `shared_ptr` manage polymorphic objects on the heap (§4.4.1, p.46)
- Member initializer lists (`: member{value}`) are the preferred way to initialize members (§4.2.1, p.40)

---

## 1. Multiple Choice (8 questions)

**Q1.** What makes a class "abstract"? (§4.3, p.43)

a) It has no constructors
b) It has at least one pure virtual function (`= 0`)
c) It is declared with the `abstract` keyword
d) It has only private members

<details><summary>Answer</summary>

**b)** A class with at least one pure virtual function is abstract and cannot be instantiated directly. Derived classes must override all pure virtual functions to be concrete.

</details>

---

**Q2.** Why should a class with virtual functions have a virtual destructor? (§4.3, p.44)

a) To make the class abstract
b) To prevent memory leaks when deleting through a base pointer
c) To enable copy construction
d) Virtual destructors are only needed for templates

<details><summary>Answer</summary>

**b)** Without a virtual destructor, `delete base_ptr` only calls the base destructor, leaking resources owned by the derived class. A virtual destructor ensures the correct destructor chain is called.

</details>

---

**Q3.** What does `override` do? (§4.3, p.44)

a) Makes a function virtual
b) Forces a compile error if the function doesn't actually override a base class virtual function
c) Prevents further overriding in derived classes
d) Is equivalent to `virtual`

<details><summary>Answer</summary>

**b)** `override` is a contextual keyword that tells the compiler to verify the function matches a base class virtual. If there's a signature mismatch (e.g., wrong parameter type), you get a compile error instead of silently creating a new function.

</details>

---

**Q4.** What is object slicing? (§4.4, p.46)

a) Dividing an object into multiple parts
b) Losing derived-class data when copying a derived object into a base variable
c) Casting a base pointer to a derived pointer
d) A technique for reducing object size

<details><summary>Answer</summary>

**b)** When a `Derived` object is assigned or passed to a `Base` variable by value, only the `Base` part is copied — the `Derived` members are "sliced off."

</details>

---

**Q5.** What does `dynamic_cast<Circle*>(shape_ptr)` return if `shape_ptr` doesn't point to a `Circle`? (§4.4.2, p.47)

a) Throws `std::bad_cast`
b) Returns `nullptr`
c) Returns an uninitialized pointer
d) Undefined behavior

<details><summary>Answer</summary>

**b)** When `dynamic_cast` is used with pointers, a failed cast returns `nullptr`. (When used with references, it throws `std::bad_cast`.)

</details>

---

**Q6.** In a concrete class like `complex`, what makes it behave "like a built-in type"? (§4.2, p.39)

a) It must be allocated on the heap
b) It can be created on the stack, copied, and moved efficiently
c) It must inherit from `std::builtin`
d) It cannot have member functions

<details><summary>Answer</summary>

**b)** Concrete types have a fixed size known at compile time, can live on the stack, and support value semantics (copy, move). Their representation is part of the class definition.

</details>

---

**Q7.** Which is the correct way to define a pure virtual function? (§4.3, p.43)

a) `virtual void draw() = delete;`
b) `virtual void draw() = 0;`
c) `void draw() = 0;`
d) `pure virtual void draw();`

<details><summary>Answer</summary>

**b)** `= 0` marks a virtual function as pure. The `virtual` keyword is required. `= delete` means something different (deleted function, not pure virtual).

</details>

---

**Q8.** When should you use a class hierarchy (inheritance) over a concrete type? (§4.3, p.43; §4.2, p.39)

a) Always, because inheritance is more powerful
b) When the set of types is open-ended and you need runtime polymorphism
c) Only for data-only structs
d) When you want to avoid virtual dispatch overhead

<details><summary>Answer</summary>

**b)** Hierarchies are appropriate when new types may be added without modifying existing code, and behavior is selected at runtime. Concrete types are better for fixed, value-like types.

</details>

---

## 2. Fill-in-the-blank Code (5 snippets)

**Snippet 1** — Abstract class with pure virtual (§4.3, p.43)

```cpp
#include <iostream>

class Shape {
public:
    ___ double area() ___ = 0;   // pure virtual
    ___ ~Shape() = default;      // virtual destructor
};

class Circle : public Shape {
    double r;
public:
    Circle(double radius) : r{radius} {}
    double area() ___ { return 3.14159 * r * r; }
};

int main() {
    Circle c(5);
    Shape& s = c;
    std::cout << s.area() << '\n';
}
```

<details><summary>Answer</summary>

```cpp
virtual double area() const = 0;
virtual ~Shape() = default;
double area() const override { return 3.14159 * r * r; }
```

</details>

---

**Snippet 2** — Concrete type with operator overloading (§4.2, p.39)

```cpp
#include <iostream>

class Complex {
    double re, im;
public:
    Complex(double r, double i) : re{r}, im{i} {}
    double real() const { return re; }
    double imag() const { return im; }

    Complex ___(const Complex& other) ___ {  // addition operator
        return {re + other.re, im + other.im};
    }
};

int main() {
    Complex a{1, 2}, b{3, 4};
    auto c = a + b;
    std::cout << c.real() << " + " << c.imag() << "i\n";
}
```

<details><summary>Answer</summary>

```cpp
Complex operator+(const Complex& other) const {
    return {re + other.re, im + other.im};
}
```

</details>

---

**Snippet 3** — dynamic_cast (§4.4.2, p.47)

```cpp
#include <iostream>

class Shape { public: virtual ~Shape() = default; };
class Circle : public Shape { public: double radius = 5; };
class Square : public Shape { public: double side = 4; };

void describe(Shape* s) {
    if (auto* c = ___<___*>(s)) {
        std::cout << "Circle with radius " << c->radius << '\n';
    } else if (auto* sq = ___<___*>(s)) {
        std::cout << "Square with side " << sq->side << '\n';
    }
}

int main() {
    Circle c;
    describe(&c);
}
```

<details><summary>Answer</summary>

```cpp
if (auto* c = dynamic_cast<Circle*>(s)) {
    ...
} else if (auto* sq = dynamic_cast<Square*>(s)) {
    ...
}
```

</details>

---

**Snippet 4** — override keyword (§4.3, p.44)

```cpp
#include <iostream>

class Base {
public:
    virtual void greet() const { std::cout << "Hello from Base\n"; }
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void greet() const ___ {  // explicitly mark as override
        std::cout << "Hello from Derived\n";
    }
};

int main() {
    Derived d;
    Base& b = d;
    b.greet();
}
```

<details><summary>Answer</summary>

```cpp
void greet() const override { ... }
```

Output: `Hello from Derived` — virtual dispatch calls the derived version.

</details>

---

**Snippet 5** — unique_ptr with polymorphism (§4.4.1, p.46)

```cpp
#include <iostream>
#include <memory>

class Animal {
public:
    virtual void speak() const = 0;
    virtual ~Animal() = default;
};

class Dog : public Animal {
public:
    void speak() const override { std::cout << "Woof!\n"; }
};

int main() {
    ___<Animal> pet = ___<Dog>();  // create on heap
    pet->speak();
}
```

<details><summary>Answer</summary>

```cpp
std::unique_ptr<Animal> pet = std::make_unique<Dog>();
```

`unique_ptr<Animal>` manages a `Dog` on the heap; the virtual destructor ensures proper cleanup.

</details>

---

## 3. Code from Scratch — LeetCode Style (3 problems)

### Problem 1: Shape hierarchy with area() (§4.3, p.43)

Implement an abstract `Shape` class with a pure virtual `area()`. Then implement `Circle` (takes radius) and `Rectangle` (takes width, height). Write a function that takes a `const Shape&` and prints the area.

```
Signatures:
  class Shape { public: virtual double area() const = 0; virtual ~Shape() = default; };
  class Circle : public Shape { ... };
  class Rectangle : public Shape { ... };
  void print_area(const Shape& s);
```

**Examples:**

| Shape | area() |
|---|---|
| `Circle(5)` | `~78.54` |
| `Rectangle(3, 4)` | `12.0` |

**Constraints:** Must use virtual dispatch. `print_area` must not know the concrete type. No `dynamic_cast`.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <cmath>

class Shape {
public:
    virtual double area() const = 0;   // pure virtual — shapes must implement this
    virtual ~Shape() = default;        // virtual dtor for safe polymorphic deletion
};

class Circle : public Shape {
    double r;
public:
    explicit Circle(double radius) : r{radius} {}
    double area() const override {     // override: compiler checks signature match
        return M_PI * r * r;
    }
};

class Rectangle : public Shape {
    double w, h;
public:
    Rectangle(double width, double height) : w{width}, h{height} {}
    double area() const override {
        return w * h;
    }
};

// Works with any Shape — no knowledge of concrete type
void print_area(const Shape& s) {
    std::cout << "Area: " << s.area() << '\n';
}

int main() {
    Circle c(5);
    Rectangle r(3, 4);
    print_area(c);  // Area: 78.5398
    print_area(r);  // Area: 12
}
```

</details>

---

### Problem 2: Concrete Complex number class (§4.2, p.39)

Implement a `Complex` class with `operator+`, `operator*`, and `operator==`. Provide `real()` and `imag()` accessors.

```
Signature:
  class Complex {
  public:
      Complex(double r, double i);
      double real() const;
      double imag() const;
      Complex operator+(const Complex&) const;
      Complex operator*(const Complex&) const;
      bool operator==(const Complex&) const;
  };
```

**Examples:**

| Expression | Result |
|---|---|
| `Complex(1,2) + Complex(3,4)` | `Complex(4,6)` |
| `Complex(1,2) * Complex(3,4)` | `Complex(-5,10)` |
| `Complex(1,2) == Complex(1,2)` | `true` |

**Constraints:** Value type — all operations on stack. No heap allocation.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <cmath>

class Complex {
    double re, im;
public:
    Complex(double r = 0, double i = 0) : re{r}, im{i} {}

    double real() const { return re; }
    double imag() const { return im; }

    // (a+bi) + (c+di) = (a+c) + (b+d)i
    Complex operator+(const Complex& rhs) const {
        return {re + rhs.re, im + rhs.im};
    }

    // (a+bi)(c+di) = (ac-bd) + (ad+bc)i
    Complex operator*(const Complex& rhs) const {
        return {re * rhs.re - im * rhs.im,
                re * rhs.im + im * rhs.re};
    }

    bool operator==(const Complex& rhs) const {
        return re == rhs.re && im == rhs.im;
    }
};

int main() {
    Complex a(1, 2), b(3, 4);
    auto sum = a + b;
    auto prod = a * b;
    std::cout << sum.real() << "+" << sum.imag() << "i\n";    // 4+6i
    std::cout << prod.real() << "+" << prod.imag() << "i\n";  // -5+10i
    std::cout << std::boolalpha << (a == Complex(1, 2)) << '\n'; // true
}
```

</details>

---

### Problem 3: Type-safe shape identifier using dynamic_cast (§4.4.2, p.47)

Given a `vector<unique_ptr<Shape>>`, write a function that counts how many shapes are `Circle`s.

```
Signature: int count_circles(const std::vector<std::unique_ptr<Shape>>& shapes);
```

**Examples:**

| Input shapes | Output |
|---|---|
| Circle, Rectangle, Circle, Circle | `3` |
| Rectangle, Rectangle | `0` |

**Constraints:** Use `dynamic_cast`. Do not add a `type()` member to Shape.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <vector>
#include <memory>

class Shape {
public:
    virtual double area() const = 0;
    virtual ~Shape() = default;
};

class Circle : public Shape {
    double r;
public:
    Circle(double radius) : r{radius} {}
    double area() const override { return 3.14159 * r * r; }
};

class Rectangle : public Shape {
    double w, h;
public:
    Rectangle(double width, double height) : w{width}, h{height} {}
    double area() const override { return w * h; }
};

int count_circles(const std::vector<std::unique_ptr<Shape>>& shapes) {
    int count = 0;
    for (const auto& s : shapes) {
        // dynamic_cast returns nullptr if s doesn't point to a Circle
        if (dynamic_cast<Circle*>(s.get()))
            ++count;
    }
    return count;
}

int main() {
    std::vector<std::unique_ptr<Shape>> shapes;
    shapes.push_back(std::make_unique<Circle>(5));
    shapes.push_back(std::make_unique<Rectangle>(3, 4));
    shapes.push_back(std::make_unique<Circle>(2));
    shapes.push_back(std::make_unique<Circle>(1));
    std::cout << count_circles(shapes) << '\n';  // 3
}
```

</details>

---

## 4. Output Prediction (4 snippets)

**Snippet 1** — Virtual dispatch (§4.3, p.43)

```cpp
#include <iostream>
class Base {
public:
    virtual void f() { std::cout << "Base\n"; }
    virtual ~Base() = default;
};
class Derived : public Base {
public:
    void f() override { std::cout << "Derived\n"; }
};
int main() {
    Derived d;
    Base* bp = &d;
    Base& br = d;
    bp->f();
    br.f();
    Base b = d;   // copy (slicing!)
    b.f();
}
```

<details><summary>Answer</summary>

Output:
```
Derived
Derived
Base
```

The first two calls use virtual dispatch through pointer/reference, calling `Derived::f()`. The third copies `d` into a `Base` object (slicing) — the `Base` copy has no knowledge of `Derived`, so `Base::f()` is called.

</details>

---

**Snippet 2** — Pure virtual (§4.3, p.43)

```cpp
#include <iostream>
class Shape {
public:
    virtual double area() const = 0;
    virtual ~Shape() = default;
};

int main() {
    Shape s;
    std::cout << s.area() << '\n';
}
```

<details><summary>Answer</summary>

**Compilation error.** `Shape` is abstract (has a pure virtual function) and cannot be instantiated.

</details>

---

**Snippet 3** — override catches bugs (§4.3, p.44)

```cpp
#include <iostream>
class Base {
public:
    virtual void process(int x) const { std::cout << "Base\n"; }
    virtual ~Base() = default;
};
class Derived : public Base {
public:
    void process(double x) const override { std::cout << "Derived\n"; }
};
int main() {
    Derived d;
    d.process(42);
}
```

<details><summary>Answer</summary>

**Compilation error.** `override` checks that the function signature matches a base class virtual. `process(double)` does not match `process(int)`, so the compiler rejects it. Without `override`, this would silently create a new function.

</details>

---

**Snippet 4** — Destructor order (§4.3, p.44)

```cpp
#include <iostream>
struct A {
    ~A() { std::cout << "~A "; }
};
struct B : A {
    ~B() { std::cout << "~B "; }
};
int main() {
    B b;
}
```

<details><summary>Answer</summary>

Output: `~B ~A `. Destructors run in reverse order of construction: derived first, then base. Note: these destructors are not virtual, which is fine here since we're not deleting through a base pointer.

</details>

---

## 5. Debug This (3 snippets)

**Bug 1** — Missing virtual destructor (§4.3, p.44)

This code is supposed to properly clean up a derived object:

```cpp
#include <memory>
#include <iostream>

class Base {
public:
    ~Base() { std::cout << "~Base\n"; }
};

class Derived : public Base {
    int* data;
public:
    Derived() : data{new int[100]} {}
    ~Derived() { delete[] data; std::cout << "~Derived\n"; }
};

int main() {
    Base* p = new Derived();
    delete p;
}
```

<details><summary>Answer</summary>

**Bug:** `Base` destructor is not `virtual`. `delete p` only calls `~Base()`, leaking the `data` array allocated in `Derived`. `~Derived()` is never called.

**Fix:** `virtual ~Base() { std::cout << "~Base\n"; }`

</details>

---

**Bug 2** — Slicing in function call (§4.4, p.46)

This code is supposed to use polymorphism to call the derived `speak()`:

```cpp
#include <iostream>

class Animal {
public:
    virtual void speak() const { std::cout << "...\n"; }
    virtual ~Animal() = default;
};

class Cat : public Animal {
public:
    void speak() const override { std::cout << "Meow!\n"; }
};

void make_sound(Animal a) {
    a.speak();
}

int main() {
    Cat c;
    make_sound(c);
}
```

<details><summary>Answer</summary>

**Bug:** `make_sound` takes `Animal` **by value**, slicing the `Cat` object. Virtual dispatch won't call `Cat::speak()` because `a` is a plain `Animal`.

**Fix:** Pass by reference: `void make_sound(const Animal& a)`.

</details>

---

**Bug 3** — Forgetting override leads to silent bug (§4.3, p.44)

```cpp
#include <iostream>

class Base {
public:
    virtual void update(int value) { std::cout << "Base: " << value << '\n'; }
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void update(int value) const {  // note the const
        std::cout << "Derived: " << value << '\n';
    }
};

int main() {
    Derived d;
    Base& b = d;
    b.update(42);
}
```

<details><summary>Answer</summary>

**Bug:** `Derived::update` has `const` but `Base::update` does not — they have different signatures, so `Derived::update` does **not** override `Base::update`. The base version is called. Adding `override` would catch this at compile time.

**Fix:** Remove `const` from `Derived::update` (or add `const` to both), and add `override`.

</details>

---

## 6. System Design / Open-ended (2 questions)

**Q1.** Design a simple plugin system where each plugin implements an interface `Plugin` with `name()`, `initialize()`, and `execute()`. Plugins are loaded into a `vector<unique_ptr<Plugin>>`. What are the tradeoffs of using abstract base classes + virtual dispatch vs. templates? When would you choose one over the other? (§4.3, p.43; §4.4.1, p.46)

---

**Q2.** You have a drawing application with shapes (Circle, Rectangle, Triangle). Should you use `dynamic_cast` to dispatch drawing commands, or should you add a `draw()` virtual function to the `Shape` base? What are the maintenance implications when a new shape is added? Relate this to the "expression problem." (§4.3, p.43; §4.4.2, p.47)

---

## 7. Rapid Fire (10 true/false)

1. A class with a pure virtual function can be instantiated. (§4.3, p.43)

<details><summary>Answer</summary>False. Abstract classes cannot be instantiated directly.</details>

2. `override` is required when overriding a virtual function. (§4.3, p.44)

<details><summary>Answer</summary>False. It's optional but strongly recommended — it catches signature mismatches at compile time.</details>

3. `dynamic_cast` on a pointer returns `nullptr` on failure. (§4.4.2, p.47)

<details><summary>Answer</summary>True. For references, it throws `std::bad_cast` instead.</details>

4. Object slicing can occur when passing a derived object by reference to a base parameter. (§4.4, p.46)

<details><summary>Answer</summary>False. Slicing occurs on copy (by value), not by reference.</details>

5. A concrete type's representation is part of its definition. (§4.2, p.39)

<details><summary>Answer</summary>True. This allows stack allocation and compile-time size determination.</details>

6. Every class with virtual functions should have a virtual destructor. (§4.3, p.44)

<details><summary>Answer</summary>True. Otherwise, deleting through a base pointer causes undefined behavior.</details>

7. `make_unique<Derived>()` returns a `unique_ptr<Derived>`. (§4.4.1, p.46)

<details><summary>Answer</summary>True. It can be implicitly converted to `unique_ptr<Base>` if Derived inherits from Base.</details>

8. Virtual dispatch works through value copies, not just pointers and references. (§4.3, p.43)

<details><summary>Answer</summary>False. Virtual dispatch requires a pointer or reference to the base class. Copies slice.</details>

9. `dynamic_cast` requires the base class to have at least one virtual function. (§4.4.2, p.47)

<details><summary>Answer</summary>True. RTTI (runtime type information) is only available for polymorphic types (those with virtual functions).</details>

10. A derived class can override a non-virtual base class function. (§4.3, p.43)

<details><summary>Answer</summary>False. Only virtual functions can be overridden. A non-virtual function in the derived class simply hides the base version.</details>

---

## 8. Capstone Implementation Challenge

### Polymorphic Expression Tree (§4.2–4.4, p.39–47)

**Motivation:** You're building a symbolic math evaluator. Expressions like `(3 + 4) * 2` are represented as a tree of nodes. Each node is either a literal number or a binary operation. The tree supports evaluation and conversion to a string representation.

**Signatures:**

```cpp
class Expr {
public:
    virtual double eval() const = 0;
    virtual std::string to_string() const = 0;
    virtual ~Expr() = default;
};

class Literal : public Expr {
    double value_;
public:
    explicit Literal(double v);
    double eval() const override;
    std::string to_string() const override;
};

class BinaryOp : public Expr {
protected:
    std::unique_ptr<Expr> left_, right_;
public:
    BinaryOp(std::unique_ptr<Expr> l, std::unique_ptr<Expr> r);
};

class Add : public BinaryOp {
public:
    using BinaryOp::BinaryOp;
    double eval() const override;
    std::string to_string() const override;
};

class Multiply : public BinaryOp {
public:
    using BinaryOp::BinaryOp;
    double eval() const override;
    std::string to_string() const override;
};

// Helper factory
std::unique_ptr<Expr> lit(double v);
std::unique_ptr<Expr> add(std::unique_ptr<Expr> l, std::unique_ptr<Expr> r);
std::unique_ptr<Expr> mul(std::unique_ptr<Expr> l, std::unique_ptr<Expr> r);
```

**Test Scenarios:**

1. `lit(42)->eval()` returns `42.0`, `to_string()` returns `"42"`.
2. `add(lit(3), lit(4))->eval()` returns `7.0`, `to_string()` returns `"(3 + 4)"`.
3. `mul(add(lit(3), lit(4)), lit(2))` evaluates to `14.0`, `to_string()` returns `"((3 + 4) * 2)"`.

**Constraints:**
- Must use virtual dispatch (no `dynamic_cast` or type tags)
- Children owned via `std::unique_ptr<Expr>` — no raw `new`/`delete` in user code
- `Expr` must have a virtual destructor
- Use `override` on all overriding functions
- No slicing — everything through pointers/references

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <string>
#include <memory>
#include <sstream>
#include <cmath>

// Abstract base — defines the expression interface
class Expr {
public:
    virtual double eval() const = 0;
    virtual std::string to_string() const = 0;
    virtual ~Expr() = default;  // virtual dtor: safe polymorphic deletion
};

// Leaf node: a numeric literal
class Literal : public Expr {
    double value_;
public:
    explicit Literal(double v) : value_{v} {}

    double eval() const override { return value_; }

    std::string to_string() const override {
        // Clean formatting: no trailing zeros for whole numbers
        if (value_ == std::floor(value_))
            return std::to_string(static_cast<int>(value_));
        std::ostringstream oss;
        oss << value_;
        return oss.str();
    }
};

// Internal node: binary operation with two children
class BinaryOp : public Expr {
protected:
    std::unique_ptr<Expr> left_;   // unique_ptr owns children
    std::unique_ptr<Expr> right_;
public:
    BinaryOp(std::unique_ptr<Expr> l, std::unique_ptr<Expr> r)
        : left_{std::move(l)}, right_{std::move(r)} {}
    // No custom destructor needed — unique_ptr handles cleanup
};

class Add : public BinaryOp {
public:
    using BinaryOp::BinaryOp;  // inherit constructor

    double eval() const override {
        return left_->eval() + right_->eval();  // virtual dispatch on children
    }

    std::string to_string() const override {
        return "(" + left_->to_string() + " + " + right_->to_string() + ")";
    }
};

class Multiply : public BinaryOp {
public:
    using BinaryOp::BinaryOp;

    double eval() const override {
        return left_->eval() * right_->eval();
    }

    std::string to_string() const override {
        return "(" + left_->to_string() + " * " + right_->to_string() + ")";
    }
};

// Factory helpers — hide make_unique details from caller
std::unique_ptr<Expr> lit(double v) {
    return std::make_unique<Literal>(v);
}

std::unique_ptr<Expr> add(std::unique_ptr<Expr> l, std::unique_ptr<Expr> r) {
    return std::make_unique<Add>(std::move(l), std::move(r));
}

std::unique_ptr<Expr> mul(std::unique_ptr<Expr> l, std::unique_ptr<Expr> r) {
    return std::make_unique<Multiply>(std::move(l), std::move(r));
}

int main() {
    // Scenario 1: literal
    auto e1 = lit(42);
    std::cout << e1->to_string() << " = " << e1->eval() << '\n';
    // 42 = 42

    // Scenario 2: addition
    auto e2 = add(lit(3), lit(4));
    std::cout << e2->to_string() << " = " << e2->eval() << '\n';
    // (3 + 4) = 7

    // Scenario 3: nested expression (3 + 4) * 2
    auto e3 = mul(add(lit(3), lit(4)), lit(2));
    std::cout << e3->to_string() << " = " << e3->eval() << '\n';
    // ((3 + 4) * 2) = 14

    // Scenario 4: deeper nesting (1 + 2) * (3 + 4)
    auto e4 = mul(add(lit(1), lit(2)), add(lit(3), lit(4)));
    std::cout << e4->to_string() << " = " << e4->eval() << '\n';
    // ((1 + 2) * (3 + 4)) = 21
}
```

Key decisions:
- **Abstract `Expr` base** with pure virtuals creates a clean interface (§4.3).
- **`unique_ptr` children** ensure automatic cleanup — no memory leaks even with deep trees (§4.4.1).
- **`override`** on every virtual function catches signature mismatches at compile time (§4.3).
- **Virtual destructor** on `Expr` ensures derived destructors run when deleting through base pointer (§4.3).
- **Factory functions** (`lit`, `add`, `mul`) provide a clean DSL — all heap allocation is hidden behind `make_unique`.

</details>
