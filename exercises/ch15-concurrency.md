# Chapter 15: Concurrency

---

## Quick Reference

- `<thread>` provides `std::thread` for launching concurrent threads of execution (§15.2, p.141)
- A `thread` must be `join()`ed or `detach()`ed before destruction — otherwise `std::terminate` is called (§15.2, p.141)
- **Mutex** (`std::mutex`) provides mutual exclusion: `lock()` and `unlock()` (§15.5, p.145)
- **`lock_guard<mutex>`** is RAII for mutexes — locks on construction, unlocks on destruction (§15.5, p.145)
- **`unique_lock<mutex>`** is more flexible: supports deferred locking, timed locking, manual unlock (§15.5, p.146)
- **Data races** occur when two threads access the same data, at least one writes, and there's no synchronization (§15.1, p.141)
- `std::condition_variable` allows threads to wait for a condition: `wait()`, `notify_one()`, `notify_all()` (§15.6, p.147)
- **`std::future`** and **`std::promise`** for one-time value passing between threads (§15.7, p.148)
- **`std::async`** launches a function asynchronously and returns a `future` (§15.7.1, p.149)
- `std::atomic<T>` provides lock-free atomic operations for simple types (§15.5, p.146)
- Avoid sharing mutable data between threads when possible — prefer message passing or immutable data (§15.1, p.141)
- `std::packaged_task` wraps a callable for asynchronous execution (§15.7.2, p.149)

---

## 1. Multiple Choice (8 questions)

**Q1.** What happens if a `std::thread` is destroyed without being joined or detached? (§15.2, p.141)

a) The thread continues running in the background
b) `std::terminate()` is called
c) The thread is automatically joined
d) Nothing — it's cleaned up by the OS

<details><summary>Answer</summary>

**b)** A joinable `thread` that is destroyed without `join()` or `detach()` calls `std::terminate()`. This is a design choice to prevent silent resource leaks or unjoined threads.

</details>

---

**Q2.** What does `lock_guard` do? (§15.5, p.145)

a) Creates a new mutex
b) Locks a mutex on construction and unlocks it on destruction (RAII)
c) Permanently locks a mutex
d) Tries to lock without blocking

<details><summary>Answer</summary>

**b)** `lock_guard` is the simplest RAII wrapper for mutexes. It acquires the lock in its constructor and releases it in its destructor, ensuring the lock is always released even if an exception occurs.

</details>

---

**Q3.** What is a data race? (§15.1, p.141)

a) Two threads reading the same variable simultaneously
b) Two threads accessing the same data where at least one writes, with no synchronization
c) Any use of multiple threads
d) Using `volatile` instead of `atomic`

<details><summary>Answer</summary>

**b)** A data race is undefined behavior. It occurs when multiple threads access shared data concurrently, at least one modifies it, and there's no synchronization (mutex, atomic, etc.).

</details>

---

**Q4.** What does `std::async` return? (§15.7.1, p.149)

a) A `thread`
b) A `promise`
c) A `future` that will hold the result of the asynchronous computation
d) A `condition_variable`

<details><summary>Answer</summary>

**c)** `std::async(f, args...)` returns a `std::future<T>` where `T` is the return type of `f`. Call `.get()` on the future to wait for and retrieve the result.

</details>

---

**Q5.** When should you use `std::atomic<T>` instead of a mutex? (§15.5, p.146)

a) Always — it's faster
b) When you need atomic read-modify-write on simple types (int, bool, pointer)
c) When you need to protect complex data structures
d) Only with floating-point types

<details><summary>Answer</summary>

**b)** `atomic` is ideal for single variables that need lock-free access (counters, flags). For protecting complex data structures with multiple invariants, use a mutex.

</details>

---

**Q6.** What does `condition_variable::wait(lock, pred)` do? (§15.6, p.147)

a) Busy-waits until the predicate is true
b) Atomically releases the lock and sleeps until notified; re-checks the predicate to guard against spurious wakeups
c) Waits for a fixed timeout
d) Blocks forever

<details><summary>Answer</summary>

**b)** `wait` releases the lock and suspends the thread. When notified, it re-acquires the lock and checks the predicate. If the predicate is false (spurious wakeup), it goes back to sleep.

</details>

---

