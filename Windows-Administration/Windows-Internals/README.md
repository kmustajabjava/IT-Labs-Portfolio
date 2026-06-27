# Windows Internals Lab: Exploring Processes, Threads, Handles, and the Windows Registry

## Overview

This lab explores fundamental Windows operating system internals using **Microsoft Sysinternals Process Explorer** and **Registry Editor (Regedit)**. The objective was to understand how Windows manages processes, threads, system handles, and configuration data stored in the Windows Registry.

---

## Objectives

* Explore active Windows processes using Process Explorer.
* Understand parent-child process relationships.
* Observe how threads are managed within a process.
* Examine system handles and the resources they reference.
* Explore the structure of the Windows Registry.
* Modify a registry value and observe its effect on application behavior.

---

## Tools Used

* Windows 10/11
* Microsoft Sysinternals Suite
* Process Explorer (procexp.exe)
* Registry Editor (regedit)
* Command Prompt

---

# Part 1 – Process Exploration

## Task 1: Inspecting Running Processes

Opened **Process Explorer** and identified the active Microsoft Edge process using the **Find Window's Process** tool.

### Observation

After terminating the Microsoft Edge process:

* The browser window immediately closed.
* The process disappeared from the active process list.

**Learning Outcome**

Applications depend on their running process. Once the process is terminated, the application exits immediately.

---

## Task 2: Parent and Child Processes

Opened Command Prompt and inspected its process hierarchy.

### Process Relationship

```text
explorer.exe
    └── cmd.exe
            └── conhost.exe
```

Started a continuous ping command.

```cmd
ping 8.8.8.8 -t
```

### Observation

A new child process appeared:

```text
PING.EXE
```

under the **cmd.exe** process.

This demonstrated how applications can dynamically create child processes during execution.

---

## Task 3: VirusTotal Integration

Used Process Explorer's built-in VirusTotal integration to analyze **conhost.exe**.

### Observation

Process Explorer uploaded the file hash and displayed the VirusTotal reputation score within the application before opening the detailed report in the web browser.

---

## Task 4: Process Dependency

Terminated the **cmd.exe** process.

### Observation

Both:

* cmd.exe
* conhost.exe

were automatically closed.

### Learning Outcome

Child processes depend on their parent process. When the parent process terminates, dependent child processes are also terminated.

---

# Part 2 – Exploring Threads

Opened the **Properties** window of **conhost.exe** and navigated to the **Threads** tab.

### Information Observed

* Thread IDs
* CPU usage
* Thread start address
* Thread state
* Thread execution time

Additional process information included:

* Environment variables
* Security details
* Performance statistics
* Process strings

---

# Part 3 – Exploring Handles

Enabled the **Lower Pane View → Handles** option in Process Explorer.

### Observation

The handles referenced various operating system resources including:

* Files
* Registry Keys
* Threads
* Events
* Synchronization Objects

### Learning Outcome

Handles act as references that allow a process to interact with operating system resources.

---

# Part 4 – Exploring the Windows Registry

Opened Registry Editor and explored the five primary registry hives.

| Registry Hive       | Purpose                                                   |
| ------------------- | --------------------------------------------------------- |
| HKEY_CLASSES_ROOT   | Stores file associations and COM registration information |
| HKEY_CURRENT_USER   | Stores settings for the currently logged-in user          |
| HKEY_LOCAL_MACHINE  | Stores system-wide hardware and software configuration    |
| HKEY_USERS          | Stores profiles for all users on the computer             |
| HKEY_CURRENT_CONFIG | Stores the current hardware profile used during boot      |

---

## Modifying a Registry Key

Navigated to:

```text
HKEY_CURRENT_USER
└── Software
    └── Sysinternals
        └── Process Explorer
```

Located the registry value:

```text
EulaAccepted
```

Changed:

```text
1
```

to

```text
0
```

### Observation

The registry entry changed to:

```text
0x00000000 (0)
```

---

## Verification

After reopening Process Explorer, the application displayed the **License Agreement (EULA)** dialog again.

### Learning Outcome

This demonstrates how applications store configuration data within the Windows Registry and use registry values to determine application behavior.

---

# Skills Demonstrated

* Windows Process Management
* Windows Internals
* Process Hierarchy Analysis
* Thread Inspection
* Handle Analysis
* Registry Navigation
* Registry Modification
* System Troubleshooting
* Sysinternals Suite
* Command Prompt

---

# Folder Structure

```text
Windows-Internals-Lab/
│
├── README.md
├── screenshots/
│   ├── process-explorer.png
│   ├── edge-process.png
│   ├── ping-process.png
│   ├── threads.png
│   ├── handles.png
│   ├── registry-editor.png
│   └── eulaaccepted.png
└── notes/
    └── observations.md
```

---

# Key Takeaways

* Learned how Windows creates and manages processes.
* Understood parent-child process relationships.
* Explored thread execution inside a running process.
* Investigated system handles and the resources they reference.
* Learned the purpose of the Windows Registry.
* Verified how registry values directly influence application behavior.

---

## Author

**Kainat Mustajab**

BS Information Technology

Aspiring Network Engineer | CCNA Learner | Cybersecurity Enthusiast
