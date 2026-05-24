# 🧑‍💻 User Environment

> The **user environment** is the collection of settings, variables, files, and configurations that define how a user interacts with the Linux shell. Every login session loads a personalized environment — from the PATH that finds commands, to aliases that shorten them, to the prompt that greets you.

---

## 1. Users and Accounts

### 1.1 Types of Users

| Type | UID Range | Description |
|---|---|---|
| **Root** | 0 | Superuser — unrestricted access to everything |
| **System users** | 1–999 | Services and daemons (no login shell) |
| **Regular users** | 1000+ | Human accounts with home directories |

```bash
id                         # Show your UID, GID, and groups
id username                # Info for another user
whoami                     # Print current username
who                        # Who is logged in (with terminal and time)
w                          # Who is logged in + what they're doing
last                       # Login history
lastb                      # Failed login attempts
finger username            # Detailed user info (if installed)
```

### 1.2 `/etc/passwd` — User Database

Each line represents one user account:

```
username:password:UID:GID:GECOS:home_dir:shell
alice:x:1001:1001:Alice Smith:/home/alice:/bin/bash
```

| Field | Meaning |
|---|---|
| `username` | Login name |
| `password` | `x` means password is in `/etc/shadow` |
| `UID` | User ID number |
| `GID` | Primary group ID |
| `GECOS` | Full name / description (comma-separated info) |
| `home_dir` | Home directory path |
| `shell` | Default login shell |

```bash
cat /etc/passwd            # View all accounts
grep alice /etc/passwd     # Find specific user
getent passwd alice        # Query via NSS (works with LDAP too)
```

### 1.3 `/etc/shadow` — Password Database

Stores hashed passwords — readable only by root.

```
alice:$6$hash...:19000:0:99999:7:::
```

| Field | Meaning |
|---|---|
| Username | Login name |
| Hashed password | `$6$` = SHA-512 hash |
| Last change | Days since epoch of last password change |
| Min age | Minimum days before password can change |
| Max age | Days until password expires |
| Warning | Days before expiry to warn user |
| Inactive | Days after expiry before account locked |
| Expire | Absolute account expiry date |

### 1.4 Managing Users

```bash
# Creating users
sudo useradd alice                      # Create user (basic)
sudo useradd -m -s /bin/bash alice      # With home dir and bash shell
sudo useradd -m -G sudo,devs alice      # Add to groups on creation
sudo adduser alice                      # Interactive version (Debian/Ubuntu)

# Setting passwords
sudo passwd alice                       # Set/change alice's password
passwd                                  # Change your own password

# Modifying users
sudo usermod -s /bin/zsh alice          # Change shell
sudo usermod -aG docker alice           # Add to group (append, keep existing)
sudo usermod -l newname alice           # Rename user
sudo usermod -d /new/home -m alice      # Move home directory
sudo usermod -L alice                   # Lock account
sudo usermod -U alice                   # Unlock account
sudo usermod -e 2025-12-31 alice        # Set account expiry

# Deleting users
sudo userdel alice                      # Delete user (keep home dir)
sudo userdel -r alice                   # Delete user + home directory

# Password aging
sudo chage -l alice                     # List password aging info
sudo chage -M 90 alice                  # Set max password age to 90 days
sudo chage -E 2025-12-31 alice          # Set account expiry date
sudo chage -d 0 alice                   # Force password change on next login
```

---

## 2. Groups

### 2.1 `/etc/group` — Group Database

```
groupname:password:GID:member1,member2
developers:x:1002:alice,bob,charlie
```

```bash
groups                     # List groups current user belongs to
groups alice               # List groups for alice
id                         # Shows all group memberships
cat /etc/group             # View all groups
getent group developers    # Query specific group
```

### 2.2 Managing Groups