**Q7.** What is the relationship between `promise` and `future`? (§15.7, p.148)

a) They are the same thing
b) `promise` sets a value; `future` retrieves it — they're a one-time communication channel
c) `promise` is for reading; `future` is for writing
d) They must be used with `async`

<details><summary>Answer</summary>

**b)** A `promise`/`future` pair is a one-shot channel. The producer calls `promise::set_value()`, and the consumer calls `future::get()` to retrieve it (blocking if not yet set).

</details>

---

**Q8.** Why is `unique_lock` more flexible than `lock_guard`? (§15.5, p.146)

a) It's faster
b) It supports deferred locking, timed locking, and manual lock/unlock
c) It doesn't need a mutex
d) It can be copied

<details><summary>Answer</summary>

**b)** `unique_lock` adds features: construct without locking (`std::defer_lock`), `try_lock()`, `try_lock_for()`, and manual `unlock()`. It's required by `condition_variable::wait()`.

</details>

---

## 2. Fill-in-the-blank Code (5 snippets)

**Snippet 1** — Basic thread (§15.2, p.141)

```cpp
#include <thread>
#include <iostream>

void hello() { std::cout << "Hello from thread!\n"; }

int main() {
    std::___ t(hello);    // launch thread
    t.___();                // wait for completion
}
```

<details><summary>Answer</summary>

```cpp
std::thread t(hello);
t.join();
```

</details>

---

**Snippet 2** — Mutex with lock_guard (§15.5, p.145)

```cpp
#include <thread>
#include <mutex>
#include <iostream>

std::___ mtx;
int counter = 0;

void increment(int n) {
    for (int i = 0; i < n; ++i) {
        std::___<std::mutex> lock(mtx);  // RAII lock
        ++counter;
    }
}

int main() {
    std::thread t1(increment, 100000);
    std::thread t2(increment, 100000);
    t1.join(); t2.join();
    std::cout << counter << '\n';  // 200000
}
```

<details><summary>Answer</summary>

```cpp
std::mutex mtx;
std::lock_guard<std::mutex> lock(mtx);
```

</details>

---

**Snippet 3** — async and future (§15.7.1, p.149)

```cpp
#include <future>
#include <iostream>

int compute(int x) { return x * x; }

int main() {
    auto f = std::___(compute, 42);  // launch async
    std::cout << "Working...\n";
    std::cout << "Result: " << f.___() << '\n';  // get result
}
```

<details><summary>Answer</summary>

```cpp
auto f = std::async(compute, 42);
std::cout << "Result: " << f.get() << '\n';
```

</details>

---

**Snippet 4** — atomic counter (§15.5, p.146)

```cpp
#include <thread>
#include <atomic>
#include <iostream>

std::___<int> counter{0};

void count_up(int n) {
    for (int i = 0; i < n; ++i)
        counter.___();  // atomic increment
}

int main() {
    std::thread t1(count_up, 100000);
    std::thread t2(count_up, 100000);
    t1.join(); t2.join();
    std::cout << counter.___() << '\n';  // 200000
}
```

<details><summary>Answer</summary>

```cpp
std::atomic<int> counter{0};
counter.fetch_add(1);   // or counter++
counter.load()          // or just counter
```

</details>

---

**Snippet 5** — condition_variable (§15.6, p.147)

```cpp
#include <thread>
#include <mutex>
#include <condition_variable>
#include <iostream>

std::mutex mtx;
std::___ cv;
bool ready = false;

void worker() {
    std::unique_lock<std::mutex> lock(mtx);
    cv.___(lock, []{ return ready; });  // wait until ready
    std::cout << "Worker running!\n";
}

int main() {
    std::thread t(worker);
    {
        std::lock_guard<std::mutex> lock(mtx);
        ready = true;
    }
    cv.___();  // wake up worker
    t.join();
}
```

<details><summary>Answer</summary>

```cpp
std::condition_variable cv;
cv.wait(lock, []{ return ready; });
cv.notify_one();
```

</details>

---

## 3. Code from Scratch — LeetCode Style (3 problems)

### Problem 1: Thread-safe counter (§15.5, p.145)

Implement a `Counter` class that can be safely incremented from multiple threads using `mutex` and `lock_guard`.

