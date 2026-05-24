# ⚙️ Processes

> A **process** is an instance of a running program. Linux is a multitasking OS — it manages hundreds of processes simultaneously, each with its own memory space, resources, and state. Understanding how to monitor, control, and manage processes is a core Linux skill.

---

## 1. What is a Process?

### 1.1 Program vs Process

| Term | Definition |
|---|---|
| **Program** | A static set of instructions stored on disk (binary/script) |
| **Process** | A running instance of a program, loaded into memory |
| **Thread** | A lightweight execution unit within a process (shared memory) |

A single program can spawn multiple processes. Each process is independent and isolated.

### 1.2 How Processes are Created

Linux uses a **fork-exec** model:

- **`fork()`** — The kernel duplicates the parent process (creates a child)
- **`exec()`** — The child replaces itself with a new program

```bash
# Every process has a parent — the first process is always PID 1
pstree -p    # View the full parent-child process tree
```

### 1.3 The init / systemd Process

The very first process started by the kernel is **PID 1**. On modern Linux systems this is `systemd`. It is the ancestor of all other processes.

```bash
ps -p 1      # Confirm PID 1
cat /proc/1/comm   # Shows: systemd
```

---

## 2. Process Identifiers

### 2.1 Key IDs

| ID | Name | Description |
|---|---|---|
| **PID** | Process ID | Unique ID assigned to each process |
| **PPID** | Parent PID | PID of the process that spawned this one |
| **UID** | User ID | The user who owns the process |
| **GID** | Group ID | The primary group of the process owner |
| **EUID** | Effective UID | UID used for permission checks (can differ with SUID) |

```bash
echo $$        # PID of current shell
echo $PPID     # PID of parent shell
id             # Show UID, GID, groups of current user
```

### 2.2 Viewing Process IDs

```bash
ps aux             # Show all processes with PID, PPID, UID
ps -ef             # BSD-style full listing
pgrep bash         # Find PIDs by name
pidof firefox      # Find PID of a named program
```

---

## 3. Process States

### 3.1 State Table

| State | Code | Description |
|---|---|---|
| **Running** | `R` | Actively using CPU or in run queue |
| **Sleeping (Interruptible)** | `S` | Waiting for an event (e.g., I/O); can be interrupted |
| **Sleeping (Uninterruptible)** | `D` | Waiting for I/O that cannot be interrupted (disk, NFS) |
| **Stopped** | `T` | Paused by a signal (e.g., `SIGSTOP` or Ctrl+Z) |
| **Zombie** | `Z` | Finished but parent hasn't read exit status yet |
| **Dead** | `X` | Fully terminated (rarely seen) |

```bash
ps aux | awk '{print $8, $11}' | head -20   # Show state + command
```

### 3.2 Zombie Processes

A **zombie** process has exited but its parent hasn't called `wait()` to collect its exit status. It holds an entry in the process table.

```bash
ps aux | grep Z    # Find zombie processes
```

!!! warning "Zombie Accumulation"
    Zombies are harmless in small numbers but if they accumulate they exhaust the process table. The fix is to restart or fix the parent process, not kill the zombie.

---

## 4. Viewing Processes

### 4.1 `ps` — Process Snapshot

```bash
ps                  # Processes in current shell session
ps aux              # All processes (a=all users, u=user format, x=no tty)
ps -ef              # Full-format listing (UID, PID, PPID, CMD)
ps -eLf             # Include threads
ps --sort=-%cpu     # Sort by CPU usage descending
ps --sort=-%mem     # Sort by memory usage descending
ps -p 1234          # Info for a specific PID
ps -u username      # Processes owned by a user
ps -C firefox       # Processes by command name
```

**Common `ps aux` columns:**

| Column | Meaning |
|---|---|
| USER | Owner of the process |
| PID | Process ID |
| %CPU | CPU usage percentage |
| %MEM | Memory usage percentage |
| VSZ | Virtual memory size (KB) |
| RSS | Resident Set Size — actual RAM used (KB) |
| TTY | Controlling terminal |
| STAT | Process state code |
| START | Start time |
| TIME | Cumulative CPU time |
| COMMAND | Command that started the process |

### 4.2 `top` — Dynamic Real-Time View

```bash
top              # Launch interactive process viewer
top -u username  # Show only processes for a user
top -p 1234      # Monitor a specific PID
top -n 5         # Run 5 iterations then exit
top -b -n 1      # Batch mode — pipe-friendly snapshot
```

