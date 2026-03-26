# Chapter 12: Algorithms

---

## Quick Reference

- Standard algorithms live in `<algorithm>` and operate on iterator ranges `[begin, end)` (§12.1, p.111)
- **Non-modifying**: `find`, `count`, `for_each`, `all_of`, `any_of`, `none_of`, `equal` (§12.2, p.112)
- **Modifying**: `copy`, `move`, `transform`, `replace`, `fill`, `generate`, `remove` (§12.3, p.114)
- **Sorting & searching**: `sort`, `stable_sort`, `partial_sort`, `nth_element`, `lower_bound`, `upper_bound`, `binary_search` (§12.4, p.116)
- `sort` requires random-access iterators and averages O(n log n) (§12.4, p.116)
- `remove` doesn't actually erase — it moves unwanted elements to the end and returns a new "logical end" (§12.3, p.115)
- The **erase-remove idiom**: `v.erase(std::remove(v.begin(), v.end(), val), v.end())` (§12.3, p.115)
- **Lambdas** are the preferred way to pass custom predicates: `[](int x){ return x > 0; }` (§12.5, p.118)
- Lambda captures: `[=]` by value, `[&]` by reference, `[x, &y]` mixed (§12.5, p.118)
- `std::transform` applies a function to each element and writes results to an output range (§12.3, p.114)
- **Parallel algorithms** (C++17): pass `std::execution::par` as the first argument (§12.6, p.119)
- Always pass the correct iterator category — passing a `list::iterator` to `sort` won't compile (§12.1, p.111)

---

## 1. Multiple Choice (8 questions)

**Q1.** What does `std::remove` actually do? (§12.3, p.115)

a) Erases elements from the container
b) Moves unwanted elements to the end and returns the new logical end iterator
c) Sets unwanted elements to zero
d) Removes elements and shrinks the container

<details><summary>Answer</summary>

**b)** `remove` is an algorithm, not a container operation. It shifts wanted elements forward and returns an iterator past the last "good" element. The container size is unchanged — you need `erase` to actually remove.

</details>

---

**Q2.** What is the erase-remove idiom? (§12.3, p.115)

a) `v.remove(val);`
b) `v.erase(std::remove(v.begin(), v.end(), val), v.end());`
c) `std::erase(v, val);`
d) `v.clear();`

<details><summary>Answer</summary>

**b)** The idiom combines `remove` (which partitions elements) with `erase` (which actually removes from the container). C++20 added `std::erase(v, val)` as a shorthand.

</details>

---

**Q3.** What does `std::transform` do? (§12.3, p.114)

a) Sorts elements
b) Applies a function to each element and writes results to an output range
c) Filters elements matching a predicate
d) Reverses a container

<details><summary>Answer</summary>

**b)** `transform(begin, end, out, f)` applies `f` to each element in `[begin, end)` and writes results starting at `out`. The output range can be the same as the input.

</details>

---

**Q4.** Which algorithm would you use to check if ALL elements satisfy a condition? (§12.2, p.112)

a) `std::find_if`
b) `std::for_each`
c) `std::all_of`
d) `std::count_if`

<details><summary>Answer</summary>

**c)** `std::all_of(begin, end, pred)` returns `true` if the predicate is true for every element.

</details>

---

**Q5.** What does `[&]` mean in a lambda capture? (§12.5, p.118)

a) Capture nothing
b) Capture all local variables by value
c) Capture all local variables by reference
d) Capture only the first variable

<details><summary>Answer</summary>

**c)** `[&]` captures all referenced local variables by reference. Changes inside the lambda affect the original variables.

</details>

---

**Q6.** What is the time complexity of `std::sort`? (§12.4, p.116)

a) O(n)
b) O(n log n) average and worst case
c) O(n^2)
d) O(log n)

<details><summary>Answer</summary>

**b)** `std::sort` uses introsort (hybrid of quicksort, heapsort, and insertion sort) to guarantee O(n log n).

</details>

---

**Q7.** What does `std::lower_bound` return? (§12.4, p.116)

a) The smallest element
b) An iterator to the first element not less than the given value
c) The lower bound of the container's memory
d) An iterator to the largest element less than the given value

<details><summary>Answer</summary>

**b)** `lower_bound` returns an iterator to the first element >= the target in a sorted range. Combined with `upper_bound`, it defines the equal range.

</details>

---

**Q8.** How do you enable parallel execution of `std::sort` in C++17? (§12.6, p.119)

a) `std::sort(std::execution::par, v.begin(), v.end());`
b) `std::parallel_sort(v.begin(), v.end());`
c) `#pragma omp parallel sort`
d) Parallel sort is not supported