```bash
sudo groupadd developers           # Create group
sudo groupadd -g 1500 devops       # Create with specific GID
sudo groupmod -n newname oldname   # Rename group
sudo groupdel developers           # Delete group
sudo gpasswd -a alice developers   # Add alice to group
sudo gpasswd -d alice developers   # Remove alice from group
sudo gpasswd -A alice developers   # Make alice group admin
newgrp developers                  # Switch primary group (temporary)
```

---

## 3. The Shell

### 3.1 What is a Shell?

The shell is a command-line interpreter — the interface between the user and the kernel. It reads commands, interprets them, and executes programs.

### 3.2 Common Shells

| Shell | Path | Description |
|---|---|---|
| **bash** | `/bin/bash` | Bourne Again Shell — most common default |
| **sh** | `/bin/sh` | POSIX shell (often symlink to dash or bash) |
| **zsh** | `/bin/zsh` | Z Shell — interactive features, Oh My Zsh |
| **fish** | `/usr/bin/fish` | Friendly Interactive Shell |
| **dash** | `/bin/dash` | Lightweight POSIX sh (fast startup) |
| **ksh** | `/bin/ksh` | KornShell — common on AIX/Solaris |
| **tcsh** | `/bin/tcsh` | C-shell with enhancements |

```bash
echo $SHELL                # Your current login shell
echo $0                    # Current running shell
cat /etc/shells            # List of valid login shells
chsh -s /bin/zsh           # Change your login shell
chsh -s /bin/zsh alice     # Change another user's shell (root)
```

### 3.3 Login vs Non-Login Shell

| Type | When | Files Loaded |
|---|---|---|
| **Login shell** | SSH login, `su -`, console login | `/etc/profile` → `~/.bash_profile` → `~/.bash_login` → `~/.profile` |
| **Interactive non-login** | Terminal emulator, new bash tab | `/etc/bash.bashrc` → `~/.bashrc` |
| **Non-interactive** | Scripts, cron jobs | `$BASH_ENV` (if set) |

```bash
# Check if current shell is a login shell
shopt -q login_shell && echo "Login shell" || echo "Non-login shell"

# Force login shell
bash --login
bash -l
su - username              # Login shell for another user
```

### 3.4 Interactive vs Non-Interactive Shell

```bash
# Check if interactive
[[ $- == *i* ]] && echo "Interactive" || echo "Non-interactive"
```

---

## 4. Environment Variables

### 4.1 What are Environment Variables?

Environment variables are named key-value pairs stored in the shell's environment. They configure behavior for the shell and programs.

```bash
env                        # Show all environment variables
printenv                   # Same as env
printenv HOME              # Print value of specific variable
printenv PATH HOME SHELL   # Multiple variables
set                        # Show all variables (including shell-only)
```

### 4.2 Setting Variables

```bash
# Shell variable (current shell only, not exported to children)
MYVAR="hello"
echo $MYVAR

# Environment variable (exported to child processes)
export MYVAR="hello"
export MYVAR                       # Export existing variable
declare -x MYVAR="hello"           # Same as export

# Set and export in one line
export EDITOR=vim
export PATH="$PATH:/opt/myapp/bin"

# Temporary variable for one command only
LANG=C ls                          # Run ls with LANG=C
DEBUG=1 ./script.sh                # Pass variable to script
```

### 4.3 Unsetting Variables

```bash
unset MYVAR                # Remove variable from environment
unset -f myfunc            # Remove function
```

### 4.4 Key Built-in Environment Variables

