# unix-shell-posix
# Unix Shell with Job Control

A Unix-like shell implemented in C using POSIX process and signal APIs.

The shell supports foreground and background execution, job control, process groups, asynchronous signal handling, and race-free process synchronization. It was developed to gain hands-on experience with core operating system concepts including processes, signals, synchronization, and process scheduling.

---

## Features

- Foreground and background job execution
- Job control using `bg` and `fg`
- Process group management
- POSIX signal handling
  - `SIGCHLD`
  - `SIGINT`
  - `SIGTSTP`
- Zombie process cleanup
- Race-free synchronization using signal masking
- Concurrent process-state management

---

## Operating System Concepts

### Process Creation

Each command is executed in a separate process using:

```c
fork();
execve();
```

The shell creates child processes and executes programs while maintaining control over foreground and background jobs.

### Process Groups

Every job is assigned its own process group:

```c
setpgid(0, 0);
```

This allows signals to be delivered to an entire job rather than a single process.

### Signal Handling

The shell handles asynchronous process events using:

```c
SIGCHLD
SIGINT
SIGTSTP
```

Signal handlers are responsible for:

- Reaping terminated children
- Forwarding Ctrl-C to foreground jobs
- Forwarding Ctrl-Z to foreground jobs
- Updating job states

### Synchronization

The shell prevents race conditions between parent and child processes using:

```c
sigprocmask();
sigsuspend();
```

This ensures that child state changes are never missed even when signals arrive at unpredictable times.

---

## Architecture

```

+----------------+
| User Command   |
+----------------+
|
v
+----------------+
| eval()         |
+----------------+
|
+----------+----------+
|                     |
v                     v
fork()           Built-in Command
|
v
+----------------+
| addjob()       |
+----------------+
|
v
+----------------+
| waitfg()       |
+----------------+
|
v
+----------------+
| Signal Handlers|
+----------------+
| SIGCHLD        |
| SIGINT         |
| SIGTSTP        |
+----------------+

```

---

## Example Usage

### Foreground Job

```bash
tsh> ./myspin 5
```

### Background Job

```bash
tsh> ./myspin 10 &
[1] (1234) ./myspin 10 &
```

### List Active Jobs

```bash
tsh> jobs
[1] (1234) Running ./myspin 10 &
```

### Resume a Job in Background

```bash
tsh> bg %1
```

### Resume a Job in Foreground

```bash
tsh> fg %1
```

---

## Key Technical Challenges

### Race Condition Between `fork()` and `SIGCHLD`

A child process may terminate before the parent inserts it into the job list.

**Solution**

1. Block `SIGCHLD`
2. Create the child process
3. Add the job to the job list
4. Restore the signal mask

```c
sigprocmask(SIG_BLOCK, &mask, &prev);
pid = fork();
addjob(...);
sigprocmask(SIG_SETMASK, &prev, NULL);
```

---

### Waiting for Foreground Jobs

Busy waiting wastes CPU cycles and introduces unnecessary overhead.

**Solution**

Use:

```c
sigsuspend();
```

to suspend execution until a relevant signal arrives.

---

### Signal Forwarding

Signals should be delivered to the entire foreground process group rather than an individual process.

```c
kill(-pid, SIGINT);
kill(-pid, SIGTSTP);
```

This ensures correct behavior for multi-process jobs.

---

### Zombie Process Reaping

Terminated child processes become zombies until collected by the parent.

The shell uses:

```c
waitpid(-1, &status, WNOHANG | WUNTRACED);
```

inside the `SIGCHLD` handler to reap all completed children without blocking.

---

## Technologies

- C
- Linux
- POSIX APIs
- Process Management
- Signals
- Synchronization
- Systems Programming

---

## Learning Outcomes

Through this project I gained practical experience with:

- Process creation and execution
- Process groups and job control
- POSIX signal handling
- Race-free synchronization
- Asynchronous event handling
- Zombie process cleanup
- Operating system design principles

---

## Repository Structure

```

.
├── tsh.c
├── tsh.h
├── Makefile
├── trace01.txt
├── ...
├── trace16.txt
└── README.md

```

---

## References

- Computer Systems: A Programmer's Perspective (CS:APP)
- POSIX Process and Signal APIs
- Linux Manual Pages
