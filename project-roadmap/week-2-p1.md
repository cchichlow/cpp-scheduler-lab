Great — now we move from “script-like program” to **actual system design**. This is where C++ starts to feel powerful.

---

#  Week 2 — Building the Scheduler Core

##  Goal

Turn your program into a structured system:

Instead of:

```cpp
for (...) task.execute();
```

You’ll build:

```cpp
Scheduler scheduler;
scheduler.addTask(...);
scheduler.run();
```

 This introduces **abstraction, ownership, and architecture**.

---

#  Step 1 — Create `Scheduler` Class

##  Add files in `core/`:

* `Scheduler.h`
* `Scheduler.cpp`

---

##  `Scheduler.h`

```cpp
#pragma once
#include <vector>
#include "Task.h"

class Scheduler {
public:
    void addTask(const Task& task);
    void run() const;

private:
    std::vector<Task> tasks;
};
```

---

##  `Scheduler.cpp`

```cpp
#include "Scheduler.h"
#include <iostream>

void Scheduler::addTask(const Task& task) {
    tasks.push_back(task);
}

void Scheduler::run() const {
    std::cout << "Running " << tasks.size() << " tasks...\n";

    int index = 1;
    for (const auto& task : tasks) {
        std::cout << "[" << index++ << "] ";
        task.execute();
    }
}
```

---

#  What’s Important Here

### 1. Encapsulation

* `tasks` is **private**
* Only `Scheduler` controls execution

 This is *core systems design thinking*

---

### 2. Passing by `const&`

```cpp
void addTask(const Task& task);
```

* Avoids copying unnecessarily
* Very important in C++

---

#  Step 2 — Refactor `main.cpp`

Now we **use the scheduler instead of raw loops**:

```cpp
#include <iostream>
#include <vector>
#include <fstream>
#include <string>
#include "core/Task.h"
#include "core/Scheduler.h"

std::vector<Task> loadTasks(const std::string& filename) {
    std::vector<Task> tasks;
    std::ifstream file(filename);

    if (!file) {
        std::cerr << "Error: Could not open file " << filename << std::endl;
        return tasks;
    }

    std::string line;
    while (std::getline(file, line)) {
        if (!line.empty() && line[0] != '#') {
            tasks.emplace_back(line);
        }
    }

    return tasks;
}

int main() {
    auto tasks = loadTasks("tasks.txt");

    Scheduler scheduler;

    for (const auto& task : tasks) {
        scheduler.addTask(task);
    }

    scheduler.run();

    return 0;
}
```

---

#  Step 3 — Run It

Output should now look like:

```text
Running 3 tasks...
[1] Executing task: Load data
[2] Executing task: Process data
[3] Executing task: Save results
```

---

#  Key Concepts You Just Learned

## 1. Separation of Concerns

* `main.cpp` ? orchestration
* `Scheduler` ? logic
* `Task` ? data + behavior

 This is how real systems are structured.

---

## 2. Data Ownership (important!)

Right now:

* `Scheduler` **owns** the tasks (copy stored in vector)

Later we’ll evolve this to:

* pointers / smart pointers
* shared ownership vs unique ownership

---

## 3. Interface Design

```cpp
scheduler.addTask(...)
scheduler.run()
```

 Clean, simple API — this matters a lot in C++

---

#  Exercises (do at least one)

##  Exercise 1 — Add `clear()`

Add:

```cpp
void clear();
```

? Removes all tasks

---

##  Exercise 2 — Count tasks

Add:

```cpp
size_t getTaskCount() const;
```

---

##  Exercise 3 — Prevent empty tasks

In `addTask`, ignore tasks with empty names

---

#  Common Mistakes

### ? Including `.cpp` instead of `.h`

Wrong:

```cpp
#include "Task.cpp"
```

Correct:

```cpp
#include "Task.h"
```

---

### ? Linker errors (again)

If you see:

```
unresolved external symbol
```

 Usually means:

* `Scheduler.cpp` not added to project

---

#  Systems Insight (important)

You just built a **basic runtime system loop**.

Real systems like:

* job schedulers
* build systems
* operating systems

 all follow this pattern:

1. Load tasks
2. Store them
3. Execute via a controller

---

#  Week 2 — Next Step (Very Important)

We now make it *interesting*:

##  Add **task dependencies**

Example:

```text
Load data
Process data depends_on Load data
Save results depends_on Process data
```

This introduces:

* Graphs
* Topological sorting
* Real algorithmic thinking

---

## ? Before moving on

Make sure:

* ? Scheduler works
* ? You understand `.h` vs `.cpp`
* ? You ran the program successfully
