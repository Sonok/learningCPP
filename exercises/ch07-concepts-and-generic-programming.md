# Chapter 7: Concepts and Generic Programming

---

## Quick Reference

- **Concepts** (C++20) constrain template arguments: `template<typename T> requires Sortable<T>` (§7.2, p.71)
- A concept defines a set of requirements on a type — like a compile-time interface (§7.2, p.71)
- Short-form: `void sort(Sortable auto& c)` constrains `c` to types satisfying `Sortable` (§7.2.1, p.72)
- `requires` clause: `template<typename T> requires std::integral<T>` (§7.2, p.71)
- Standard concepts: `std::same_as`, `std::derived_from`, `std::convertible_to`, `std::integral`, `std::floating_point`, `std::regular`, `std::totally_ordered` (§7.2.4, p.75)
- Concept-based overloading: the most constrained viable function wins (§7.2.3, p.74)
- `requires` expressions test whether code is well-formed: `requires(T a) { a + a; }` (§7.2.2, p.73)
- Concepts produce **clear error messages** at the call site, unlike SFINAE (§7.2, p.71)
- **Generic programming**: write algorithms in terms of concepts, not specific types (§7.1, p.69)
- **Lifting**: identify the minimal requirements an algorithm needs → define a concept for those (§7.3, p.76)
- Concepts subsume other concepts: `Sortable` might require `Regular` + `TotallyOrdered` (§7.2.3, p.74)
- Use concepts to document assumptions about template parameters (§7.2, p.71)

---

## 1. Multiple Choice (8 questions)

**Q1.** What problem do concepts solve? (§7.2, p.71)

a) Runtime type checking
b) Constraining template parameters with clear, readable requirements and better error messages
c) Replacing virtual functions
d) Memory management

<details><summary>Answer</summary>

**b)** Without concepts, template errors are notoriously verbose — failures deep inside implementation code. Concepts check requirements at the call site, producing clear, actionable errors.

</details>

---

**Q2.** What is the short-form syntax for a constrained function template? (§7.2.1, p.72)

a) `template<Sortable T> void sort(T& c);`
b) `void sort(Sortable auto& c);`
c) `void sort(concept Sortable& c);`
d) `void sort<Sortable>(auto& c);`

<details><summary>Answer</summary>

**b)** `Sortable auto& c` is the abbreviated syntax. `auto` is deduced, and `Sortable` constrains the deduced type.

</details>

---

**Q3.** What does this `requires` expression test? (§7.2.2, p.73)

```cpp
requires(T a, T b) { a + b; }
```

a) Whether `a + b` produces a positive result
b) Whether the expression `a + b` is well-formed (compiles) for type `T`
c) Whether `T` is an arithmetic type
d) Whether `a + b` doesn't throw

<details><summary>Answer</summary>

**b)** A `requires` expression tests whether the enclosed expressions are syntactically valid (well-formed) for the given types. It does not evaluate them.

</details>

---

**Q4.** When two overloads match and one is more constrained, which is selected? (§7.2.3, p.74)

a) The less constrained one
b) Compilation error — ambiguous
c) The more constrained one (subsumption)
d) The first one declared

<details><summary>Answer</summary>

**c)** Concept-based overload resolution selects the most constrained viable candidate. This is called subsumption — a more specific concept "subsumes" a more general one.

</details>

---

**Q5.** Which standard concept requires that a type can be compared with `==` and `!=`, is copyable, and has a default constructor? (§7.2.4, p.75)

a) `std::integral`
b) `std::totally_ordered`
c) `std::regular`
d) `std::movable`

<details><summary>Answer</summary>

**c)** `std::regular` combines `std::semiregular` (default constructible, copyable) with `std::equality_comparable` (supports `==` and `!=`).

</details>

---

**Q6.** What is "lifting" in the context of generic programming? (§7.3, p.76)

a) Moving code to a higher namespace
b) Identifying the minimal set of operations an algorithm needs and abstracting them into a concept
c) Promoting integers to floating point
d) Moving objects from stack to heap

<details><summary>Answer</summary>

