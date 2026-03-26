# Chapter 10: Input and Output

---

## Quick Reference

- The I/O library is based on `ostream` (output) and `istream` (input) types (§10.1, p.95)
- `cout`, `cerr`, `cin` are the standard stream objects for output, errors, and input (§10.2, p.96)
- `<<` is the output operator; `>>` is the input operator — both return the stream for chaining (§10.2, p.96)
- I/O state flags: `good()`, `eof()`, `fail()`, `bad()` — check after operations (§10.3, p.97)
- `getline(cin, str)` reads an entire line including spaces (§10.2, p.96)
- File I/O: `std::ifstream` for reading, `std::ofstream` for writing, `std::fstream` for both (§10.5, p.99)
- `std::istringstream` and `std::ostringstream` for in-memory string I/O (§10.6, p.100)
- Formatting: `setw()`, `setprecision()`, `fixed`, `hex`, `oct`, `left`, `right` from `<iomanip>` (§10.4, p.98)
- Streams use RAII — files are closed automatically when the stream object is destroyed (§10.5, p.99)
- User-defined types support I/O by overloading `operator<<` and `operator>>` (§10.2, p.96)
- `cin >> x` skips whitespace by default; to read whitespace use `get()` or `getline()` (§10.2, p.96)

---

## 1. Multiple Choice (8 questions)

**Q1.** What does `cin >> x` do when it encounters non-matching input? (§10.3, p.97)

a) Throws an exception
b) Sets the fail bit and leaves the bad input in the buffer
c) Skips the bad input and continues
d) Terminates the program

<details><summary>Answer</summary>

**b)** The stream enters a fail state (`fail()` returns true), and the bad input remains in the buffer. You must call `cin.clear()` and `cin.ignore()` to recover.

</details>

---

**Q2.** Which header provides `setw` and `setprecision`? (§10.4, p.98)

a) `<iostream>`
b) `<fstream>`
c) `<iomanip>`
d) `<format>`

<details><summary>Answer</summary>

**c)** `<iomanip>` provides I/O manipulators like `setw`, `setprecision`, `setfill`, `left`, `right`, etc.

</details>

---

**Q3.** How does `std::ofstream` handle file cleanup? (§10.5, p.99)

a) You must call `close()` explicitly
b) The destructor closes the file (RAII)
c) The OS handles it when the program exits
d) Files are never closed automatically

<details><summary>Answer</summary>

**b)** The `ofstream` destructor calls `close()` automatically. This is RAII applied to file handles. You can call `close()` explicitly if you need earlier cleanup.

</details>

---

**Q4.** What's the difference between `cerr` and `clog`? (§10.2, p.96)

a) `cerr` is unbuffered; `clog` is buffered
b) They are identical
c) `cerr` goes to a file; `clog` goes to the screen
d) `clog` is for warnings; `cerr` is for fatal errors

<details><summary>Answer</summary>

**a)** `cerr` is unbuffered — output appears immediately. `clog` is buffered — output may be delayed. Both write to standard error.

</details>

---

**Q5.** What does `std::istringstream` allow you to do? (§10.6, p.100)

a) Write to a string
b) Read/parse from a string as if it were an input stream
c) Convert a stream to a string
d) Compress a string

<details><summary>Answer</summary>

**b)** `istringstream` wraps a `string` in an `istream` interface, allowing you to use `>>` to parse values from it.

</details>

---

**Q6.** What happens if you try to open a non-existent file with `ifstream`? (§10.5, p.99)

a) The file is created
b) An exception is thrown
c) The stream enters a fail state
d) The program crashes

<details><summary>Answer</summary>

**c)** The stream's fail bit is set. You should check `if (file)` or `if (file.is_open())` after construction.

</details>

---

**Q7.** How do you read an entire line including spaces from `cin`? (§10.2, p.96)

a) `cin >> line;`
b) `cin.read(line);`
c) `std::getline(cin, line);`
d) `cin.getline(line);`

<details><summary>Answer</summary>

