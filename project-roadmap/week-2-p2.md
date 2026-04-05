Excellent — now we hit the **first real algorithmic milestone**.
This is where your project stops being “just structured code” and becomes a **systems + algorithms hybrid**.

---

#  Week 2 — Part 2: Task Dependencies (Graph + Ordering)

##  Goal

Allow tasks to depend on other tasks:

Example:

```text
Load data
Process data depends_on Load data
Save results depends_on Process data
```

 Your scheduler must:

* Respect dependencies
* Execute in the **correct order**
* Detect invalid setups (cycles later)

---

#  Core Idea

This is a **graph problem**:

* Each task = a node
* Dependency = directed edge

```
Load ? Process ? Save
```

 We must compute a valid execution order
? This is called **topological sorting**

---

#  Step 1 — Extend `Task`

We need to store dependencies.

##  Update `Task.h`

```cpp
#pragma once
#include <string>
#include <vector>

class Task {
public:
    Task(const std::string& name);

    void addDependency(const std::string& dependency);
    const std::vector<std::string>& getDependencies() const;

    void execute() const;
    std::string getName() const;

private:
    std::string name;
    std::vector<std::string> dependencies;
};
```

---

##  Update `Task.cpp`

```cpp
#include "Task.h"
#include <iostream>

Task::Task(const std::string& name)
    : name(name) {}

void Task::addDependency(const std::string& dependency) {
    dependencies.push_back(dependency);
}

const std::vector<std::string>& Task::getDependencies() const {
    return dependencies;
}

void Task::execute() const {
    std::cout << "Executing task: " << name << std::endl;
}

std::string Task::getName() const {
    return name;
}
```

---

#  Step 2 — Parse Dependencies from File

Update your input format:

```text
Load data
Process data depends_on Load data
Save results depends_on Process data
```

---

##  Update `loadTasks`

Replace parsing logic in `main.cpp`:

```cpp
std::vector<Task> loadTasks(const std::string& filename) {
    std::vector<Task> tasks;
    std::ifstream file(filename);

    if (!file) {
        std::cerr << "Error: Could not open file " << filename << std::endl;
        return tasks;
    }

    std::string line;
    while (std::getline(file, line)) {
        if (line.empty() || line[0] == '#') continue;

        size_t pos = line.find("depends_on");

        if (pos == std::string::npos) {
            tasks.emplace_back(line);
        } else {
            std::string name = line.substr(0, pos - 1);
            std::string dependency = line.substr(pos + 11);

            Task task(name);
            task.addDependency(dependency);
            tasks.push_back(task);
        }
    }

    return tasks;
}
```

---

#  Important Limitation (for now)

Right now:

* Dependencies are stored as **strings**
* We don’t yet “connect” them to actual tasks

 That’s the next step.

---

#  Step 3 — Prepare Scheduler for Lookup

We need fast lookup by task name.

##  Update `Scheduler.h`

```cpp
#pragma once
#include <vector>
#include <unordered_map>
#include "Task.h"

class Scheduler {
public:
    void addTask(const Task& task);
    void run();

private:
    std::vector<Task> tasks;
    std::unordered_map<std::string, int> taskIndex;
};
```

---

##  Update `Scheduler.cpp`

```cpp
#include "Scheduler.h"
#include <iostream>

void Scheduler::addTask(const Task& task) {
    taskIndex[task.getName()] = tasks.size();
    tasks.push_back(task);
}
```

---

#  Why This Matters

```cpp
taskIndex[name] ? index in vector
```

 This is a classic systems trick:

* Combine **fast array storage**
* With **fast lookup (hash map)**

---

#  Step 4 — Simple Dependency Handling (First Version)

We won’t implement full topological sort yet.

Instead:
 Only run tasks when dependencies are satisfied

---

##  Replace `run()` with:

```cpp
void Scheduler::run() {
    std::vector<bool> executed(tasks.size(), false);

    size_t completed = 0;

    while (completed < tasks.size()) {
        bool progress = false;

        for (size_t i = 0; i < tasks.size(); ++i) {
            if (executed[i]) continue;

            const auto& task = tasks[i];
            bool ready = true;

            for (const auto& dep : task.getDependencies()) {
                if (taskIndex.find(dep) == taskIndex.end()) {
                    std::cerr << "Error: Unknown dependency " << dep << std::endl;
                    return;
                }

                if (!executed[taskIndex[dep]]) {
                    ready = false;
                    break;
                }
            }

            if (ready) {
                task.execute();
                executed[i] = true;
                completed++;
                progress = true;
            }
        }

        if (!progress) {
            std::cerr << "Error: Circular dependency detected!" << std::endl;
            return;
        }
    }
}
```

---

#  What You Just Built

This is essentially:

* A **dependency resolver**
* A **basic scheduler**
* A **cycle detector**

 This is already similar to:

* build systems (like `make`)
* job scheders
* workflow engines

---

#  Example Output

Input:

```text
Load data
Process data depends_on Load data
Save results depends_on Process data
```

Output:

```text
Executing task: Load data
Executing task: Process data
Executing task: Save results
```

---

#  Exercises (important!)

##  1. Add multiple dependencies

Support:

```text
TaskC depends_on TaskA,TaskB
```

---

##  2. Better error messages

Show:

```text
Task "X" depends on missing task "Y"
```

---

##  3. Detect duplicate tasks

Prevent adding same task twice

---

#  Systems Insight

You just implemented:

* Dependency resolution loop
* Execution scheduling
* Deadlock detection (cycle case)

 This is *real systems programming logic*

---

#  Next Step (Week 3 Preview)

We will upgrade this to:

##  Proper Topological Sorting

* More efficient
* Cleaner logic
* Scales better

AND introduce:

* Better data modeling
* Separation of parsing vs execution

---

## ? Checkpoint

Before moving on:

* ? Dependencies work
* ? You understand the loop logic
* ? You tested a failure case (cycle or missing dep)
