Perfect — this is where your system becomes **flexible and “real-world usable”** instead of rigid.

Take it step by step — this week is conceptually important but not too long.

---

#  Week 6 — Lambdas & Flexible Task Behavior

##  Goal

Right now your `Task` always does:

```cpp
task.execute(); ? prints name
```

 That’s **hardcoded behavior**

---

##  After this week:

Each task can have **custom logic**, like:

* print something
* simulate work
* run calculations

 This is how real systems work.

---

#  Step 1 — Add Behavior to `Task`

We give each task a **function it can execute**.

---

##  Update `Task.h`

Add:

```cpp
#include <functional>
```

Then modify the class:

```cpp
std::function<void()> action;
```

Full version:

```cpp
#pragma once
#include <string>
#include <vector>
#include <functional>

class Task {
public:
    Task(const std::string& name);

    void addDependency(const std::string& dependency);
    const std::vector<std::string>& getDependencies() const;

    void setAction(std::function<void()> action);
    void execute() const;

    std::string getName() const;

private:
    std::string name;
    std::vector<std::string> dependencies;
    std::function<void()> action;
};
```

---

#  What is `std::function`?

 A wrapper for “anything callable”

It can store:

* normal functions
* lambdas
* function objects

---

#  Step 2 — Implement It

##  Update `Task.cpp`

Add:

```cpp
void Task::setAction(std::function<void()> actionFunc) {
    action = actionFunc;
}
```

---

##  Modify `execute()`

Replace:

```cpp
std::cout << "Executing task: " << name << std::endl;
```

With:

```cpp
std::cout << "Executing task: " << name << std::endl;

if (action) {
    action();
}
```

---

#  Important

```cpp
if (action)
```

 Checks if a function is assigned
(prevents crashes)

---

#  Step 3 — Use Lambdas in `main.cpp`

Now we assign behavior dynamically.

---

##  Modify task creation

Where you previously did:

```cpp
scheduler.addTask(std::move(task));
```

Change to:

```cpp
task->setAction([name = task->getName()]() {
    std::cout << "  -> Running logic for: " << name << std::endl;
});

scheduler.addTask(std::move(task));
```

---

#  What is this?

```cpp
[name = task->getName()]
```

 This is a **lambda capture**

We store the name inside the lambda so it remains valid after move.

---

#  Step 4 — Run It

Expected output:

```text
Executing task: Load data
  -> Running logic for: Load data
Executing task: Process data
  -> Running logic for: Process data
...
```

---

#  You Just Learned

## 1. Lambdas

```cpp
[]() { ... }
```

---

## 2. Captures

```cpp
[name = task->getName()]
```

 Stores variables inside the lambda

---

## 3. Behavior as data

You are now passing **functions like variables**

 This is HUGE in modern C++

---

#  Step 5 — Make It Interesting

Now let’s simulate real work.

---

##  Add delay

Include:

```cpp
#include <thread>
#include <chrono>
```

---

##  Modify lambda:

```cpp
task->setAction([name = task->getName()]() {
    std::cout << "  -> Working on: " << name << std::endl;
    std::this_thread::sleep_for(std::chrono::milliseconds(500));
    std::cout << "  -> Done: " << name << std::endl;
});
```

---

#  Output:

```text
Executing task: Load data
  -> Working on: Load data
  -> Done: Load data
```

---

#  Why This Matters

You now have:

* execution logic
* timing simulation
* extensible behavior

 This is basically a **job system**

---

#  Common Mistakes

### ? Capturing by reference (dangerous here)

```cpp
[&name] // ? can break after move
```

---

### ? Forgetting capture entirely

```cpp
[]() { std::cout << name; } // ? won't compile
```

---

### ? Using moved object

```cpp
std::move(task);
task->getName(); // ? unsafe
```

---

#  Exercises (important)

##  1. Different behavior per task

Example:

* "Load" ? prints loading message
* "Process" ? simulate computation
* "Save" ? print saving message

---

##  2. Add random duration

Use:

```cpp
rand() % 1000
```

---

##  3. Add execution time logging

Measure start/end time

---

#  Systems Insight

You just implemented:

* Function-as-data model
* Dynamic execution behavior
* Extensible architecture

 This is used in:

* task schedulers
* game engines
* async frameworks

---

#  Next Step (Week 7)

Now we go **parallel**:

##  Multi-threaded execution

You will:

* run tasks concurrently
* deal with synchronization
* face real-world problems (race conditions!)

---

## ? Checkpoint

Before moving on:

* ? Lambdas work
* ? You understand captures
* ? Tasks run custom logic
