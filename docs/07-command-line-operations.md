# 💻 Command Line Operations

> The command line is the most powerful interface on a Linux system. Mastering the shell, navigation, file operations, redirection, and job control unlocks the full potential of Linux — whether on a local machine, remote server, or automated script.

---

## 1. The Shell

### 1.1 What is a Shell?

The **shell** is a command-line interpreter that reads your input, executes commands, and returns output. It acts as the interface between the user and the Linux kernel.

| Shell | Config File | Notes |
|-------|------------|-------|
| **bash** | `~/.bashrc`, `~/.bash_profile` | Default on most distros |
| **zsh** | `~/.zshrc` | Powerful, macOS default, Oh My Zsh |
| **fish** | `~/.config/fish/config.fish` | User-friendly, autosuggestions |
| **dash** | `/etc/environment` | Minimal, fast, POSIX-only |
| **ksh** | `~/.kshrc` | Korn shell, enterprise Unix |
| **tcsh** | `~/.tcshrc` | C shell variant |

```bash
# Check current shell
echo $SHELL

# Check available shells
cat /etc/shells

# Change default shell
chsh -s /usr/bin/zsh

# Open a specific shell temporarily
bash
zsh
fish
```

### 1.2 Shell Prompt

The default bash prompt shows: `username@hostname:current_directory$`

```bash
user@ubuntu:~$          # Standard user prompt ($ = normal user)
root@ubuntu:~#          # Root user prompt (# = root)
user@ubuntu:/etc$       # Inside /etc directory
```

| Symbol | Meaning |
|--------|---------|
| `~` | Home directory shorthand |
| `$` | Normal user prompt |
| `#` | Root / superuser prompt |
| `@` | Separates username from hostname |

### 1.3 Shell Types

| Type | Description |
|------|-------------|
| **Interactive** | Reads input from user in a terminal |
| **Non-interactive** | Runs a script without user input |
| **Login shell** | Started at login — reads `~/.bash_profile` |
| **Non-login shell** | Started after login (e.g. new terminal tab) — reads `~/.bashrc` |

!!! tip
    Always put persistent settings (aliases, exports, PATH changes) in `~/.bashrc` for non-login shells and source `~/.bashrc` from `~/.bash_profile` to cover both.

---

## 2. Navigating the Filesystem

### 2.1 Essential Navigation Commands

```bash
# Print working (current) directory
pwd

# List directory contents
ls
ls -l           # Long format (permissions, owner, size, date)
ls -a           # Include hidden files (dotfiles)
ls -lh          # Human-readable sizes
ls -lt          # Sort by modification time
ls -lS          # Sort by file size
ls -R           # Recursive listing
ls -la /etc     # Long listing of /etc including hidden

# Change directory
cd /etc                  # Absolute path
cd Documents             # Relative path
cd ..                    # Up one level
cd ../..                 # Up two levels
cd ~                     # Home directory
cd -                     # Previous directory
cd /                     # Root directory
```

### 2.2 Absolute vs Relative Paths

| Type | Starts With | Example |
|------|------------|---------|
| **Absolute** | `/` | `/home/user/Documents/file.txt` |
| **Relative** | Current dir | `Documents/file.txt` or `../etc/hosts` |

```bash
# Absolute path — always works regardless of current location
cat /etc/hosts

# Relative path — depends on where you are
cd /etc
cat hosts           # Same file, relative from /etc
```

### 2.3 Directory Shortcuts

| Shortcut | Meaning |
|----------|---------|
| `.` | Current directory |
| `..` | Parent directory |
| `~` | Current user's home (`/home/username`) |
| `~user` | Another user's home (`/home/user`) |
| `-` | Previous directory (used with `cd`) |
| `/` | Root of the filesystem |

---

## 3. Working with Files and Directories

### 3.1 Creating Files and Directories

```bash
# Create an empty file
touch file.txt
touch file1.txt file2.txt file3.txt    # Multiple files

# Create a file with content
echo "Hello World" > file.txt
cat > file.txt << EOF
Line 1
Line 2
EOF

# Create a directory
mkdir mydir
mkdir -p parent/child/grandchild       # Create nested dirs
mkdir -p project/{src,docs,tests}      # Create multiple subdirs

# Create a directory and move into it
mkdir newdir && cd newdir
```

### 3.2 Copying Files and Directories

