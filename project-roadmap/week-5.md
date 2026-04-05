Now we get to one of the *most important* (and most “C++-specific”) topics:

 **Move semantics & performance**

This is where C++ stops being just “manual memory” and becomes a **high-performance language you control precisely**.

---

#  Week 5 — Move Semantics & Efficient Objects

##  Goal

Understand and control:

* When objects are **copied**
* When they are **moved**
* How to make your code **fast and safe**

---

#  The Core Problem

Right now, this happens:

```cpp
std::make_unique<Task>(task);
```

 This **copies** `Task`

If `Task` becomes large (which it will later), this is:

* slow
* unnecessary

---

#  Key Idea

## Copy vs Move

### Copy:

```cpp
Task b = a;
```

* duplicates data
* expensive

---

### Move:

```cpp
Task b = std::move(a);
```

* transfers resources
* leaves `a` in a valid but empty state
* very fast

---

#  Step 1 — Instrument Your Class

We want to *see* what’s happening.

---

##  Update `Task.h`

```cpp
Task(const Task& other);            // copy constructor
Task(Task&& other) noexcept;        // move constructor
```

---

##  Update `Task.cpp`

```cpp
Task::Task(const Task& other)
    : name(other.name), dependencies(other.dependencies) {
    std::cout << "Copying task: " << name << std::endl;
}

Task::Task(Task&& other) noexcept
    : name(std::move(other.name)),
      dependencies(std::move(other.dependencies)) {
    std::cout << "Moving task: " << name << std::endl;
}
```

---

#  What This Does

Now when your program runs, you’ll see:

```text
Copying task: Load data
Moving task: Process data
```

 This gives you **visibility into performance**

---

#  Step 2 — Enable Move Instead of Copy

Currently:

```cpp
for (const auto& task : tasks)
```

 This prevents moving (because it's `const`)

---

##  Change to:

```cpp
for (auto& task : tasks)
```

---

##  Then update:

```cpp
scheduler.addTask(std::make_unique<Task>(std::move(task)));
```

---

#  What Just Happened

```cpp
std::move(task)
```

 Converts `task` into a **movable object**

? Your move constructor is used instead of copy

---

#  Step 3 — Observe the Output

Before:

```text
Copying task: Load data
Copying task: Process data
```

After:

```text
Moving task: Load data
Moving task: Process data
```

 This is a *real performance improvement*

---

#  Important Rule

> **Only move objects you won’t use again**

After:

```cpp
std::move(task)
```

 `task` is now in a **valid but unspecified state**

DO NOT:

```cpp
task.getName(); // ? unsafe logic
```

---

#  Step 4 — Improve `loadTasks`

Now let’s remove unnecessary copies entirely.

---

##  Change return type:

```cpp
std::vector<std::unique_ptr<Task>> loadTasks(...)
```

---

##  Inside:

```cpp
tasks.push_back(std::make_unique<Task>(line));
```

---

##  Then in `main.cpp`:

```cpp
auto tasks = loadTasks("tasks.txt");

Scheduler scheduler;

for (auto& task : tasks) {
    scheduler.addTask(std::move(task));
}
```

---

#  Result

* No copying
* Only moving pointers
* Very efficient

---

#  Big Concept: “Zero-Cost Abstraction”

Your system now:

* Uses dynamic memory
* Has ownership safety
* Avoids unnecessary copies

 AND runs fast

---

#  Common Mistakes

### ? Moving twice

```cpp
scheduler.addTask(std::move(task));
scheduler.addTask(std::move(task)); // ?
```

---

### ? Using moved object

```cpp
auto t2 = std::move(t1);
t1.getName(); // ?
```

---

### ? Forgetting `noexcept`

```cpp
Task(Task&& other) noexcept
```

 Important for STL optimizations

---

#  Exercises (important)

##  1. Add move assignment operator

```cpp
Task& operator=(Task&& other) noexcept;
```

---

##  2. Count copies vs moves

Add counters:

```cpp
static int copyCount;
static int moveCount;
```

---

##  3. Stress test

Create 10,000 tasks and observe:

* Copy vs move behavior
* Performance difference

---

#  Systems Insight

You now understand:

* How memory is transferred, not duplicated
* How STL containers optimize operations
* Why C++ can outperform higher-level languages

 This is used everywhere in:

* game engines
* compilers
* trading systems

---

#  Week 6 Preview

Now we unlock **Modern C++ expressiveness**:

##  Lambdas & Functional Patterns

You’ll learn:

* Lambda functions
* Passing behavior as data
* Custom execution logic per task

 This makes your scheduler **flexible and extensible**

---

## ? Checkpoint

Before moving on:

* ? You see move vs copy output
* ? You understand `std::move`
* ? Your system avoids unnecessary copies