**Interactive `top` keys:**

| Key | Action |
|---|---|
| `h` | Help |
| `q` | Quit |
| `k` | Kill a process (enter PID) |
| `r` | Renice a process |
| `M` | Sort by memory |
| `P` | Sort by CPU |
| `T` | Sort by cumulative time |
| `u` | Filter by user |
| `1` | Toggle individual CPU cores |
| `d` | Change refresh delay |
| `f` | Add/remove display fields |
| `W` | Save config to `~/.toprc` |

**`top` header explained:**

```
top - 14:32:01 up 3 days,  2:15,  2 users,  load average: 0.52, 0.48, 0.45
Tasks: 215 total,   1 running, 214 sleeping,   0 stopped,   0 zombie
%Cpu(s):  3.2 us,  1.1 sy,  0.0 ni, 95.4 id,  0.0 wa,  0.0 hi,  0.3 si,  0.0 st
MiB Mem :  15886.3 total,   4123.7 free,   7845.2 used,   3917.4 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   7654.3 avail Mem
```

| Field | Meaning |
|---|---|
| `us` | User space CPU % |
| `sy` | Kernel/system CPU % |
| `ni` | Nice'd process CPU % |
| `id` | Idle CPU % |
| `wa` | I/O wait CPU % |
| `hi` | Hardware interrupt % |
| `si` | Software interrupt % |
| `st` | CPU stolen by hypervisor (VMs) |

### 4.3 `htop` — Enhanced Interactive Viewer

```bash
htop             # Colorful, mouse-friendly process viewer
```

!!! tip "Install htop"
    ```bash
    sudo apt install htop      # Debian/Ubuntu
    sudo dnf install htop      # Fedora/RHEL
    ```

**`htop` advantages over `top`:**
- Color-coded CPU/memory bars
- Mouse click support
- Easier kill/renice via F-keys
- Horizontal and vertical process tree view
- Easier searching with `F3`

### 4.4 `pgrep` and `pidof`

```bash
pgrep firefox            # Find PID(s) by name
pgrep -u root sshd       # sshd processes owned by root
pgrep -l bash            # List PID + name
pidof nginx              # All PIDs of nginx
```

### 4.5 `/proc` Filesystem

Every process has a directory at `/proc/<PID>/` exposing its internals:

```bash
ls /proc/1/              # PID 1 (systemd) info
cat /proc/1/status       # Detailed status
cat /proc/1/cmdline      # Command-line args (null-separated)
cat /proc/1/environ      # Environment variables
ls -l /proc/1/fd/        # Open file descriptors
cat /proc/1/maps         # Memory maps
cat /proc/self/status    # Your own process info
```

| File | Contents |
|---|---|
| `status` | Human-readable state, memory, UID |
| `cmdline` | Command used to start the process |
| `environ` | Environment variables |
| `fd/` | Open file descriptors |
| `maps` | Virtual memory mappings |
| `stat` | Machine-readable stats (used by `ps`) |
| `io` | I/O statistics |
| `limits` | Resource limits |

---

## 5. Signals

### 5.1 What are Signals?

Signals are software interrupts sent to processes to notify them of events. They are the primary way to communicate with and control processes.

### 5.2 Common Signals

| Signal | Number | Default Action | Meaning |
|---|---|---|---|
| `SIGHUP` | 1 | Terminate | Hangup — often used to reload config |
| `SIGINT` | 2 | Terminate | Interrupt from keyboard (Ctrl+C) |
| `SIGQUIT` | 3 | Core dump | Quit from keyboard (Ctrl+\\) |
| `SIGKILL` | 9 | Terminate | Force kill — cannot be caught or ignored |
| `SIGTERM` | 15 | Terminate | Graceful termination request (default) |
| `SIGSTOP` | 19 | Stop | Pause process — cannot be caught or ignored |
| `SIGCONT` | 18 | Continue | Resume a stopped process |
| `SIGCHLD` | 17 | Ignore | Child process stopped or terminated |
| `SIGUSR1` | 10 | Terminate | User-defined signal 1 |
| `SIGUSR2` | 12 | Terminate | User-defined signal 2 |
| `SIGPIPE` | 13 | Terminate | Broken pipe (write to closed socket) |
| `SIGALRM` | 14 | Terminate | Alarm clock timeout |
| `SIGSEGV` | 11 | Core dump | Segmentation fault (invalid memory access) |

```bash
kill -l          # List all signal names and numbers
```