**b)** Lifting is the process of abstracting from concrete types to their essential operations, forming a concept that captures exactly what the algorithm requires — no more, no less.

</details>

---

**Q7.** Can a concept constrain a non-type template parameter? (§7.2, p.71)

a) Yes, always
b) No, concepts only constrain type parameters
c) Only with `requires` clauses
d) Only integral types

<details><summary>Answer</summary>

**c)** While concepts primarily constrain types, you can use `requires` clauses to constrain non-type parameters: `requires (N > 0)`.

</details>

---

**Q8.** What happens if a type doesn't satisfy a concept constraint? (§7.2, p.71)

a) Runtime error
b) Undefined behavior
c) Clear compile error at the call site
d) The template is silently skipped

<details><summary>Answer</summary>

**c)** The compiler produces an error at the point of use, stating which requirement the type fails to meet. This is a major improvement over unconstrained templates.

</details>

---

## 2. Fill-in-the-blank Code (5 snippets)

**Snippet 1** — Basic concept definition (§7.2, p.71)

```cpp
#include <concepts>
#include <iostream>

template<typename T>
___ Addable = requires(T a, T b) {
    { a + b } -> std::convertible_to<T>;
};

template<___ T>  // constrain with Addable
T add(T a, T b) {
    return a + b;
}

int main() {
    std::cout << add(3, 4) << '\n';
}
```

<details><summary>Answer</summary>

```cpp
concept Addable = requires(T a, T b) { ... };
template<Addable T>
T add(T a, T b) { ... }
```

</details>

---

**Snippet 2** — requires clause (§7.2, p.71)

```cpp
#include <concepts>
#include <iostream>

template<typename T>
    ___ std::integral<T>  // constraint
T gcd(T a, T b) {
    while (b != 0) {
        T temp = b;
        b = a % b;
        a = temp;
    }
    return a;
}

int main() {
    std::cout << gcd(12, 8) << '\n';  // 4
}
```

<details><summary>Answer</summary>

```cpp
template<typename T>
    requires std::integral<T>
T gcd(T a, T b) { ... }
```

</details>

---

**Snippet 3** — Short-form constrained parameter (§7.2.1, p.72)

```cpp
#include <concepts>
#include <iostream>

void print_number(___ ___ value) {  // any floating_point type
    std::cout << "Float: " << value << '\n';
}

void print_number(___ ___ value) {  // any integral type
    std::cout << "Int: " << value << '\n';
}

int main() {
    print_number(3.14);
    print_number(42);
}
```

<details><summary>Answer</summary>

```cpp
void print_number(std::floating_point auto value) { ... }
void print_number(std::integral auto value) { ... }
```

Concept-based overloading selects the correct function.

</details>

---

**Snippet 4** — requires expression with compound requirements (§7.2.2, p.73)

```cpp
#include <concepts>
#include <string>

template<typename T>
concept Printable = requires(T a) {
    { a.to_string() } -> std::___<std::string>;  // must return string
};

struct Widget {
    std::string to_string() const { return "Widget"; }
};

template<Printable T>
void print(const T& obj) {
    std::cout << obj.to_string() << '\n';
}
```

<details><summary>Answer</summary>

```cpp
{ a.to_string() } -> std::convertible_to<std::string>;
```

The compound requirement checks both that `to_string()` is valid and that its return type is convertible to `std::string`.

</details>

---

**Snippet 5** — Concept subsumption for overload resolution (§7.2.3, p.74)

```cpp
#include <concepts>
#include <iostream>

template<typename T>
concept Number = std::is_arithmetic_v<T>;

template<typename T>
concept Integer = Number<T> && std::___<T>;  // more constrained

void process(Number auto val) {
    std::cout << "Number: " << val << '\n';
}

void process(Integer auto val) {
    std::cout << "Integer: " << val << '\n';
}

int main() {
    process(3.14);  // calls Number version
    process(42);    // calls Integer version (more constrained)
}
```

<details><summary>Answer</summary>

```cpp
concept Integer = Number<T> && std::integral<T>;
```

