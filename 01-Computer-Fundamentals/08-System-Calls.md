> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
A System Call (syscall) is the programmatic way in which a computer program requests a service from the kernel of the operating system. It is the fundamental bridge between user applications and the actual hardware.

# Why It Exists
Operating Systems are divided into two distinct zones: **User Mode** and **Kernel Mode**. User Mode is an unprivileged sandbox where normal applications run. Kernel Mode has absolute, unrestricted access to the hardware. If a rogue application in User Mode tries to directly access the hard drive, the OS blocks it to prevent crashes or security breaches. System Calls were invented as the official, secure doorway for an application to politely ask the Kernel to do something dangerous on its behalf.

# Problem It Solves
It solves the problem of security and stability.

**Before:**
Any program could write data anywhere on the hard drive, easily overwriting the Operating System itself and destroying the computer.

**After:**
Programs run in a secure sandbox. If they want to write to the hard drive, they must make a System Call. The Kernel intercepts the call, checks if the program has permission, and safely performs the action.

# Core Concepts
1. **User Mode:** The safe, restricted environment where your web browser, games, and code run.
2. **Kernel Mode:** The highly privileged environment where the core OS runs.
3. **The Trap (Context Switch):** The mechanism of pausing the program, crossing the boundary from User Mode to Kernel Mode, and handing control to the OS.
4. **APIs vs Syscalls:** You rarely write raw syscalls. You use APIs (like standard libraries in Python/C) which wrap the ugly syscalls in friendly code.

# Architecture / Components
```text
+---------------------------------------------------+
|                   USER MODE                       |
|                                                   |
|  Your JavaScript App:   fs.readFileSync()         |
|                             ↓                     |
|  Node.js Library:       calls C++ wrapper         |
+-----------------------------|---------------------+
                              |  <-- THE SYSCALL BOUNDARY
+-----------------------------|---------------------+
|                  KERNEL MODE                      |
|                             ↓                     |
|  OS Kernel:          executes sys_read()          |
|                             ↓                     |
|  Hardware:           Reads physical Hard Drive    |
+---------------------------------------------------+
```

# Workflow
1. A Python script wants to print "Hello" to the screen.
2. The script calls the Python `print("Hello")` function.
3. The Python interpreter translates this into a System Call (e.g., `write()` in Linux).
4. The CPU triggers a "Trap", pausing the Python script and switching to Kernel Mode.
5. The OS checks permissions and executes the `write()` command on the physical monitor.
6. The OS returns control to the Python script, switching back to User Mode.

# Real World Examples
Think of a System Call like interacting with a **Bank Teller**:
* **User Mode:** You standing in the lobby. You cannot walk into the vault.
* **Hardware:** The Vault full of money.
* **The Kernel:** The Bank Teller behind the bulletproof glass.
* **System Call:** You passing a withdrawal slip (request) under the glass. The teller checks your ID (permissions), goes into the vault, gets the money, and hands it back to you.

# Implementation
You almost never write raw syscalls unless you are writing an operating system or low-level C code. 
Every time your code opens a file, opens a network socket, creates a thread, or even gets the current time, it is silently making a System Call under the hood.

# Best Practices
* **Batch operations:** Crossing the boundary from User Mode to Kernel Mode is computationally expensive (slow). Reading a file 1 byte at a time requires 1,000 syscalls for a 1KB file. Reading the whole 1KB chunk at once requires 1 syscall. Always buffer/batch your data.

# Industry Standards
* **POSIX:** A standard that defines a common set of System Calls across Unix-like operating systems (Linux, macOS). This is why a C program written on Mac can often compile on Linux. Windows has its own entirely different set of syscalls (Windows API).

# Common Mistakes
* **Over-calling the OS:** Writing a loop that prints to the console a million times will be incredibly slow because it triggers a million expensive context switches to the Kernel.

# Security & Performance Considerations
* **The Attack Vector:** Hackers look for vulnerabilities in System Calls. If a hacker can pass malicious data through a syscall that tricks the Kernel, they can gain "root" or "admin" privileges and take over the entire machine.
* **Syscall Overhead:** High-performance software (like heavy databases or stock trading engines) goes to extreme lengths to minimize the number of system calls they make to avoid the microsecond delays of switching to Kernel Mode.

# Related Technologies
* 03. Operating Systems
* 06. Processes & Threads

# Summary
## What We Learned
- System Calls are how applications ask the OS to do hardware-level tasks.
- They provide a secure boundary between the unprivileged User Mode and the absolute power of Kernel Mode.
- Standard programming functions (like reading files) are just wrappers around syscalls.

## Key Takeaways
- System calls are "expensive" in terms of CPU time.
- High-performance code minimizes interactions with the Kernel by batching data.

# Keywords
- System Call (Syscall)
- User Mode
- Kernel Mode
- Context Switch
- POSIX

# Glossary
| Term | Meaning |
|---|---|
| Syscall | A programmatic request to the OS kernel. |
| User Mode | Restricted execution environment for standard apps. |
| Kernel Mode | Unrestricted execution environment for the OS. |

## Next Recommended Chapters
- 09. Virtualization
