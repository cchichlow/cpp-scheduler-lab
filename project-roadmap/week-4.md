Now we get into the part that really separates C++ from Python and even Fortran:

 **Memory ownership and performance control**

This is *the* core skill for systems programming in C++.

---

#  Week 4 — Memory, Ownership & Smart Pointers

##  Goal

Right now your system:

* Copies `Task` objects around
* Doesn’t explicitly model ownership

We will:

* Introduce **ownership semantics**
* Replace copies with **smart pointers**
* Make your scheduler more “systems-like”

---

#  The Big Idea

In C++ you must answer:

> ? *Who owns this object?*

---

## Current situation

```cpp
std::vector<Task> tasks;
```

 Problem:

* Tasks are copied into the vector
* Harder to control lifetime
* Not scalable for complex systems

---

## Target design

```cpp
std::vector<std::unique_ptr<Task>> tasks;
```

 Meaning:

* Scheduler **owns** all tasks
* No accidental copies
* Memory is freed automatically

---

#  Step 1 — Introduce `std::unique_ptr`

##  Update `Scheduler.h`

```cpp
#pragma once
#include <vector>
#include <unordered_map>
#include <memory>
#include "Task.h"

class Scheduler {
public:
    void addTask(std::unique_ptr<Task> task);
    void run();

private:
    std::vector<std::unique_ptr<Task>> tasks;
    std::unordered_map<std::string, int> taskIndex;
    std::vector<std::vector<int>> adjacencyList;
};
```

---

#  New Concept: `std::unique_ptr`

 A smart pointer that:

* Owns exactly one object
* Cannot be copied
* Can only be **moved**

---

#  Step 2 — Update `Scheduler.cpp`

##  Modify `addTask`

```cpp
void Scheduler::addTask(std::unique_ptr<Task> task) {
    taskIndex[task->getName()] = tasks.size();
    tasks.push_back(std::move(task));
}
```

---

#  Key Line

```cpp
std::move(task)
```

 Transfers ownership into the vector

After this:

* Original pointer is empty
* Vector owns the object

---

#  Step 3 — Update `run()` Access

Anywhere you had:

```cpp
tasks[i].getDependencies()
```

Change to:

```cpp
tasks[i]->getDependencies()
```

 Because now you're using pointers

---

#  Step 4 — Update `main.cpp`

Now we must **create tasks dynamically**

---

##  Replace scheduler population:

```cpp
Scheduler scheduler;

for (const auto& task : tasks) {
    scheduler.addTask(std::make_unique<Task>(task));
}
```

---

#  What’s happening here?

```cpp
std::make_unique<Task>(task)
```

 Creates a **heap-allocated Task**

---

#  Important Detail

This still copies `task` into the new object.

 That’s OK for now — we’ll optimize later.

---

#  Ownership Flow (Very Important)

```text
main() ? creates unique_ptr
        ?
Scheduler takes ownership
        ?
Scheduler destroys tasks automatically
```

 No manual `delete` needed
 No memory leaks

---

#  Step 5 — Prove It Works

Add a destructor to `Task`:

##  In `Task.h`

```cpp
~Task();
```

---

##  In `Task.cpp`

```cpp
Task::~Task() {
    std::cout << "Destroying task: " << name << std::endl;
}
```

---

### Expected output (end of program):

```text
Destroying task: Load data
Destroying task: Process data
Destroying task: Save results
```

 This proves:

* Memory is cleaned up automatically

---

#  Concepts You Just Learned

## 1. RAII (core C++ concept)

* Resource tied to object lifetime
* Destructor cleans up automatically

---

## 2. Ownership semantics

* `unique_ptr` = single owner
* Prevents bugs by design

---

## 3. Move semantics (intro)

```cpp
std::move(task)
```

 Transfers ownership instead of copying

---

#  Common Mistakes

### ? Forgetting `std::move`

```cpp
tasks.push_back(task); // ? won't compile
```

---

### ? Using pointer like object

```cpp
task.getName(); // ?
task->getName(); // ?
```

---

### ? Double ownership

Never do:

```cpp
std::unique_ptr<Task> a = ...;
std::unique_ptr<Task> b = a; // ?
```

---

#  Exercises (important!)

##  1. Remove unnecessary copies

Modify `loadTasks` to return:

```cpp
std::vector<std::unique_ptr<Task>>
```

---

##  2. Add logging

Print when tasks are:

* Created
* Executed
* Destroyed

---

##  3. Try breaking it

Remove `std::move` and observe compiler error

 This teaches ownership rules deeply

---

#  Systems Insight

You just implemented:

* Explicit ownership model
* Automatic memory management (without GC!)
* Zero-cost abstractions

 This is why C++ is used in:

* operating systems
* game engines
* high-performance systems

---

#  Week 5 Preview

Now we go deeper into performance:

##  Move Semantics + Efficiency

You’ll learn:

* Copy vs move constructors
* Avoiding expensive copies
* Writing efficient classes

 This is where C++ becomes *fast by design*

---

## ? Checkpoint

Before moving on:

* ? Code compiles with `unique_ptr`
* ? You understand `std::move`
* ? You see destructor being called