```bash
# Copy a file
cp source.txt destination.txt
cp source.txt /path/to/dest/           # Copy into directory

# Copy and preserve attributes (timestamps, permissions)
cp -p source.txt dest.txt

# Copy a directory recursively
cp -r sourcedir/ destdir/
cp -rv sourcedir/ destdir/             # Verbose output

# Copy multiple files into a directory
cp file1.txt file2.txt /path/to/dir/

# Interactive copy (ask before overwrite)
cp -i source.txt dest.txt
```

### 3.3 Moving and Renaming

```bash
# Move a file (also used to rename)
mv oldname.txt newname.txt             # Rename
mv file.txt /path/to/directory/        # Move to directory
mv dir1/ dir2/                         # Rename/move directory

# Move multiple files
mv file1.txt file2.txt /destination/

# Interactive move (ask before overwrite)
mv -i source.txt dest.txt
```

### 3.4 Deleting Files and Directories

```bash
# Remove a file
rm file.txt
rm -i file.txt                         # Interactive (prompt first)
rm -f file.txt                         # Force (no prompt)

# Remove multiple files
rm file1.txt file2.txt *.log

# Remove an empty directory
rmdir emptydir

# Remove a directory and all contents
rm -r directory/
rm -rf directory/                      # Force, no prompts

# Safer deletion — move to trash (requires trash-cli)
trash file.txt
```

!!! warning
    `rm -rf` is irreversible. There is no recycle bin in the terminal. Always double-check your command, especially with wildcards. **Never run `rm -rf /` or `rm -rf /*`.**

### 3.5 Viewing File Contents

```bash
# Print entire file
cat file.txt

# Print with line numbers
cat -n file.txt

# Concatenate two files
cat file1.txt file2.txt

# View file one screen at a time
less file.txt         # Navigate: arrows, PgUp/Dn, /search, q to quit
more file.txt         # Older, simpler pager

# View first N lines (default 10)
head file.txt
head -n 20 file.txt

# View last N lines (default 10)
tail file.txt
tail -n 20 file.txt

# Follow a file in real time (logs)
tail -f /var/log/syslog
tail -F /var/log/syslog   # Follow even if file rotates
```

### 3.6 File Information

```bash
# Determine file type
file document.pdf
file image.jpg
file script.sh

# Detailed file info (inode, permissions, size, timestamps)
stat file.txt

# Count lines, words, characters
wc file.txt
wc -l file.txt            # Lines only
wc -w file.txt            # Words only
wc -c file.txt            # Bytes only

# Show first bytes of a file (hex)
xxd file.bin | head
od -c file.bin | head
```

---

## 4. Input/Output Redirection

### 4.1 Standard Streams

Every Linux process has three standard I/O streams:

| Stream | Name | File Descriptor | Default |
|--------|------|----------------|---------|
| **stdin** | Standard Input | `0` | Keyboard |
| **stdout** | Standard Output | `1` | Terminal screen |
| **stderr** | Standard Error | `2` | Terminal screen |

### 4.2 Redirection Operators

```bash
# Redirect stdout to a file (overwrite)
command > output.txt
ls -la > listing.txt

# Redirect stdout to a file (append)
command >> output.txt
echo "new line" >> log.txt

# Redirect stdin from a file
command < input.txt
sort < names.txt

# Redirect stderr to a file
command 2> errors.txt
find / -name "*.conf" 2> /dev/null

# Redirect both stdout and stderr to a file
command > output.txt 2>&1
command &> output.txt          # Shorthand (bash 4+)

# Redirect stderr to stdout (combine streams)
command 2>&1

# Discard output (send to null device)
command > /dev/null
command > /dev/null 2>&1       # Discard everything
```

### 4.3 Pipes

**Pipes** (`|`) connect the stdout of one command to the stdin of the next, creating powerful command chains.

```bash
# Basic pipe
ls -la | less                          # Page through ls output
ps aux | grep nginx                    # Filter processes

# Chain multiple commands
cat /etc/passwd | cut -d: -f1 | sort   # Get sorted usernames
du -sh /* | sort -rh | head -10        # Largest root dirs
ls -la | awk '{print $9, $5}' | sort   # Names and sizes

# Count occurrences
cat access.log | grep "404" | wc -l
ls *.txt | wc -l                       # Count .txt files
```

### 4.4 Here Documents and Here Strings

