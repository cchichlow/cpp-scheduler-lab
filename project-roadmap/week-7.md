Now we cross an important line:

 From **single-threaded logic** ? **real concurrent system**

This is where things get powerful *and* tricky.

---

#  Week 7 — Multithreading Basics

##  Goal

Run tasks **in parallel** instead of one-by-one.

Right now:

```cpp
task.execute(); // sequential
```

After this week:
 Multiple tasks run **at the same time**

---

#  Important Mindset Shift

In single-threaded code:

* Order is predictable

In multithreaded code:

* Order is **NOT guaranteed**

 You must design for that.

---

#  Step 1 — Your First Thread

##  Include headers

In `Scheduler.cpp`:

```cpp
#include <thread>
```

---

##  Replace execution loop

Find where you execute tasks:

```cpp
tasks[current]->execute();
```

---

##  Replace with:

```cpp
std::thread t([this, current]() {
    tasks[current]->execute();
});

t.join();
```

---

#  What this does

```cpp
std::thread t(...)
```

 Starts a new thread

```cpp
t.join();
```

 Waits for it to finish

---

##  Important

This is still effectively sequential because:

* You start a thread
* Immediately wait for it

 Next step fixes that.

---

#  Step 2 — Run Multiple Tasks in Parallel

Now we allow **multiple threads at once**

---

##  Modify `run()`

Replace execution section with:

```cpp
std::vector<std::thread> threads;
```

---

Then inside loop:

```cpp
threads.emplace_back([this, current]() {
    tasks[current]->execute();
});
```

---

After the loop finishes:

```cpp
for (auto& t : threads) {
    t.join();
}
```

---

#  What changed?

Before:

```text
Start ? wait ? Start ? wait
```

Now:

```text
Start ? Start ? Start ? wait all
```

 True parallelism

---

#  Step 3 — Run It

Now output may look like:

```text
Executing task: Load data
Executing task: Process data
Executing task: Save results
```

 Order may vary!

---

#  This is NORMAL

Threads run independently:

* CPU scheduling decides order
* Not your code

---

#  Step 4 — Fix Output Race Condition

You may see messy output like:

```text
Executing task: LoExecuting task: Process data
ad data
```

 That’s a **race condition**

---

#  Step 4 — Add Mutex

##  Include:

```cpp
#include <mutex>
```

---

##  Add global mutex (temporary):

```cpp
std::mutex coutMutex;
```

---

##  Protect output

In `Task::execute()`:

```cpp
extern std::mutex coutMutex;

void Task::execute() const {
    std::lock_guard<std::mutex> lock(coutMutex);

    std::cout << "Executing task: " << name << std::endl;

    if (action) {
        action();
    }
}
```

---

#  What is this?

```cpp
std::lock_guard<std::mutex>
```

 Locks automatically:

* Locks at start
* Unlocks when leaving scope

---

#  Why needed?

Multiple threads writing to:

```cpp
std::cout
```

 causes corruption without synchronization

---

#  Step 5 — Make Parallelism Visible

Modify your lambda:

```cpp
task->setAction([name = task->getName()]() {
    std::cout << "  -> Working on: " << name << std::endl;

    std::this_thread::sleep_for(std::chrono::milliseconds(1000));

    std::cout << "  -> Done: " << name << std::endl;
});
```

---

##  Expected behavior:

Instead of:

```text
Task1 ? Task2 ? Task3 (slow)
```

You get:

```text
Task1
Task2
Task3
(all running together)
```

 Much faster overall

---

#  Concepts You Just Learned

## 1. Threads

```cpp
std::thread
```

---

## 2. Parallel execution

Multiple tasks running simultaneously

---

## 3. Race conditions

Uncontrolled shared access

---

## 4. Mutex

```cpp
std::mutex
```

 Prevents conflicts

---

#  Common Mistakes

### ? Forgetting `join()`

```cpp
std::thread t(...);
// no join ?  crash
```

---

### ? Capturing incorrectly

```cpp
[this, &current] // ? dangerous
```

Correct:

```cpp
[this, current] // ?
```

---

### ? Over-locking

Lock only what's necessary

---

#  Exercises

##  1. Limit number of threads

Only allow e.g. 2 threads at a time

---

##  2. Print thread ID

```cpp
std::this_thread::get_id()
```

---

##  3. Remove mutex and observe chaos

 Helps you *feel* race conditions

---

#  Systems Insight

You now built:

* Parallel execution engine
* Basic synchronization system

 This is the foundation of:

* operating systems
* game engines
* high-performance servers

---

#  Week 8 — REAL Scheduler

Right now:

* You create threads per task (expensive)

Next:

##  Thread Pool

You will build:

```text
Worker Threads + Task Queue
```

 This is how real systems scale

---

## ? Checkpoint

Before moving on:

* ? Tasks run in parallel
* ? You understand mutex
* ? You saw race conditions
