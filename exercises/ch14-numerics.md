# Chapter 14: Numerics

---

## Quick Reference

- `<cmath>` provides standard math functions: `sqrt`, `pow`, `abs`, `sin`, `cos`, `log`, `exp`, `ceil`, `floor` (§14.2, p.133)
- `<numeric>` provides `accumulate`, `inner_product`, `partial_sum`, `adjacent_difference`, `iota`, `gcd`, `lcm` (§14.3, p.135)
- `std::complex<double>` provides complex number arithmetic (§14.4, p.136)
- `<random>` provides proper random number generation: engines + distributions (§14.5, p.137)
- **Engines**: `default_random_engine`, `mt19937`, `random_device` (§14.5, p.137)
- **Distributions**: `uniform_int_distribution`, `uniform_real_distribution`, `normal_distribution`, `bernoulli_distribution` (§14.5, p.138)
- `random_device` provides non-deterministic random numbers (seed source) (§14.5, p.137)
- `std::valarray<T>` supports element-wise operations on numeric arrays (§14.6, p.139)
- `std::numeric_limits<T>` provides min, max, epsilon, etc. for numeric types (§14.2, p.134)
- Prefer `<random>` over `rand()`/`srand()` — the old C functions have poor distribution and limited range (§14.5, p.137)
- `std::reduce` (C++17) is like `accumulate` but allows parallel execution (§14.3, p.135)

---

## 1. Multiple Choice (8 questions)

**Q1.** Why is `<random>` preferred over `rand()/srand()`? (§14.5, p.137)

a) `rand()` is not available in C++
b) `<random>` provides better distributions, longer periods, and is not shared global state
c) `rand()` is slower
d) They are equivalent

<details><summary>Answer</summary>

**b)** `rand()` has a short period, poor distribution quality, and uses global state (not thread-safe). `<random>` provides multiple engines with excellent statistical properties and separates engines from distributions.

</details>

---

**Q2.** What does `std::accumulate` do? (§14.3, p.135)

a) Sorts and accumulates
b) Computes the sum (or custom fold) of a range, starting from an initial value
c) Counts elements
d) Finds the maximum

<details><summary>Answer</summary>

**b)** `accumulate(begin, end, init)` folds the range with `+` starting from `init`. A custom binary operation can be provided as the fourth argument.

</details>

---

**Q3.** What is `std::random_device` used for? (§14.5, p.137)

a) Generating the same sequence every run
b) Providing non-deterministic random numbers, typically used to seed other engines
c) Replacing `mt19937`
d) Generating distributions

<details><summary>Answer</summary>

**b)** `random_device` provides entropy from the OS. It's typically used once to seed a faster engine like `mt19937`.

</details>

---

**Q4.** What does `std::iota(begin, end, start_val)` do? (§14.3, p.135)

a) Fills with zeros
b) Fills with `start_val` then increments: `start_val, start_val+1, start_val+2, ...`
c) Fills with random values
d) Reverses the range

<details><summary>Answer</summary>

**b)** `iota` fills the range with sequentially increasing values starting from `start_val`.

</details>

---

**Q5.** What is `numeric_limits<double>::epsilon()`? (§14.2, p.134)

a) The smallest `double` value
b) The difference between 1.0 and the next representable double
c) Zero
d) The largest `double` value

<details><summary>Answer</summary>

**b)** Machine epsilon is the smallest value such that `1.0 + epsilon != 1.0`. It characterizes floating-point precision.

</details>

---

**Q6.** What is the correct way to generate a uniform integer in [1, 6]? (§14.5, p.138)

a) `rand() % 6 + 1`
b) `uniform_int_distribution<int>(1, 6)(engine)`
c) `engine() % 6 + 1`
d) `random_device() % 6 + 1`

<details><summary>Answer</summary>

**b)** The distribution object properly maps the engine's output to the desired range with uniform probability. `rand() % 6` has modulo bias.

</details>

---

**Q7.** What does `std::inner_product` compute? (§14.3, p.135)

a) The cross product of two vectors
b) The dot product: sum of element-wise products
c) The outer product matrix
d) The norm of a vector

<details><summary>Answer</summary>

**b)** `inner_product(a.begin(), a.end(), b.begin(), 0)` computes `a[0]*b[0] + a[1]*b[1] + ...` — the dot product.