<details><summary>Answer</summary>

**a)** Pass an execution policy as the first argument. `std::execution::par` enables parallel execution. Also available: `seq` (sequential) and `par_unseq` (parallel + vectorized).

</details>

---

## 2. Fill-in-the-blank Code (5 snippets)

**Snippet 1** — find and count (§12.2, p.112)

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 2, 4, 2, 5};
    auto it = std::___(v.begin(), v.end(), 3);  // find value 3
    int n = std::___(v.begin(), v.end(), 2);    // count value 2
    std::cout << "Found 3 at index " << (it - v.begin()) << '\n';
    std::cout << "Count of 2: " << n << '\n';
}
```

<details><summary>Answer</summary>

```cpp
auto it = std::find(v.begin(), v.end(), 3);
int n = std::count(v.begin(), v.end(), 2);
```

</details>

---

**Snippet 2** — sort with custom comparator (§12.4, p.116)

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {5, 3, 8, 1, 9};
    std::sort(v.begin(), v.end(), [](int a, int b) {
        return a ___ b;  // descending order
    });
    for (auto x : v) std::cout << x << " ";  // 9 8 5 3 1
}
```

<details><summary>Answer</summary>

```cpp
return a > b;
```

</details>

---

**Snippet 3** — transform (§12.3, p.114)

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    std::vector<int> result(v.size());
    std::___(v.begin(), v.end(), result.begin(),
             [](int x) { return x * ___; });  // square each
    for (auto x : result) std::cout << x << " ";  // 1 4 9 16 25
}
```

<details><summary>Answer</summary>

```cpp
std::transform(v.begin(), v.end(), result.begin(),
               [](int x) { return x * x; });
```

</details>

---

**Snippet 4** — erase-remove idiom (§12.3, p.115)

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5, 6};
    // Remove all even numbers
    v.___(std::___(v.begin(), v.end(),
                     [](int x) { return x % 2 == 0; }),
           v.end());
    for (auto x : v) std::cout << x << " ";  // 1 3 5
}
```

<details><summary>Answer</summary>

```cpp
v.erase(std::remove_if(v.begin(), v.end(),
                        [](int x) { return x % 2 == 0; }),
         v.end());
```

</details>

---

**Snippet 5** — any_of / all_of / none_of (§12.2, p.112)

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {2, 4, 6, 8};
    bool allEven  = std::___(v.begin(), v.end(), [](int x) { return x%2==0; });
    bool anyNeg   = std::___(v.begin(), v.end(), [](int x) { return x<0; });
    bool noneZero = std::___(v.begin(), v.end(), [](int x) { return x==0; });
    std::cout << std::boolalpha << allEven << " " << anyNeg << " " << noneZero;
}
```

<details><summary>Answer</summary>

```cpp
bool allEven  = std::all_of(...);   // true
bool anyNeg   = std::any_of(...);   // false
bool noneZero = std::none_of(...);  // true
```

</details>

---

## 3. Code from Scratch — LeetCode Style (3 problems)

### Problem 1: Partition and transform (§12.3, p.114)

Given a vector of integers, separate positives and negatives. Return two vectors: one with the absolute values of negatives, one with the doubled positives. Use `std::partition` or `std::partition_copy` and `std::transform`.

```
Signature:
  std::pair<std::vector<int>, std::vector<int>>
  split_transform(const std::vector<int>& nums);
  // returns {abs_negatives, doubled_positives}
```

**Examples:**

| Input | Output |
|---|---|
| `{-3, 5, -1, 2, -4}` | `{3,1,4}, {10,4}` |

**Constraints:** Use standard algorithms. No manual loops.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <cmath>

std::pair<std::vector<int>, std::vector<int>>
split_transform(const std::vector<int>& nums) {
    // Separate into negatives and positives
    std::vector<int> negs, poss;
    std::partition_copy(nums.begin(), nums.end(),
                        std::back_inserter(negs),
                        std::back_inserter(poss),
                        [](int x) { return x < 0; });

    // Transform negatives to absolute values
    std::transform(negs.begin(), negs.end(), negs.begin(),
                   [](int x) { return std::abs(x); });

    // Transform positives to doubled values
    std::transform(poss.begin(), poss.end(), poss.begin(),
                   [](int x) { return x * 2; });

    return {negs, poss};
}

int main() {
    auto [neg, pos] = split_transform({-3, 5, -1, 2, -4});
    for (auto x : neg) std::cout << x << " ";  // 3 1 4
    std::cout << "| ";
    for (auto x : pos) std::cout << x << " ";  // 10 4
}
```

</details>

---

### Problem 2: Custom sort by multiple criteria (§12.4, p.116)

