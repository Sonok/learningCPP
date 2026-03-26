# Chapter 11: Containers

---

## Quick Reference

- **Sequence containers**: `vector` (dynamic array), `list` (doubly-linked), `deque` (double-ended queue), `forward_list` (singly-linked) (§11.2, p.103)
- **Associative containers**: `map`, `set` (sorted, balanced tree); `unordered_map`, `unordered_set` (hash-based) (§11.4, p.107)
- `vector` is the default container — use it unless you have a specific reason not to (§11.2, p.103)
- `vector` provides `push_back()`, `pop_back()`, `size()`, `operator[]`, `at()`, iterators (§11.2, p.103)
- `map<K,V>` stores key-value pairs sorted by key; `operator[]` inserts if key is missing (§11.4, p.107)
- `unordered_map` has O(1) average lookup vs `map`'s O(log n) (§11.4, p.108)
- **Iterator invalidation**: inserting/erasing from a `vector` can invalidate iterators (§11.3, p.106)
- `emplace_back()` constructs elements in-place, avoiding copies (§11.2, p.104)
- Container adaptors: `stack`, `queue`, `priority_queue` wrap underlying containers (§11.5, p.109)
- All standard containers provide `begin()`, `end()`, `size()`, `empty()` (§11.2, p.103)
- `set` contains unique elements; `multiset` allows duplicates (§11.4, p.107)
- Prefer `reserve()` on `vector` if you know the size in advance — avoids reallocations (§11.2, p.104)

---

## 1. Multiple Choice (8 questions)

**Q1.** What is the time complexity of `vector::push_back` (amortized)? (§11.2, p.103)

a) O(n)
b) O(log n)
c) O(1) amortized
d) O(n log n)

<details><summary>Answer</summary>

**c)** `push_back` is amortized O(1). The vector doubles its capacity when full, so most pushes are constant time; the occasional reallocation is amortized across all pushes.

</details>

---

**Q2.** What happens when you use `map[key]` and `key` doesn't exist? (§11.4, p.107)

a) Returns a null reference
b) Throws `std::out_of_range`
c) Inserts a default-constructed value for that key and returns a reference to it
d) Returns an iterator to `end()`

<details><summary>Answer</summary>

**c)** `operator[]` on `map` performs an insert if the key is absent. This is a common gotcha — use `find()` or `count()` if you don't want insertion.

</details>

---

**Q3.** When is `unordered_map` preferred over `map`? (§11.4, p.108)

a) When you need sorted iteration
b) When you need O(1) average-case lookup and don't need ordering
c) When keys don't support hashing
d) When the map is very small

<details><summary>Answer</summary>

**b)** `unordered_map` uses hashing for O(1) average lookup. `map` uses a balanced tree for O(log n) but maintains sorted order.

</details>

---

**Q4.** What does `vector::reserve(n)` do? (§11.2, p.104)

a) Resizes the vector to `n` elements
b) Allocates capacity for at least `n` elements without changing size
c) Fills the vector with `n` default values
d) Limits the vector to `n` elements maximum

<details><summary>Answer</summary>

**b)** `reserve` allocates storage but doesn't create elements. `size()` stays the same; `capacity()` becomes at least `n`. This prevents reallocations when you know how many elements you'll add.

</details>

---

**Q5.** Which operation on a `vector` can invalidate ALL iterators? (§11.3, p.106)

a) `front()`
b) `push_back()` (when reallocation occurs)
c) `operator[]`
d) `size()`

<details><summary>Answer</summary>

**b)** If `push_back` triggers reallocation (size exceeds capacity), all iterators, pointers, and references to elements are invalidated because the data moves to a new memory block.

</details>

---

**Q6.** What is the difference between `set` and `multiset`? (§11.4, p.107)

a) `set` is sorted; `multiset` is not
b) `set` stores unique elements; `multiset` allows duplicates
c) `multiset` is hash-based
d) They are identical

<details><summary>Answer</summary>

**b)** Both are sorted. `set` enforces uniqueness (insert of a duplicate is a no-op); `multiset` allows multiple elements with the same value.

</details>

---

**Q7.** What does `emplace_back` do differently from `push_back`? (§11.2, p.104)

a) It's slower but safer
b) It constructs the element in-place, potentially avoiding a copy/move
c) It adds elements at the front
d) It reserves capacity first

<details><summary>Answer</summary>

**b)** `emplace_back` forwards arguments to the element's constructor, constructing it directly in the vector's memory. `push_back` requires an already-constructed object (though moves are cheap).

</details>