!!! warning "SIGKILL vs SIGTERM"
    Always try `SIGTERM` (15) first — it lets the process clean up. Use `SIGKILL` (9) only if the process doesn't respond. `SIGKILL` cannot be caught, blocked, or ignored.

### 5.3 Sending Signals with `kill`

```bash
kill 1234              # Send SIGTERM to PID 1234
kill -9 1234           # Send SIGKILL to PID 1234
kill -SIGTERM 1234     # Explicit name
kill -15 1234          # Explicit number
kill -HUP 1234         # Reload config (SIGHUP)
```

### 5.4 `killall` and `pkill`

```bash
killall firefox          # Kill all processes named "firefox"
killall -9 firefox       # Force kill all firefox processes
pkill -u alice           # Kill all processes owned by alice
pkill -f "python script.py"   # Kill by full command match
```

### 5.5 Keyboard Shortcuts for Signals

| Shortcut | Signal Sent | Effect |
|---|---|---|
| `Ctrl+C` | `SIGINT` (2) | Interrupt/terminate foreground process |
| `Ctrl+Z` | `SIGTSTP` (20) | Suspend/stop foreground process |
| `Ctrl+\` | `SIGQUIT` (3) | Quit with core dump |

---

## 6. Process Priority and Niceness

### 6.1 Concept

Linux assigns CPU time using a priority scheduler. **Niceness** is a user-adjustable value that influences priority.

| Value | Range | Meaning |
|---|---|---|
| **Nice** | -20 to 19 | Lower = higher priority; higher = more "nice" to others |
| **Priority (PR)** | 0 to 39 (in `top`) | Kernel's actual scheduling priority |

!!! info "Priority Math"
    `Priority = 20 + Nice` (in top's display). So nice -20 → PR 0 (highest). Nice 19 → PR 39 (lowest).

### 6.2 Starting a Process with a Nice Value

```bash
nice -n 10 ./backup.sh         # Start with niceness +10 (lower priority)
nice -n -5 ./urgent-task       # Start with niceness -5 (higher priority, needs sudo)
sudo nice -n -20 ./critical    # Highest priority
```

### 6.3 Changing Priority of Running Process with `renice`

```bash
renice 10 -p 1234              # Set niceness of PID 1234 to 10
renice -5 -p 1234              # Increase priority (needs sudo)
sudo renice -10 -p 1234        # High priority
renice 5 -u username           # Renice all processes of a user
```

### 6.4 Viewing Priority in `top`

In `top`, the `PR` and `NI` columns show priority and niceness. Processes with `PR = rt` are real-time processes with the highest possible priority.

---

## 7. Foreground and Background Jobs

### 7.1 Concept

The shell can run processes in:
- **Foreground** — occupies the terminal, you must wait for it
- **Background** — runs independently, shell remains usable

### 7.2 Running in Background

```bash
./long-script.sh &       # Start in background (append &)
sleep 100 &              # Background sleep
```

The shell prints `[job_number] PID` when backgrounding a process.

### 7.3 Job Control Commands

```bash
jobs               # List all background/stopped jobs
jobs -l            # Include PIDs
fg                 # Bring most recent background job to foreground
fg %2              # Bring job number 2 to foreground
bg                 # Resume most recent stopped job in background
bg %2              # Resume job 2 in background
```

### 7.4 Suspending and Resuming

```bash
# While a process is running in foreground:
Ctrl+Z             # Suspend it (sends SIGTSTP)

# Then:
bg                 # Resume it in background
fg                 # Resume it in foreground
```

### 7.5 `nohup` — Survive Terminal Logout

By default, background jobs receive `SIGHUP` when the terminal closes. `nohup` prevents this:

```bash
nohup ./backup.sh &               # Continues after logout
nohup ./script.sh > output.log 2>&1 &   # With log file
```

Output goes to `nohup.out` by default if no redirection is specified.

### 7.6 `disown` — Detach from Shell

```bash
./script.sh &
disown             # Detach most recent background job from shell
disown %2          # Detach job 2
disown -h %1       # Mark to not receive SIGHUP on shell exit
```

---

## 8. `screen` and `tmux` — Terminal Multiplexers

### 8.1 Why Use Them?

Terminal multiplexers let you create persistent sessions that survive SSH disconnections, and manage multiple terminal windows in one SSH connection.

### 8.2 `screen` Basics

```bash
screen             # Start new session
screen -S mysession      # Named session
screen -ls         # List sessions
screen -r mysession      # Reattach to session
screen -r          # Reattach (if only one)
```

**Inside screen:**

| Shortcut | Action |
|---|---|
| `Ctrl+A, D` | Detach session |
| `Ctrl+A, C` | New window |
| `Ctrl+A, N` | Next window |
| `Ctrl+A, P` | Previous window |
| `Ctrl+A, "` | List windows |
| `Ctrl+A, K` | Kill current window |