**c)** `std::getline(cin, line)` reads until newline. `cin >> line` stops at whitespace.

</details>

---

**Q8.** What is the effect of `cout << hex << 255`? (§10.4, p.98)

a) Prints `hex255`
b) Prints `ff`
c) Prints `0xff`
d) Prints `255` (no effect)

<details><summary>Answer</summary>

**b)** `hex` changes the output base to hexadecimal. 255 in hex is `ff`. Note: this is a sticky manipulator — subsequent integers also print in hex.

</details>

---

## 2. Fill-in-the-blank Code (5 snippets)

**Snippet 1** — File output (§10.5, p.99)

```cpp
#include <fstream>
#include <iostream>

int main() {
    std::___ out("data.txt");  // open for writing
    if (!out) {
        std::cerr << "Cannot open file\n";
        return 1;
    }
    out << "Line 1\n" << "Line 2\n";
    // file closed automatically by ___
}
```

<details><summary>Answer</summary>

```cpp
std::ofstream out("data.txt");
// file closed automatically by destructor (RAII)
```

</details>

---

**Snippet 2** — Reading with getline (§10.2, p.96)

```cpp
#include <iostream>
#include <string>

int main() {
    std::string line;
    std::cout << "Enter a sentence: ";
    std::___(std::cin, line);  // read entire line
    std::cout << "You said: " << line << '\n';
}
```

<details><summary>Answer</summary>

```cpp
std::getline(std::cin, line);
```

</details>

---

**Snippet 3** — Formatting with iomanip (§10.4, p.98)

```cpp
#include <iostream>
#include <___>

int main() {
    double pi = 3.14159265;
    std::cout << std::___ << std::___(4) << pi << '\n';  // "3.1416"
}
```

<details><summary>Answer</summary>

```cpp
#include <iomanip>
std::cout << std::fixed << std::setprecision(4) << pi << '\n';
```

</details>

---

**Snippet 4** — String stream parsing (§10.6, p.100)

```cpp
#include <sstream>
#include <string>
#include <iostream>

int main() {
    std::string data = "42 3.14 hello";
    std::___ iss(data);
    int i; double d; std::string s;
    iss >> i >> d >> s;
    std::cout << i << " " << d << " " << s << '\n';
}
```

<details><summary>Answer</summary>

```cpp
std::istringstream iss(data);
```

</details>

---

**Snippet 5** — Custom type I/O (§10.2, p.96)

```cpp
#include <iostream>

struct Point { double x, y; };

std::ostream& operator___(std::ostream& os, const Point& p) {
    return os << "(" << p.x << ", " << p.y << ")";
}

std::istream& operator___(std::istream& is, Point& p) {
    return is >> p.x >> p.y;
}

int main() {
    Point p{3, 4};
    std::cout << p << '\n';
}
```

<details><summary>Answer</summary>

```cpp
std::ostream& operator<<(std::ostream& os, const Point& p) { ... }
std::istream& operator>>(std::istream& is, Point& p) { ... }
```

</details>

---

## 3. Code from Scratch — LeetCode Style (3 problems)

### Problem 1: CSV line parser using stringstream (§10.6, p.100)

Write a function that takes a comma-separated string and returns a vector of trimmed fields.

```
Signature: std::vector<std::string> parse_csv(const std::string& line);
```

**Examples:**

| Input | Output |
|---|---|
| `"Alice,30,New York"` | `{"Alice","30","New York"}` |
| `"a,,b"` | `{"a","","b"}` |

**Constraints:** Use `getline` with `','` as delimiter. No regex.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <sstream>
#include <string>
#include <vector>

std::vector<std::string> parse_csv(const std::string& line) {
    std::vector<std::string> fields;
    std::istringstream iss(line);
    std::string field;
    // getline with ',' as delimiter splits on commas
    while (std::getline(iss, field, ',')) {
        fields.push_back(field);
    }
    return fields;
}