Sort a vector of `{name, age}` pairs first by age ascending, then by name alphabetically for equal ages.

```
Signature: void sort_people(std::vector<std::pair<std::string, int>>& people);
```

**Examples:**

| Input | Output |
|---|---|
| `{{"Charlie",25},{"Alice",30},{"Bob",25}}` | `{{"Bob",25},{"Charlie",25},{"Alice",30}}` |

**Constraints:** Use `std::sort` with a lambda comparator.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <algorithm>

void sort_people(std::vector<std::pair<std::string, int>>& people) {
    std::sort(people.begin(), people.end(),
        [](const auto& a, const auto& b) {
            if (a.second != b.second)
                return a.second < b.second;   // age ascending
            return a.first < b.first;          // name alphabetically for ties
        });
}

int main() {
    std::vector<std::pair<std::string, int>> people = {
        {"Charlie", 25}, {"Alice", 30}, {"Bob", 25}
    };
    sort_people(people);
    for (const auto& [name, age] : people)
        std::cout << name << "(" << age << ") ";
    // Bob(25) Charlie(25) Alice(30)
}
```

</details>

---

### Problem 3: Binary search on sorted data (§12.4, p.116)

Given a sorted vector, implement a function that finds the range of indices where a target appears, using `lower_bound` and `upper_bound`.

```
Signature: std::pair<int,int> find_range(const std::vector<int>& sorted, int target);
// returns {first_index, count}, or {-1, 0} if not found
```

**Examples:**

| Input | Output |
|---|---|
| `{1,2,2,2,3,4}, target=2` | `{1, 3}` |
| `{1,3,5,7}, target=4` | `{-1, 0}` |

**Constraints:** Use `std::lower_bound` and `std::upper_bound`. O(log n).

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

std::pair<int,int> find_range(const std::vector<int>& sorted, int target) {
    auto lo = std::lower_bound(sorted.begin(), sorted.end(), target);
    auto hi = std::upper_bound(sorted.begin(), sorted.end(), target);
    int count = static_cast<int>(hi - lo);
    if (count == 0)
        return {-1, 0};
    int first = static_cast<int>(lo - sorted.begin());
    return {first, count};
}

int main() {
    auto [idx, cnt] = find_range({1,2,2,2,3,4}, 2);
    std::cout << "Index: " << idx << " Count: " << cnt << '\n';  // 1, 3
    auto [idx2, cnt2] = find_range({1,3,5,7}, 4);
    std::cout << "Index: " << idx2 << " Count: " << cnt2 << '\n';  // -1, 0
}
```

</details>

---

## 4. Output Prediction (4 snippets)

**Snippet 1** — remove doesn't erase (§12.3, p.115)

```cpp
#include <algorithm>
#include <vector>
#include <iostream>
int main() {
    std::vector<int> v = {1, 2, 3, 2, 4};
    std::remove(v.begin(), v.end(), 2);
    std::cout << v.size() << '\n';
    for (auto x : v) std::cout << x << " ";
}
```

<details><summary>Answer</summary>

Output:
```
5
1 3 4 2 4
```

`remove` shifts elements but doesn't change the vector's size. The size remains 5. The elements after the "logical end" are unspecified (often leftover values).

</details>

---

**Snippet 2** — Lambda capture by reference (§12.5, p.118)

```cpp
#include <algorithm>
#include <vector>
#include <iostream>
int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    int total = 0;
    std::for_each(v.begin(), v.end(), [&total](int x) { total += x; });
    std::cout << total << '\n';
}
```

<details><summary>Answer</summary>

Output: `15`. The lambda captures `total` by reference and accumulates the sum: 1+2+3+4+5 = 15.

</details>

---

**Snippet 3** — sort stability (§12.4, p.116)

```cpp
#include <algorithm>
#include <vector>
#include <iostream>
int main() {
    std::vector<std::pair<int,char>> v = {{1,'b'},{2,'a'},{1,'a'},{2,'b'}};
    std::stable_sort(v.begin(), v.end(),
        [](const auto& a, const auto& b) { return a.first < b.first; });
    for (auto [n,c] : v) std::cout << n << c << " ";
}
```

<details><summary>Answer</summary>

Output: `1b 1a 2a 2b `. `stable_sort` preserves the original order of equal elements. The two 1s keep their original order (b before a), and the two 2s keep theirs (a before b).

</details>

---

**Snippet 4** — find_if (§12.2, p.112)

```cpp
#include <algorithm>
#include <vector>
#include <iostream>
int main() {
    std::vector<int> v = {2, 4, 6, 7, 8, 10};
    auto it = std::find_if(v.begin(), v.end(), [](int x) { return x % 2 != 0; });
    if (it != v.end())
        std::cout << *it << " at index " << (it - v.begin()) << '\n';
}
```