</details>

---

**Q8.** What is `std::valarray`? (§14.6, p.139)

a) A dynamically-sized array with virtual dispatch
b) An array type optimized for element-wise numeric operations
c) A replacement for `std::vector`
d) A fixed-size array

<details><summary>Answer</summary>

**b)** `valarray` supports element-wise operations: `v1 + v2`, `v1 * 2.0`, `sin(v1)` etc. It's designed for numeric computation, though in practice `vector` + algorithms is more common.

</details>

---

## 2. Fill-in-the-blank Code (5 snippets)

**Snippet 1** — accumulate (§14.3, p.135)

```cpp
#include <numeric>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    int sum = std::___(v.begin(), v.end(), ___);  // sum starting from 0
    int product = std::___(v.begin(), v.end(), 1,
                           std::___<int>());       // product
    std::cout << sum << " " << product << '\n';     // 15 120
}
```

<details><summary>Answer</summary>

```cpp
int sum = std::accumulate(v.begin(), v.end(), 0);
int product = std::accumulate(v.begin(), v.end(), 1, std::multiplies<int>());
```

</details>

---

**Snippet 2** — Random number generation (§14.5, p.137)

```cpp
#include <random>
#include <iostream>

int main() {
    std::___ rd;                          // entropy source
    std::___ gen(rd());                    // Mersenne Twister, seeded
    std::uniform_int_distribution<int> dist(___, ___);  // [1, 6]

    for (int i = 0; i < 10; ++i)
        std::cout << dist(gen) << " ";
}
```

<details><summary>Answer</summary>

```cpp
std::random_device rd;
std::mt19937 gen(rd());
std::uniform_int_distribution<int> dist(1, 6);
```

</details>

---

**Snippet 3** — iota (§14.3, p.135)

```cpp
#include <numeric>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v(10);
    std::___(v.begin(), v.end(), ___);  // fill with 0,1,2,...,9
    for (auto x : v) std::cout << x << " ";
}
```

<details><summary>Answer</summary>

```cpp
std::iota(v.begin(), v.end(), 0);
```

</details>

---

**Snippet 4** — numeric_limits (§14.2, p.134)

```cpp
#include <limits>
#include <iostream>

int main() {
    std::cout << "int max: "    << std::numeric_limits<int>::___() << '\n';
    std::cout << "double eps: " << std::numeric_limits<double>::___() << '\n';
    std::cout << "float min: "  << std::numeric_limits<float>::___() << '\n';
}
```

<details><summary>Answer</summary>

```cpp
std::numeric_limits<int>::max()
std::numeric_limits<double>::epsilon()
std::numeric_limits<float>::min()      // smallest positive normalized value
```

</details>

---

**Snippet 5** — gcd and lcm (§14.3, p.135)

```cpp
#include <numeric>
#include <iostream>

int main() {
    std::cout << std::___(12, 8) << '\n';   // 4
    std::cout << std::___(12, 8) << '\n';   // 24
}
```

<details><summary>Answer</summary>

```cpp
std::cout << std::gcd(12, 8) << '\n';   // 4
std::cout << std::lcm(12, 8) << '\n';   // 24
```

</details>

---

## 3. Code from Scratch — LeetCode Style (3 problems)

### Problem 1: Statistical summary (§14.3, p.135)

Write a function that takes a `vector<double>` and returns the mean, variance, and standard deviation using `<numeric>` and `<cmath>`.

```
Signature: std::tuple<double,double,double> stats(const std::vector<double>& data);
// returns {mean, variance, stddev}
```

**Examples:**

| Input | Output (approx) |
|---|---|
| `{2, 4, 4, 4, 5, 5, 7, 9}` | `{5.0, 4.0, 2.0}` |

**Constraints:** Use `std::accumulate`. No manual loops for the sum.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <vector>
#include <numeric>
#include <cmath>
#include <tuple>

std::tuple<double,double,double> stats(const std::vector<double>& data) {
    double n = data.size();
    // Mean: sum / n
    double mean = std::accumulate(data.begin(), data.end(), 0.0) / n;

    // Variance: average of squared deviations
    double var = std::accumulate(data.begin(), data.end(), 0.0,
        [mean](double acc, double x) {
            return acc + (x - mean) * (x - mean);
        }) / n;

    return {mean, var, std::sqrt(var)};
}