int main() {
    auto fields = parse_csv("Alice,30,New York");
    for (const auto& f : fields)
        std::cout << "[" << f << "] ";
    // [Alice] [30] [New York]
}
```

</details>

---

### Problem 2: File word counter (§10.5, p.99)

Write a function that opens a file, counts the number of words, lines, and characters, and returns them as a tuple.

```
Signature: std::tuple<int,int,int> count_file(const std::string& filename);
// returns {words, lines, chars}
```

**Examples:**

| File content | Output |
|---|---|
| `"hello world\nfoo bar\n"` | `{4, 2, 20}` |

**Constraints:** Use `ifstream`. Must check if file opens successfully.

<details><summary>Solution</summary>

```cpp
#include <fstream>
#include <tuple>
#include <string>
#include <stdexcept>
#include <sstream>
#include <iostream>

std::tuple<int,int,int> count_file(const std::string& filename) {
    std::ifstream file(filename);
    if (!file)
        throw std::runtime_error("Cannot open: " + filename);

    int words = 0, lines = 0, chars = 0;
    std::string line;
    while (std::getline(file, line)) {
        ++lines;
        chars += line.size() + 1;  // +1 for the newline getline consumed
        std::istringstream iss(line);
        std::string word;
        while (iss >> word)
            ++words;
    }
    return {words, lines, chars};
}

int main() {
    auto [w, l, c] = count_file("test.txt");
    std::cout << "Words: " << w << " Lines: " << l << " Chars: " << c << '\n';
}
```

</details>

---

### Problem 3: Formatted table printer (§10.4, p.98)

Write a function that prints a table of name-score pairs with aligned columns using `setw` and `left`/`right`.

```
Signature: void print_table(const std::vector<std::pair<std::string,int>>& data);
```

**Expected output:**
```
Name            Score
-----------     -----
Alice              95
Bob                87
Carol              92
```

**Constraints:** Use `<iomanip>`. Names left-aligned in 15 chars, scores right-aligned in 5 chars.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <iomanip>
#include <vector>
#include <string>

void print_table(const std::vector<std::pair<std::string,int>>& data) {
    // Header
    std::cout << std::left << std::setw(15) << "Name"
              << std::right << std::setw(5) << "Score" << '\n';
    std::cout << std::string(15, '-') << " " << std::string(5, '-') << '\n';

    // Data rows
    for (const auto& [name, score] : data) {
        std::cout << std::left  << std::setw(15) << name
                  << std::right << std::setw(5) << score << '\n';
    }
}

int main() {
    std::vector<std::pair<std::string,int>> data = {
        {"Alice", 95}, {"Bob", 87}, {"Carol", 92}
    };
    print_table(data);
}
```

</details>

---

## 4. Output Prediction (4 snippets)

**Snippet 1** (§10.4, p.98)

```cpp
#include <iostream>
#include <iomanip>
int main() {
    std::cout << std::setw(10) << std::right << 42 << '\n';
    std::cout << 42 << '\n';
}
```

<details><summary>Answer</summary>

Output:
```
        42
42
```

`setw(10)` pads to width 10 with right alignment. `setw` is NOT sticky — the second `42` has no padding.

</details>

---

**Snippet 2** (§10.4, p.98)

```cpp
#include <iostream>
int main() {
    std::cout << std::hex << 255 << '\n';
    std::cout << 16 << '\n';
    std::cout << std::dec << 16 << '\n';
}
```

<details><summary>Answer</summary>

Output:
```
ff
10
16
```

`hex` is sticky — 255 prints as `ff`, and 16 prints as `10` (hex). After `std::dec`, 16 prints as `16` (decimal).

</details>

---

**Snippet 3** (§10.3, p.97)

```cpp
#include <iostream>
int main() {
    int x;
    std::cout << "Enter a number: ";
    if (std::cin >> x)
        std::cout << "Got " << x << '\n';
    else
        std::cout << "Failed\n";
}
```

If the user types `abc`:

<details><summary>Answer</summary>

Output: `Failed`. `cin >> x` fails because `abc` can't be parsed as `int`. The stream enters a fail state, and the `if` condition is false.