---

**Q8.** Which is NOT a container adaptor? (§11.5, p.109)

a) `stack`
b) `queue`
c) `deque`
d) `priority_queue`

<details><summary>Answer</summary>

**c)** `deque` is a sequence container, not an adaptor. `stack`, `queue`, and `priority_queue` are adaptors that wrap underlying containers.

</details>

---

## 2. Fill-in-the-blank Code (5 snippets)

**Snippet 1** — vector basics (§11.2, p.103)

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3};
    v.___(4);               // add to end
    v.___(v.begin(), 0);    // insert at beginning
    std::cout << v.___ << " " << v.___ << '\n';  // first and last
}
```

<details><summary>Answer</summary>

```cpp
v.push_back(4);
v.insert(v.begin(), 0);
std::cout << v.front() << " " << v.back() << '\n';  // 0 4
```

</details>

---

**Snippet 2** — map usage (§11.4, p.107)

```cpp
#include <map>
#include <string>
#include <iostream>

int main() {
    std::map<std::string, int> ages;
    ages[___] = 30;       // insert key-value
    ages[___] = 25;

    for (const auto& [___, ___] : ages)  // structured binding
        std::cout << ___ << ": " << ___ << '\n';
}
```

<details><summary>Answer</summary>

```cpp
ages["Alice"] = 30;
ages["Bob"] = 25;
for (const auto& [name, age] : ages)
    std::cout << name << ": " << age << '\n';
```

</details>

---

**Snippet 3** — unordered_map word counting (§11.4, p.108)

```cpp
#include <unordered_map>
#include <string>
#include <sstream>
#include <iostream>

int main() {
    std::string text = "the cat sat on the mat";
    std::___<std::string, int> freq;
    std::istringstream iss(text);
    std::string word;
    while (iss >> word)
        ___[word]++;
    for (const auto& [w, c] : freq)
        std::cout << w << ": " << c << '\n';
}
```

<details><summary>Answer</summary>

```cpp
std::unordered_map<std::string, int> freq;
freq[word]++;
```

</details>

---

**Snippet 4** — reserve to avoid reallocation (§11.2, p.104)

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v;
    v.___(1000);        // allocate space for 1000 elements
    for (int i = 0; i < 1000; ++i)
        v.push_back(i);
    std::cout << "size: " << v.___() << " capacity: " << v.___() << '\n';
}
```

<details><summary>Answer</summary>

```cpp
v.reserve(1000);
std::cout << "size: " << v.size() << " capacity: " << v.capacity() << '\n';
```

</details>

---

**Snippet 5** — set with custom ordering (§11.4, p.107)

```cpp
#include <set>
#include <iostream>

int main() {
    std::set<int, std::___<int>> s = {3, 1, 4, 1, 5};  // descending
    for (auto x : s)
        std::cout << x << " ";  // 5 4 3 1
}
```

<details><summary>Answer</summary>

```cpp
std::set<int, std::greater<int>> s = {3, 1, 4, 1, 5};
// Duplicates removed (1 appears once), sorted descending
```

</details>

---

## 3. Code from Scratch — LeetCode Style (3 problems)

### Problem 1: Word frequency — return top 3 (§11.4, p.108)

Given a vector of strings (words), use a `map` to count frequencies, then return the top 3 most frequent words.

```
Signature: std::vector<std::string> top3(const std::vector<std::string>& words);
```

**Examples:**

| Input | Output |
|---|---|
| `{"the","cat","the","sat","the","cat"}` | `{"the","cat","sat"}` |
| `{"a","b","c"}` | `{"a","b","c"}` |

**Constraints:** Use `map` or `unordered_map` for counting. Sort by frequency descending.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <unordered_map>
#include <algorithm>

std::vector<std::string> top3(const std::vector<std::string>& words) {
    // Step 1: count frequencies
    std::unordered_map<std::string, int> freq;
    for (const auto& w : words)
        ++freq[w];

    // Step 2: move to a vector for sorting
    std::vector<std::pair<std::string, int>> entries(freq.begin(), freq.end());

    // Step 3: sort by frequency descending
    std::sort(entries.begin(), entries.end(),
        [](const auto& a, const auto& b) { return a.second > b.second; });

    // Step 4: extract top 3 names
    std::vector<std::string> result;
    int limit = std::min(3, static_cast<int>(entries.size()));
    for (int i = 0; i < limit; ++i)
        result.push_back(entries[i].first);
    return result;
}