| Variable | Description |
|---|---|
| `HOME` | Current user's home directory (`~`) |
| `USER` | Current username |
| `LOGNAME` | Login name (may differ from USER in su) |
| `SHELL` | Path to login shell |
| `PATH` | Colon-separated list of command search directories |
| `PWD` | Current working directory |
| `OLDPWD` | Previous working directory |
| `EDITOR` | Default text editor (used by git, cron, etc.) |
| `VISUAL` | Default visual (full-screen) editor |
| `PAGER` | Default pager (less, more) |
| `LANG` | System locale / language |
| `LC_ALL` | Override all locale settings |
| `TZ` | Timezone (e.g., `America/New_York`) |
| `TERM` | Terminal type (e.g., `xterm-256color`) |
| `DISPLAY` | X11 display (e.g., `:0`) |
| `HISTFILE` | Path to bash history file |
| `HISTSIZE` | Number of commands in memory |
| `HISTFILESIZE` | Number of commands saved to file |
| `PS1` | Primary shell prompt string |
| `PS2` | Secondary prompt (continuation) |
| `RANDOM` | Random number between 0–32767 |
| `SECONDS` | Seconds since shell started |
| `LINENO` | Current line number in script |
| `PPID` | Parent process ID |
| `UID` | Current user's UID |
| `EUID` | Effective UID |
| `HOSTNAME` | Machine hostname |
| `IFS` | Internal Field Separator (default: space, tab, newline) |
| `CDPATH` | Search path for `cd` command |

### 4.5 The PATH Variable

`PATH` is the most important environment variable — it tells the shell where to look for commands.

```bash
echo $PATH
# /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/home/alice/.local/bin

# Add a directory to PATH
export PATH="$PATH:/opt/myapp/bin"          # Append (lower priority)
export PATH="/opt/myapp/bin:$PATH"          # Prepend (higher priority)

# Add to PATH permanently (add to ~/.bashrc)
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc

# Which directory is a command found in?
which python3
type -a python3            # Shows all matches
command -v python3         # POSIX-compatible which
```

!!! warning "PATH Security"
    Never add `.` (current directory) to PATH — it allows malicious scripts named after common commands to be executed accidentally. Always use `./script.sh` to run scripts in the current directory.

---

## 5. Shell Startup Files

### 5.1 Bash Startup File Order

```
Login Shell startup:
  /etc/profile
        │
        ├── /etc/profile.d/*.sh   (sourced by /etc/profile)
        │
  ~/.bash_profile  (or ~/.bash_login, or ~/.profile — first found wins)
        │
        └── usually sources ~/.bashrc

Interactive Non-Login Shell startup:
  /etc/bash.bashrc  (or /etc/bashrc)
        │
  ~/.bashrc

Logout:
  ~/.bash_logout
```

### 5.2 Key Startup Files

| File | Scope | When Loaded |
|---|---|---|
| `/etc/profile` | System-wide | Login shell |
| `/etc/profile.d/*.sh` | System-wide | Sourced by `/etc/profile` |
| `/etc/bash.bashrc` | System-wide | Interactive bash |
| `~/.bash_profile` | User | Login shell (bash) |
| `~/.bash_login` | User | Login shell (if no `.bash_profile`) |
| `~/.profile` | User | Login shell (if no `.bash_profile` or `.bash_login`) |
| `~/.bashrc` | User | Interactive non-login bash |
| `~/.bash_logout` | User | On logout from login shell |
| `~/.bash_history` | User | Command history storage |

### 5.3 Sourcing Files

```bash
source ~/.bashrc           # Reload bashrc without logging out
. ~/.bashrc                # Same (dot command — POSIX compatible)
source ~/.bash_profile     # Reload bash_profile
```

### 5.4 Typical `~/.bashrc` Structure

```bash
# ~/.bashrc — example

# ── Don't run if non-interactive ────────────────────────────────────
[[ $- != *i* ]] && return

# ── Environment Variables ────────────────────────────────────────────
export EDITOR=vim
export VISUAL=vim
export PAGER=less
export HISTSIZE=10000
export HISTFILESIZE=20000
export HISTCONTROL=ignoredups:erasedups   # No duplicate history
export HISTTIMEFORMAT="%F %T "           # Timestamps in history

# ── PATH ─────────────────────────────────────────────────────────────
export PATH="$HOME/.local/bin:$HOME/bin:$PATH"

# ── Aliases ──────────────────────────────────────────────────────────
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
alias grep='grep --color=auto'
alias df='df -h'
alias du='du -h'
alias cp='cp -i'
alias mv='mv -i'
alias rm='rm -i'
alias ..='cd ..'
alias ...='cd ../..'
alias update='sudo apt update && sudo apt upgrade'

# ── Functions ────────────────────────────────────────────────────────
mkcd() { mkdir -p "$1" && cd "$1"; }
backup() { cp "$1" "$1.bak.$(date +%Y%m%d)"; }

# ── Prompt ───────────────────────────────────────────────────────────
PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '

# ── Completion ───────────────────────────────────────────────────────
if [ -f /usr/share/bash-completion/bash_completion ]; then
    . /usr/share/bash-completion/bash_completion
fi
```