```bash
# Here document — multi-line input to a command
cat << EOF
Line 1
Line 2
Line 3
EOF

# Here document to create a file
cat > config.txt << EOF
[settings]
debug=true
port=8080
EOF

# Here string — single string as stdin
grep "pattern" <<< "string to search in"
bc <<< "5 * 8 + 3"                     # Quick calculation
```

### 4.5 tee — Write to File and stdout Simultaneously

```bash
# Write output to file AND display on screen
command | tee output.txt
ls -la | tee listing.txt

# Append mode
command | tee -a output.txt

# Send to multiple files
command | tee file1.txt file2.txt
```

---

## 5. Searching and Finding

### 5.1 find — Search Files by Attributes

```bash
# Find by name
find /home -name "*.txt"
find . -name "config.yml"              # In current directory
find / -name "passwd" -type f          # Files named passwd

# Case-insensitive name search
find /home -iname "readme*"

# Find by type
find /etc -type f                      # Files only
find /var -type d                      # Directories only
find /dev -type l                      # Symbolic links only

# Find by size
find / -size +100M                     # Larger than 100MB
find / -size -1k                       # Smaller than 1KB
find . -size +10M -size -100M          # Between 10MB and 100MB

# Find by modification time
find /var/log -mtime -7                # Modified in last 7 days
find /tmp -mtime +30                   # Modified more than 30 days ago
find . -newer reference.txt            # Modified more recently than reference

# Find by permissions
find / -perm 777                       # Exactly 777
find / -perm -u+s                      # SUID files (security check)

# Find by owner
find /home -user alice
find /var -group www-data

# Execute a command on found files
find . -name "*.log" -exec rm {} \;    # Delete all .log files
find . -name "*.txt" -exec grep "error" {} \;
find . -type f -exec chmod 644 {} \;   # Set permissions

# Find and print with details
find /etc -name "*.conf" -ls
```

### 5.2 locate — Fast Filename Search

```bash
# Search for file by name (uses a database)
locate passwd
locate "*.conf"

# Update the locate database
sudo updatedb

# Case-insensitive search
locate -i readme

# Limit results
locate -n 10 python
```

!!! info
    `locate` uses a pre-built database and is much faster than `find`, but may be out of date. Run `sudo updatedb` to refresh it. New files won't appear until the database is updated.

### 5.3 grep — Search File Contents

```bash
# Basic search
grep "pattern" file.txt
grep "error" /var/log/syslog

# Case-insensitive
grep -i "error" file.txt

# Recursive search in directory
grep -r "TODO" /home/user/projects/

# Show line numbers
grep -n "pattern" file.txt

# Invert match (lines NOT containing pattern)
grep -v "debug" logfile.txt

# Count matching lines
grep -c "error" logfile.txt

# Show only the matching part
grep -o "pattern" file.txt

# Extended regex
grep -E "error|warning|critical" log.txt
grep -E "^[0-9]{4}-" logfile.txt       # Lines starting with 4 digits

# Show context around matches
grep -A 3 "error" log.txt              # 3 lines after
grep -B 2 "error" log.txt              # 2 lines before
grep -C 2 "error" log.txt              # 2 lines around

# Search compressed files
zgrep "pattern" file.gz
```

### 5.4 which, whereis, whatis

```bash
# Find location of a command executable
which python3
which -a python3           # All matches in PATH

# Find binary, source, and man page locations
whereis ls
whereis -b python3         # Binary only

# Brief description of a command
whatis ls
whatis grep
whatis -w "ls*"            # Wildcard search

# Detailed command info
info ls
man ls
```

---

## 6. Aliases and Shell History

### 6.1 Creating Aliases

```bash
# Temporary alias (current session only)
alias ll='ls -lh --color=auto'
alias la='ls -lah'
alias grep='grep --color=auto'
alias ..='cd ..'
alias ...='cd ../..'
alias cls='clear'

# View all current aliases
alias

# Remove an alias
unalias ll

# Persistent aliases — add to ~/.bashrc
echo "alias ll='ls -lh --color=auto'" >> ~/.bashrc
source ~/.bashrc                       # Reload without logging out
```

### 6.2 Useful Default Aliases to Add