</details>

---

**Snippet 4** (§10.6, p.100)

```cpp
#include <sstream>
#include <iostream>
int main() {
    std::ostringstream oss;
    oss << "Value: " << 42 << " and " << 3.14;
    std::cout << oss.str() << '\n';
}
```

<details><summary>Answer</summary>

Output: `Value: 42 and 3.14`. `ostringstream` builds a string in memory using stream syntax. `.str()` extracts it.

</details>

---

## 5. Debug This (3 snippets)

**Bug 1** — Not checking file open (§10.5, p.99)

```cpp
#include <fstream>
#include <iostream>

int main() {
    std::ifstream file("nonexistent.txt");
    int x;
    file >> x;
    std::cout << x << '\n';
}
```

<details><summary>Answer</summary>

**Bug:** The file doesn't exist, so `file` is in a fail state. `file >> x` does nothing; `x` is uninitialized — reading it is undefined behavior.

**Fix:** Check `if (!file)` before reading, and initialize `x`.

</details>

---

**Bug 2** — getline after >> leaves newline in buffer (§10.2, p.96)

```cpp
#include <iostream>
#include <string>

int main() {
    int age;
    std::string name;
    std::cout << "Age: ";
    std::cin >> age;
    std::cout << "Name: ";
    std::getline(std::cin, name);
    std::cout << "Hello " << name << ", age " << age << '\n';
}
```

<details><summary>Answer</summary>

**Bug:** After `cin >> age`, the newline character remains in the buffer. `getline` immediately reads that newline and returns an empty string.

**Fix:** Add `std::cin.ignore()` (or `std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n')`) between `>>` and `getline`.

</details>

---

**Bug 3** — Sticky formatting (§10.4, p.98)

```cpp
#include <iostream>
int main() {
    std::cout << std::hex << 255 << '\n';
    std::cout << 100 << '\n';  // expects decimal 100
}
```

<details><summary>Answer</summary>

**Bug:** `hex` is a sticky manipulator — it stays in effect. `100` prints as `64` (hex), not `100` (decimal).

**Fix:** Reset with `std::cout << std::dec;` when you want decimal output again.

</details>

---

## 6. System Design / Open-ended (2 questions)

**Q1.** You're writing a configuration file parser. The file has lines like `key = value`. Design the parsing using `ifstream` and `istringstream`. How would you handle comments (lines starting with `#`), blank lines, and values with spaces? (§10.5, p.99; §10.6, p.100)

---

**Q2.** Compare C++ stream I/O (`<<`/`>>`) with C's `printf`/`scanf`. What are the type-safety advantages of streams? What about performance? When might you still prefer `printf`-style formatting? (§10.1, p.95; §10.2, p.96)

---

## 7. Rapid Fire (10 true/false)

1. `cerr` output is buffered. (§10.2, p.96)

<details><summary>Answer</summary>False. `cerr` is unbuffered — output appears immediately.</details>

2. `setw()` is a sticky manipulator. (§10.4, p.98)

<details><summary>Answer</summary>False. `setw` only affects the next output operation. It resets after each use.</details>

3. `hex`, `oct`, and `dec` are sticky manipulators. (§10.4, p.98)

<details><summary>Answer</summary>True. They remain in effect until changed.</details>

4. `ifstream` automatically closes the file in its destructor. (§10.5, p.99)

<details><summary>Answer</summary>True. RAII ensures cleanup.</details>

5. `cin >> s` (where `s` is a string) reads an entire line. (§10.2, p.96)

<details><summary>Answer</summary>False. `>>` reads until whitespace. Use `getline` for full lines.</details>

6. `ostringstream` writes to a file. (§10.6, p.100)

<details><summary>Answer</summary>False. It writes to an in-memory string buffer.</details>

7. A failed stream can be reused after calling `clear()`. (§10.3, p.97)

<details><summary>Answer</summary>True. `clear()` resets the error flags, allowing further operations (after dealing with any bad input in the buffer).</details>