### 5.5 Typical `~/.bash_profile` Structure

```bash
# ~/.bash_profile — runs for login shells

# Load .bashrc if it exists and we're interactive
if [ -f ~/.bashrc ]; then
    source ~/.bashrc
fi

# Login-specific environment (not needed in every shell)
export PATH="$HOME/bin:$PATH"
umask 022
```

---

## 6. Aliases

### 6.1 Creating Aliases

```bash
alias ll='ls -alF'                     # Create alias
alias grep='grep --color=auto'         # Override command with options
alias update='sudo apt update && sudo apt upgrade'
alias myip='curl ifconfig.me'
alias ports='ss -tulnp'
alias alert='notify-send --urgency=low "Command finished"'

alias                                  # List all aliases
alias ll                               # Show definition of one alias
unalias ll                             # Remove alias
unalias -a                             # Remove all aliases

# Bypass an alias (run the real command)
\ls                                    # Backslash prefix
command ls                             # command builtin
'ls'                                   # Quoted command
```

### 6.2 Useful Alias Examples

```bash
# Safety aliases
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# Navigation
alias ..='cd ..'
alias ...='cd ../..'
alias ....='cd ../../..'
alias ~='cd ~'

# Listing
alias ll='ls -alFh'
alias la='ls -A'
alias lsd='ls -d */'                   # Directories only

# System info
alias meminfo='free -h'
alias cpuinfo='lscpu'
alias diskinfo='df -h'
alias ports='ss -tulnp'
alias myip='curl -s ifconfig.me'
alias localip='hostname -I'

# Git shortcuts
alias gs='git status'
alias ga='git add'
alias gc='git commit'
alias gp='git push'
alias gl='git log --oneline --graph'

# Safety and convenience
alias please='sudo'
alias fucking='sudo'                   # Humor alias
alias c='clear'
alias h='history'
alias j='jobs -l'
```

### 6.3 Making Aliases Permanent

Add aliases to `~/.bashrc` (for interactive shells) then reload:

```bash
echo "alias ll='ls -alFh'" >> ~/.bashrc
source ~/.bashrc
```

---

## 7. Shell Functions

Functions are more powerful than aliases — they can accept arguments, use logic, and run multiple commands.

### 7.1 Defining Functions

```bash
# Syntax 1
function greet() {
    echo "Hello, $1!"
}

# Syntax 2 (POSIX compatible)
greet() {
    echo "Hello, $1!"
}

greet Alice                # Call function: Hello, Alice!
```

### 7.2 Function Arguments

```bash
myfunc() {
    echo "Function name: $0"
    echo "First arg: $1"
    echo "Second arg: $2"
    echo "All args: $@"
    echo "Arg count: $#"
}
```

### 7.3 Useful Function Examples

