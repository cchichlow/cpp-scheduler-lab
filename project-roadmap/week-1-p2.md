Perfect — now we turn your program from a “demo” into something that actually behaves like a real system component.

---

# Week 1 — Part 2: Reading Tasks from a File

## Goal

Instead of hardcoding tasks, your program will:

* Read tasks from a file
* Create `Task` objects dynamically
* Execute them

 This introduces **file I/O + basic parsing**, which is essential for systems programming.

---

#  Step 1 — Create an Input File

In your project directory (same level as `.vcxproj`), create:

```
tasks.txt
```

### Example content:

```
Load data
Process data
Save results
```

---

#  Step 2 — Add File Reading Logic

Modify `main.cpp`:

```cpp
#include <iostream>
#include <vector>
#include <fstream>
#include <string>
#include "core/Task.h"

std::vector<Task> loadTasks(const std::string& filename) {
    std::vector<Task> tasks;
    std::ifstream file(filename);

    if (!file) {
        std::cerr << "Error: Could not open file " << filename << std::endl;
        return tasks;
    }

    std::string line;
    while (std::getline(file, line)) {
        if (!line.empty()) {
            tasks.emplace_back(line);
        }
    }

    return tasks;
}

int main() {
    std::vector<Task> tasks = loadTasks("tasks.txt");

    if (tasks.empty()) {
        std::cout << "No tasks found." << std::endl;
        return 1;
    }

    for (const auto& task : tasks) {
        task.execute();
    }

    return 0;
}
```

---

#  Concepts You Just Learned

## 1. File Input (`std::ifstream`)

```cpp
std::ifstream file(filename);
```

* Opens a file for reading
* Automatically closes when it goes out of scope (**RAII**)

---

## 2. Error Handling (basic but important)

```cpp
if (!file)
```

* Checks if file opened successfully

---

## 3. Line-by-line reading

```cpp
std::getline(file, line)
```

---

## 4. Returning containers safely

```cpp
return tasks;
```

 In C++ this is efficient (thanks to move semantics / RVO)

---

#  Step 3 — Run It

### Important (Visual Studio quirk!)

Your program runs in:

```
ProjectFolder/x64/Debug/
```

 So either:

* Put `tasks.txt` there
  **OR**
* Change working directory:

  * Project ? Properties ? Debugging ? Working Directory
    ? Set to:

  ```
  $(ProjectDir)
  ```

---

#  Common Issues

### ? File not found

You’ll see:

```
Error: Could not open file tasks.txt
```

Fix:

* Check working directory
* Or use full path temporarily

---

### ? Empty lines causing weird behavior

We handled this with:

```cpp
if (!line.empty())
```

---

#  Mini Exercise (very important)

Modify the system:

### 1. Add numbering to tasks

Output should be:

```
[1] Load data
[2] Process data
[3] Save results
```

 Hint:

* Use a counter in the loop

---

### 2. Ignore comment lines

Update file:

```
# This is a comment
Load data
Process data
# Another comment
Save results
```

 Skip lines starting with `#`

---

#  Why This Matters (Systems Thinking)

You just implemented:

* Input parsing
* Error handling
* Resource management (RAII)

 This is exactly how real tools (compilers, schedulers, etc.) start.

---

#  Next Step (Week 2 Preview)

We will transform this into a **real system component**:

### You’ll build:

 A `Scheduler` class

Instead of:

```cpp
for (...) task.execute();
```

You’ll have:

```cpp
Scheduler scheduler;
scheduler.addTask(...);
scheduler.run();
```

---

### New concepts coming:

* Class design
* Separation of concerns
* Header/source architecture at scale

---

## ? Checkpoint

Before moving on, make sure:

* ? File loading works
* ? You understand each part
* ? You did at least one mini exercise