int main() {
    auto [m, v, sd] = stats({2, 4, 4, 4, 5, 5, 7, 9});
    std::cout << "Mean: " << m << " Var: " << v << " StdDev: " << sd << '\n';
    // Mean: 5 Var: 4 StdDev: 2
}
```

</details>

---

### Problem 2: Dice simulator (§14.5, p.137)

Write a function that simulates rolling `n` dice and returns a frequency map of the results.

```
Signature: std::map<int,int> roll_dice(int n, int sides = 6);
```

**Examples:**

| Input | Output (varies) |
|---|---|
| `n=1000, sides=6` | `{1: ~167, 2: ~167, ...}` |

**Constraints:** Use `mt19937` seeded by `random_device`. Use `uniform_int_distribution`.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <map>
#include <random>

std::map<int,int> roll_dice(int n, int sides = 6) {
    std::random_device rd;
    std::mt19937 gen(rd());                       // good-quality engine
    std::uniform_int_distribution<int> dist(1, sides);

    std::map<int,int> freq;
    for (int i = 0; i < n; ++i)
        ++freq[dist(gen)];                        // roll and tally
    return freq;
}

int main() {
    auto results = roll_dice(6000);
    for (auto [face, count] : results)
        std::cout << face << ": " << count << '\n';
    // Each face appears ~1000 times
}
```

</details>

---

### Problem 3: Dot product using inner_product (§14.3, p.135)

Write a function that computes the cosine similarity between two vectors: `dot(a,b) / (|a| * |b|)`.

```
Signature: double cosine_similarity(const std::vector<double>& a, const std::vector<double>& b);
```

**Examples:**

| Input | Output |
|---|---|
| `{1,0,0}, {1,0,0}` | `1.0` |
| `{1,0}, {0,1}` | `0.0` (orthogonal) |

**Constraints:** Use `std::inner_product` for the dot product. Use `<cmath>` for `sqrt`.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <vector>
#include <numeric>
#include <cmath>

double cosine_similarity(const std::vector<double>& a, const std::vector<double>& b) {
    // Dot product: sum of a[i]*b[i]
    double dot = std::inner_product(a.begin(), a.end(), b.begin(), 0.0);
    // Magnitudes
    double mag_a = std::sqrt(std::inner_product(a.begin(), a.end(), a.begin(), 0.0));
    double mag_b = std::sqrt(std::inner_product(b.begin(), b.end(), b.begin(), 0.0));
    if (mag_a == 0 || mag_b == 0) return 0.0;
    return dot / (mag_a * mag_b);
}

int main() {
    std::cout << cosine_similarity({1,0,0}, {1,0,0}) << '\n';  // 1
    std::cout << cosine_similarity({1,0}, {0,1}) << '\n';        // 0
    std::cout << cosine_similarity({1,1}, {1,1}) << '\n';        // 1
}
```

</details>

---

## 4. Output Prediction (4 snippets)

**Snippet 1** (§14.3, p.135)

```cpp
#include <numeric>
#include <vector>
#include <iostream>
int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    std::cout << std::accumulate(v.begin(), v.end(), 0) << '\n';
    std::cout << std::accumulate(v.begin(), v.end(), 1, std::multiplies<int>()) << '\n';
}
```

<details><summary>Answer</summary>

Output:
```
15
120
```

Sum: 1+2+3+4+5=15. Product: 1*1*2*3*4*5=120.

</details>

---

**Snippet 2** (§14.3, p.135)

```cpp
#include <numeric>
#include <vector>
#include <iostream>
int main() {
    std::vector<int> v(5);
    std::iota(v.begin(), v.end(), 10);
    for (auto x : v) std::cout << x << " ";
}
```

<details><summary>Answer</summary>

Output: `10 11 12 13 14`. `iota` fills with sequentially increasing values starting from 10.

</details>

---

**Snippet 3** — accumulate with wrong initial type (§14.3, p.135)

```cpp
#include <numeric>
#include <vector>
#include <iostream>
int main() {
    std::vector<double> v = {1.5, 2.5, 3.5};
    auto sum = std::accumulate(v.begin(), v.end(), 0);
    std::cout << sum << '\n';
}
```

<details><summary>Answer</summary>

Output: `6` (not `7.5`!). The initial value `0` is an `int`, so accumulate uses `int` arithmetic. Each addition truncates: `0+1.5=1`, `1+2.5=3`, `3+3.5=6`. Fix: use `0.0` as the initial value.

</details>

---

**Snippet 4** (§14.2, p.134)

```cpp
#include <limits>
#include <iostream>
int main() {
    std::cout << std::numeric_limits<int>::max() << '\n';
    std::cout << std::numeric_limits<int>::min() << '\n';
    std::cout << std::numeric_limits<int>::is_signed << '\n';
}
```

<details><summary>Answer</summary>

Output (typical 32-bit int):
```
2147483647
-2147483648
1
```

`max()` and `min()` give the range boundaries. `is_signed` is `true` (1) for `int`.

</details>

---

## 5. Debug This (3 snippets)

**Bug 1** — Using rand() % n (biased) (§14.5, p.137)

```cpp
#include <cstdlib>
#include <iostream>