`Integer` subsumes `Number`, so for `int` arguments the more constrained `Integer` overload wins.

</details>

---

## 3. Code from Scratch — LeetCode Style (3 problems)

### Problem 1: Constrained container sum (§7.2, p.71)

Write a function template `sum` that takes any container of arithmetic elements and returns their sum. Use concepts to constrain both the container (must support range-for) and the element type (must be arithmetic).

```
Signature: template<typename Container> auto sum(const Container& c);
```

**Examples:**

| Input | Output |
|---|---|
| `vector<int>{1,2,3}` | `6` |
| `vector<double>{1.5, 2.5}` | `4.0` |

**Constraints:** Use `std::ranges::range` and `std::is_arithmetic_v`. Must not compile for `vector<string>`.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <vector>
#include <concepts>
#include <ranges>
#include <type_traits>

// Concept: container must be a range whose elements are arithmetic
template<typename C>
concept ArithmeticRange = std::ranges::range<C> &&
    std::is_arithmetic_v<std::ranges::range_value_t<C>>;

// Constrained function — only accepts arithmetic ranges
template<ArithmeticRange Container>
auto sum(const Container& c) {
    using T = std::ranges::range_value_t<Container>;
    T total{};  // zero-initialize
    for (const auto& elem : c)
        total += elem;
    return total;
}

int main() {
    std::vector<int> vi = {1, 2, 3};
    std::cout << sum(vi) << '\n';  // 6

    std::vector<double> vd = {1.5, 2.5};
    std::cout << sum(vd) << '\n';  // 4

    // std::vector<std::string> vs = {"a"};
    // sum(vs);  // compile error: string is not arithmetic
}
```

</details>

---

### Problem 2: Concept for "has .size() and operator[]" (§7.2.2, p.73)

Define a concept `Indexable` that requires a type to have `.size()` returning something convertible to `size_t` and `operator[]` accepting an integer. Write a function that prints all elements of any `Indexable`.

```
Signature:
  template<typename T> concept Indexable = ...;
  template<Indexable C> void print_all(const C& c);
```

**Examples:**

| Type | Works? |
|---|---|
| `std::vector<int>` | Yes |
| `std::string` | Yes |
| `std::list<int>` | No (no operator[]) |

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <concepts>

template<typename T>
concept Indexable = requires(const T c, std::size_t i) {
    { c.size() } -> std::convertible_to<std::size_t>;
    { c[i] };  // operator[] must be valid
};

template<Indexable C>
void print_all(const C& c) {
    for (std::size_t i = 0; i < c.size(); ++i)
        std::cout << c[i] << " ";
    std::cout << '\n';
}

int main() {
    std::vector<int> v = {1, 2, 3};
    print_all(v);  // 1 2 3

    std::string s = "hello";
    print_all(s);  // h e l l o

    // std::list<int> l = {1, 2};
    // print_all(l);  // compile error: no operator[]
}
```

</details>

---

### Problem 3: Concept-constrained max with custom comparison (§7.2, p.71; §7.2.2, p.73)

Write a `max_by` function that takes two values and a comparison function. Constrain: values must be `std::regular`, and the comparison must be invocable with two `T`s and return something convertible to `bool`.

```
Signature:
  template<std::regular T, typename Cmp>
      requires std::predicate<Cmp, T, T>
  const T& max_by(const T& a, const T& b, Cmp cmp);
```

**Examples:**

| Call | Output |
|---|---|
| `max_by(3, 7, std::less{})` | `7` |
| `max_by("apple"s, "banana"s, [](auto& a, auto& b){ return a.size() < b.size(); })` | `"banana"` |

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <string>
#include <concepts>
#include <functional>

template<std::regular T, typename Cmp>
    requires std::predicate<Cmp, T, T>
const T& max_by(const T& a, const T& b, Cmp cmp) {
    return cmp(a, b) ? b : a;  // if a < b (per cmp), return b
}