8. `operator<<` for user-defined types must be a member function. (§10.2, p.96)

<details><summary>Answer</summary>False. It must be a non-member (or friend) function because the left operand is `ostream`, not the user type.</details>

9. `std::endl` flushes the stream; `'\n'` does not. (§10.2, p.96)

<details><summary>Answer</summary>True. `endl` writes a newline AND flushes. `'\n'` only writes a newline.</details>

10. `getline` includes the delimiter in the returned string. (§10.2, p.96)

<details><summary>Answer</summary>False. `getline` extracts and discards the delimiter.</details>

---

## 8. Capstone Implementation Challenge

### CSV Report Generator (§10.2–10.6, p.96–100)

**Motivation:** You're building a reporting tool that reads student records from a CSV file, computes statistics, and generates a formatted report to both the console and a file. This exercises file I/O, string streams, formatting, and custom type I/O.

**Signatures:**

```cpp
struct Student {
    std::string name;
    int age;
    double gpa;

    friend std::ostream& operator<<(std::ostream& os, const Student& s);
    friend std::istream& operator>>(std::istream& is, Student& s);
};

class ReportGenerator {
public:
    void load_csv(const std::string& filename);
    void load_from_string(const std::string& csv_data);  // for testing

    struct Summary {
        int count;
        double avg_gpa;
        double max_gpa;
        std::string top_student;
    };

    Summary compute_summary() const;
    void print_table(std::ostream& os) const;  // formatted table
    void save_report(const std::string& filename) const;

    int count() const;
};
```

**Test Scenarios:**

1. Load CSV data with 5 students. `count()` returns 5. `compute_summary()` has correct avg/max GPA.
2. `print_table(cout)` outputs a nicely aligned table with headers, separator, and rows.
3. `save_report("report.txt")` writes the table + summary to a file. Verify the file exists and is readable.
4. Malformed CSV lines (missing fields, non-numeric GPA) are skipped with a warning to `cerr`.

**Constraints:**
- Use `ifstream` and `ofstream` for file I/O with RAII
- Use `istringstream` for parsing CSV lines
- Use `<iomanip>` for formatted table output (`setw`, `left`, `right`, `fixed`, `setprecision`)
- Overload `operator<<` and `operator>>` for `Student`
- Check stream state after I/O operations
- Use `cerr` for error reporting

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <fstream>
#include <sstream>
#include <iomanip>
#include <vector>
#include <string>
#include <algorithm>

struct Student {
    std::string name;
    int age;
    double gpa;

    // Output: "Name (age=XX, gpa=X.XX)"
    friend std::ostream& operator<<(std::ostream& os, const Student& s) {
        return os << s.name << " (age=" << s.age
                  << ", gpa=" << std::fixed << std::setprecision(2) << s.gpa << ")";
    }

    // Input from CSV-style stream: "Name,Age,GPA"
    friend std::istream& operator>>(std::istream& is, Student& s) {
        std::string line;
        if (!std::getline(is, line)) return is;

        std::istringstream iss(line);
        std::string age_str, gpa_str;

        if (!std::getline(iss, s.name, ',') ||
            !std::getline(iss, age_str, ',') ||
            !std::getline(iss, gpa_str, ',')) {
            is.setstate(std::ios::failbit);
            return is;
        }

        try {
            s.age = std::stoi(age_str);
            s.gpa = std::stod(gpa_str);
        } catch (...) {
            is.setstate(std::ios::failbit);
        }
        return is;
    }
};

class ReportGenerator {
    std::vector<Student> students_;

public:
    // Load from file
    void load_csv(const std::string& filename) {
        std::ifstream file(filename);
        if (!file) {
            std::cerr << "Error: cannot open " << filename << '\n';
            return;
        }
        std::string header;
        std::getline(file, header);  // skip header line

        std::string line;
        int line_num = 1;
        while (std::getline(file, line)) {
            ++line_num;
            std::istringstream iss(line);
            Student s;
            if (iss >> s) {
                students_.push_back(std::move(s));
            } else {
                std::cerr << "Warning: skipping malformed line " << line_num << '\n';
            }
        }
        // file closed by RAII destructor
    }