int main() {
    srand(42);
    for (int i = 0; i < 10; ++i)
        std::cout << rand() % 6 + 1 << " ";  // dice roll
}
```

<details><summary>Answer</summary>

**Bug:** `rand() % 6` has modulo bias (if `RAND_MAX` is not a multiple of 6, some values are slightly more probable). Also, `rand()` uses global state and has poor randomness.

**Fix:** Use `mt19937` + `uniform_int_distribution<int>(1, 6)`.

</details>

---

**Bug 2** — accumulate with int initial for doubles (§14.3, p.135)

```cpp
#include <numeric>
#include <vector>
#include <iostream>

int main() {
    std::vector<double> v = {0.1, 0.2, 0.3};
    auto sum = std::accumulate(v.begin(), v.end(), 0);
    std::cout << sum << '\n';  // expects 0.6, gets 0
}
```

<details><summary>Answer</summary>

**Bug:** Initial value `0` is `int`. The accumulator uses `int` arithmetic: `0 + 0.1 = 0` (truncated), etc.

**Fix:** `std::accumulate(v.begin(), v.end(), 0.0)` — use `0.0` to ensure `double` arithmetic.

</details>

---

**Bug 3** — Reseeding engine every call (§14.5, p.137)

```cpp
#include <random>
#include <iostream>

int random_number() {
    std::random_device rd;
    std::mt19937 gen(rd());
    std::uniform_int_distribution<int> dist(1, 100);
    return dist(gen);
}
```

<details><summary>Answer</summary>

**Bug:** Creating a new `random_device` and `mt19937` every call is expensive and can reduce randomness quality. Some `random_device` implementations may return the same value if called rapidly.

**Fix:** Make `gen` `static` or pass it as a parameter:
```cpp
static std::mt19937 gen(std::random_device{}());
```

</details>

---

## 6. System Design / Open-ended (2 questions)

**Q1.** You're building a Monte Carlo simulation that needs billions of random samples. Discuss: (a) why `rand()` is inadequate, (b) how `mt19937` improves things, (c) how you'd seed for reproducibility vs. true randomness, (d) what `random_device` provides and its limitations. (§14.5, p.137)

---

**Q2.** Numerical precision matters: `accumulate` summing millions of small floats can lose precision. Research and explain Kahan summation. How would you implement it using `accumulate` with a custom operation? (§14.3, p.135; §14.2, p.134)

---

## 7. Rapid Fire (10 true/false)

1. `std::accumulate` is in `<algorithm>`. (§14.3, p.135)

<details><summary>Answer</summary>False. It's in `<numeric>`.</details>

2. `random_device` is guaranteed to be non-deterministic. (§14.5, p.137)

<details><summary>Answer</summary>False. The standard allows implementations where `random_device` is deterministic (rare in practice).</details>

3. `mt19937` has a period of 2^19937 - 1. (§14.5, p.137)

<details><summary>Answer</summary>True. That's what the name means — Mersenne Twister with that period.</details>

4. `std::iota` fills a range with the same value. (§14.3, p.135)

<details><summary>Answer</summary>False. It fills with sequentially increasing values. Use `std::fill` for a constant value.</details>

5. `numeric_limits<double>::epsilon()` is exactly 0. (§14.2, p.134)

<details><summary>Answer</summary>False. It's a small positive value (~2.22e-16) representing the precision limit.</details>

6. `std::gcd` and `std::lcm` are available since C++17. (§14.3, p.135)

<details><summary>Answer</summary>True. They were added to `<numeric>` in C++17.</details>

7. `valarray` supports element-wise arithmetic operators. (§14.6, p.139)

<details><summary>Answer</summary>True. `v1 + v2`, `v1 * 2.0`, etc. work element-wise.</details>

8. `std::inner_product` requires sorted inputs. (§14.3, p.135)

<details><summary>Answer</summary>False. It works on any two ranges of equal length.</details>

9. `uniform_real_distribution<double>(0, 1)` can return exactly 1.0. (§14.5, p.138)

<details><summary>Answer</summary>False. The range is [0, 1) — the upper bound is excluded.</details>

10. `std::reduce` (C++17) differs from `accumulate` in that it allows out-of-order evaluation. (§14.3, p.135)

<details><summary>Answer</summary>True. `reduce` doesn't guarantee left-to-right evaluation, enabling parallelism.</details>

---

## 8. Capstone Implementation Challenge

### Monte Carlo Option Pricer (§14.2–14.5, p.133–138)

**Motivation:** You're building a financial tool that prices European call options using Monte Carlo simulation. This requires generating millions of random stock price paths using geometric Brownian motion, computing the payoff of each path, and averaging the discounted payoffs.

**Signatures:**

```cpp
struct OptionParams {
    double spot;         // current stock price
    double strike;       // option strike price
    double rate;         // risk-free interest rate (annual)
    double volatility;   // annual volatility (sigma)
    double time_years;   // time to expiration in years
};