```
Signature:
  class Counter {
  public:
      void increment();
      int get() const;
  };
```

**Examples:**

| Scenario | Result |
|---|---|
| 4 threads each increment 250,000 times | `get() == 1,000,000` |

**Constraints:** Use `std::mutex` and `std::lock_guard`. No `atomic`.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <vector>

class Counter {
    mutable std::mutex mtx_;   // mutable so const methods can lock
    int count_ = 0;
public:
    void increment() {
        std::lock_guard<std::mutex> lock(mtx_);  // RAII: lock here, unlock at scope end
        ++count_;
    }

    int get() const {
        std::lock_guard<std::mutex> lock(mtx_);
        return count_;
    }
};

int main() {
    Counter c;
    std::vector<std::thread> threads;

    // Launch 4 threads, each incrementing 250k times
    for (int i = 0; i < 4; ++i)
        threads.emplace_back([&c]() {
            for (int j = 0; j < 250000; ++j)
                c.increment();
        });

    for (auto& t : threads) t.join();
    std::cout << c.get() << '\n';  // 1000000
}
```

</details>

---

### Problem 2: Parallel map with async (§15.7.1, p.149)

Write a function that applies a transformation to each element of a vector in parallel using `std::async`, collecting results via `future`.

```
Signature:
  template<typename T, typename F>
  std::vector<T> parallel_map(const std::vector<T>& input, F func);
```

**Examples:**

| Input | func | Output |
|---|---|---|
| `{1,2,3,4}` | `[](int x){ return x*x; }` | `{1,4,9,16}` |

**Constraints:** Use `std::async`. Each element processed independently.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <vector>
#include <future>

template<typename T, typename F>
std::vector<T> parallel_map(const std::vector<T>& input, F func) {
    // Launch an async task for each element
    std::vector<std::future<T>> futures;
    for (const auto& x : input)
        futures.push_back(std::async(std::launch::async, func, x));

    // Collect results
    std::vector<T> result;
    for (auto& f : futures)
        result.push_back(f.get());  // blocks until each task completes
    return result;
}

int main() {
    auto result = parallel_map(std::vector<int>{1,2,3,4},
                               [](int x) { return x * x; });
    for (auto x : result) std::cout << x << " ";  // 1 4 9 16
}
```

</details>

---

### Problem 3: Producer-consumer queue (§15.6, p.147)

Implement a thread-safe queue where producers push items and consumers pop them. Use `condition_variable` for efficient waiting.

```
Signature:
  template<typename T>
  class ThreadSafeQueue {
  public:
      void push(T val);
      T pop();  // blocks until an item is available
  };
```

**Examples:**

| Producer pushes | Consumer pops |
|---|---|
| 1, 2, 3 | 1, 2, 3 (FIFO) |

**Constraints:** Use `mutex`, `condition_variable`, `unique_lock`. Use `std::queue` internally.

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <queue>
#include <thread>
#include <mutex>
#include <condition_variable>

template<typename T>
class ThreadSafeQueue {
    std::queue<T> queue_;
    std::mutex mtx_;
    std::condition_variable cv_;
public:
    void push(T val) {
        {
            std::lock_guard<std::mutex> lock(mtx_);
            queue_.push(std::move(val));
        }
        cv_.notify_one();   // wake one waiting consumer
    }

    T pop() {
        std::unique_lock<std::mutex> lock(mtx_);
        // Wait until queue is non-empty; predicate guards against spurious wakeups
        cv_.wait(lock, [this]{ return !queue_.empty(); });
        T val = std::move(queue_.front());
        queue_.pop();
        return val;
    }
};