int main() {
    auto result = top3({"the","cat","the","sat","the","cat"});
    for (const auto& w : result)
        std::cout << w << " ";  // the cat sat
}
```

</details>

---

### Problem 2: LRU-style deque (§11.2, p.103)

Implement a bounded buffer using `deque`: when it's full and a new element is added, the oldest element is removed.

```
Signature:
  class BoundedBuffer {
  public:
      BoundedBuffer(int capacity);
      void push(int val);
      std::vector<int> contents() const;
  };
```

**Examples:**

| Operations (capacity=3) | contents() |
|---|---|
| push(1), push(2), push(3) | `{1,2,3}` |
| push(4) | `{2,3,4}` (1 dropped) |

**Constraints:** Use `std::deque`. No heap allocation beyond the deque itself.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <deque>
#include <vector>

class BoundedBuffer {
    std::deque<int> buf_;
    int cap_;
public:
    BoundedBuffer(int capacity) : cap_{capacity} {}

    void push(int val) {
        if (static_cast<int>(buf_.size()) >= cap_)
            buf_.pop_front();   // remove oldest
        buf_.push_back(val);
    }

    std::vector<int> contents() const {
        return {buf_.begin(), buf_.end()};
    }
};

int main() {
    BoundedBuffer b(3);
    b.push(1); b.push(2); b.push(3);
    for (auto x : b.contents()) std::cout << x << " ";  // 1 2 3
    std::cout << '\n';
    b.push(4);
    for (auto x : b.contents()) std::cout << x << " ";  // 2 3 4
}
```

</details>

---

### Problem 3: Set intersection (§11.4, p.107)

Given two `vector<int>`s, return their intersection (common elements, no duplicates) using `set`.

```
Signature: std::vector<int> intersect(const std::vector<int>& a, const std::vector<int>& b);
```

**Examples:**

| Input | Output |
|---|---|
| `{1,2,3,4}, {3,4,5,6}` | `{3,4}` |
| `{1,2}, {3,4}` | `{}` |

**Constraints:** Use `std::set` for deduplication and lookup. Return sorted.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <vector>
#include <set>

std::vector<int> intersect(const std::vector<int>& a, const std::vector<int>& b) {
    std::set<int> sa(a.begin(), a.end());   // deduplicate a
    std::set<int> sb(b.begin(), b.end());   // deduplicate b
    std::vector<int> result;
    // Check each element of the smaller set against the larger
    for (int x : sa) {
        if (sb.count(x))
            result.push_back(x);
    }
    return result;  // already sorted since we iterate a set
}

int main() {
    auto r = intersect({1,2,3,4}, {3,4,5,6});
    for (auto x : r) std::cout << x << " ";  // 3 4
}
```

</details>

---

## 4. Output Prediction (4 snippets)

**Snippet 1** — map insertion via operator[] (§11.4, p.107)

```cpp
#include <map>
#include <iostream>
int main() {
    std::map<std::string, int> m;
    m["x"] = 1;
    m["y"] = 2;
    std::cout << m["z"] << '\n';
    std::cout << m.size() << '\n';
}
```

<details><summary>Answer</summary>

Output:
```
0
3
```

`m["z"]` inserts a default-constructed `int` (0) for key `"z"`. The map now has 3 entries.

</details>

---

**Snippet 2** — vector iterator invalidation (§11.3, p.106)

```cpp
#include <vector>
#include <iostream>
int main() {
    std::vector<int> v = {1, 2, 3};
    auto it = v.begin();
    v.push_back(4);
    v.push_back(5);
    v.push_back(6);
    std::cout << *it << '\n';
}
```

<details><summary>Answer</summary>

**Undefined behavior.** The `push_back` calls may trigger reallocation, invalidating `it`. Dereferencing an invalidated iterator is UB.

</details>

---

**Snippet 3** — set uniqueness (§11.4, p.107)

```cpp
#include <set>
#include <iostream>
int main() {
    std::set<int> s = {3, 1, 4, 1, 5, 9, 2, 6, 5};
    std::cout << s.size() << '\n';
    for (auto x : s) std::cout << x << " ";
}
```

<details><summary>Answer</summary>

Output:
```
7
1 2 3 4 5 6 9
```

Duplicates (1, 5) are removed. Elements are sorted in ascending order. 7 unique values remain.

</details>

---

**Snippet 4** — deque operations (§11.2, p.103)

```cpp
#include <deque>
#include <iostream>
int main() {
    std::deque<int> d = {2, 3, 4};
    d.push_front(1);
    d.push_back(5);
    d.pop_front();
    std::cout << d.front() << " " << d.back() << " " << d.size() << '\n';
}
```

<details><summary>Answer</summary>

Output: `2 5 4`. After push_front(1): {1,2,3,4,5}. After push_back(5): already there. After pop_front: {2,3,4,5}. Front=2, back=5, size=4.

</details>

---

## 5. Debug This (3 snippets)

**Bug 1** — Modifying map while iterating (§11.3, p.106)

```cpp
#include <map>
#include <iostream>