```bash
# Create directory and cd into it
mkcd() {
    mkdir -p "$1" && cd "$1"
}

# Extract any archive
extract() {
    case "$1" in
        *.tar.gz|*.tgz)  tar -xzvf "$1" ;;
        *.tar.bz2|*.tbz) tar -xjvf "$1" ;;
        *.tar.xz)        tar -xJvf "$1" ;;
        *.tar)           tar -xvf  "$1" ;;
        *.gz)            gunzip    "$1" ;;
        *.bz2)           bunzip2   "$1" ;;
        *.zip)           unzip     "$1" ;;
        *.7z)            7z x      "$1" ;;
        *)               echo "Unknown archive: $1" ;;
    esac
}

# Backup a file with datestamp
backup() {
    cp "$1" "$1.bak.$(date +%Y%m%d_%H%M%S)"
    echo "Backup created: $1.bak.$(date +%Y%m%d_%H%M%S)"
}

# Quick search in files
ftext() {
    grep -rn "$1" .
}

# Show top N memory-consuming processes
topmem() {
    ps aux --sort=-%mem | head -"${1:-10}"
}

# Go up N directories
up() {
    local d=""
    for ((i=0; i<${1:-1}; i++)); do d="../$d"; done
    cd "$d" || return
}
```

### 7.4 Listing and Removing Functions

```bash
declare -f                 # List all defined functions with bodies
declare -F                 # List all function names only
declare -f greet           # Show definition of specific function
unset -f greet             # Remove function
type greet                 # Show function type and definition
```

---

## 8. The Shell Prompt (PS1)

### 8.1 Prompt Variables

| Variable | Description |
|---|---|
| `PS1` | Primary prompt (shown before each command) |
| `PS2` | Continuation prompt (shown when command spans lines) |
| `PS3` | Prompt for `select` statements |
| `PS4` | Trace prompt (shown with `set -x`) |

### 8.2 PS1 Escape Sequences

| Escape | Expands to |
|---|---|
| `\u` | Current username |
| `\h` | Hostname (up to first `.`) |
| `\H` | Full hostname |
| `\w` | Current working directory (full path) |
| `\W` | Basename of current directory |
| `\d` | Date (e.g., `Mon May 10`) |
| `\t` | Time in 24h HH:MM:SS |
| `\T` | Time in 12h HH:MM:SS |
| `\@` | Time in 12h am/pm |
| `\n` | Newline |
| `\$` | `#` if root, `$` otherwise |
| `\!` | History number of this command |
| `\#` | Command number of this command |
| `\j` | Number of background jobs |
| `\s` | Shell name |
| `\v` | Bash version |
| `\\` | Literal backslash |

### 8.3 ANSI Color Codes in PS1

```bash
# Format: \[\033[CODEm\]text\[\033[00m\]
# Codes: 30-37 = foreground colors, 40-47 = background, 01 = bold

# Colors
Black='\[\033[0;30m\]'
Red='\[\033[0;31m\]'
Green='\[\033[0;32m\]'
Yellow='\[\033[0;33m\]'
Blue='\[\033[0;34m\]'
Purple='\[\033[0;35m\]'
Cyan='\[\033[0;36m\]'
White='\[\033[0;37m\]'
Bold='\[\033[1m\]'
Reset='\[\033[0m\]'
```

### 8.4 PS1 Examples

```bash
# Minimal: user@host:dir$
PS1='\u@\h:\w\$ '

# Colorful: green user@host, blue path
PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '

# Two-line prompt with time
PS1='\[\033[0;33m\][\t]\[\033[0m\] \[\033[01;32m\]\u@\h\[\033[0m\]:\[\033[01;34m\]\w\[\033[0m\]\n\$ '

# Show git branch in prompt (requires git-aware function)
parse_git_branch() {
    git branch 2>/dev/null | grep '\*' | sed 's/\* //'
}
PS1='\u@\h:\w\[\033[0;33m\]$(parse_git_branch)\[\033[0m\]\$ '

# Root warning — red prompt for root
if [ "$EUID" -eq 0 ]; then
    PS1='\[\033[01;31m\]\u@\h:\w\#\[\033[00m\] '
else
    PS1='\[\033[01;32m\]\u@\h:\w\$\[\033[00m\] '
fi
```

---

## 9. Command History

### 9.1 History Basics