int main() {
    ThreadSafeQueue<int> q;

    // Producer
    std::thread prod([&q]() {
        for (int i = 1; i <= 5; ++i) {
            q.push(i);
        }
    });

    // Consumer
    std::thread cons([&q]() {
        for (int i = 0; i < 5; ++i) {
            std::cout << q.pop() << " ";
        }
        std::cout << '\n';
    });

    prod.join();
    cons.join();
    // Output: 1 2 3 4 5
}
```

</details>

---

## 4. Output Prediction (4 snippets)

**Snippet 1** — Thread ordering (§15.2, p.141)

```cpp
#include <thread>
#include <iostream>
void f() { std::cout << "A"; }
void g() { std::cout << "B"; }
int main() {
    std::thread t1(f);
    std::thread t2(g);
    t1.join();
    t2.join();
    std::cout << "C\n";
}
```

<details><summary>Answer</summary>

Output: Either `ABC`, `BAC`, or interleaved. The order of A and B is non-deterministic — threads may execute in any order. `C` always comes last because both threads are joined first.

</details>

---

**Snippet 2** — Data race (§15.1, p.141)

```cpp
#include <thread>
#include <iostream>
int x = 0;
void inc() { for (int i = 0; i < 1000000; ++i) ++x; }
int main() {
    std::thread t1(inc);
    std::thread t2(inc);
    t1.join(); t2.join();
    std::cout << x << '\n';
}
```

<details><summary>Answer</summary>

**Undefined behavior (data race).** Both threads modify `x` without synchronization. The result is unpredictable — likely less than 2,000,000. Could be any value.

</details>

---

**Snippet 3** — async with deferred (§15.7.1, p.149)

```cpp
#include <future>
#include <iostream>
int main() {
    auto f = std::async(std::launch::deferred, []{ return 42; });
    std::cout << "Before get\n";
    std::cout << f.get() << '\n';
}
```

<details><summary>Answer</summary>

Output:
```
Before get
42
```

With `std::launch::deferred`, the function runs lazily — only when `.get()` is called. It executes synchronously in the calling thread.

</details>

---

**Snippet 4** — Forgotten join (§15.2, p.141)

```cpp
#include <thread>
#include <iostream>
int main() {
    std::thread t([]{ std::cout << "hello\n"; });
    // forgot t.join() or t.detach()
}
```

<details><summary>Answer</summary>

**`std::terminate()` is called.** A joinable thread destroyed without `join()` or `detach()` terminates the program. The actual output before termination is unpredictable.

</details>

---

## 5. Debug This (3 snippets)

**Bug 1** — Forgetting to join (§15.2, p.141)

```cpp
#include <thread>
#include <iostream>

void work() { std::cout << "Working\n"; }

int main() {
    std::thread t(work);
    std::cout << "Done\n";
}
```

<details><summary>Answer</summary>

**Bug:** `t` is joinable when `main` exits → `std::terminate()`.

**Fix:** Add `t.join();` before `main` returns (or `t.detach()` if you don't need to wait).

</details>

---

**Bug 2** — Lock guard scope too wide (§15.5, p.145)

```cpp
#include <thread>
#include <mutex>
#include <iostream>
#include <chrono>

std::mutex mtx;

