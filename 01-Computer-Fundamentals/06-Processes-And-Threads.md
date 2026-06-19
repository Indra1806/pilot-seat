> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
Processes and Threads are the fundamental units of execution in a computer system. They are how the Operating System tracks and manages running software.

# Why It Exists
In the earliest days, a computer ran exactly one program from start to finish. If that program got stuck waiting for the printer, the entire computer froze. Engineers needed a way for computers to "multitask." By splitting execution into "Processes" (isolated programs) and "Threads" (tasks within a program), the CPU can rapidly switch between them, making it feel like everything is happening at once.

# Problem It Solves
It solves the problem of idle time and system responsiveness.

**Before:**
While saving a massive file, your entire computer would freeze until the save was complete.

**After:**
The "Save" task runs on a background Thread, while the main Thread keeps the UI responsive so you can keep typing.

# Core Concepts
1. **Process:** A heavyweight, fully isolated program running in memory. It has its own dedicated memory space. (e.g., Google Chrome as a whole).
2. **Thread:** A lightweight "mini-process" that lives *inside* a Process. All threads within a process share the same memory. (e.g., Each open tab in Chrome).
3. **Context Switching:** The rapid, invisible act of the CPU pausing one thread, saving its state, and jumping to another thread.
4. **Concurrency vs Parallelism:** Concurrency is the *illusion* of doing two things at once by switching very fast. Parallelism is *actually* doing two things at once using multiple CPU cores.

# Architecture / Components
```text
+---------------------------------------------------+
|                     PROCESS                       |
|  (Has its own dedicated RAM and File Handles)     |
|                                                   |
|   +-----------+   +-----------+   +-----------+   |
|   | Thread 1  |   | Thread 2  |   | Thread 3  |   |
|   | (UI)      |   | (Network) |   | (Save Disk) |   |
|   +-----------+   +-----------+   +-----------+   |
+---------------------------------------------------+
```

# Workflow
1. You launch Spotify (Process starts).
2. Spotify creates **Thread 1** to display the user interface.
3. You click "Play" on a song.
4. Spotify creates **Thread 2** to stream the audio from the internet.
5. **Thread 1** and **Thread 2** run concurrently. If Thread 2 buffers (waits for network), Thread 1 still works, so the app doesn't freeze.

# Real World Examples
Think of a Process like a **Restaurant**, and Threads like the **Workers inside**:
* The **Process (Restaurant)** has its own building, ingredients, and money (Memory Space).
* The **Threads (Workers)** include a Chef, a Waiter, and a Host.
* They all share the same ingredients and kitchen (Shared Memory). If the Chef drops a cake, it affects the Waiter. 
* Another Process would be the Bank next door. The Bank workers cannot touch the Restaurant's kitchen.

# Implementation
Developers explicitly create threads to keep applications fast. In Python, you might use the `threading` module. In JavaScript, which is famously "single-threaded", developers use asynchronous callbacks or Web Workers to achieve the same non-blocking effect.

# Best Practices
* **Never block the Main Thread:** The Main Thread handles the User Interface. If you put a heavy calculation on it, the app will say "(Not Responding)".
* **Thread Safety:** Because threads share memory, if two threads try to change the same variable at the exact same millisecond, the data corrupts. Use "Locks" or "Mutexes" to prevent this.

# Industry Standards
Modern servers (like Nginx) and databases handle thousands of simultaneous users by assigning a lightweight Thread (or an asynchronous event loop) to every single incoming connection.

# Common Mistakes
* **Race Conditions:** Forgetting to lock shared memory. Thread A reads $10, Thread B reads $10. Both add $5 and save $15. The final total should be $20, but because they "raced", it's $15. 
* **Too Many Threads:** Creating 10,000 threads will crash your app. Context Switching takes CPU power; if you have too many threads, the CPU spends 100% of its time just switching between them and 0% doing actual work (Thrashing).

# Security & Performance Considerations
* **Deadlocks:** If Thread A locks Resource X and waits for Resource Y, and Thread B locks Resource Y and waits for Resource X, they wait forever. The application freezes permanently.
* **Process Isolation:** OS security relies on Processes being unable to read each other's memory.

# Related Technologies
* 03. Operating Systems
* 04. Memory Management
* Asynchronous Programming

# Summary
## What We Learned
- A Process is a running program with isolated memory.
- A Thread is a lightweight worker inside a Process sharing that memory.
- The OS switches between them rapidly to create the illusion of multitasking.

## Key Takeaways
- Offload heavy tasks to background threads to keep applications responsive.
- Be extremely careful when multiple threads access the same data.

# Keywords
- Process
- Thread
- Context Switch
- Concurrency
- Deadlock
- Race Condition

# Glossary
| Term | Meaning |
|---|---|
| Process | An isolated instance of a running program. |
| Thread | A path of execution within a process. |
| Context Switch | The CPU pausing one task to work on another. |
| Deadlock | When two or more threads are permanently blocked waiting for each other. |

## Next Recommended Chapters
- 07. File Systems