```bash
history                    # Show command history
history 20                 # Show last 20 commands
history -c                 # Clear history for current session
history -w                 # Write history to file now
history -r                 # Read history from file
!!                         # Repeat last command
!50                        # Run command #50 from history
!-2                        # Run 2nd-to-last command
!ssh                       # Run last command starting with "ssh"
!?pattern                  # Run last command containing pattern
^old^new                   # Quick substitution in last command
```

### 9.2 History Search

```bash
Ctrl+R                     # Reverse incremental history search
Ctrl+R (again)             # Find previous match
Ctrl+G                     # Cancel history search
Ctrl+P                     # Previous command (like ↑)
Ctrl+N                     # Next command (like ↓)
```

### 9.3 History Configuration

```bash
# In ~/.bashrc:
export HISTSIZE=10000              # Commands stored in memory
export HISTFILESIZE=20000          # Lines stored in file
export HISTFILE=~/.bash_history    # History file location
export HISTCONTROL=ignoredups      # Skip duplicate consecutive commands
export HISTCONTROL=ignorespace     # Skip commands starting with space
export HISTCONTROL=ignoreboth      # Both of the above
export HISTCONTROL=erasedups       # Remove all older duplicates
export HISTTIMEFORMAT="%F %T "     # Add timestamps

# Ignore specific commands from history
export HISTIGNORE="ls:ll:cd:pwd:clear:history"

# Append to history on exit (instead of overwriting)
shopt -s histappend

# Save history immediately after each command
PROMPT_COMMAND="history -a; history -c; history -r"
```

### 9.4 History Expansion

| Expansion | Meaning |
|---|---|
| `!!` | Last command |
| `!n` | Command number n |
| `!-n` | n-th previous command |
| `!string` | Last command starting with string |
| `!?string?` | Last command containing string |
| `!!:s/old/new` | Substitute in last command |
| `^old^new` | Quick substitute in last command |
| `!!:$` | Last argument of last command |
| `!$` | Last argument of last command |
| `!^` | First argument of last command |
| `!*` | All arguments of last command |
| `Alt+.` | Insert last argument of previous command |

---

## 10. Shell Options (`shopt` and `set`)

### 10.1 `shopt` — Shell Options

```bash
shopt                      # List all options and their state
shopt -s histappend        # Enable option
shopt -u histappend        # Disable option
shopt -q histappend        # Query (exit code 0=on, 1=off)
```

**Useful `shopt` options:**

| Option | Description |
|---|---|
| `histappend` | Append to history file instead of overwrite |
| `cdspell` | Auto-correct minor typos in `cd` paths |
| `dirspell` | Auto-correct directory name spelling in completion |
| `autocd` | Type directory name to cd into it |
| `globstar` | `**` matches files in all subdirectories |
| `nocaseglob` | Case-insensitive filename globbing |
| `checkwinsize` | Update `LINES`/`COLUMNS` after each command |
| `cmdhist` | Save multi-line commands as single history entry |
| `dotglob` | Include hidden files in glob patterns |
| `extglob` | Enable extended globbing patterns |
| `nullglob` | Unmatched globs expand to empty (not literal) |

### 10.2 `set` — Shell Behavior

```bash
set -e                     # Exit immediately on error
set -u                     # Treat unset variables as errors
set -x                     # Print each command before executing (debug)
set -o pipefail            # Pipe fails if any command fails
set +x                     # Disable debug mode
set -o vi                  # Use vi keybindings in shell
set -o emacs               # Use emacs keybindings (default)
set -n                     # Read commands but don't execute (syntax check)
```

---

## 11. Keyboard Shortcuts (readline)

The bash command line uses GNU readline for editing. These shortcuts work in bash, Python REPL, MySQL, and many other interactive tools.

### 11.1 Movement

| Shortcut | Action |
|---|---|
| `Ctrl+A` | Move to beginning of line |
| `Ctrl+E` | Move to end of line |
| `Ctrl+F` | Move forward one character |
| `Ctrl+B` | Move backward one character |
| `Alt+F` | Move forward one word |
| `Alt+B` | Move backward one word |
| `Ctrl+XX` | Toggle between current and line start |