void slow_work() {
    std::lock_guard<std::mutex> lock(mtx);  // locked for entire function
    // critical section (fast)
    int x = 42;
    // non-critical section (slow I/O)
    std::this_thread::sleep_for(std::chrono::seconds(2));
    std::cout << x << '\n';
}
```

<details><summary>Answer</summary>

**Bug:** The mutex is held during the slow non-critical section, blocking other threads unnecessarily.

**Fix:** Limit the lock scope to just the critical section:
```cpp
void slow_work() {
    int x;
    { std::lock_guard<std::mutex> lock(mtx); x = 42; }  // unlock here
    std::this_thread::sleep_for(std::chrono::seconds(2));
    std::cout << x << '\n';
}
```

</details>

---

**Bug 3** — Condition variable without predicate (§15.6, p.147)

```cpp
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void waiter() {
    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock);  // no predicate!
    // proceed...
}
```

<details><summary>Answer</summary>

**Bug:** `wait` without a predicate is susceptible to **spurious wakeups**. The thread may wake up even though `ready` is still false.

**Fix:** `cv.wait(lock, []{ return ready; });` — the predicate is re-checked after each wakeup.

</details>

---

## 6. System Design / Open-ended (2 questions)

**Q1.** Design a thread pool with a fixed number of worker threads and a task queue. Workers take tasks from the queue and execute them. What synchronization primitives do you need? How do you signal workers to shut down gracefully? How does this compare to launching a new `std::async` for each task? (§15.2, p.141; §15.5, p.145; §15.6, p.147)

---

**Q2.** You're building a cache that multiple threads read from and occasionally write to. Compare these approaches: (a) a single `mutex`, (b) a `shared_mutex` (readers-writer lock), (c) lock-free using `atomic`. What are the tradeoffs in terms of contention, complexity, and correctness? (§15.5, p.145–146)

---

## 7. Rapid Fire (10 true/false)

1. `std::thread::detach()` makes the thread run in the background. (§15.2, p.141)

<details><summary>Answer</summary>True. A detached thread runs independently; the program doesn't wait for it.</details>

2. A data race is undefined behavior. (§15.1, p.141)

<details><summary>Answer</summary>True. Two threads accessing shared data without synchronization where at least one writes is UB.</details>

3. `lock_guard` can be manually unlocked before scope end. (§15.5, p.145)

<details><summary>Answer</summary>False. `lock_guard` has no `unlock()` method. Use `unique_lock` for manual control.</details>

4. `std::async` always creates a new thread. (§15.7.1, p.149)

<details><summary>Answer</summary>False. With `std::launch::deferred`, it runs lazily in the calling thread. Without specifying a policy, the implementation may choose either.</details>

5. `condition_variable::wait` requires a `unique_lock`, not a `lock_guard`. (§15.6, p.147)

<details><summary>Answer</summary>True. `wait` needs to unlock/relock the mutex, which `lock_guard` cannot do.</details>

6. `std::atomic<int>` operations are always lock-free. (§15.5, p.146)

<details><summary>Answer</summary>False. The standard doesn't guarantee lock-free for all types. Check `atomic::is_lock_free()`.</details>

7. `future::get()` can be called multiple times. (§15.7, p.148)

<details><summary>Answer</summary>False. `get()` moves the value out on first call. Calling again is undefined. Use `shared_future` for multiple reads.</details>

8. Spurious wakeups can occur with `condition_variable`. (§15.6, p.147)

<details><summary>Answer</summary>True. That's why you always pass a predicate to `wait()`.</details>

9. `std::thread` copies its arguments by default. (§15.2, p.141)

<details><summary>Answer</summary>True. Arguments are copied into the thread. Use `std::ref()` to pass by reference.</details>

10. A `mutex` can be locked recursively by the same thread. (§15.5, p.145)

<details><summary>Answer</summary>False. `std::mutex` is not recursive — locking it twice from the same thread is UB. Use `std::recursive_mutex` if needed.</details>

---

## 8. Capstone Implementation Challenge

### Thread Pool with Future-Based Results (§15.2–15.7, p.141–149)

**Motivation:** You're building a thread pool for a web server. The pool has a fixed number of worker threads that pick up tasks from a shared queue. Each submitted task returns a `future` so the caller can retrieve the result later. This is the foundation of concurrent server architectures.

**Signatures:**

```cpp
class ThreadPool {
public:
    explicit ThreadPool(int num_threads);
    ~ThreadPool();

    // Submit a callable and get a future for its result
    template<typename F, typename... Args>
    auto submit(F&& func, Args&&... args) -> std::future<std::invoke_result_t<F, Args...>>;

    // Graceful shutdown: finish all queued tasks, then stop
    void shutdown();

