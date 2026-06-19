> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Linux Fundamentals** is the study of the Linux operating system, command-line interface (CLI), directory filesystem structure, permissions settings, and process controls. Because virtually all production web servers, cloud platforms, and database instances run on Linux, mastering its commands is a foundational requirement for software development and DevOps operations.

# Why It Exists
Most personal computers run user-friendly operating systems like Windows or macOS, which use Graphical User Interfaces (GUIs) with icons, folders, and clickable menus. However, running a GUI on a high-traffic production server is highly inefficient, consuming significant CPU and memory resources just to render graphics. Furthermore, graphical menus cannot be easily automated using scripts. Engineers created the lightweight, open-source Linux operating system to run servers using text-only command lines, maximizing server performance and allowing administrators to automate system management tasks using script code.

# Problem It Solves
Linux fundamentals solve server resource waste, system administration automation blocks, and server configuration drift.

### Before Linux Server Management (GUI Focus):
- Web servers wasted 20% of their RAM and CPU just drawing a desktop screen.
- Backing up files, updating software, or configuring ports required manual clicking, which was slow and prone to human error.
- Checking system status required connecting via remote desktop, which saturated network bandwidth.

### After Linux Server Management (CLI Focus):
- Servers operate with maximum efficiency, dedicating 100% of their hardware to running application databases and backends.
- Administrators write automated bash scripts to manage user accounts, back up databases, and update software across thousands of servers simultaneously.
- Server metrics and logs are queried instantly over lightweight text connections (SSH), requiring minimal network bandwidth.

# Core Concepts
To navigate and operate a Linux server, you must master the directory structure, file permissions, and process management:

1. **The Linux Filesystem Tree:** Unlike Windows (which divides storage into drives like `C:` and `D:`), Linux organizes everything into a single hierarchical directory tree starting at the **Root** directory (`/`). Core subdirectories include:
   - `/bin`: Essential system command binaries (like `ls` or `mkdir`).
   - `/etc`: Configuration files for system services.
   - `/var`: Volatile files that change frequently, such as system logs.
   - `/home`: Personal folders for standard user accounts.
2. **File Permissions (chmod):** A security system that restricts access to files and directories based on three roles: Owner (User), Group, and Others. For each role, permissions define who can Read (`r`), Write (`w`), or Execute (`x`) the file.
3. **Processes:** Any active, running program on the system. Every process is assigned a unique Process ID (PID) by the kernel.

# Architecture / Components
The structure of a Linux operating system showing how the User Shell communicates with the underlying hardware via the Kernel:

```text
       [ User / Admin ]
              │
              ▼ (Types commands like 'ls' or 'mkdir')
          [ Shell ] (Command-line Interpreter, e.g. Bash)
              │
              ▼ (Translates shell commands to system calls)
          [ Kernel ] (Core OS Engine)
              │
              ├──────────────────────┬──────────────────────┤
              ▼                      ▼                      ▼
          [ CPU ]                 [ RAM ]               [ Disk / SSD ]
```

- **Shell:** The command-line program (e.g. Bash, Zsh) that reads typed commands, interprets them, and calls the appropriate system utilities.
- **Kernel:** The core engine of the operating system that directly controls the computer's physical hardware (CPU, RAM, Disks) on behalf of the Shell.

# Workflow
The standard command navigation workflow inside a Linux terminal:

```text
Step 1: Administrator logs in and opens the terminal. The terminal defaults to the user's `/home/username` directory.
                             ↓
Step 2: Type `pwd` (Print Working Directory) to verify current location.
                             ↓
Step 3: Type `ls -l` (List Files) to see a detailed list of folders, files, and their permission keys.
                             ↓
Step 4: Type `cd /var/log` (Change Directory) to walk down the filesystem tree to the system log room.
                             ↓
Step 5: Type `cat syslog` (Concatenate) to read the contents of the system log file on screen.
```

# Real World Examples
Think of operating a Linux server as **managing a smart house using a text control panel**.
- A desktop operating system (like Windows) is like walking through a house and manually flipping light switches, opening doors, and looking inside drawers. It is easy and visual, but takes time.
- Linux is like sitting at a keyboard control panel in the lobby. You cannot see the rooms, so you must type commands to operate the house:
  - **Filesystem Rooms:** `/` is the front lobby entrance. `/home` represents private bedrooms. `/bin` is the kitchen tool drawer.
  - `pwd` (Print Working Directory): Ask the panel: *"Which room am I standing in right now?"*
  - `cd` (Change Directory): Type `cd bedroom` to walk through the bedroom door.
  - `ls` (List): Type `ls` to list all objects sitting on the bedroom table.
  - `chmod` (Permissions): Locking drawers. You type a command: *"Only the parent (Owner) can open this drawer (Read/Write). Guests (Others) cannot look inside."*
  - **Processes:** Every active appliance (washing machine, AC, oven) is a process with an appliance label (PID). If the washing machine starts spinning out of control, you don't run to pull the plug; you sit at the keyboard, look up the washing machine's code (PID 504), and type: `kill 504` to shut it down instantly.

# Implementation
Here is a list of the most critical daily Linux command-line tools along with examples of how developers use them:

### 1. Filesystem Navigation and Management
```bash
# Print current directory location
pwd
# Output: /home/admin_user

# List all files (including hidden files beginning with a dot) in a detailed list format
ls -la

# Create a new directory named 'my_app'
mkdir my_app

# Navigate into the newly created folder
cd my_app

# Create a new empty text file named 'config.txt'
touch config.txt

# Copy 'config.txt' to a backup file
cp config.txt config_backup.txt

# Delete the backup file
rm config_backup.txt
```

### 2. Checking and Modifying File Permissions
Permissions are represented as a string (e.g. `-rwxr-xr-x`). Let's dissect how to read and change them:

```bash
# View detailed permissions of files
ls -l config.txt
# Output: -rw-r--r-- 1 admin_user admin_group 0 Jun 19 16:00 config.txt
# Read as: User (admin_user) can Read/Write (rw-). Group can Read (r--). Others can Read (r--).

# Change permissions so only the Owner can Read and Write, and nobody else can access
chmod 600 config.txt
# New permissions: -rw-------

# Make a script file executable for everyone
chmod +x run_app.sh
# New permissions: -rwxr-xr-x (x denotes executable)
```

### 3. Monitoring System Health and Running Processes
```bash
# Display live, real-time list of running processes sorted by CPU usage
top

# Find the Process ID (PID) of a running node application
ps aux | grep node
# Output: admin_user  12345  0.5  2.1  ... node server.js  (PID is 12345)

# Force-terminate the runaway node application using its PID
kill -9 12345
```

# Best Practices
- **Never Run Commands as Root (Superuser) by Default:** Avoid logging into your server directly as the `root` administrator account. If you make a mistake (like typing a delete command with a typo), you can delete the entire operating system. Log in as a standard user and use `sudo` (Superuser Do) only when administrative access is required.
- **Learn Keyboard Shortcuts:** Speed up command-line navigation by mastering basic terminal shortcuts (e.g. press `Tab` to auto-complete filenames, `Ctrl + C` to cancel a hanging command, and `up arrow` to scroll through command history).
- **Utilize SSH Keys for Server Access:** Never configure Linux servers to accept password-based logins over the network. Password logins are vulnerable to brute-force attacks. Always configure SSH keys (cryptographic key pairs) to log in securely.

# Industry Standards
Most cloud and enterprise environments standardized on **Ubuntu Server** or **Red Hat Enterprise Linux (RHEL)** distributions. DevOps engineers write automation configurations using standard shells like **Bash** to ensure their setup scripts can run on any Linux instance.

# Common Mistakes
- **The Accidental Wipeout Command:** Running `rm -rf /` as the root user. The `-r` flag means recursive (delete folders and their contents), and `-f` means force (do not ask for confirmation). Running this command at the root directory `/` will delete every single file on the server's hard drive instantly, bricking the machine.
- **Incorrect Relative Path Navigation:** Typing `cd var/log` instead of `cd /var/log`. Leaving out the leading slash tells the shell to look for a folder named `var` inside your *current* folder, rather than starting at the Root lobby.
- **Ignoring Directory Permissions on Script Executions:** Writing a custom deploy script, attempting to run it, and getting a "Permission Denied" error. Developers often waste time debugging code when they simply forgot to run `chmod +x script.sh` to allow execution.

# Security & Performance Considerations
- **Open SSH Port Vulnerability:** By default, Linux listens for SSH terminal connections on Port 22. Hackers run automated scripts scanning the internet for Port 22, attempting millions of password guesses. Administrators must change the default SSH port or block public traffic using network firewalls.
- **Memory Swap Bottlenecks:** When a server runs out of RAM, the Linux kernel moves idle data from RAM to a designated sector of the slow hard drive (swap space). While this prevents the server from crashing immediately, reading from swap is thousands of times slower, causing application response times to lag severely.

# Related Technologies
- **Bash (Bourne Again Shell):** The default command line interpreter shell on most Linux distributions.
- **SSH (Secure Shell):** A cryptographic network protocol used to connect securely to a remote Linux terminal.
- **WSL (Windows Subsystem for Linux):** A feature allowing developers to run a full Linux terminal directly inside Windows without virtual machines.

# Summary

## What We Learned
- Linux is a lightweight, GUI-free operating system designed for maximum server performance and script automation.
- The Linux filesystem is a single unified tree starting at the Root lobby (`/`).
- Permissions enforce security by defining who (Owner, Group, Others) can Read, Write, or Execute files.
- Active programs operate as Processes with unique PIDs that can be monitored and terminated.

## Key Takeaways
- Use standard user accounts paired with `sudo` rather than logging in directly as the root administrator.
- Always configure SSH cryptographic keys for remote server terminal access instead of simple passwords.
- Verify file permissions (`chmod +x`) when deployment scripts return "Permission Denied" errors.

# Keywords
- Linux
- Command Line Interface (CLI)
- Kernel
- Shell
- Root Directory
- chmod
- Process ID (PID)
- SSH
- sudo
- swap space

# Glossary

| Term | Meaning |
|---|---|
| Command Line | A text-based user interface where users type commands to operate the system. |
| Kernel | The core layer of the operating system that communicates directly with the computer hardware. |
| Shell | A program that interprets typed command text and translates it into kernel actions. |
| Root | The top-most directory in the Linux filesystem tree, represented by a single slash (`/`). |
| sudo | Superuser Do; a command that temporarily grants administrative privileges to execute a specific task. |
| Process | A running instance of an application or script on a computer. |

## Next Recommended Chapters
- 02-Git-And-GitHub.md
