# Week 1 — Setup + First Working Program

## Goal of Week 1

By the end, you will:

* Understand how a C++ project is structured in Visual Studio
* Write your first **multi-file C++ program**
* Build a **basic task runner** (hardcoded tasks for now)

---

# Step 1 — Create the Project (Visual Studio)

1. Open Visual Studio
2. Click **Create a new project**
3. Choose:
    **Console App (C++)**
4. Name it:

   ```
   TaskScheduler
   ```
5. Choose a location and create it

---

## Configure it properly (important!)

### Set C++ standard:

* Right click project **Properties**
* Go to:

  ```
  C/C++ ? Language ? C++ Language Standard
  ```
* Set to:

  ```
  ISO C++20 Standard (/std:c++20)
  ```

### Enable warnings:

* Go to:

  ```
  C/C++ ? General ? Warning Level
  ```
* Set:

  ```
  Level 4 (/W4)
  ```

---

# Step 2 — Create Project Structure

In **Solution Explorer**:

### Add folders:

* `core`
* `utils`

### Add files:

#### In `core/`

* `Task.h`
* `Task.cpp`

#### In root:

* `main.cpp` (already exists)

---

# Step 3 — Your First Class (Task)

## `Task.h`

```cpp
#pragma once
#include <string>

class Task {
public:
    Task(const std::string& name);

    void execute() const;
    std::string getName() const;

private:
    std::string name;
};
```

---

## `Task.cpp`

```cpp
#include "Task.h"
#include <iostream>

Task::Task(const std::string& name)
    : name(name) {}

void Task::execute() const {
    std::cout << "Executing task: " << name << std::endl;
}

std::string Task::getName() const {
    return name;
}
```

---

# What you just learned

* Header (`.h`) = interface
* Source (`.cpp`) = implementation
* `#pragma once` prevents double inclusion
* Constructor initialization list:

  ```cpp
  : name(name)
  ```

---

# Step 4 — Use It in `main.cpp`

Replace your `main.cpp` with:

```cpp
#include <iostream>
#include <vector>
#include "core/Task.h"

int main() {
    std::vector<Task> tasks;

    tasks.emplace_back("Load data");
    tasks.emplace_back("Process data");
    tasks.emplace_back("Save results");

    for (const auto& task : tasks) {
        task.execute();
    }

    return 0;
}
```

---

# Concepts Introduced Here

### 1. `std::vector`

Dynamic array (like Python list, but typed and efficient)

### 2. `emplace_back`

Constructs object **in-place** (important later for performance)

### 3. Range-based loop

```cpp
for (const auto& task : tasks)
```

* `auto` ? type inference
* `const &` ? no copy, read-only access

---

# Step 5 — Run It

You should see:

```
Executing task: Load data
Executing task: Process data
Executing task: Save results
```

---

# Common Beginner Pitfalls (you might hit these)

### “Cannot open source file Task.h”

Fix:

* Make sure include path is correct:

```cpp
#include "core/Task.h"
```

---

### Linker errors

Usually means:

* `.cpp` file not added to project
  ? Right-click project ? Add ? Existing Item

---

# Mini Exercise (important)

Modify your program:

### Add:

* A function:

```cpp
void runTasks(const std::vector<Task>& tasks);
```

Move the loop into that function.

This reinforces:

* Passing by reference
* Separation of logic

---

# What You Achieved

You already:

* Built a **modular C++ program**
* Used **classes + STL**
* Understood **project structure**