struct PricingResult {
    double price;        // estimated option price
    double std_error;    // standard error of the estimate
    int num_paths;
};

class MonteCarloPricer {
public:
    explicit MonteCarloPricer(unsigned seed = 42);

    // Price a European call option
    PricingResult price_call(const OptionParams& params, int num_paths) const;

    // Price a European put option
    PricingResult price_put(const OptionParams& params, int num_paths) const;

    // Generate a single price path (for visualization/debugging)
    std::vector<double> generate_path(const OptionParams& params, int steps) const;
};
```

**Test Scenarios:**

1. Price a call option: spot=100, strike=100, rate=5%, vol=20%, T=1yr, 100k paths. Result should be near Black-Scholes analytical value (~10.45).
2. Put-call parity: `call - put ≈ spot - strike * exp(-rT)`. Verify this holds within standard error.
3. Increasing volatility increases both call and put prices.

**Constraints:**
- Use `<random>` with `mt19937` and `normal_distribution`
- Use `<cmath>` for `exp`, `sqrt`, `log`
- Use `std::accumulate` for averaging payoffs
- Use `<numeric>` for computing standard deviation
- Use `numeric_limits` for any precision checks
- No `rand()`/`srand()`

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <vector>
#include <random>
#include <cmath>
#include <numeric>
#include <iomanip>

struct OptionParams {
    double spot;
    double strike;
    double rate;
    double volatility;
    double time_years;
};

struct PricingResult {
    double price;
    double std_error;
    int num_paths;
};

class MonteCarloPricer {
    mutable std::mt19937 gen_;  // mutable: const methods need to draw random numbers

public:
    explicit MonteCarloPricer(unsigned seed = 42) : gen_{seed} {}

    // Simulate terminal stock price using geometric Brownian motion:
    // S_T = S_0 * exp((r - 0.5*sigma^2)*T + sigma*sqrt(T)*Z)
    // where Z ~ N(0,1)
    double simulate_terminal_price(const OptionParams& p) const {
        std::normal_distribution<double> norm(0.0, 1.0);
        double z = norm(gen_);
        double drift = (p.rate - 0.5 * p.volatility * p.volatility) * p.time_years;
        double diffusion = p.volatility * std::sqrt(p.time_years) * z;
        return p.spot * std::exp(drift + diffusion);
    }

    PricingResult price_call(const OptionParams& params, int num_paths) const {
        std::vector<double> payoffs(num_paths);

        // Generate payoffs: max(S_T - K, 0)
        for (int i = 0; i < num_paths; ++i) {
            double s_t = simulate_terminal_price(params);
            payoffs[i] = std::max(s_t - params.strike, 0.0);
        }

        // Discount factor: e^(-rT)
        double discount = std::exp(-params.rate * params.time_years);

        // Mean payoff using accumulate
        double mean = std::accumulate(payoffs.begin(), payoffs.end(), 0.0) / num_paths;

        // Standard deviation for confidence interval
        double variance = std::accumulate(payoffs.begin(), payoffs.end(), 0.0,
            [mean](double acc, double x) {
                return acc + (x - mean) * (x - mean);
            }) / (num_paths - 1);

        double std_dev = std::sqrt(variance);
        double std_error = discount * std_dev / std::sqrt(static_cast<double>(num_paths));

        return {discount * mean, std_error, num_paths};
    }

    PricingResult price_put(const OptionParams& params, int num_paths) const {
        std::vector<double> payoffs(num_paths);

        for (int i = 0; i < num_paths; ++i) {
            double s_t = simulate_terminal_price(params);
            payoffs[i] = std::max(params.strike - s_t, 0.0);  // put payoff
        }

        double discount = std::exp(-params.rate * params.time_years);
        double mean = std::accumulate(payoffs.begin(), payoffs.end(), 0.0) / num_paths;

        double variance = std::accumulate(payoffs.begin(), payoffs.end(), 0.0,
            [mean](double acc, double x) {
                return acc + (x - mean) * (x - mean);
            }) / (num_paths - 1);

        double std_error = discount * std::sqrt(variance) / std::sqrt(static_cast<double>(num_paths));
        return {discount * mean, std_error, num_paths};
    }

    // Generate a multi-step price path for visualization
    std::vector<double> generate_path(const OptionParams& params, int steps) const {
        std::normal_distribution<double> norm(0.0, 1.0);
        double dt = params.time_years / steps;
        double drift = (params.rate - 0.5 * params.volatility * params.volatility) * dt;
        double vol_sqrt_dt = params.volatility * std::sqrt(dt);

        std::vector<double> path(steps + 1);
        path[0] = params.spot;
        for (int i = 1; i <= steps; ++i) {
            double z = norm(gen_);
            path[i] = path[i-1] * std::exp(drift + vol_sqrt_dt * z);
        }
        return path;
    }
};

int main() {
    MonteCarloPricer pricer(42);

    OptionParams params{100.0, 100.0, 0.05, 0.20, 1.0};
    int paths = 100000;

    // Scenario 1: price a call
    auto call = pricer.price_call(params, paths);
    std::cout << std::fixed << std::setprecision(4);
    std::cout << "Call price: $" << call.price
              << " (SE: " << call.std_error << ")\n";
    // Should be near Black-Scholes ~$10.45

    // Scenario 2: price a put and verify put-call parity
    auto put = pricer.price_put(params, paths);
    std::cout << "Put price:  $" << put.price
              << " (SE: " << put.std_error << ")\n";

    double parity_lhs = call.price - put.price;
    double parity_rhs = params.spot - params.strike * std::exp(-params.rate * params.time_years);
    std::cout << "Put-call parity check: "
              << parity_lhs << " ≈ " << parity_rhs << '\n';

    // Scenario 3: higher volatility increases prices
    OptionParams high_vol = params;
    high_vol.volatility = 0.40;
    auto call_hv = pricer.price_call(high_vol, paths);
    std::cout << "\nHigh-vol call: $" << call_hv.price
              << " (vs base: $" << call.price << ")\n";

    // Bonus: visualize a path
    auto path = pricer.generate_path(params, 12);
    std::cout << "\nSample 12-month path: ";
    for (auto p : path) std::cout << std::setprecision(1) << p << " ";
    std::cout << '\n';
}
```

Key decisions:
- **`mt19937`** for high-quality random generation with reproducible seed (§14.5).
- **`normal_distribution`** for the Z ~ N(0,1) random variable in GBM (§14.5).
- **`std::accumulate`** for computing mean and variance — no manual loops for aggregation (§14.3).
- **`std::exp`, `std::sqrt`, `std::log`** from `<cmath>` for the GBM formula (§14.2).
- **`mutable` on the engine** — const methods need to generate random numbers without modifying logical state.
- **Standard error** = discounted σ / √n — quantifies estimate reliability.

</details>