### 8.3 `tmux` Basics

```bash
tmux               # Start new session
tmux new -s mysession    # Named session
tmux ls            # List sessions
tmux attach -t mysession # Attach to session
```

**Inside tmux (prefix = `Ctrl+B`):**

| Shortcut | Action |
|---|---|
| `Ctrl+B, D` | Detach session |
| `Ctrl+B, C` | New window |
| `Ctrl+B, N` | Next window |
| `Ctrl+B, %` | Vertical split |
| `Ctrl+B, "` | Horizontal split |
| `Ctrl+B, Arrow` | Move between panes |
| `Ctrl+B, X` | Close pane |
| `Ctrl+B, ?` | Help / key list |

---

## 9. Scheduling Processes

### 9.1 `at` — One-Time Scheduled Jobs

```bash
at 10:30pm                    # Schedule a job interactively
at now + 2 hours              # 2 hours from now
at midnight                   # Tonight at midnight
at 10:30pm 2025-12-25         # Specific date

# Inside at prompt, type commands, then Ctrl+D to submit
echo "backup.sh" | at 2am
```

```bash
atq                    # List pending at jobs
atrm 3                 # Remove job number 3
```

### 9.2 `cron` — Recurring Scheduled Jobs

`cron` runs jobs on a schedule defined in a **crontab** file.

```bash
crontab -e             # Edit your crontab
crontab -l             # List your crontab
crontab -r             # Remove your crontab
sudo crontab -e -u alice    # Edit another user's crontab
```

**Crontab format:**

```
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day of month (1 - 31)
│ │ │ ┌───────────── month (1 - 12)
│ │ │ │ ┌───────────── day of week (0 - 7, Sun=0 or 7)
│ │ │ │ │
* * * * *  command to execute
```

**Crontab examples:**

```bash
# Run backup every day at 2:30 AM
30 2 * * * /home/user/backup.sh

# Run every 5 minutes
*/5 * * * * /usr/local/bin/monitor.sh

# Run every Monday at 9 AM
0 9 * * 1 /usr/local/bin/report.sh

# Run at noon on the 1st of every month
0 12 1 * * /usr/local/bin/monthly.sh

# Run every weekday (Mon-Fri) at 8 PM
0 20 * * 1-5 /usr/local/bin/notify.sh

# Redirect output
0 3 * * * /home/user/backup.sh >> /var/log/backup.log 2>&1
```

**Special crontab strings:**

| String | Equivalent | Meaning |
|---|---|---|
| `@reboot` | — | Run once at startup |
| `@hourly` | `0 * * * *` | Run every hour |
| `@daily` | `0 0 * * *` | Run every day at midnight |
| `@weekly` | `0 0 * * 0` | Run every Sunday at midnight |
| `@monthly` | `0 0 1 * *` | Run on the 1st of every month |
| `@yearly` | `0 0 1 1 *` | Run on Jan 1st at midnight |

### 9.3 `anacron` — For Non-24/7 Systems

`cron` misses jobs if the machine is off. `anacron` reruns missed jobs on next boot.

```bash
cat /etc/anacrontab      # View anacron schedule
```

---

## 10. Resource Limits

### 10.1 `ulimit` — Per-Shell Resource Limits

```bash
ulimit -a                # Show all current limits
ulimit -n                # Max open file descriptors
ulimit -u                # Max user processes
ulimit -v                # Max virtual memory (KB)
ulimit -s                # Max stack size (KB)
ulimit -c                # Max core dump size (0 = disabled)
ulimit -t                # Max CPU time (seconds)

# Set limits (soft)
ulimit -n 4096           # Increase open file limit for this session
ulimit -Hn               # Show hard limit
ulimit -Sn               # Show soft limit
```

!!! info "Soft vs Hard Limits"
    **Soft limit** — the actual enforced limit; users can raise up to the hard limit.  
    **Hard limit** — ceiling that only root can raise. Set in `/etc/security/limits.conf`.

### 10.2 `/etc/security/limits.conf`

