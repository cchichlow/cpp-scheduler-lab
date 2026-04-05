Perfect — now we refine what you built into something **clean, efficient, and closer to real-world systems**.

Week 2 worked, but it had a limitation:

* It repeatedly scanned all tasks ? inefficient
* Logic was harder to reason about

Now we fix that properly.

---

#  Week 3 — Proper Topological Sorting (Kahn’s Algorithm)

##  Goal

Replace your current “trial-and-error loop” with a **real algorithm**:

 Deterministic
 Efficient
 Scales well

---

#  Core Idea (Important)

We transform the problem:

Instead of asking:

> “Is this task ready?”

We track:

> “How many dependencies are still unmet?”

---

## Concept:

Each task has:

```text
in-degree = number of dependencies
```

### Example:

```text
Load data        (0)
Process data     (1)
Save results     (1)
```

---

## Algorithm Flow

1. Find all tasks with **0 dependencies**
2. Execute them
3. Reduce dependency count of dependent tasks
4. Repeat

---

#  Step 1 — Update Scheduler Data Model

We need to track relationships explicitly.

---

##  Update `Scheduler.h`

```cpp
#pragma once
#include <vector>
#include <unordered_map>
#include <queue>
#include "Task.h"

class Scheduler {
public:
    void addTask(const Task& task);
    void run();

private:
    std::vector<Task> tasks;
    std::unordered_map<std::string, int> taskIndex;

    // Graph representation
    std::vector<std::vector<int>> adjacencyList;
};
```

---

#  Step 2 — Build the Graph

We now explicitly build connections between tasks.

---

##  Update `Scheduler.cpp`

### Add this helper inside `run()` (or separate later):

```cpp
void Scheduler::run() {
    size_t n = tasks.size();

    adjacencyList.clear();
    adjacencyList.resize(n);

    std::vector<int> inDegree(n, 0);

    // Build graph
    for (size_t i = 0; i < n; ++i) {
        for (const auto& dep : tasks[i].getDependencies()) {

            if (taskIndex.find(dep) == taskIndex.end()) {
                std::cerr << "Error: Unknown dependency " << dep << std::endl;
                return;
            }

            int depIndex = taskIndex[dep];

            adjacencyList[depIndex].push_back(i);
            inDegree[i]++;
        }
    }
```

---

#  What This Does

```text
A depends_on B
```

Becomes:

```text
B ? A
```

 Because:

* B must run before A

---

#  Step 3 — Kahn’s Algorithm

Continue inside `run()`:

```cpp
    std::queue<int> readyQueue;

    // Step 1: find tasks with no dependencies
    for (size_t i = 0; i < n; ++i) {
        if (inDegree[i] == 0) {
            readyQueue.push(i);
        }
    }

    size_t executedCount = 0;

    while (!readyQueue.empty()) {
        int current = readyQueue.front();
        readyQueue.pop();

        tasks[current].execute();
        executedCount++;

        // Reduce dependencies of neighbors
        for (int neighbor : adjacencyList[current]) {
            inDegree[neighbor]--;

            if (inDegree[neighbor] == 0) {
                readyQueue.push(neighbor);
            }
        }
    }

    if (executedCount != n) {
        std::cerr << "Error: Circular dependency detected!" << std::endl;
    }
}
```

---

#  Why This Is Better

### Old approach:

* O(n²) worst case
* Hard to reason about

### New approach:

* O(n + edges)
* Clean graph logic
* Industry standard

---

#  Example Execution

Input:

```text
Load data
Process data depends_on Load data
Save results depends_on Process data
```

Execution flow:

```text
Queue: [Load]
? Execute Load
? Unlock Process
? Execute Process
? Unlock Save
? Execute Save
```

---

#  Concepts You Just Learned

## 1. Graph Representation

```cpp
std::vector<std::vector<int>> adjacencyList;
```

---

## 2. In-degree tracking

```cpp
std::vector<int> inDegree;
```

---

## 3. Queue-based processing

```cpp
std::queue<int>
```

 Very common in:

* schedulers
* compilers
* OS task systems

---

#  Exercises (do at least one)

##  1. Print execution order explicitly

Add:

```text
Execution order:
1 ? Load data
2 ? Process data
3 ? Save results
```

---

##  2. Detect isolated tasks

Warn if:

* Task has no dependencies
* AND no one depends on it

---

##  3. Improve error message

Instead of:

```text
Circular dependency detected!
```

Try:

```text
Cycle detected involving remaining tasks
```

---

#  Common Pitfalls

### ? Forgetting to resize adjacency list

```cpp
adjacencyList.resize(n);
```

---

### ? Using wrong direction of edge

Wrong:

```text
A ? B (incorrect)
```

Correct:

```text
B ? A
```

---

### ? Modifying container while iterating

Avoid changing `tasks` during execution

---

#  Systems Insight (Important)

You now have:

* A **deterministic scheduler**
* A **graph execution engine**
* A **cycle detection mechanism**

 This is basically a simplified version of:

* build systems (like Make, Ninja)
* workflow engines (Airflow, etc.)

---

#  Week 4 Preview

Now we level up into **real C++ power**:

##  Memory + Ownership

We will:

* Replace copies with **smart pointers**
* Introduce:

  * `std::unique_ptr`
  * move semantics
* Avoid unnecessary copies

 This is where C++ becomes fundamentally different from Python.

---

## ? Checkpoint

Before moving on:

* ? Program runs with dependencies
* ? You understand in-degree concept
* ? You understand WHY this is better than Week 2