### 11.2 Editing

| Shortcut | Action |
|---|---|
| `Ctrl+D` | Delete character under cursor (or EOF if empty) |
| `Ctrl+H` | Delete character before cursor (Backspace) |
| `Alt+D` | Delete word forward |
| `Alt+Backspace` | Delete word backward |
| `Ctrl+K` | Kill (cut) from cursor to end of line |
| `Ctrl+U` | Kill from cursor to beginning of line |
| `Ctrl+W` | Kill word backward |
| `Ctrl+Y` | Yank (paste) last killed text |
| `Alt+Y` | Yank previous kill (cycle) |
| `Ctrl+T` | Transpose characters |
| `Alt+T` | Transpose words |
| `Alt+U` | Uppercase word |
| `Alt+L` | Lowercase word |
| `Alt+C` | Capitalize word |

### 11.3 History and Control

| Shortcut | Action |
|---|---|
| `Ctrl+P` | Previous command (up arrow) |
| `Ctrl+N` | Next command (down arrow) |
| `Ctrl+R` | Reverse search history |
| `Ctrl+G` | Cancel history search |
| `Ctrl+C` | Cancel current command |
| `Ctrl+Z` | Suspend current process |
| `Ctrl+L` | Clear screen (like `clear`) |
| `Ctrl+S` | Pause output (XOFF) |
| `Ctrl+Q` | Resume output (XON) |
| `Ctrl+D` | Log out (if line empty) |
| `Tab` | Auto-complete command, file, or variable |
| `Tab Tab` | Show all completion options |

---

## 12. Tab Completion

### 12.1 Built-in Completion

Bash can auto-complete:
- Commands and executables in PATH
- Filenames and directories
- Environment variable names (after `$`)
- Usernames (after `~`)
- Hostnames (after `@`)

```bash
ls /et<Tab>                # Completes to /etc/
echo $HO<Tab>              # Completes to $HOME
cd ~ali<Tab>               # Completes to ~alice
ssh user@host<Tab>         # Completes hostnames from ~/.ssh/known_hosts
```

### 12.2 `bash-completion` Package

```bash
sudo apt install bash-completion       # Install (Debian/Ubuntu)
sudo dnf install bash-completion       # Install (Fedora)

# Enable in ~/.bashrc
if [ -f /usr/share/bash-completion/bash_completion ]; then
    . /usr/share/bash-completion/bash_completion
fi
```

With this package installed, bash gains context-aware completion for hundreds of commands (`git`, `apt`, `systemctl`, `ssh`, `make`, etc.).

### 12.3 `complete` — Custom Completion

```bash
complete -W "start stop restart status" myservice   # Custom options
complete -f -X '!*.py' python3                      # Only .py files for python3
complete -d cd                                      # Only directories for cd
complete -p                                         # Show all completion specs
complete -r command                                 # Remove completion for command
```

---

## 13. Directory Stack

```bash
pushd /etc                 # Push /etc onto stack and cd to it
pushd /var/log             # Push /var/log and cd
dirs                       # Show directory stack
dirs -v                    # Show stack with indices
popd                       # Pop top of stack and cd there
popd +2                    # Pop item at index 2
cd ~2                      # cd to item at index 2 in stack
```

---

## 14. Locale and Language

### 14.1 Locale Settings

```bash
locale                     # Show all locale settings
locale -a                  # List all available locales
echo $LANG                 # Current language/locale
echo $LC_ALL               # Override for all categories
```

**Common locale categories:**

| Variable | Controls |
|---|---|
| `LANG` | Default for all unset LC_* |
| `LC_COLLATE` | String sorting order |
| `LC_CTYPE` | Character classification |
| `LC_MESSAGES` | Language for messages |
| `LC_NUMERIC` | Number formatting |
| `LC_TIME` | Date/time formatting |
| `LC_MONETARY` | Currency formatting |
| `LC_ALL` | Override all categories |