Persistent limits for users/groups:

```bash
# /etc/security/limits.conf format:
# <domain>  <type>  <item>   <value>
alice        soft    nofile   4096
alice        hard    nofile   8192
@developers  soft    nproc    200
*            hard    core     0
```

### 10.3 `cgroups` — Control Groups

`cgroups` (control groups) is the kernel feature that lets you limit, account for, and isolate resource usage of process groups. It's the backbone of containers (Docker, LXC).

```bash
# View cgroup hierarchy
ls /sys/fs/cgroup/
cat /proc/self/cgroup      # Your process's cgroups

# systemd uses cgroups by default
systemd-cgls               # View cgroup tree
systemctl status           # Shows cgroup info per service
```

---

## 11. Process Monitoring Tools

### 11.1 `vmstat` — System-Wide Stats

```bash
vmstat                     # One-time snapshot
vmstat 2                   # Update every 2 seconds
vmstat 2 10                # 10 updates every 2 seconds
```

Key columns: `r` (run queue), `b` (blocked), `si`/`so` (swap in/out), `us`/`sy`/`id`/`wa` (CPU).

### 11.2 `iostat` — CPU and I/O Stats

```bash
iostat                     # CPU + disk I/O snapshot
iostat -x 2                # Extended stats every 2s
iostat -d /dev/sda         # Specific device
```

### 11.3 `mpstat` — Per-CPU Stats

```bash
mpstat                     # All CPUs combined
mpstat -P ALL 2            # All individual CPUs every 2s
mpstat -P 0                # CPU 0 only
```

### 11.4 `sar` — System Activity Reporter

```bash
sar                        # CPU usage from today's log
sar -u 2 5                 # CPU, 5 samples every 2s
sar -r 2 5                 # Memory usage
sar -d 2 5                 # Disk activity
sar -n DEV 2 5             # Network interface stats
```

### 11.5 `lsof` — List Open Files

Every process holds open files, sockets, devices. `lsof` lists them all.

```bash
lsof                       # All open files (huge output)
lsof -p 1234               # Open files for PID 1234
lsof -u username           # Open files by user
lsof /var/log/syslog       # Who has this file open?
lsof -i                    # All network connections
lsof -i :80                # Processes using port 80
lsof -i TCP                # All TCP connections
lsof +D /tmp               # All files under /tmp
```

### 11.6 `strace` — System Call Tracer

Traces every system call a process makes — invaluable for debugging.

```bash
strace ls                  # Trace system calls of 'ls'
strace -p 1234             # Attach to running PID 1234
strace -e open,read ls     # Trace only open() and read() calls
strace -o trace.log ls     # Write trace to file
strace -c ls               # Summary count of calls
strace -f ./script.sh      # Follow child processes too
```

### 11.7 `perf` — Performance Analysis

```bash
sudo perf top              # Live CPU profiling (like top but by function)
sudo perf stat ls          # Count hardware events for 'ls'
sudo perf record ./app     # Record profile data
sudo perf report           # View recorded data
```

---

## 12. Load Average

### 12.1 What is Load Average?

Load average represents the **average number of processes waiting for CPU time** over 1, 5, and 15 minutes. Shown in `top`, `uptime`, and `w`.

```bash
uptime
# 14:45:03 up 3 days, 2:28, 2 users, load average: 1.20, 0.95, 0.85
#                                                    1min  5min  15min
```

### 12.2 Interpreting Load Average

| Load vs CPUs | Meaning |
|---|---|
| Load < # CPUs | System is fine, CPUs have idle time |
| Load ≈ # CPUs | System is fully utilized |
| Load > # CPUs | System is overloaded, processes are queuing |

```bash
nproc                      # Number of CPU cores
cat /proc/cpuinfo | grep processor | wc -l   # Alternative
```

!!! tip "Rule of Thumb"
    On a 4-core machine, a sustained load average of 4.0 means fully utilized. 6.0 means overloaded.

---

## 13. Memory and Process Memory Usage

### 13.1 `free` — Memory Overview

```bash
free -h                    # Human-readable (MB/GB)
free -m                    # In megabytes
free -s 2                  # Update every 2 seconds
```

### 13.2 Per-Process Memory

```bash
ps aux --sort=-%mem | head -10     # Top 10 memory consumers
top                                # M key to sort by memory

cat /proc/1234/status | grep -i vm # Virtual memory stats for PID 1234
```