int main() {
    std::cout << max_by(3, 7, std::less<>{}) << '\n';  // 7

    using namespace std::string_literals;
    auto by_length = [](const std::string& a, const std::string& b) {
        return a.size() < b.size();
    };
    std::cout << max_by("apple"s, "banana"s, by_length) << '\n';  // banana
}
```

</details>

---

## 4. Output Prediction (4 snippets)

**Snippet 1** — Concept constraint failure (§7.2, p.71)

```cpp
#include <concepts>
void f(std::integral auto x) { }
int main() {
    f(3.14);
}
```

<details><summary>Answer</summary>

**Compilation error.** `double` does not satisfy `std::integral`. The error message will clearly state that the constraint is not satisfied.

</details>

---

**Snippet 2** — Concept-based overloading (§7.2.3, p.74)

```cpp
#include <iostream>
#include <concepts>

void f(auto x)                { std::cout << "generic\n"; }
void f(std::integral auto x)  { std::cout << "integral\n"; }

int main() {
    f(42);
    f(3.14);
    f("hi");
}
```

<details><summary>Answer</summary>

Output:
```
integral
generic
generic
```

For `42` (int), the more constrained `integral` overload wins. For `3.14` (double) and `"hi"` (const char*), only the generic overload matches.

</details>

---

**Snippet 3** — requires expression returning bool (§7.2.2, p.73)

```cpp
#include <iostream>
#include <vector>

template<typename T>
constexpr bool has_push_back = requires(T c, typename T::value_type v) {
    c.push_back(v);
};

int main() {
    std::cout << std::boolalpha;
    std::cout << has_push_back<std::vector<int>> << '\n';
    std::cout << has_push_back<int> << '\n';
}
```

<details><summary>Answer</summary>

Output:
```
true
false
```

`vector<int>` has `push_back`, so the requires expression is satisfied. `int` doesn't have `push_back` or `value_type`.

</details>

---

**Snippet 4** — Subsumption order (§7.2.3, p.74)

```cpp
#include <iostream>
#include <concepts>

template<typename T> concept A = std::is_arithmetic_v<T>;
template<typename T> concept B = A<T> && std::integral<T>;

void g(A auto x) { std::cout << "A\n"; }
void g(B auto x) { std::cout << "B\n"; }

int main() {
    g(42);
    g(3.14);
}
```

<details><summary>Answer</summary>

Output:
```
B
A
```

`42` is `int`, which satisfies both A and B. B subsumes A, so B wins. `3.14` is `double`, which satisfies A but not B (not integral).

</details>

---

## 5. Debug This (3 snippets)

**Bug 1** — Missing `concept` keyword (§7.2, p.71)

```cpp
template<typename T>
bool Numeric = std::is_arithmetic_v<T>;

void f(Numeric auto x) { }
```

<details><summary>Answer</summary>

**Bug:** `Numeric` is declared as a `bool` variable template, not a concept. It cannot be used as a constraint.

**Fix:** `template<typename T> concept Numeric = std::is_arithmetic_v<T>;`

</details>

---

**Bug 2** — requires expression used instead of requires clause (§7.2.2, p.73)

```cpp
template<typename T>
requires(T a) { a + a; }  // this is a requires expression, not a clause
T double_it(T val) {
    return val + val;
}
```

<details><summary>Answer</summary>

**Bug:** `requires(T a) { a + a; }` is a requires *expression* (evaluates to bool), but it's used in the position of a requires *clause*. The syntax is wrong.

**Fix:** `requires requires(T a) { a + a; }` (requires clause containing a requires expression) or define a concept first.

</details>

---

**Bug 3** — Concept not checking what's needed (§7.2.2, p.73)

```cpp
template<typename T>
concept Printable = requires(T a) {
    std::cout << a;  // this checks if it compiles, not if it "works"
};