```bash
# Add to ~/.bashrc
alias ll='ls -lh --color=auto'
alias la='ls -lah --color=auto'
alias l='ls -CF'
alias ..='cd ..'
alias ...='cd ../..'
alias grep='grep --color=auto'
alias df='df -h'
alias du='du -h'
alias free='free -h'
alias ps='ps auxf'
alias ports='ss -tulnp'
alias myip='curl ifconfig.me'
alias update='sudo apt update && sudo apt upgrade -y'
```

### 6.3 Command History

```bash
# View command history
history
history 20                             # Last 20 commands
history | grep ssh                     # Search history

# Re-run commands from history
!!                                     # Repeat last command
!n                                     # Run command number n
!ssh                                   # Run last command starting with "ssh"
!$                                     # Last argument of previous command
!^                                     # First argument of previous command

# Reverse history search (most useful)
Ctrl + R                               # Type to search, Enter to run, Ctrl+R again for next match

# History settings in ~/.bashrc
export HISTSIZE=10000                  # Commands to keep in memory
export HISTFILESIZE=20000              # Commands to keep in file
export HISTCONTROL=ignoredups          # Skip duplicate entries
export HISTTIMEFORMAT="%F %T "        # Add timestamps

# Clear history
history -c                             # Clear session history
history -w                             # Write history to file
```

---

## 7. Job Control

### 7.1 Foreground and Background Jobs

```bash
# Run a command in the background
command &
sleep 100 &

# Send a running foreground job to background
Ctrl + Z                               # Suspend (pause) the job
bg                                     # Resume it in background
bg %1                                  # Resume job number 1

# Bring a background job to foreground
fg
fg %1                                  # Bring job number 1 to foreground

# List current jobs
jobs
jobs -l                                # With PIDs
```

### 7.2 nohup — Survive Shell Exit

```bash
# Run command that continues after logout
nohup command &
nohup python3 server.py &

# Output goes to nohup.out by default
nohup command > output.log 2>&1 &

# Disown a background job (detach from shell)
command &
disown %1
```

### 7.3 Killing Processes

```bash
# Kill by PID
kill PID
kill -9 PID                            # Force kill (SIGKILL)
kill -15 PID                           # Graceful terminate (SIGTERM, default)
kill -l                                # List all signals

# Kill by name
pkill process_name
pkill -9 firefox
killall process_name

# Interactive process viewer
htop                                   # Press F9 or k to kill
```

### 7.4 Common Signals

| Signal | Number | Meaning |
|--------|--------|---------|
| `SIGTERM` | 15 | Graceful termination (default) |
| `SIGKILL` | 9 | Force kill (cannot be caught) |
| `SIGHUP` | 1 | Hangup — reload config |
| `SIGINT` | 2 | Interrupt (Ctrl+C) |
| `SIGSTOP` | 19 | Pause process (Ctrl+Z) |
| `SIGCONT` | 18 | Continue paused process |

---

## 8. Environment Variables

### 8.1 Viewing and Setting Variables

```bash
# Print all environment variables
env
printenv

# Print a specific variable
echo $HOME
echo $PATH
echo $USER
printenv PATH                          # Without $ sign

# Set a variable (current session only)
MY_VAR="hello"
echo $MY_VAR

# Export variable to child processes
export MY_VAR="hello"
export PATH="$PATH:/opt/myapp/bin"     # Append to PATH

# Unset a variable
unset MY_VAR
```

### 8.2 Important Environment Variables

| Variable | Description | Example Value |
|----------|-------------|---------------|
| `HOME` | Current user's home directory | `/home/alice` |
| `USER` | Current username | `alice` |
| `PATH` | Colon-separated list of executable dirs | `/usr/bin:/usr/local/bin:...` |
| `SHELL` | Current shell path | `/bin/bash` |
| `PWD` | Current working directory | `/home/alice/docs` |
| `OLDPWD` | Previous working directory | `/home/alice` |
| `EDITOR` | Default text editor | `vim` |
| `VISUAL` | Default visual editor | `nano` |
| `LANG` | System locale/language | `en_US.UTF-8` |
| `TERM` | Terminal type | `xterm-256color` |
| `HOSTNAME` | Machine hostname | `myserver` |
| `UID` | Current user ID | `1000` |
| `EUID` | Effective user ID | `1000` |
| `HISTFILE` | History file location | `~/.bash_history` |

### 8.3 Persistent Environment Variables

