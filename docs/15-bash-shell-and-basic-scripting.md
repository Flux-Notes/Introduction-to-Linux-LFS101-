# 🐚 The Bash Shell and Basic Scripting

> **Bash** (Bourne Again SHell) is the default shell on most Linux systems — it's both an interactive command interpreter and a full scripting language. Mastering Bash lets you automate repetitive tasks, build tools, manage systems, and glue together the entire Linux ecosystem with just a text file.

---

## 1. What is Bash?

Bash is a Unix shell and command language written as a free replacement for the Bourne Shell (`sh`). It is the default login shell on most Linux distributions and macOS (pre-Catalina).

| Shell | Path | Notes |
|---|---|---|
| `bash` | `/bin/bash` | Default on most Linux distros |
| `sh` | `/bin/sh` | POSIX shell (may link to bash/dash) |
| `dash` | `/bin/dash` | Faster, minimal POSIX shell (Ubuntu default sh) |
| `zsh` | `/bin/zsh` | Extended, popular on macOS |
| `fish` | `/usr/bin/fish` | User-friendly, non-POSIX |
| `ksh` | `/bin/ksh` | Korn shell |

```bash
echo $SHELL                     # your current login shell
echo $0                         # current shell or script name
cat /etc/shells                 # list of valid shells on system
chsh -s /bin/zsh                # change login shell permanently
bash --version                  # bash version info
```

---

## 2. Interactive Shell Basics

### 2.1 Command History

```bash
history                         # show command history
history 20                      # show last 20 commands
!!                              # re-run last command
!n                              # re-run command number n
!ssh                            # re-run last command starting with "ssh"
!$                              # last argument of previous command
!*                              # all arguments of previous command
^old^new                        # replace "old" with "new" in last command
Ctrl+R                          # reverse search through history
Ctrl+G                          # cancel history search
history -c                      # clear history
history -d 42                   # delete entry 42
```

```bash
# History control (in ~/.bashrc)
HISTSIZE=10000                  # commands kept in memory
HISTFILESIZE=20000              # commands saved to file
HISTFILE=~/.bash_history        # history file location
HISTCONTROL=ignoredups          # don't record duplicate commands
HISTCONTROL=ignorespace         # don't record commands starting with space
HISTCONTROL=ignoreboth          # both of the above
HISTTIMEFORMAT="%F %T "         # timestamp in history
```

### 2.2 Keyboard Shortcuts (Readline)

| Shortcut | Action |
|---|---|
| `Ctrl+A` | Move to beginning of line |
| `Ctrl+E` | Move to end of line |
| `Ctrl+B` | Move back one character |
| `Ctrl+F` | Move forward one character |
| `Alt+B` | Move back one word |
| `Alt+F` | Move forward one word |
| `Ctrl+W` | Delete word before cursor |
| `Ctrl+U` | Delete from cursor to beginning |
| `Ctrl+K` | Delete from cursor to end |
| `Ctrl+Y` | Paste (yank) deleted text |
| `Ctrl+L` | Clear screen |
| `Ctrl+C` | Cancel current command |
| `Ctrl+D` | Exit shell / EOF |
| `Ctrl+Z` | Suspend current process |
| `Tab` | Auto-complete |
| `Tab Tab` | Show all completions |

### 2.3 Job Control

```bash
command &                       # run command in background
jobs                            # list background jobs
jobs -l                         # list with PIDs
fg                              # bring last job to foreground
fg %2                           # bring job 2 to foreground
bg                              # resume suspended job in background
bg %2                           # resume job 2 in background
Ctrl+Z                          # suspend foreground job
kill %1                         # kill job 1
disown %1                       # detach job from shell (survives logout)
nohup command &                 # run immune to hangup (survives logout)
```

---

## 3. Shell Configuration Files

Bash reads different files depending on how it is invoked.

| File | When Read |
|---|---|
| `/etc/profile` | System-wide, login shell |
| `/etc/profile.d/*.sh` | System-wide snippets |
| `~/.bash_profile` | User login shell (read first) |
| `~/.bash_login` | User login shell (if no `.bash_profile`) |
| `~/.profile` | User login shell (if neither above) |
| `~/.bashrc` | Interactive non-login shell |
| `~/.bash_logout` | On logout |
| `~/.inputrc` | Readline key bindings |