    // Load from string (for testing without files)
    void load_from_string(const std::string& csv_data) {
        std::istringstream iss(csv_data);
        std::string header;
        std::getline(iss, header);  // skip header

        std::string line;
        while (std::getline(iss, line)) {
            std::istringstream line_stream(line);
            Student s;
            if (line_stream >> s) {
                students_.push_back(std::move(s));
            } else {
                std::cerr << "Warning: skipping malformed line\n";
            }
        }
    }

    struct Summary {
        int count;
        double avg_gpa;
        double max_gpa;
        std::string top_student;
    };

    Summary compute_summary() const {
        Summary sum{0, 0.0, 0.0, ""};
        if (students_.empty()) return sum;

        sum.count = static_cast<int>(students_.size());
        double total_gpa = 0;
        for (const auto& s : students_) {
            total_gpa += s.gpa;
            if (s.gpa > sum.max_gpa) {
                sum.max_gpa = s.gpa;
                sum.top_student = s.name;
            }
        }
        sum.avg_gpa = total_gpa / sum.count;
        return sum;
    }

    // Print formatted table
    void print_table(std::ostream& os) const {
        // Header
        os << std::left << std::setw(20) << "Name"
           << std::right << std::setw(5) << "Age"
           << std::setw(8) << "GPA" << '\n';
        os << std::string(33, '-') << '\n';

        // Rows
        for (const auto& s : students_) {
            os << std::left  << std::setw(20) << s.name
               << std::right << std::setw(5)  << s.age
               << std::fixed << std::setprecision(2)
               << std::setw(8) << s.gpa << '\n';
        }
    }

    // Save full report to file
    void save_report(const std::string& filename) const {
        std::ofstream out(filename);
        if (!out) {
            std::cerr << "Error: cannot write to " << filename << '\n';
            return;
        }

        out << "=== Student Report ===\n\n";
        print_table(out);  // reuse same formatting logic

        auto sum = compute_summary();
        out << "\n--- Summary ---\n"
            << "Students: " << sum.count << '\n'
            << "Avg GPA:  " << std::fixed << std::setprecision(2) << sum.avg_gpa << '\n'
            << "Top:      " << sum.top_student
            << " (" << sum.max_gpa << ")\n";
        // file closed by RAII
    }

    int count() const { return static_cast<int>(students_.size()); }
};

int main() {
    ReportGenerator gen;

    // Load from string (simulating file)
    gen.load_from_string(
        "Name,Age,GPA\n"
        "Alice,22,3.85\n"
        "Bob,20,3.42\n"
        "Carol,21,3.91\n"
        "Dave,23,3.15\n"
        "Eve,22,3.78\n"
    );

    std::cout << "Loaded " << gen.count() << " students\n\n";

    // Print table
    gen.print_table(std::cout);

    // Summary
    auto sum = gen.compute_summary();
    std::cout << "\nAvg GPA: " << std::fixed << std::setprecision(2) << sum.avg_gpa
              << "  Top: " << sum.top_student << " (" << sum.max_gpa << ")\n";

    // Save to file
    gen.save_report("/tmp/report.txt");
    std::cout << "\nReport saved to /tmp/report.txt\n";
}
```

Key decisions:
- **`operator>>` for Student** parses CSV with `getline(stream, field, ',')` — reusable and testable (§10.2).
- **`print_table` takes `ostream&`** — works for both `cout` and file output without duplication (§10.2).
- **`<iomanip>`** formatting: `setw` for column alignment (non-sticky), `fixed`+`setprecision` for GPA (sticky) (§10.4).
- **RAII file handles**: both `ifstream` and `ofstream` close automatically on scope exit (§10.5).
- **`cerr`** for errors: unbuffered, appears immediately, separate from normal output (§10.2).
- **`istringstream`** for both CSV parsing and test input — same code path either way (§10.6).

</details>