template<Printable T>
void print(const T& val) {
    std::cout << val << '\n';
}
```

Calling `print` with a `const Widget&` but the concept checks with a non-const `T`:

```cpp
struct Widget {
    friend std::ostream& operator<<(std::ostream&, const Widget&);
};
```

<details><summary>Answer</summary>

**Bug:** The concept checks `requires(T a)` (non-const), but `print` takes `const T&`. This mismatch means the concept may pass even if `operator<<` only works with `const`, or vice versa.

**Fix:** `concept Printable = requires(const T& a) { std::cout << a; };` — match the actual usage.

</details>

---

## 6. System Design / Open-ended (2 questions)

**Q1.** You're designing a generic `sort` function. What minimal set of operations does sorting require? Define a `Sortable` concept by "lifting" from a concrete sort implementation. How does this compare to just documenting requirements in comments? (§7.3, p.76)

---

**Q2.** Concepts vs. `static_assert` vs. SFINAE: all three can constrain templates. Compare them in terms of: (a) error message quality, (b) overload resolution, (c) readability. When would you choose each? (§7.2, p.71)

---

## 7. Rapid Fire (10 true/false)

1. Concepts are evaluated at runtime. (§7.2, p.71)

<details><summary>Answer</summary>False. Concepts are compile-time constraints.</details>

2. `void f(auto x)` is an unconstrained function template. (§7.2.1, p.72)

<details><summary>Answer</summary>True. `auto` with no concept prefix is unconstrained.</details>

3. `requires requires` (double requires) is valid C++ syntax. (§7.2.2, p.73)

<details><summary>Answer</summary>True. The first is a requires-clause, the second starts a requires-expression.</details>

4. Concepts can be used with `static_assert`. (§7.2, p.71)

<details><summary>Answer</summary>True. `static_assert(std::integral<int>)` is valid.</details>

5. A more constrained overload is always preferred over a less constrained one. (§7.2.3, p.74)

<details><summary>Answer</summary>True. This is concept subsumption — the most constrained viable candidate wins.</details>

6. `std::regular` requires move semantics but not copy semantics. (§7.2.4, p.75)

<details><summary>Answer</summary>False. `regular` requires both copy and move, plus default construction and equality comparison.</details>

7. You can define a concept inside a function. (§7.2, p.71)

<details><summary>Answer</summary>False. Concepts must be defined at namespace scope.</details>

8. Concepts replace the need for `enable_if` and SFINAE in most cases. (§7.2, p.71)

<details><summary>Answer</summary>True. Concepts are the modern replacement for SFINAE-based constraints.</details>

9. A `requires` expression can contain statements like `if` and `for`. (§7.2.2, p.73)

<details><summary>Answer</summary>False. Requires expressions test whether expressions are well-formed, not whether statements execute.</details>

10. `std::totally_ordered` implies `std::equality_comparable`. (§7.2.4, p.75)

<details><summary>Answer</summary>True. Total ordering requires `<`, `>`, `<=`, `>=`, and `==`, `!=`.</details>

---

## 8. Capstone Implementation Challenge

### Concept-Constrained Generic Pipeline (§7.2–7.3, p.71–76)

**Motivation:** You're building a data processing pipeline for a financial analytics system. Each stage transforms data, and stages can be chained. The pipeline must be type-safe: each stage's output type must match the next stage's input type. Use concepts to enforce this at compile time.

**Signatures:**

```cpp
// Concept: a stage must be callable with an input and produce an output
template<typename Stage, typename Input>
concept PipelineStage = requires(Stage s, Input in) {
    { s(in) } -> std::convertible_to<typename Stage::output_type>;
};

// A typed stage wrapping any callable
template<typename F, typename In, typename Out>
struct Stage {
    using input_type = In;
    using output_type = Out;
    F func;
    Out operator()(In input) const { return func(input); }
};

// Pipeline chains stages
template<typename... Stages>
class Pipeline {
public:
    Pipeline(Stages... stages);
    auto process(auto input) const;
};