| Field | Meaning |
|---|---|
| **VSZ** | Virtual memory size (all mapped memory, including not-yet-used) |
| **RSS** | Resident Set Size (actual RAM currently in use) |
| **SHR** | Shared memory (shared with other processes) |

### 13.3 `pmap` — Process Memory Map

```bash
pmap 1234                  # Memory map of PID 1234
pmap -x 1234               # Extended with RSS and dirty pages
```

---

## 14. Orphan and Zombie Processes

### 14.1 Orphan Processes

An **orphan** process is one whose parent has died. The kernel automatically re-parents orphans to **PID 1 (init/systemd)**, which then cleans them up properly.

### 14.2 Zombie Processes

A **zombie** process has exited but its parent has not called `wait()` to collect its exit status, so its entry remains in the process table.

```bash
ps aux | awk '$8 == "Z"'   # Find zombies (state = Z)
```

To eliminate zombies: kill or fix the parent process. On the next `wait()` call (or when the parent dies), the zombie is reaped.

---

## 15. Executable Formats and Libraries

### 15.1 ELF Format

Linux executables use the **ELF (Executable and Linkable Format)**.

```bash
file /bin/ls               # Confirms it's an ELF executable
readelf -h /bin/ls         # ELF header details
objdump -f /bin/ls         # File format info
```

### 15.2 Static vs Dynamic Linking

| Type | Description | Size |
|---|---|---|
| **Static** | All libraries compiled into the binary | Larger binary |
| **Dynamic** | Libraries loaded at runtime from system | Smaller binary |

```bash
ldd /bin/ls                # List dynamic library dependencies
ldd /usr/bin/python3       # Dependencies for python3
```

### 15.3 `ldconfig` and `LD_LIBRARY_PATH`

```bash
sudo ldconfig              # Rebuild dynamic library cache
ldconfig -p                # Print all cached libraries
echo $LD_LIBRARY_PATH      # Custom library search paths
export LD_LIBRARY_PATH=/opt/mylib:$LD_LIBRARY_PATH
```

---

## 16. Quick Reference Cheat Sheet

```bash
# ─── Viewing Processes ───────────────────────────────────────────────
ps aux                     # Snapshot of all processes
ps -ef                     # Full-format listing
top                        # Interactive real-time view
htop                       # Enhanced interactive view
pgrep firefox              # Find PID by name
pidof nginx                # PID of named program
cat /proc/<PID>/status     # Detailed process info

# ─── Signals ─────────────────────────────────────────────────────────
kill -15 <PID>             # Graceful terminate (SIGTERM)
kill -9 <PID>              # Force kill (SIGKILL)
kill -HUP <PID>            # Reload config (SIGHUP)
killall firefox            # Kill by name
pkill -u alice             # Kill by user
Ctrl+C                     # Interrupt foreground (SIGINT)
Ctrl+Z                     # Suspend foreground (SIGTSTP)

# ─── Priority / Niceness ─────────────────────────────────────────────
nice -n 10 ./script.sh     # Start with niceness 10
sudo nice -n -5 ./app      # Start with higher priority
renice 5 -p <PID>          # Change running process niceness

# ─── Job Control ─────────────────────────────────────────────────────
./script.sh &              # Run in background
jobs -l                    # List jobs with PIDs
fg %1                      # Foreground job 1
bg %1                      # Background job 1
nohup ./script.sh &        # Survive logout
disown %1                  # Detach job from shell

# ─── Scheduling ──────────────────────────────────────────────────────
at 10pm                    # One-time scheduled job
atq                        # List pending at jobs
atrm 3                     # Remove at job 3
crontab -e                 # Edit recurring cron jobs
crontab -l                 # List cron jobs

# ─── Resource Limits ─────────────────────────────────────────────────
ulimit -a                  # Show all limits
ulimit -n 4096             # Set open files limit

# ─── Monitoring ──────────────────────────────────────────────────────
vmstat 2                   # System stats every 2s
iostat -x 2                # I/O stats every 2s
lsof -p <PID>              # Open files of a process
lsof -i :80                # Process using port 80
strace -p <PID>            # Trace system calls
uptime                     # Load averages

# ─── Memory ──────────────────────────────────────────────────────────
free -h                    # Memory overview
pmap <PID>                 # Process memory map
ps aux --sort=-%mem        # Sort by memory usage

# ─── Libraries ───────────────────────────────────────────────────────
ldd /bin/ls                # Dynamic library dependencies
ldconfig -p                # List cached libraries
```