```bash
source ~/.bashrc                # reload config without logging out
. ~/.bashrc                     # same (. is alias for source)
bash -x script.sh               # trace execution (debug)
bash -n script.sh               # syntax check only (no execution)
```

!!! tip "Login vs Non-Login Shell"
    A **login shell** (SSH, TTY) reads `.bash_profile`. A **non-login interactive shell** (new terminal tab) reads `.bashrc`. Put environment variables in `.bash_profile`, aliases and functions in `.bashrc`, and source `.bashrc` from `.bash_profile` for consistency.

---

## 4. Variables

### 4.1 Defining and Using Variables

```bash
name="Alice"                    # assign (no spaces around =)
echo $name                      # read variable
echo ${name}                    # preferred: curly brace form
echo "${name}"                  # always quote to handle spaces safely

age=30
readonly PI=3.14159             # constant (cannot be changed)
unset name                      # delete variable
```

### 4.2 Variable Types

```bash
# String
greeting="Hello, World"

# Integer
count=42

# Array
fruits=("apple" "banana" "cherry")
echo ${fruits[0]}               # apple
echo ${fruits[@]}               # all elements
echo ${#fruits[@]}              # number of elements
fruits+=("date")                # append element
unset fruits[1]                 # delete element at index 1

# Associative Array (Bash 4+)
declare -A colors
colors["red"]="#FF0000"
colors["green"]="#00FF00"
echo ${colors["red"]}
echo ${!colors[@]}              # all keys
echo ${colors[@]}               # all values
```

### 4.3 Variable Scope

```bash
x=10                            # global variable
function myfunc {
    local y=20                  # local to function
    echo $x $y
}
myfunc                          # 10 20
echo $y                         # empty (y is local)

export MYVAR="shared"           # export to child processes
```

### 4.4 Special Variables

| Variable | Meaning |
|---|---|
| `$0` | Script name |
| `$1` … `$9` | Positional arguments 1–9 |
| `${10}` | Positional argument 10+ |
| `$#` | Number of arguments |
| `$@` | All arguments (each quoted) |
| `$*` | All arguments (single string) |
| `$$` | Current script's PID |
| `$!` | PID of last background process |
| `$?` | Exit status of last command |
| `$_` | Last argument of last command |
| `$-` | Current shell options |
| `$PPID` | Parent process ID |
| `$PWD` | Current directory |
| `$OLDPWD` | Previous directory |
| `$HOME` | Home directory |
| `$USER` | Current username |
| `$HOSTNAME` | Machine hostname |
| `$RANDOM` | Random integer 0–32767 |
| `$LINENO` | Current line number in script |
| `$SECONDS` | Seconds since shell started |
| `$BASH_VERSION` | Bash version string |
| `$IFS` | Internal Field Separator |

---

## 5. Parameter Expansion

Parameter expansion provides powerful string manipulation without external tools.

### 5.1 Basic Expansion

```bash
var="Hello World"
echo ${var}                     # Hello World
echo ${#var}                    # 11 (length)
echo ${var:-"default"}          # value if set, else "default"
echo ${var:="default"}          # assign default if unset
echo ${var:?"error message"}    # error and exit if unset
echo ${var:+"alternate"}        # "alternate" if var is set, else empty
```

### 5.2 Substring Extraction

```bash
var="Hello World"
echo ${var:6}                   # World (from index 6)
echo ${var:6:3}                 # Wor (3 chars from index 6)
echo ${var: -5}                 # World (last 5 chars, note the space)
echo ${var:0:5}                 # Hello (first 5 chars)
```

### 5.3 Pattern Removal

```bash
file="archive.tar.gz"
echo ${file%.gz}                # archive.tar     (remove shortest suffix match)
echo ${file%%.*}                # archive         (remove longest suffix match)
echo ${file#*.}                 # tar.gz          (remove shortest prefix match)
echo ${file##*.}                # gz              (extension only)
echo ${file##*/}                # filename from path (basename equivalent)
echo ${file%/*}                 # directory from path (dirname equivalent)
```

### 5.4 Substitution

```bash
var="foo bar foo"
echo ${var/foo/baz}             # baz bar foo   (replace first)
echo ${var//foo/baz}            # baz bar baz   (replace all)
echo ${var/#foo/baz}            # baz bar foo   (replace if at start)
echo ${var/%foo/baz}            # foo bar baz   (replace if at end)
```

### 5.5 Case Conversion (Bash 4+)