int main() {
    std::map<std::string, int> m = {{"a",1},{"b",2},{"c",3}};
    for (auto it = m.begin(); it != m.end(); ++it) {
        if (it->second % 2 == 0)
            m.erase(it);
    }
}
```

<details><summary>Answer</summary>

**Bug:** `erase(it)` invalidates `it`, so `++it` in the loop is undefined behavior.

**Fix:** `it = m.erase(it);` (erase returns the next valid iterator), and don't increment in the loop header for that case:
```cpp
for (auto it = m.begin(); it != m.end(); ) {
    if (it->second % 2 == 0)
        it = m.erase(it);
    else
        ++it;
}
```

</details>

---

**Bug 2** — Using [] on a const map (§11.4, p.107)

```cpp
#include <map>
#include <iostream>

void print_value(const std::map<std::string, int>& m) {
    std::cout << m["key"] << '\n';
}
```

<details><summary>Answer</summary>

**Bug:** `operator[]` is non-const because it may insert. Cannot use `[]` on a `const` map.

**Fix:** Use `m.at("key")` (throws if not found) or `m.find("key")`.

</details>

---

**Bug 3** — Forgetting that set elements are const (§11.4, p.107)

```cpp
#include <set>

int main() {
    std::set<int> s = {1, 2, 3};
    for (auto& x : s)
        x *= 2;  // trying to double each element
}
```

<details><summary>Answer</summary>

**Bug:** Set elements are `const` — you cannot modify them in place because it would break the sorted invariant.

**Fix:** Erase and re-insert, or use a different container if you need to modify elements.

</details>

---

## 6. System Design / Open-ended (2 questions)

**Q1.** You need to implement an autocomplete system. As the user types characters, you suggest completions from a dictionary. Which container would you use and why? Compare `set` (sorted, with `lower_bound`), `unordered_map` (prefix hash), and `vector` (sorted + binary search). What are the tradeoffs? (§11.4, p.107)

---

**Q2.** A real-time event processor receives events with timestamps. You need: (a) O(1) lookup by event ID, (b) ability to iterate events in time order, (c) removal of events older than a threshold. Which combination of containers would you use? Can one container do it all? (§11.2, p.103; §11.4, p.107)

---

## 7. Rapid Fire (10 true/false)

1. `vector` is the recommended default container. (§11.2, p.103)

<details><summary>Answer</summary>True. Stroustrup recommends vector unless you have a specific reason for another container.</details>

2. `map::operator[]` never inserts elements. (§11.4, p.107)

<details><summary>Answer</summary>False. It inserts a default-constructed value if the key is missing.</details>

3. `unordered_map` iterates in sorted key order. (§11.4, p.108)

<details><summary>Answer</summary>False. Iteration order is unspecified in hash-based containers.</details>

4. `vector::reserve()` changes the vector's `size()`. (§11.2, p.104)

<details><summary>Answer</summary>False. `reserve` changes `capacity()`, not `size()`. Use `resize()` to change size.</details>

5. `emplace_back` can be more efficient than `push_back` for complex types. (§11.2, p.104)

<details><summary>Answer</summary>True. It constructs in-place, potentially avoiding a copy or move.</details>

6. `deque` supports O(1) insertion at both front and back. (§11.2, p.103)

<details><summary>Answer</summary>True. Unlike vector, deque can efficiently push/pop at both ends.</details>

7. Iterator invalidation never occurs with `std::list`. (§11.3, p.106)

<details><summary>Answer</summary>False. Erasing an element invalidates the iterator pointing to that element. But other iterators remain valid (unlike vector).</details>

8. `set` allows duplicate elements. (§11.4, p.107)

<details><summary>Answer</summary>False. `set` stores unique elements. Use `multiset` for duplicates.</details>

9. `priority_queue` is a container adaptor. (§11.5, p.109)

<details><summary>Answer</summary>True. It wraps a container (default: `vector`) with heap operations.</details>

10. `map::find()` returns `end()` if the key is not found. (§11.4, p.107)

<details><summary>Answer</summary>True. Compare the returned iterator against `map.end()` to check for absence.</details>