```bash
# User-level — add to ~/.bashrc or ~/.bash_profile
export EDITOR=vim
export PATH="$PATH:/opt/myapp/bin"
export JAVA_HOME="/usr/lib/jvm/java-17-openjdk"

# System-wide — add to /etc/environment (no export keyword)
EDITOR=vim
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin"

# Or add a file in /etc/profile.d/
sudo nano /etc/profile.d/myapp.sh
# Content: export MYAPP_HOME=/opt/myapp
```

---

## 9. Keyboard Shortcuts

### 9.1 Essential Bash Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl + C` | Interrupt (kill) running command |
| `Ctrl + Z` | Suspend (pause) running command |
| `Ctrl + D` | Send EOF / logout of shell |
| `Ctrl + L` | Clear screen (same as `clear`) |
| `Ctrl + A` | Move cursor to start of line |
| `Ctrl + E` | Move cursor to end of line |
| `Ctrl + U` | Delete from cursor to start of line |
| `Ctrl + K` | Delete from cursor to end of line |
| `Ctrl + W` | Delete word before cursor |
| `Ctrl + Y` | Paste (yank) last deleted text |
| `Ctrl + R` | Reverse history search |
| `Ctrl + G` | Cancel history search |
| `Alt + F` | Move forward one word |
| `Alt + B` | Move backward one word |
| `Alt + D` | Delete word after cursor |
| `Tab` | Autocomplete command or filename |
| `Tab Tab` | Show all completions |
| `↑ / ↓` | Navigate command history |

### 9.2 Terminal Multiplexer Shortcuts (tmux)

```bash
# Start tmux
tmux
tmux new -s mysession           # Named session

# Prefix key: Ctrl + B (then release, then shortcut)
Ctrl+B  c        # New window
Ctrl+B  n        # Next window
Ctrl+B  p        # Previous window
Ctrl+B  ,        # Rename window
Ctrl+B  %        # Split vertically
Ctrl+B  "        # Split horizontally
Ctrl+B  arrows   # Switch panes
Ctrl+B  d        # Detach session
Ctrl+B  [        # Enter scroll/copy mode (q to exit)

# Attach to existing session
tmux attach
tmux attach -t mysession

# List sessions
tmux ls
```

---

## 📋 Quick Reference Cheat Sheet

```bash
# --- Navigation ---
pwd                            # Current directory
ls -lah                        # Detailed listing with hidden files
cd -                           # Previous directory
cd ~                           # Home directory

# --- Files ---
touch file.txt                 # Create empty file
mkdir -p dir/subdir            # Create nested directory
cp -r src/ dest/               # Copy directory recursively
mv old.txt new.txt             # Move / rename
rm -rf directory/              # Remove directory (careful!)
stat file.txt                  # File metadata

# --- Viewing ---
cat file.txt                   # Print file
less file.txt                  # Page through file
head -n 20 file.txt            # First 20 lines
tail -f /var/log/syslog        # Follow log in real time
wc -l file.txt                 # Count lines

# --- Redirection ---
command > file.txt             # Redirect stdout (overwrite)
command >> file.txt            # Redirect stdout (append)
command 2> errors.txt          # Redirect stderr
command &> all.txt             # Redirect stdout + stderr
command | tee file.txt         # Write to file AND stdout

# --- Searching ---
find . -name "*.txt"           # Find by name
find / -size +100M             # Find large files
grep -r "pattern" dir/         # Recursive content search
grep -n "error" log.txt        # Show line numbers
locate filename                # Fast filename search

# --- Jobs ---
command &                      # Run in background
Ctrl+Z                         # Suspend job
bg                             # Resume in background
fg                             # Bring to foreground
jobs                           # List jobs
kill %1                        # Kill job 1
nohup command &                # Survive shell exit

# --- History ---
history 20                     # Last 20 commands
Ctrl+R                         # Reverse search
!!                             # Repeat last command
!$                             # Last argument of last command

# --- Environment ---
echo $PATH                     # Print PATH
export MY_VAR=value            # Set env variable
env                            # List all variables
```

| Command | Description |
|---------|-------------|
| `pwd` | Print working directory |
| `ls -lah` | List all files with details |
| `cd -` | Go to previous directory |
| `find . -name "*.log"` | Find files by name |
| `grep -r "text" dir/` | Recursive text search |
| `command > file` | Redirect output to file |
| `cmd1 \| cmd2` | Pipe output to next command |
| `tail -f logfile` | Follow log file live |
| `Ctrl+R` | Search command history |
| `nohup cmd &` | Run command after logout |