```bash
var="Hello World"
echo ${var,,}                   # hello world   (all lowercase)
echo ${var^^}                   # HELLO WORLD   (all uppercase)
echo ${var^}                    # Hello World   (capitalize first char)
echo ${var,}                    # hELLO WORLD   (lowercase first char)
```

---

## 6. Writing Your First Script

### 6.1 Script Structure

```bash
#!/bin/bash
# Script name: hello.sh
# Description: My first bash script
# Author: Your Name
# Date: 2025-01-01

# --- Variables ---
NAME="World"

# --- Main Logic ---
echo "Hello, ${NAME}!"
echo "Today is: $(date)"
echo "Running as: $(whoami)"
echo "Script path: $0"
```

### 6.2 Making Scripts Executable

```bash
chmod +x hello.sh               # make executable
./hello.sh                      # run from current directory
bash hello.sh                   # run with bash explicitly
sh hello.sh                     # run with sh (POSIX mode)
source hello.sh                 # run in current shell (shares variables)
```

### 6.3 The Shebang Line

```bash
#!/bin/bash                     # use /bin/bash
#!/usr/bin/env bash             # portable: find bash in PATH (recommended)
#!/bin/sh                       # POSIX sh (more portable, fewer features)
#!/usr/bin/env python3          # works for any language
```

!!! tip "Use `env` in Shebangs"
    `#!/usr/bin/env bash` is more portable than `#!/bin/bash` because it finds bash wherever it is in `PATH`. This matters on systems where bash is at `/usr/local/bin/bash` (e.g., macOS with Homebrew).

---

## 7. Input and Output

### 7.1 `echo` and `printf`

```bash
echo "Hello"                    # print with newline
echo -n "No newline"            # no trailing newline
echo -e "Line1\nLine2"          # interpret escape sequences
echo -e "\t Tabbed"             # tab character

printf "Name: %s\n" "Alice"     # formatted string
printf "%-15s %5d\n" "item" 42  # left-align, right-align
printf "%05d\n" 7               # zero-padded: 00007
printf "%.2f\n" 3.14159         # 2 decimal places: 3.14
printf "%x\n" 255               # hex output: ff
```

### 7.2 `read` — User Input

```bash
read name                           # read into $name
read -p "Enter your name: " name    # with prompt
read -s password                    # silent input (passwords)
read -t 10 answer                   # timeout after 10 seconds
read -n 1 key                       # read only 1 character
read -r line                        # raw (don't interpret backslashes)
read -a arr                         # read into array (space-separated)

# Read a file line by line
while IFS= read -r line; do
    echo "$line"
done < file.txt

# Read multiple variables
read first last <<< "Alice Smith"
echo "$first"                       # Alice
echo "$last"                        # Smith
```

### 7.3 Redirection

```bash
command > file.txt              # stdout to file (overwrite)
command >> file.txt             # stdout to file (append)
command < file.txt              # stdin from file
command 2> errors.txt           # stderr to file
command 2>> errors.txt          # stderr append to file
command > out.txt 2>&1          # both stdout and stderr to file
command &> out.txt              # same (bash shorthand)
command 2>/dev/null             # discard stderr
command > /dev/null 2>&1        # discard all output
command1 | command2             # pipe stdout of cmd1 to stdin of cmd2
command1 |& command2            # pipe both stdout and stderr
```

### 7.4 Here Documents and Here Strings

```bash
# Here document
cat << EOF
This is line 1
This is line 2
Current user: $USER
EOF

# Prevent variable expansion (quote the delimiter)
cat << 'EOF'
This $variable will NOT be expanded
EOF

# Strip leading tabs
cat <<- EOF
    Indented content here
    More indented content
EOF

# Here string (feed single string as stdin)
grep "pattern" <<< "search in this string"
wc -w <<< "count these words"
```

---

## 8. Quoting

Understanding quoting is essential to avoid bugs with spaces and special characters.

| Quote Type | Example | Effect |
|---|---|---|
| No quotes | `$var`, `*.txt` | Word splitting + globbing |
| Double quotes `"..."` | `"$var"` | Variable expansion, no splitting |
| Single quotes `'...'` | `'$var'` | Literal — no expansion at all |
| Backslash `\` | `\$var` | Escape single character |
| `$'...'` | `$'\n'` | ANSI-C quoting (escape sequences) |

```bash
name="Alice Smith"
echo $name                      # may break if spaces: prints 2 words
echo "$name"                    # safe: prints "Alice Smith"
echo '$name'                    # literal: prints $name
echo "Today is $(date)"         # command substitution inside quotes works
echo 'Today is $(date)'         # literal: no substitution