<details><summary>Answer</summary>

Output: `7 at index 3`. `find_if` returns an iterator to the first element satisfying the predicate. 7 is the first odd number.

</details>

---

## 5. Debug This (3 snippets)

**Bug 1** — remove without erase (§12.3, p.115)

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 2, 4};
    std::remove(v.begin(), v.end(), 2);
    // expects v to have 3 elements
    std::cout << "size: " << v.size() << '\n';  // prints 5, not 3!
}
```

<details><summary>Answer</summary>

**Bug:** `std::remove` doesn't change the container size. It returns the new logical end.

**Fix:** `v.erase(std::remove(v.begin(), v.end(), 2), v.end());`

</details>

---

**Bug 2** — binary_search on unsorted data (§12.4, p.116)

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {5, 3, 1, 4, 2};
    bool found = std::binary_search(v.begin(), v.end(), 3);
    std::cout << std::boolalpha << found << '\n';  // might print false!
}
```

<details><summary>Answer</summary>

**Bug:** `binary_search` requires a **sorted** range. Using it on unsorted data is undefined behavior — it may return incorrect results.

**Fix:** Sort first: `std::sort(v.begin(), v.end());` then call `binary_search`.

</details>

---

**Bug 3** — Lambda capturing loop variable by reference (§12.5, p.118)

```cpp
#include <vector>
#include <functional>
#include <iostream>

int main() {
    std::vector<std::function<int()>> funcs;
    for (int i = 0; i < 5; ++i)
        funcs.push_back([&i]() { return i; });
    for (auto& f : funcs)
        std::cout << f() << " ";  // expects 0 1 2 3 4
}
```

<details><summary>Answer</summary>

**Bug:** `i` is captured by reference. By the time the lambdas are called, the loop has ended and `i` is 5 (or its lifetime has ended — UB in this case since `i` is a loop-local variable).

**Fix:** Capture by value: `[i]() { return i; }` or `[=]() { return i; }`.

</details>

---

## 6. System Design / Open-ended (2 questions)

**Q1.** You need to process a large dataset (millions of records) where you must: filter out invalid entries, transform valid entries, sort by a key, and write the top 1000 to a file. Chain the appropriate algorithms. Would you benefit from parallel execution policies? What are the requirements for using `std::execution::par`? (§12.3, p.114; §12.6, p.119)

---

**Q2.** The STL separates algorithms from containers via iterators. This means `std::sort` works on any random-access range. Discuss: what would change if algorithms were member functions of containers instead? Why did the STL designers choose the current approach? (§12.1, p.111)

---

## 7. Rapid Fire (10 true/false)

1. `std::sort` works on `std::list`. (§12.4, p.116)

<details><summary>Answer</summary>False. `sort` requires random-access iterators; `list` only provides bidirectional. Use `list::sort()` instead.</details>

2. `std::remove` changes the container's size. (§12.3, p.115)

<details><summary>Answer</summary>False. It returns a new "logical end" but doesn't erase. Use the erase-remove idiom.</details>

3. `std::transform` can write to the same range it reads from. (§12.3, p.114)

<details><summary>Answer</summary>True. You can use the input range as the output: `transform(v.begin(), v.end(), v.begin(), f)`.</details>

4. `[=]` captures all variables by reference. (§12.5, p.118)

<details><summary>Answer</summary>False. `[=]` captures by value. `[&]` captures by reference.</details>

5. `std::binary_search` returns an iterator to the found element. (§12.4, p.116)

<details><summary>Answer</summary>False. It returns `bool`. Use `lower_bound` to get an iterator.</details>

6. `std::for_each` returns the function object passed to it. (§12.2, p.112)

<details><summary>Answer</summary>True. This allows you to extract state from a stateful function object.</details>

7. `std::stable_sort` preserves the relative order of equal elements. (§12.4, p.116)

<details><summary>Answer</summary>True. That's the key difference from `sort`.</details>

8. Algorithms operate on half-open ranges `[begin, end)`. (§12.1, p.111)

<details><summary>Answer</summary>True. `end` points one past the last element.</details>

9. `std::execution::par` guarantees deterministic ordering. (§12.6, p.119)

<details><summary>Answer</summary>False. Parallel execution may process elements in any order. Use `par_unseq` for even more relaxed guarantees.</details>

10. A lambda with no capture clause `[]` can be converted to a function pointer. (§12.5, p.118)

<details><summary>Answer</summary>True. Captureless lambdas are convertible to function pointers of the matching signature.</details>