```bash
# Change locale temporarily
LANG=en_US.UTF-8 date
LC_TIME=de_DE.UTF-8 date

# Generate and set locale
sudo locale-gen en_US.UTF-8
sudo update-locale LANG=en_US.UTF-8

# Set in ~/.bashrc
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
```

### 14.2 Timezone

```bash
date                       # Current date and time
timedatectl                # Full timezone and NTP status
timedatectl list-timezones # All available timezones
sudo timedatectl set-timezone Asia/Kolkata   # Set timezone

# Temporary timezone change
TZ="America/New_York" date
export TZ="Asia/Kolkata"   # Permanent for session

# System timezone file
cat /etc/timezone          # Current timezone
ls -la /etc/localtime      # Symlink to zone file
```

---

## 15. Quick Reference Cheat Sheet

```bash
# ─── Users ───────────────────────────────────────────────────────────
whoami                     # Current username
id                         # UID, GID, groups
who / w                    # Who is logged in
last                       # Login history
sudo useradd -m -s /bin/bash alice    # Create user
sudo passwd alice          # Set password
sudo usermod -aG docker alice         # Add to group
sudo userdel -r alice      # Delete user + home

# ─── Groups ──────────────────────────────────────────────────────────
groups                     # My groups
sudo groupadd developers   # Create group
sudo gpasswd -a alice devs # Add user to group
newgrp developers          # Switch primary group

# ─── Shell ───────────────────────────────────────────────────────────
echo $SHELL                # Current shell
cat /etc/shells            # Available shells
chsh -s /bin/zsh           # Change login shell

# ─── Environment Variables ───────────────────────────────────────────
env                        # Show all env vars
export MYVAR="value"       # Set env variable
unset MYVAR                # Remove variable
echo $PATH                 # Show PATH
export PATH="$HOME/bin:$PATH"  # Add to PATH

# ─── Startup Files ───────────────────────────────────────────────────
source ~/.bashrc           # Reload bashrc
. ~/.bash_profile          # Reload bash_profile (dot command)

# ─── Aliases ─────────────────────────────────────────────────────────
alias ll='ls -alFh'        # Create alias
alias                      # List all aliases
unalias ll                 # Remove alias
\ls                        # Bypass alias

# ─── Functions ───────────────────────────────────────────────────────
mkcd() { mkdir -p "$1" && cd "$1"; }   # Define function
declare -F                 # List all functions
unset -f functionname      # Remove function

# ─── Prompt ──────────────────────────────────────────────────────────
echo $PS1                  # Show current prompt
PS1='\u@\h:\w\$ '          # Set simple prompt

# ─── History ─────────────────────────────────────────────────────────
history                    # Full history
history 20                 # Last 20 commands
!!                         # Repeat last command
!string                    # Last command starting with string
!$                         # Last argument of last command
Ctrl+R                     # Reverse search history
history -c                 # Clear session history

# ─── Keyboard Shortcuts ──────────────────────────────────────────────
Ctrl+A / Ctrl+E            # Start / end of line
Ctrl+K / Ctrl+U            # Kill to end / start of line
Ctrl+Y                     # Paste killed text
Alt+F / Alt+B              # Forward / backward word
Ctrl+L                     # Clear screen
Ctrl+R                     # Reverse history search

# ─── Shell Options ───────────────────────────────────────────────────
shopt -s histappend        # Append history (don't overwrite)
shopt -s cdspell           # Auto-correct cd typos
set -x                     # Debug mode (trace commands)
set +x                     # Disable debug mode

# ─── Locale / Timezone ───────────────────────────────────────────────
locale                     # Show locale settings
timedatectl                # Show timezone/NTP info
sudo timedatectl set-timezone Asia/Kolkata
TZ="America/New_York" date # Temporary timezone
```