# ANSI-C quoting
echo $'Line1\nLine2'            # prints two lines
echo $'Tab\there'               # tab character

# Common mistake
file="my file.txt"
cat $file                       # ERROR: cat my file.txt (two args)
cat "$file"                     # CORRECT: cat "my file.txt"
```

!!! warning "Always Quote Variables"
    The most common bash bugs come from unquoted variables. Always use `"$variable"` unless you explicitly need word splitting or globbing.

---

## 9. Conditional Logic

### 9.1 `if` / `elif` / `else`

```bash
if condition; then
    # commands
elif condition2; then
    # commands
else
    # commands
fi
```

```bash
# Example
if [ "$USER" == "root" ]; then
    echo "Running as root"
elif [ "$USER" == "alice" ]; then
    echo "Hello Alice"
else
    echo "Unknown user: $USER"
fi
```

### 9.2 Test Conditions — `[ ]` and `[[ ]]`

**String Tests:**

```bash
[ "$a" == "$b" ]                # equal
[ "$a" != "$b" ]                # not equal
[ -z "$a" ]                     # empty string
[ -n "$a" ]                     # non-empty string
[[ "$a" == *"pattern"* ]]       # glob match (double brackets)
[[ "$a" =~ ^[0-9]+$ ]]          # regex match (double brackets)
```

**Numeric Tests:**

```bash
[ "$a" -eq "$b" ]               # equal
[ "$a" -ne "$b" ]               # not equal
[ "$a" -lt "$b" ]               # less than
[ "$a" -le "$b" ]               # less than or equal
[ "$a" -gt "$b" ]               # greater than
[ "$a" -ge "$b" ]               # greater than or equal
```

**File Tests:**

```bash
[ -e file ]                     # exists (any type)
[ -f file ]                     # exists and is regular file
[ -d dir ]                      # exists and is directory
[ -L link ]                     # is a symbolic link
[ -r file ]                     # readable
[ -w file ]                     # writable
[ -x file ]                     # executable
[ -s file ]                     # exists and not empty (size > 0)
[ -O file ]                     # owned by current user
[ -G file ]                     # owned by current group
[ file1 -nt file2 ]             # file1 newer than file2
[ file1 -ot file2 ]             # file1 older than file2
[ file1 -ef file2 ]             # same file (hard link)
```

**Logical Operators:**

```bash
# Single brackets
[ cond1 ] && [ cond2 ]          # AND
[ cond1 ] || [ cond2 ]          # OR
[ ! cond ]                      # NOT

# Double brackets (preferred)
[[ cond1 && cond2 ]]            # AND
[[ cond1 || cond2 ]]            # OR
[[ ! cond ]]                    # NOT
```

!!! info "[ ] vs [[ ]]"
    `[[ ]]` is a bash built-in and is safer and more powerful — it supports regex (`=~`), glob matching, and doesn't need quoting for word splitting. `[ ]` is POSIX-compatible. Prefer `[[ ]]` in bash scripts.

### 9.3 `case` Statement

```bash
case "$variable" in
    pattern1)
        commands
        ;;
    pattern2 | pattern3)        # multiple patterns with |
        commands
        ;;
    prefix*)                    # glob pattern
        commands
        ;;
    *)                          # default (catch-all)
        commands
        ;;
esac
```

```bash
# Example: day of the week
day=$(date +%A)
case "$day" in
    Monday)    echo "Start of the week" ;;
    Friday)    echo "Almost weekend!" ;;
    Saturday | Sunday) echo "Weekend!" ;;
    *)         echo "Regular day" ;;
esac
```

### 9.4 Short-Circuit Evaluation

```bash
command1 && command2            # run cmd2 only if cmd1 succeeds
command1 || command2            # run cmd2 only if cmd1 fails
command1 && command2 || command3  # if/else one-liner

# Common patterns
[ -d /tmp/mydir ] || mkdir /tmp/mydir    # create if not exists
ping -c1 host && echo "Up" || echo "Down"
```

---

## 10. Loops

### 10.1 `for` Loop

```bash
# List-based for loop
for item in apple banana cherry; do
    echo "$item"