    int thread_count() const;
    bool is_running() const;
};
```

**Test Scenarios:**

1. Create pool with 4 threads. Submit 20 tasks (compute squares). Collect all results via futures. All 20 results are correct.
2. Submit tasks that take varying amounts of time. All complete. Order of completion doesn't matter.
3. `shutdown()` waits for all in-progress tasks, then pool destructor is clean (no `terminate`).

**Constraints:**
- Use `std::thread` for worker threads
- Use `std::mutex` and `std::condition_variable` for the task queue
- Use `std::packaged_task` to wrap callables and produce futures
- Use `std::function<void()>` for type-erased task storage
- Use `std::unique_lock` (not `lock_guard`) with condition_variable
- Workers must be joined in the destructor — no detaching
- Must handle the shutdown sequence correctly (no data races, no deadlocks)

<details><summary>Solution</summary>

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <future>
#include <functional>
#include <type_traits>

class ThreadPool {
    std::vector<std::thread> workers_;
    std::queue<std::function<void()>> tasks_;
    std::mutex mtx_;
    std::condition_variable cv_;
    bool stop_ = false;

    // Worker loop: wait for tasks, execute them
    void worker_loop() {
        while (true) {
            std::function<void()> task;
            {
                std::unique_lock<std::mutex> lock(mtx_);
                // Wait until there's a task or we're shutting down
                cv_.wait(lock, [this] {
                    return stop_ || !tasks_.empty();
                });
                // If shutting down and no tasks left, exit
                if (stop_ && tasks_.empty())
                    return;
                // Take the next task
                task = std::move(tasks_.front());
                tasks_.pop();
            }
            task();  // execute outside the lock
        }
    }

public:
    explicit ThreadPool(int num_threads) {
        for (int i = 0; i < num_threads; ++i)
            workers_.emplace_back(&ThreadPool::worker_loop, this);
    }

    ~ThreadPool() {
        shutdown();
    }

    // Submit: wrap callable in packaged_task, store as function<void()>
    template<typename F, typename... Args>
    auto submit(F&& func, Args&&... args)
        -> std::future<std::invoke_result_t<F, Args...>>
    {
        using ReturnType = std::invoke_result_t<F, Args...>;

        // Bind arguments into the callable
        auto bound = std::bind(std::forward<F>(func), std::forward<Args>(args)...);

        // packaged_task wraps the callable and provides a future
        auto task = std::make_shared<std::packaged_task<ReturnType()>>(std::move(bound));
        auto future = task->get_future();

        {
            std::lock_guard<std::mutex> lock(mtx_);
            if (stop_)
                throw std::runtime_error("submit on stopped pool");
            // Type-erase the task as function<void()>
            tasks_.push([task]() { (*task)(); });
        }
        cv_.notify_one();  // wake one worker
        return future;
    }

    void shutdown() {
        {
            std::lock_guard<std::mutex> lock(mtx_);
            if (stop_) return;  // already shut down
            stop_ = true;
        }
        cv_.notify_all();  // wake all workers so they can exit

        // Join all threads — blocks until each finishes its current task
        for (auto& w : workers_) {
            if (w.joinable())
                w.join();
        }
    }

    int thread_count() const { return static_cast<int>(workers_.size()); }
    bool is_running() const { return !stop_; }
};

int main() {
    // Scenario 1: compute squares
    {
        ThreadPool pool(4);
        std::vector<std::future<int>> futures;

        for (int i = 0; i < 20; ++i) {
            futures.push_back(pool.submit([](int x) { return x * x; }, i));
        }

        std::cout << "Squares: ";
        for (auto& f : futures)
            std::cout << f.get() << " ";
        std::cout << '\n';
    }  // pool destroyed here — shutdown + join

    // Scenario 2: mixed-duration tasks
    {
        ThreadPool pool(2);

        auto f1 = pool.submit([] {
            std::this_thread::sleep_for(std::chrono::milliseconds(50));
            return std::string("slow");
        });

        auto f2 = pool.submit([] {
            return std::string("fast");
        });

        auto f3 = pool.submit([] {
            std::this_thread::sleep_for(std::chrono::milliseconds(25));
            return std::string("medium");
        });

        std::cout << "Results: " << f1.get() << ", "
                  << f2.get() << ", " << f3.get() << '\n';
    }

    // Scenario 3: explicit shutdown
    {
        ThreadPool pool(2);
        pool.submit([] { return 42; });
        pool.shutdown();
        std::cout << "Pool shut down cleanly\n";

        // Submitting after shutdown throws
        try {
            pool.submit([] { return 0; });
        } catch (const std::runtime_error& e) {
            std::cout << "Caught: " << e.what() << '\n';
        }
    }
}
```

Key decisions:
- **`std::packaged_task`** bridges the gap between type-erased `function<void()>` storage and typed `future<T>` results (§15.7.2).
- **`shared_ptr<packaged_task>`** is needed because `function<void()>` requires copyability, but `packaged_task` is move-only. Wrapping in `shared_ptr` makes the lambda copyable.
- **`condition_variable::wait(lock, predicate)`** guards against spurious wakeups and serves as the shutdown signal (§15.6).
- **Workers check `stop_ && tasks_.empty()`** — they finish existing tasks before exiting (graceful shutdown).
- **Destructor calls `shutdown()`** — RAII ensures threads are always joined (§15.2).
- **`unique_lock`** is required by `condition_variable::wait` (not `lock_guard`) (§15.5).

</details>