// Helper to create stages
template<typename In, typename Out, typename F>
auto make_stage(F f) -> Stage<F, In, Out>;
```

**Test Scenarios:**

1. Pipeline: `string -> int (parse)`, `int -> double (scale by 1.5)`, `double -> string (format)`. Process `"42"` -> `"63.0"`.
2. Attempting to chain `string -> int` then `double -> string` produces a compile error (type mismatch caught by concepts).
3. Pipeline with a single stage works correctly.

**Constraints:**
- Use C++20 concepts to constrain stage compatibility
- Use `requires` expressions to validate that stages chain correctly
- Define at least one custom concept
- Use variadic templates for the pipeline
- Use `if constexpr` or fold expressions where appropriate

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <string>
#include <sstream>
#include <concepts>
#include <tuple>

// Concept: a callable that transforms In -> Out
template<typename F, typename In>
concept Transformer = requires(F f, In input) {
    { f(input) };  // must be callable with In
};

// Stage wrapper: bundles a callable with its type information
template<typename In, typename Out, typename F>
struct Stage {
    using input_type = In;
    using output_type = Out;
    F func;

    explicit Stage(F f) : func{std::move(f)} {}
    Out operator()(In input) const { return func(input); }
};

// Factory: creates a Stage with explicit In/Out types
template<typename In, typename Out, typename F>
auto make_stage(F f) {
    return Stage<In, Out, F>{std::move(f)};
}

// Pipeline: chains an arbitrary number of stages
template<typename... Stages>
class Pipeline {
    std::tuple<Stages...> stages_;

    // Recursive process: apply stage 0, then pass result to remaining stages
    template<std::size_t I, typename Input>
    auto process_impl(Input&& input) const {
        auto& stage = std::get<I>(stages_);
        auto result = stage(std::forward<Input>(input));

        if constexpr (I + 1 < sizeof...(Stages)) {
            // More stages — chain the result forward
            return process_impl<I + 1>(std::move(result));
        } else {
            // Last stage — return final result
            return result;
        }
    }

public:
    explicit Pipeline(Stages... stages) : stages_{std::move(stages)...} {}

    // Entry point: starts processing from stage 0
    template<typename Input>
    auto process(Input input) const {
        return process_impl<0>(std::move(input));
    }
};

// Deduction guide: allows Pipeline(stage1, stage2, ...) without explicit types
template<typename... Stages>
Pipeline(Stages...) -> Pipeline<Stages...>;

int main() {
    // Stage 1: string -> int (parse)
    auto parse = make_stage<std::string, int>([](const std::string& s) {
        return std::stoi(s);
    });

    // Stage 2: int -> double (scale)
    auto scale = make_stage<int, double>([](int n) {
        return n * 1.5;
    });

    // Stage 3: double -> string (format)
    auto format = make_stage<double, std::string>([](double d) {
        std::ostringstream oss;
        oss << d;
        return oss.str();
    });

    // Build pipeline: string -> int -> double -> string
    Pipeline pipe(parse, scale, format);

    // Scenario 1: process "42" through the pipeline
    auto result = pipe.process(std::string("42"));
    std::cout << "Result: " << result << '\n';  // "63"

    // Scenario 2: single-stage pipeline
    Pipeline single(parse);
    std::cout << "Single: " << single.process(std::string("99")) << '\n';  // 99

    // Scenario 3: type mismatch would be caught at compile time
    // If you tried Pipeline(parse, format) — parse outputs int,
    // format expects double — you'd get a compile error in process_impl
    // when trying to call format(int).

    // Another pipeline: int -> int -> int
    auto doubler = make_stage<int, int>([](int n) { return n * 2; });
    auto adder = make_stage<int, int>([](int n) { return n + 10; });
    Pipeline math(doubler, adder);
    std::cout << "Math: " << math.process(5) << '\n';  // (5*2)+10 = 20
}
```

Key decisions:
- **`Transformer` concept** constrains callables to those accepting the correct input type (§7.2).
- **`if constexpr`** in `process_impl` enables compile-time recursion through the tuple of stages (§7.2, using concepts chapter's if-constexpr pattern).
- **Type mismatch** between stages produces a compile error at the stage call site — e.g., passing an `int` to a stage expecting `double` fails clearly.
- **Variadic templates + `std::tuple`** store the heterogeneous stage types (§6.2.4 + §7.2).
- **CTAD deduction guide** lets you write `Pipeline(s1, s2, s3)` without specifying all template parameters.

</details>