done

# Range with brace expansion
for i in {1..10}; do
    echo "Number $i"
done

# Range with step
for i in {0..20..5}; do    # 0, 5, 10, 15, 20
    echo "$i"
done

# C-style for loop
for ((i=0; i<10; i++)); do
    echo "$i"
done

# Iterate over array
fruits=("apple" "banana" "cherry")
for fruit in "${fruits[@]}"; do
    echo "$fruit"
done

# Iterate over files
for file in /etc/*.conf; do
    echo "Found: $file"
done

# Iterate over command output
for user in $(cut -d: -f1 /etc/passwd); do
    echo "$user"
done
```

### 10.2 `while` Loop

```bash
# Basic while
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    ((count++))
done

# Infinite loop
while true; do
    echo "Running..."
    sleep 1
done

# Read file line by line (safest method)
while IFS= read -r line; do
    echo "$line"
done < /etc/passwd

# Read command output line by line
while IFS= read -r line; do
    echo "$line"
done < <(find . -name "*.sh")

# Loop with condition check
while ping -c1 -W1 host &>/dev/null; do
    echo "Host is up"
    sleep 5
done
echo "Host went down"
```

### 10.3 `until` Loop

```bash
# until is the opposite of while (runs while condition is false)
count=1
until [ $count -gt 5 ]; do
    echo "Count: $count"
    ((count++))
done

# Wait for a service to start
until nc -zv localhost 8080 &>/dev/null; do
    echo "Waiting for service..."
    sleep 2
done
echo "Service is up!"
```

### 10.4 Loop Control

```bash
for i in {1..10}; do
    [ $i -eq 5 ] && break       # stop loop at 5
    echo "$i"
done

for i in {1..10}; do
    [ $((i % 2)) -eq 0 ] && continue   # skip even numbers
    echo "$i"
done

# Nested loops with labels (via break N)
for i in {1..3}; do
    for j in {1..3}; do
        [ $j -eq 2 ] && break 2    # break out of both loops
        echo "$i $j"
    done
done
```

---

## 11. Functions

### 11.1 Defining Functions

```bash
# Method 1: function keyword
function greet {
    echo "Hello, $1!"
}

# Method 2: POSIX style (preferred for portability)
greet() {
    echo "Hello, $1!"
}

# Call function
greet "Alice"                   # Hello, Alice!
greet "Bob"                     # Hello, Bob!
```

### 11.2 Arguments and Return Values

```bash
add() {
    local a=$1
    local b=$2
    local sum=$((a + b))
    echo $sum                   # "return" value via stdout
}

result=$(add 3 5)               # capture output
echo "Sum: $result"             # Sum: 8

# Return status code (0-255 only)
is_even() {
    [ $(($1 % 2)) -eq 0 ]
}

if is_even 4; then
    echo "Even"
fi
echo $?                         # 0 (success = true)
```

### 11.3 Local Variables and Scope

```bash
x=100                           # global

modify() {
    local x=200                 # local: doesn't affect global
    x=300                       # without local: modifies global
    echo "Inside: $x"
}

modify
echo "Outside: $x"              # 300 if no local, 100 if local used
```

### 11.4 Recursive Functions

```bash
factorial() {
    local n=$1
    if [ $n -le 1 ]; then
        echo 1
    else
        local prev=$(factorial $((n - 1)))
        echo $((n * prev))
    fi
}

echo $(factorial 5)             # 120
```

### 11.5 Useful Function Patterns

```bash
# Function with default argument
greet() {
    local name="${1:-World}"
    echo "Hello, $name!"
}
greet           # Hello, World!
greet "Alice"   # Hello, Alice!

# Logging function
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" | tee -a /var/log/myscript.log
}
log "Starting backup..."

# Check if command exists
require() {
    command -v "$1" &>/dev/null || { echo "Error: $1 not found"; exit 1; }
}
require curl
require jq

# Usage/help function
usage() {
    cat << EOF
Usage: $(basename "$0") [OPTIONS] FILE

Options:
  -h    Show this help
  -v    Verbose mode
  -o    Output file

EOF
    exit 0
}
```

---

## 12. Exit Status and Error Handling

### 12.1 Exit Status

Every command returns an exit status (0 = success, non-zero = failure).

```bash
ls /tmp
echo $?                         # 0 (success)

ls /nonexistent
echo $?                         # 2 (failure)

# Exit from script
exit 0                          # success
exit 1                          # general error
exit 2                          # misuse of command

# Custom exit codes
exit 127                        # command not found
exit 130                        # script interrupted by Ctrl+C
```

### 12.2 `set` Options for Robust Scripts

```bash
set -e                          # exit immediately on error
set -u                          # treat unset variables as error
set -o pipefail                 # pipe fails if any command fails
set -x                          # print each command before executing (debug)
set -v                          # print each line as read (verbose)

# Best practice: use at the top of every serious script
set -euo pipefail
```

!!! tip "The Safe Script Header"
    Start every important script with `set -euo pipefail`. This catches unset variables (`-u`), command failures (`-e`), and failures inside pipes (`-o pipefail`) — the three most common sources of silent bugs in bash scripts.

### 12.3 `trap` — Signal Handling

```bash
# Run cleanup on exit
cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/mylock
}
trap cleanup EXIT               # always runs on exit

# Handle Ctrl+C
trap 'echo "Interrupted!"; exit 1' INT

# Ignore signals
trap '' SIGTERM                 # ignore SIGTERM

# Reset trap
trap - EXIT                     # clear EXIT trap

# Common signals
# EXIT   - script exits (any reason)
# INT    - Ctrl+C
# TERM   - kill command (default)
# HUP    - terminal closed
# ERR    - any command fails (with set -e)
```

```bash
# Full cleanup pattern
TMPFILE=$(mktemp)
trap "rm -f $TMPFILE" EXIT

# ... work with $TMPFILE ...
# File is always cleaned up even if script errors
```

### 12.4 Error Handling Patterns

```bash
# Check return value explicitly
if ! mkdir /tmp/mydir; then
    echo "Failed to create directory" >&2
    exit 1
fi

# Die function
die() {
    echo "ERROR: $*" >&2
    exit 1
}

# Use it
[ -f "$1" ] || die "File not found: $1"
command -v jq &>/dev/null || die "jq is required but not installed"

# Conditional execution
mkdir -p /backup || die "Cannot create backup directory"
cd /backup || die "Cannot enter backup directory"
```

---

## 13. Arithmetic

### 13.1 Integer Arithmetic

```bash
# Arithmetic expansion (preferred)
echo $((3 + 4))                 # 7
echo $((10 - 3))                # 7
echo $((4 * 5))                 # 20
echo $((17 / 5))                # 3 (integer division)
echo $((17 % 5))                # 2 (remainder)
echo $((2 ** 8))                # 256 (exponentiation)

# Increment / decrement
i=5
((i++))                         # post-increment: i becomes 6
((i--))                         # post-decrement: i becomes 5
((i+=10))                       # i becomes 15
((i*=2))                        # i becomes 30

# Assign result
result=$((100 / 4))
echo $result                    # 25

# Conditional arithmetic
(( x > 5 )) && echo "greater"
```

### 13.2 `let` and `expr` (Legacy)

```bash
let "result = 5 * 4"            # let command
let i++                         # increment
expr 5 + 3                      # external command (slow, avoid)
expr 10 \* 4                    # must escape * for expr
```

### 13.3 Floating-Point with `bc`

Bash does not support floating-point — use `bc` or `awk`.

```bash
echo "scale=2; 10 / 3" | bc     # 3.33
echo "scale=4; sqrt(2)" | bc -l # 1.4142
echo "3.14 * 2^2" | bc -l       # 12.56
result=$(echo "scale=2; $a / $b" | bc)

# Using awk for float
awk "BEGIN {printf \"%.4f\n\", 10/3}"   # 3.3333
```

---

## 14. String Operations

### 14.1 Common String Tasks

```bash
str="Hello, World!"

# Length
echo ${#str}                    # 13

# Uppercase / Lowercase (Bash 4+)
echo ${str,,}                   # hello, world!
echo ${str^^}                   # HELLO, WORLD!

# Trim whitespace (no built-in — use parameter expansion)
str="  hello  "
trimmed="${str#"${str%%[![:space:]]*}"}"  # trim leading
trimmed="${trimmed%"${trimmed##*[![:space:]]}"}"  # trim trailing

# String contains
if [[ "$str" == *"World"* ]]; then
    echo "Found World"
fi

# String starts/ends with
[[ "$str" == Hello* ]] && echo "Starts with Hello"
[[ "$str" == *"!" ]] && echo "Ends with !"

# Regex match
if [[ "$str" =~ ^Hello ]]; then
    echo "Matches regex"
    echo "Match: ${BASH_REMATCH[0]}"    # full match
fi

# Replace
echo "${str/World/Linux}"       # Hello, Linux!

# Substring
echo "${str:7:5}"               # World
```

### 14.2 String Splitting

```bash
# Using IFS
IFS=',' read -ra parts <<< "a,b,c,d"
for part in "${parts[@]}"; do echo "$part"; done

# Using array
str="one:two:three"
IFS=':' read -ra arr <<< "$str"
echo "${arr[0]}"                # one
echo "${arr[2]}"                # three
```

---

## 15. Arrays

### 15.1 Indexed Arrays

```bash
# Define
arr=("one" "two" "three" "four")
arr[0]="one"                    # assign by index
arr+=("five")                   # append

# Access
echo ${arr[0]}                  # one
echo ${arr[-1]}                 # five (last element)
echo ${arr[@]}                  # all elements
echo ${arr[*]}                  # all elements (single word if quoted)
echo ${#arr[@]}                 # length (4)
echo ${!arr[@]}                 # all indices: 0 1 2 3 4

# Slice
echo ${arr[@]:1:2}              # elements from index 1, length 2: two three

# Delete element
unset arr[2]                    # remove index 2

# Iterate
for item in "${arr[@]}"; do
    echo "$item"
done

# Iterate with index
for i in "${!arr[@]}"; do
    echo "$i: ${arr[$i]}"
done
```

### 15.2 Associative Arrays (Bash 4+)

```bash
declare -A config
config["host"]="localhost"
config["port"]="5432"
config["user"]="admin"

echo ${config["host"]}          # localhost
echo ${!config[@]}              # all keys
echo ${config[@]}               # all values
echo ${#config[@]}              # number of entries

for key in "${!config[@]}"; do
    echo "$key = ${config[$key]}"
done
```

---

## 16. Script Arguments and Options

### 16.1 Positional Parameters

```bash
#!/bin/bash
echo "Script: $0"
echo "First arg: $1"
echo "Second arg: $2"
echo "All args: $@"
echo "Arg count: $#"

# Shift arguments
shift                           # $2 becomes $1, $3 becomes $2...
shift 2                         # shift by 2 positions

# Loop through all args
for arg in "$@"; do
    echo "Arg: $arg"
done
```

### 16.2 `getopts` — Option Parsing

```bash
#!/bin/bash
usage() { echo "Usage: $0 [-v] [-o outfile] [-h] file"; exit 1; }

verbose=false
outfile=""

while getopts "vo:h" opt; do
    case $opt in
        v) verbose=true ;;
        o) outfile="$OPTARG" ;;
        h) usage ;;
        ?) usage ;;
    esac
done

shift $((OPTIND - 1))           # remove parsed options, leave positional args
remaining_args=("$@")

$verbose && echo "Verbose mode on"
[ -n "$outfile" ] && echo "Output: $outfile"
echo "Remaining args: ${remaining_args[@]}"
```

---

## 17. File and Directory Operations in Scripts

```bash
# Check before acting
[ -f "$file" ] || { echo "File not found: $file"; exit 1; }
[ -d "$dir" ]  || mkdir -p "$dir"
[ -r "$file" ] || { echo "Cannot read: $file"; exit 1; }

# Safe temporary files
tmpfile=$(mktemp)               # /tmp/tmp.XXXXXXXX
tmpdir=$(mktemp -d)             # temp directory
trap "rm -rf $tmpfile $tmpdir" EXIT

# Absolute paths
script_dir="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
config_file="${script_dir}/config.conf"

# Find and process files
find /var/log -name "*.log" -mtime +30 | while IFS= read -r f; do
    gzip "$f" && echo "Compressed: $f"
done
```

---

## 18. Debugging Scripts

```bash
bash -x script.sh               # trace every command
bash -n script.sh               # syntax check (no execution)
bash -v script.sh               # print script lines as read

# Enable tracing inside script
set -x                          # start tracing
# ... code to debug ...
set +x                          # stop tracing

# Custom debug function
DEBUG=${DEBUG:-0}
debug() {
    [ "$DEBUG" -eq 1 ] && echo "[DEBUG] $*" >&2
}
debug "This only prints when DEBUG=1"

# Run with debug: DEBUG=1 ./script.sh

# PS4: customize trace prefix
export PS4='+(${BASH_SOURCE}:${LINENO}): ${FUNCNAME[0]:+${FUNCNAME[0]}(): }'
set -x
```

---

## 19. Best Practices

### 19.1 Script Template

```bash
#!/usr/bin/env bash
# ============================================================
# Script Name: template.sh
# Description: A well-structured bash script template
# Usage:       ./template.sh [-v] [-h] <input_file>
# Author:      Your Name
# Date:        2025-01-01
# ============================================================

set -euo pipefail
IFS=$'\n\t'

# --- Constants ---
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_NAME="$(basename "$0")"
readonly LOG_FILE="/var/log/${SCRIPT_NAME%.sh}.log"

# --- Defaults ---
VERBOSE=false

# --- Functions ---
log()   { echo "[$(date '+%F %T')] $*" | tee -a "$LOG_FILE"; }
die()   { echo "ERROR: $*" >&2; exit 1; }
usage() { echo "Usage: $SCRIPT_NAME [-v] [-h] <file>"; exit 0; }

cleanup() {
    log "Script finished."
}
trap cleanup EXIT

# --- Argument Parsing ---
while getopts "vh" opt; do
    case $opt in
        v) VERBOSE=true ;;
        h) usage ;;
        ?) die "Unknown option. Use -h for help." ;;
    esac
done
shift $((OPTIND - 1))

[ $# -lt 1 ] && die "Missing required argument. Use -h for help."
INPUT_FILE="$1"
[ -f "$INPUT_FILE" ] || die "File not found: $INPUT_FILE"

# --- Main ---
log "Starting ${SCRIPT_NAME}..."
$VERBOSE && log "Verbose mode enabled."
log "Processing: $INPUT_FILE"

# Your logic here

log "Done."
```

### 19.2 Key Rules

| Rule | Why |
|---|---|
| Always `set -euo pipefail` | Catch silent failures |
| Always quote `"$variables"` | Handle spaces and special chars |
| Use `local` in functions | Prevent scope leakage |
| Use `$(command)` not `` `command` `` | Backticks are hard to nest and read |
| Redirect errors to stderr: `echo "err" >&2` | Separate errors from normal output |
| Use `[[ ]]` not `[ ]` | Safer, more features |
| Use `readonly` for constants | Prevent accidental modification |
| Use `mktemp` for temp files | Unique, safe filenames |
| Always `trap cleanup EXIT` | Guarantee resource cleanup |
| Validate all inputs | Never trust user-supplied data |

---

## 20. Quick Reference Cheat Sheet

### Variables

| Syntax | Meaning |
|---|---|
| `$var` / `${var}` | Variable value |
| `${#var}` | String length |
| `${var:-default}` | Use default if unset |
| `${var:=default}` | Assign default if unset |
| `${var:offset:len}` | Substring |
| `${var//old/new}` | Replace all |
| `${var##*.}` | Strip longest prefix |
| `${var%%.*}` | Strip longest suffix |
| `${var^^}` | Uppercase |
| `${var,,}` | Lowercase |

### Tests

| Test | Meaning |
|---|---|
| `-f file` | Regular file exists |
| `-d dir` | Directory exists |
| `-z str` | String is empty |
| `-n str` | String is non-empty |
| `-eq / -ne` | Numeric equal / not equal |
| `-lt / -gt` | Less than / greater than |
| `=~ regex` | Regex match (double bracket) |

### Loops

```bash
for i in {1..10}; do ... done
for ((i=0; i<n; i++)); do ... done
while [ cond ]; do ... done
until [ cond ]; do ... done
while IFS= read -r line; do ... done < file
```

### Functions

```bash
myfunc() {
    local var="$1"
    echo "$var"
}
result=$(myfunc "arg")
```

### Error Handling

```bash
set -euo pipefail
trap cleanup EXIT
die() { echo "ERROR: $*" >&2; exit 1; }
[ -f "$f" ] || die "Not found: $f"
```

### Special Variables

| Variable | Value |
|---|---|
| `$0` | Script name |
| `$1`–`$9` | Arguments |
| `$#` | Argument count |
| `$@` | All arguments |
| `$?` | Last exit status |
| `$$` | Current PID |
| `$!` | Last background PID |
| `$RANDOM` | Random integer |