# 🔧 More on Bash Shell Scripting

> **Advanced Bash scripting** builds on the fundamentals — here you'll go deeper into string manipulation, regular expressions, process management, file handling, subshells, and real-world scripting patterns that separate a basic script from a production-ready tool.

---

## 1. Advanced String Manipulation

### 1.1 String Testing and Comparison

```bash
a="hello"
b="world"

# Lexicographic comparison (double brackets)
[[ "$a" < "$b" ]] && echo "$a comes before $b alphabetically"
[[ "$a" > "$b" ]] && echo "$a comes after $b alphabetically"
[[ "$a" == "$b" ]] && echo "Equal"
[[ "$a" != "$b" ]] && echo "Not equal"

# Case-insensitive comparison
a_lower="${a,,}"
b_lower="${b,,}"
[[ "$a_lower" == "$b_lower" ]] && echo "Case-insensitively equal"
```

### 1.2 String Padding and Alignment

```bash
# printf for alignment
printf "%-20s %s\n" "Name:" "Alice"
printf "%-20s %s\n" "Department:" "Engineering"
printf "%-20s %d\n" "Employee ID:" 12345

# Pad a number with zeros
num=7
printf "%05d\n" $num              # 00007

# Repeat a character (no built-in — use printf + tr)
printf '%0.s-' {1..40}; echo     # 40 dashes
printf '%*s\n' 40 '' | tr ' ' '-' # same result
```

### 1.3 Multi-line Strings

```bash
# Assign multi-line string
text="Line 1
Line 2
Line 3"

# Count lines in string
echo "$text" | wc -l

# Iterate over lines
while IFS= read -r line; do
    echo ">> $line"
done <<< "$text"

# Join lines with a delimiter
echo "$text" | paste -sd ',' -    # Line 1,Line 2,Line 3

# Split by delimiter into array
IFS=$'\n' read -rd '' -a lines <<< "$text"
echo "${lines[0]}"                # Line 1
```

### 1.4 String Validation

```bash
# Check if string is a valid integer
is_integer() {
    [[ "$1" =~ ^-?[0-9]+$ ]]
}
is_integer "42"   && echo "yes"   # yes
is_integer "3.14" && echo "yes"   # no
is_integer "-5"   && echo "yes"   # yes
is_integer "abc"  && echo "yes"   # no

# Check if string is a valid float
is_float() {
    [[ "$1" =~ ^-?[0-9]+(\.[0-9]+)?$ ]]
}

# Check if string is a valid IPv4
is_ipv4() {
    local ip="$1"
    local re='^([0-9]{1,3}\.){3}[0-9]{1,3}$'
    [[ "$ip" =~ $re ]] || return 1
    IFS='.' read -ra octets <<< "$ip"
    for o in "${octets[@]}"; do
        (( o >= 0 && o <= 255 )) || return 1
    done
}

# Check if variable is set and non-empty
is_set() { [[ -n "${!1+x}" && -n "${!1}" ]]; }
```

---

## 2. Regular Expressions in Bash

### 2.1 Bash Regex with `=~`

Bash supports ERE (Extended Regular Expressions) natively in `[[ ]]`.

```bash
string="Hello 42 World"

# Basic match
[[ "$string" =~ [0-9]+ ]] && echo "Contains digits"

# Capture groups with BASH_REMATCH
[[ "$string" =~ ([0-9]+) ]]
echo "${BASH_REMATCH[0]}"       # 42  (full match)
echo "${BASH_REMATCH[1]}"       # 42  (capture group 1)

# Multiple groups
date_str="2025-06-15"
[[ "$date_str" =~ ^([0-9]{4})-([0-9]{2})-([0-9]{2})$ ]]
echo "Year:  ${BASH_REMATCH[1]}"   # 2025
echo "Month: ${BASH_REMATCH[2]}"   # 06
echo "Day:   ${BASH_REMATCH[3]}"   # 15
```

### 2.2 Common Regex Patterns

```bash
# Email validation
email_re='^[a-zA-Z0-9._%+\-]+@[a-zA-Z0-9.\-]+\.[a-zA-Z]{2,}$'
[[ "user@example.com" =~ $email_re ]] && echo "Valid email"

# URL detection
url_re='^https?://[a-zA-Z0-9.\-]+(:[0-9]+)?(/.*)?$'
[[ "https://example.com/path" =~ $url_re ]] && echo "Valid URL"

# Hostname
hostname_re='^[a-zA-Z0-9]([a-zA-Z0-9\-]{0,61}[a-zA-Z0-9])?(\.[a-zA-Z]{2,})+$'

# Only digits
[[ "$var" =~ ^[0-9]+$ ]] && echo "All digits"

# Only letters
[[ "$var" =~ ^[a-zA-Z]+$ ]] && echo "All letters"

# Starts with http or https
[[ "$url" =~ ^https?:// ]]

# Date format YYYY-MM-DD
[[ "$date" =~ ^[0-9]{4}-[0-9]{2}-[0-9]{2}$ ]]

# MAC address
[[ "$mac" =~ ^([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}$ ]]
```

!!! warning "Store Regex in Variables"
    Never quote your regex when using `=~`. Store complex patterns in a variable first: `re='^[0-9]+$'; [[ "$val" =~ $re ]]`. Quoting the pattern on the right-hand side forces literal string matching.

### 2.3 `grep` with Regex in Scripts

```bash
# grep exit status in conditionals
if grep -q "^root" /etc/passwd; then
    echo "root user exists"
fi

# Extract with grep -o
echo "Error code: 404" | grep -oE '[0-9]+'   # 404

# Count matches
count=$(grep -c "^ERROR" logfile.txt)
echo "Found $count errors"

# Named capture with perl-compatible regex (grep -P)
echo "user=alice" | grep -oP 'user=\K\w+'     # alice (using \K lookbehind)
echo "foo123bar" | grep -oP '(?<=foo)\d+'     # 123
```

---

## 3. Process Management

### 3.1 Running Processes

```bash
# Background execution
sleep 10 &                      # run in background
pid=$!                          # capture PID immediately
echo "PID: $pid"

# Wait for background jobs
wait                            # wait for ALL background jobs
wait $pid                       # wait for specific PID
wait $pid; echo "Exit: $?"      # capture exit status

# Run multiple in parallel and wait
for host in host1 host2 host3; do
    ping -c3 "$host" &
done
wait
echo "All pings done"

# Parallel with tracking
pids=()
for file in *.log; do
    process_file "$file" &
    pids+=($!)
done

for pid in "${pids[@]}"; do
    wait "$pid" || echo "Process $pid failed"
done
```

### 3.2 Subshells

```bash
# Subshell with ( )
(cd /tmp && ls)                 # changes directory only inside subshell
echo "$PWD"                     # still in original directory

# Subshell for variable isolation
x=10
(x=99; echo "Inside: $x")       # Inside: 99
echo "Outside: $x"              # Outside: 10

# Capture subshell output
result=$(cd /tmp && ls -1 | wc -l)
echo "Files in /tmp: $result"

# Group commands in subshell and redirect
(echo "header"; cat data.txt; echo "footer") > output.txt

# Subshell in pipeline (each pipe stage runs in subshell)
count=0
echo "one two three" | while read word; do
    ((count++))                 # this count is LOCAL to subshell!
done
echo $count                     # 0 — variable not shared!

# Fix: use process substitution or lastpipe
shopt -s lastpipe               # last pipe runs in current shell
echo "one two three" | while read word; do
    ((count++))
done
echo $count                     # 3
```

### 3.3 Command Grouping with `{ }`

```bash
# Group in current shell (no subshell)
{ echo "line1"; echo "line2"; echo "line3"; } > output.txt
{ echo "stdout"; echo "stderr" >&2; } 2>errors.txt

# Group for conditional
{ update_package && restart_service; } || die "Update failed"

# Group with pipes
{ cat header.txt; cat body.txt; cat footer.txt; } | mail -s "Report" user@host
```

!!! info "( ) vs { }"
    `( commands )` runs in a **subshell** — changes to variables, directories, etc. are isolated. `{ commands; }` runs in the **current shell** — changes persist. Note the required space after `{` and semicolon before `}`.

### 3.4 Process Substitution

```bash
# Feed command output as a file
diff <(sort file1.txt) <(sort file2.txt)        # compare sorted versions
comm <(sort list1) <(sort list2)                # lines in common

# Read from process substitution
while IFS= read -r line; do
    echo "$line"
done < <(find . -name "*.sh" -newer ref.txt)

# Write to process substitution
tee >(gzip > out.gz) >(wc -l) > /dev/null <<< "data"

# Avoid subshell variable scope issue
while IFS= read -r line; do
    ((count++))
    last="$line"
done < <(cat file.txt)
echo "Lines: $count, Last: $last"   # variables accessible here!
```

---

## 4. Advanced I/O and File Descriptors

### 4.1 File Descriptors

```bash
# Standard descriptors
# 0 = stdin, 1 = stdout, 2 = stderr

# Redirect stderr to stdout
command 2>&1

# Redirect stdout to stderr (for error messages)
echo "This is an error" >&2

# Swap stdout and stderr
command 3>&1 1>&2 2>&3 3>&-

# Send both to file and terminal
command | tee output.txt

# Discard stderr
command 2>/dev/null

# Discard everything
command &>/dev/null
```

### 4.2 Custom File Descriptors

```bash
# Open file descriptor for reading
exec 3< /etc/passwd             # fd 3 = read from /etc/passwd
while IFS= read -r -u3 line; do
    echo "$line"
done
exec 3>&-                       # close fd 3

# Open file descriptor for writing
exec 4> output.txt              # fd 4 = write to output.txt
echo "Line 1" >&4
echo "Line 2" >&4
exec 4>&-                       # close fd 4

# Open for both read and write
exec 5<>/tmp/fifo

# Log everything to a file while keeping terminal output
exec 1> >(tee -a /var/log/myscript.log)
exec 2>&1
echo "This goes to both terminal and log"
```

### 4.3 Reading Files Efficiently

```bash
# Read entire file into variable
content=$(<file.txt)

# Read first line
read -r first_line < file.txt

# Read into array (one element per line)
mapfile -t lines < file.txt               # bash 4+
readarray -t lines < file.txt             # same as mapfile

echo "${lines[0]}"                        # first line
echo "${#lines[@]}"                       # number of lines

# Process CSV
while IFS=',' read -r field1 field2 field3; do
    echo "Name: $field1, Age: $field2, City: $field3"
done < data.csv

# Skip header line
{ read header; while IFS=',' read -r f1 f2 f3; do
    echo "$f1 $f2 $f3"
done; } < data.csv
```

---

## 5. Here Documents — Advanced Usage

```bash
# Variable expansion (default)
name="Alice"
cat << EOF
Hello, $name!
Today is $(date).
Your home: $HOME
EOF

# Suppress expansion with quoted delimiter
cat << 'EOF'
Literal: $name $(date) $HOME
All of this is printed as-is.
EOF

# Indent with dash (strip leading tabs only)
if true; then
    cat <<- EOF
        This is indented with tabs
        and they are stripped.
    EOF
fi

# Heredoc into a variable
message=$(cat << EOF
Subject: Report
Date: $(date)
User: $USER
EOF
)

# Heredoc as script input
bash << 'EOF'
echo "Running in a subshell"
for i in {1..3}; do echo "Line $i"; done
EOF

# Heredoc for file creation
cat > /etc/myapp/config.conf << EOF
host=localhost
port=8080
debug=false
user=$USER
EOF

# Heredoc for SSH commands
ssh user@host << 'EOF'
echo "On remote machine"
df -h
uptime
EOF
```

---

## 6. Advanced Functions

### 6.1 Functions Returning Multiple Values

```bash
# Method 1: stdout (capture with $())
get_stats() {
    local file="$1"
    local lines=$(wc -l < "$file")
    local words=$(wc -w < "$file")
    local size=$(stat -c%s "$file")
    echo "$lines $words $size"
}

read lines words size <<< "$(get_stats /etc/passwd)"
echo "Lines: $lines, Words: $words, Size: $size"

# Method 2: global or nameref variables
get_info() {
    local -n _result=$1         # nameref (bash 4.3+)
    _result[name]="Alice"
    _result[age]=30
}

declare -A info
get_info info
echo "${info[name]} is ${info[age]}"

# Method 3: write to multiple variables
parse_date() {
    IFS='-' read -r year month day <<< "$1"
}
parse_date "2025-06-15"
echo "Year: $year, Month: $month, Day: $day"
```

### 6.2 Function Libraries

```bash
# lib/logging.sh
#!/usr/bin/env bash
LOG_LEVEL="${LOG_LEVEL:-INFO}"

_log() {
    local level="$1"; shift
    echo "[$(date '+%F %T')] [$level] $*" >&2
}

log_info()  { [[ "$LOG_LEVEL" =~ DEBUG|INFO ]]  && _log "INFO " "$@"; }
log_debug() { [[ "$LOG_LEVEL" == "DEBUG" ]]      && _log "DEBUG" "$@"; }
log_warn()  { [[ "$LOG_LEVEL" =~ DEBUG|INFO|WARN ]] && _log "WARN " "$@"; }
log_error() { _log "ERROR" "$@"; }
log_fatal() { _log "FATAL" "$@"; exit 1; }
```

```bash
# main.sh — source the library
#!/usr/bin/env bash
source "$(dirname "$0")/lib/logging.sh"

log_info "Script started"
log_debug "Debug info"   # only if LOG_LEVEL=DEBUG
log_warn "Something odd"
log_error "Something failed"
```

### 6.3 Memoization

```bash
# Cache expensive function results
declare -A _memo_cache

expensive() {
    local key="$1"
    if [[ -v _memo_cache[$key] ]]; then
        echo "${_memo_cache[$key]}"
        return
    fi
    local result
    result=$(do_expensive_work "$key")   # only runs once per key
    _memo_cache[$key]="$result"
    echo "$result"
}
```

---

## 7. Script Organisation and Modularity

### 7.1 Sourcing Files

```bash
# Source a file (runs in current shell)
source /path/to/lib.sh
. /path/to/lib.sh                       # POSIX equivalent

# Source only if exists
[[ -f ~/.bash_aliases ]] && source ~/.bash_aliases

# Source with guard (prevent double-sourcing)
# In lib.sh:
[[ -n "${_LIB_LOADED:-}" ]] && return 0
readonly _LIB_LOADED=1
# ... rest of library ...
```

### 7.2 Script as Both Library and Executable

```bash
#!/usr/bin/env bash
# Can be sourced as a library OR run directly

my_function() {
    echo "Hello from library"
}

# Only run main() if script is executed directly
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    my_function
    echo "Running as script"
fi
```

### 7.3 Configuration Files

```bash
# Load INI-style config
parse_config() {
    local config_file="$1"
    local section=""
    while IFS= read -r line || [[ -n "$line" ]]; do
        line="${line%%#*}"             # strip comments
        line="${line%"${line##*[^[:space:]]}"}"   # trim trailing whitespace
        [[ -z "$line" ]] && continue
        if [[ "$line" =~ ^\[(.+)\]$ ]]; then
            section="${BASH_REMATCH[1]}"
        elif [[ "$line" =~ ^([^=]+)=(.*)$ ]]; then
            local key="${BASH_REMATCH[1]}"
            local val="${BASH_REMATCH[2]}"
            key="${section:+${section}_}${key// /_}"
            printf -v "$key" '%s' "$val"
        fi
    done < "$config_file"
}

# config.ini:
# [database]
# host = localhost
# port = 5432

parse_config config.ini
echo "$database_host"       # localhost
echo "$database_port"       # 5432
```

---

## 8. Signals and Traps — Advanced

```bash
# All trappable signals
trap -l                             # list all signal names

# Multiple signals
trap 'echo "Caught signal"' INT TERM HUP

# Trap ERR for error handling
set -E                              # ERR trap inherited by functions
trap 'echo "Error at line $LINENO: $BASH_COMMAND"' ERR

# Trap DEBUG (runs before every command)
trap 'echo "About to run: $BASH_COMMAND"' DEBUG

# Comprehensive cleanup trap
TMPFILES=()
TMPDIRS=()

add_tmpfile() { TMPFILES+=("$1"); }
add_tmpdir()  { TMPDIRS+=("$1"); }

cleanup() {
    local exit_code=$?
    rm -f "${TMPFILES[@]}"
    rm -rf "${TMPDIRS[@]}"
    [[ $exit_code -ne 0 ]] && echo "Script failed with code $exit_code" >&2
    exit $exit_code
}
trap cleanup EXIT

# Usage
tmpf=$(mktemp); add_tmpfile "$tmpf"
tmpd=$(mktemp -d); add_tmpdir "$tmpd"
```

---

## 9. Advanced Loops and Iteration

### 9.1 Loop with Index and Value

```bash
arr=("apple" "banana" "cherry" "date")

# Indexed loop
for i in "${!arr[@]}"; do
    printf "[%d] %s\n" "$i" "${arr[$i]}"
done

# Reversed loop
for ((i=${#arr[@]}-1; i>=0; i--)); do
    echo "${arr[$i]}"
done

# Paired arrays
keys=("name" "age" "city")
vals=("Alice" "30" "Chennai")

for i in "${!keys[@]}"; do
    printf "%-10s = %s\n" "${keys[$i]}" "${vals[$i]}"
done
```

### 9.2 Loop Over Command Output

```bash
# Process substitution (avoids subshell — variables persist)
while IFS= read -r file; do
    echo "Processing: $file"
done < <(find /etc -name "*.conf" 2>/dev/null)

# Iterate over lines with IFS control
while IFS=: read -r user pw uid gid gecos home shell; do
    printf "%-15s UID=%-5s Shell=%s\n" "$user" "$uid" "$shell"
done < /etc/passwd

# Loop over null-separated output (safe for filenames with spaces)
while IFS= read -r -d '' file; do
    echo "Found: $file"
done < <(find . -name "*.txt" -print0)
```

### 9.3 Parallel Loops with Rate Limiting

```bash
# Run N jobs in parallel
PARALLEL=4
running=0

for item in "${items[@]}"; do
    process "$item" &
    ((running++))
    if (( running >= PARALLEL )); then
        wait -n 2>/dev/null || wait   # wait for any one job (-n is bash 4.3+)
        ((running--))
    fi
done
wait    # wait for remaining jobs
```

---

## 10. Text Processing Patterns

### 10.1 Parsing Structured Output

```bash
# Parse key=value output
while IFS='=' read -r key val; do
    key="${key// /}"                  # trim spaces
    val="${val// /}"
    declare "config_$key=$val"
done < <(grep -v '^#' config.conf | grep '=')

# Parse space/tab-separated columns
awk 'NR>1 {print $1, $3}' data.txt | while read name value; do
    echo "$name: $value"
done

# Parse JSON with jq (if available)
if command -v jq &>/dev/null; then
    name=$(jq -r '.name' data.json)
    echo "Name: $name"
fi
```

### 10.2 Generating Reports

```bash
generate_report() {
    local title="$1"
    local width=60

    printf '%s\n' "$(printf '%*s' $width '' | tr ' ' '=')"
    printf '  %s\n' "$title"
    printf '%s\n' "$(printf '%*s' $width '' | tr ' ' '=')"
    printf '  Generated: %s\n' "$(date)"
    printf '  Host:      %s\n' "$(hostname)"
    printf '%s\n' "$(printf '%*s' $width '' | tr ' ' '-')"
}

generate_report "System Status Report"
```

### 10.3 Progress Bar

```bash
progress_bar() {
    local current=$1 total=$2 width=${3:-40}
    local filled=$(( current * width / total ))
    local empty=$(( width - filled ))
    local pct=$(( current * 100 / total ))

    printf '\r['
    printf '%*s' $filled '' | tr ' ' '#'
    printf '%*s' $empty  '' | tr ' ' '-'
    printf '] %3d%% (%d/%d)' $pct $current $total
    [[ $current -eq $total ]] && echo
}

total=100
for ((i=1; i<=total; i++)); do
    sleep 0.05
    progress_bar $i $total
done
```

---

## 11. Working with Dates and Times

```bash
# Current date/time
date                            # default format
date '+%Y-%m-%d'                # 2025-06-15
date '+%H:%M:%S'                # 14:30:00
date '+%Y-%m-%d %H:%M:%S'       # 2025-06-15 14:30:00
date '+%s'                      # Unix timestamp (seconds since epoch)
date '+%A, %B %d %Y'            # Monday, June 15 2025
date -u                         # UTC time

# Arithmetic with dates (GNU date)
date -d "tomorrow"
date -d "yesterday"
date -d "7 days ago"
date -d "+2 weeks"
date -d "last monday"
date -d "2025-01-01 + 30 days" '+%Y-%m-%d'

# Convert timestamp to date
date -d @1700000000 '+%Y-%m-%d %H:%M:%S'

# Measure execution time
start=$(date +%s)
sleep 2
end=$(date +%s)
echo "Elapsed: $((end - start)) seconds"

# High-resolution timing
start=$SECONDS
sleep 3
echo "Elapsed: $((SECONDS - start)) seconds"

# TIMEFORMAT for time builtin
TIMEFORMAT='Elapsed: %R seconds'
time sleep 1
```

---

## 12. Networking in Scripts

```bash
# Check if host is reachable
is_reachable() {
    ping -c1 -W2 "$1" &>/dev/null
}
is_reachable google.com && echo "Online"

# Check if port is open
is_port_open() {
    local host="$1" port="$2"
    nc -zw2 "$host" "$port" &>/dev/null
}
is_port_open localhost 80 && echo "HTTP is up"

# Wait for service to become available
wait_for_service() {
    local host="$1" port="$2" timeout="${3:-30}"
    local elapsed=0
    while ! nc -zw1 "$host" "$port" &>/dev/null; do
        (( elapsed++ ))
        (( elapsed >= timeout )) && { echo "Timeout waiting for $host:$port"; return 1; }
        echo "Waiting for $host:$port... ($elapsed/${timeout}s)"
        sleep 1
    done
    echo "$host:$port is ready"
}

# Download with retry
download_retry() {
    local url="$1" dest="$2" retries="${3:-3}"
    local attempt=0
    while (( attempt < retries )); do
        wget -q -O "$dest" "$url" && return 0
        (( attempt++ ))
        echo "Download failed, attempt $attempt/$retries"
        sleep 2
    done
    return 1
}

# Get public IP
get_public_ip() {
    curl -s https://ipinfo.io/ip || curl -s https://api.ipify.org
}

# Simple HTTP health check
http_check() {
    local url="$1"
    local code
    code=$(curl -s -o /dev/null -w "%{http_code}" --max-time 5 "$url")
    echo "HTTP $code for $url"
    [[ "$code" == "200" ]]
}
```

---

## 13. System Information Scripts

```bash
# CPU usage
cpu_usage() {
    top -bn1 | grep "Cpu(s)" | awk '{print $2}' | tr -d '%us,'
}

# Memory info
mem_info() {
    free -h | awk 'NR==2 {printf "Used: %s / %s (%.0f%%)\n", $3, $2, $3/$2*100}'
}

# Disk usage with alert
disk_alert() {
    local threshold="${1:-80}"
    df -h | awk -v t="$threshold" '
        NR>1 && /\// {
            pct = $5+0
            if (pct >= t)
                printf "ALERT: %s is %s full (threshold: %d%%)\n", $6, $5, t
        }
    '
}

# Top memory processes
top_mem() {
    local n="${1:-10}"
    ps aux --sort=-%mem | awk 'NR<='"$((n+1))"' {printf "%-6s %-5s %-5s %s\n", $1,$2,$4,$11}'
}

# System summary
system_summary() {
    echo "=== System Summary ==="
    echo "Hostname:  $(hostname -f)"
    echo "OS:        $(. /etc/os-release && echo "$NAME $VERSION")"
    echo "Kernel:    $(uname -r)"
    echo "Uptime:    $(uptime -p)"
    echo "Load avg:  $(uptime | awk -F'load average:' '{print $2}')"
    echo "Memory:    $(free -h | awk 'NR==2{print $3"/"$2}')"
    echo "Disk (/):  $(df -h / | awk 'NR==2{print $3"/"$2" ("$5")"}')"
    echo "Users:     $(who | wc -l) logged in"
    echo "Processes: $(ps aux | wc -l)"
}
```

---

## 14. Locking and Concurrency

### 14.1 File Locking

```bash
# Prevent multiple instances with a lockfile
LOCKFILE="/var/run/myscript.lock"

acquire_lock() {
    if ! mkdir "$LOCKFILE" 2>/dev/null; then
        echo "Script is already running (lock: $LOCKFILE)"
        exit 1
    fi
    trap "rm -rf $LOCKFILE" EXIT
}

acquire_lock
echo "Running with lock..."
```

### 14.2 `flock` — Advisory File Locking

```bash
# Lock a file descriptor
exec 200>/var/lock/myscript.lock
flock -n 200 || { echo "Already running"; exit 1; }

# One-liner with flock
flock -n /var/lock/myscript.lock -c '/path/to/myscript.sh'

# Wait up to 10 seconds for lock
flock -w 10 /var/lock/myscript.lock || { echo "Could not acquire lock"; exit 1; }

# flock in a script block
(
    flock -n 9 || exit 1
    echo "Critical section"
    sleep 5
) 9>/var/lock/critical.lock
```

---

## 15. Secure Scripting Practices

```bash
# Restrict PATH to known safe directories
export PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

# Use full paths for critical commands
RM=/bin/rm
CP=/bin/cp
FIND=/usr/bin/find

# Validate all input
validate_filename() {
    local f="$1"
    # Reject paths with traversal, null bytes, or leading dash
    [[ "$f" == *".."* ]] && die "Path traversal detected: $f"
    [[ "$f" == -* ]]     && die "Filename starts with dash: $f"
    [[ -z "$f" ]]        && die "Empty filename"
}

# Sanitise for use in filenames
sanitise() {
    echo "$1" | tr -cd 'A-Za-z0-9._-'
}

# Avoid `eval` with untrusted input
# BAD:
eval "result=$user_input"          # arbitrary code execution risk

# GOOD: use arrays and parameter expansion
args=("$@")
command "${args[@]}"

# Use mktemp, not predictable names
tmpfile=$(mktemp /tmp/myscript.XXXXXXXXXX) || die "mktemp failed"

# Protect sensitive variables
password=$(read -rs -p "Password: " pw; echo "$pw")
unset password                      # clear after use

# Set restrictive umask
umask 077                           # new files: 600, dirs: 700
```

---

## 16. Testing Bash Scripts

### 16.1 Manual Testing Patterns

```bash
# Dry run flag
DRY_RUN="${DRY_RUN:-0}"

run_cmd() {
    if [[ "$DRY_RUN" -eq 1 ]]; then
        echo "[DRY RUN] $*"
    else
        "$@"
    fi
}

run_cmd rm -rf /tmp/old_data        # prints command instead of running it
# Usage: DRY_RUN=1 ./script.sh
```

### 16.2 Simple Unit Tests with `assert`

```bash
#!/usr/bin/env bash
# Simple test framework

PASS=0; FAIL=0

assert_eq() {
    local desc="$1" expected="$2" actual="$3"
    if [[ "$expected" == "$actual" ]]; then
        echo "  PASS: $desc"
        ((PASS++))
    else
        echo "  FAIL: $desc"
        echo "        Expected: $expected"
        echo "        Actual:   $actual"
        ((FAIL++))
    fi
}

assert_true() {
    local desc="$1"; shift
    if "$@" 2>/dev/null; then
        echo "  PASS: $desc"
        ((PASS++))
    else
        echo "  FAIL: $desc (returned false)"
        ((FAIL++))
    fi
}

# --- Source the functions to test ---
source ./lib/utils.sh

# --- Tests ---
echo "Testing string functions..."
assert_eq "to_upper 'hello'" "HELLO" "$(to_upper 'hello')"
assert_eq "trim '  hi  '" "hi" "$(trim '  hi  ')"

echo "Testing file checks..."
assert_true "is_readable /etc/passwd" is_readable /etc/passwd

# --- Summary ---
echo ""
echo "Results: $PASS passed, $FAIL failed"
[[ $FAIL -eq 0 ]] && exit 0 || exit 1
```

### 16.3 Using Bats (Bash Automated Testing System)

```bash
# Install: npm install -g bats-core
# test/test_utils.bats

@test "to_upper converts lowercase to uppercase" {
    source lib/utils.sh
    result=$(to_upper "hello")
    [ "$result" = "HELLO" ]
}

@test "is_integer returns true for 42" {
    source lib/utils.sh
    is_integer 42
}

@test "is_integer returns false for abc" {
    source lib/utils.sh
    ! is_integer "abc"
}

# Run tests
# bats test/test_utils.bats
```

---

## 17. Practical Script Examples

### 17.1 Backup Script

```bash
#!/usr/bin/env bash
set -euo pipefail

BACKUP_SRC="${1:?Usage: $0 SOURCE DEST}"
BACKUP_DEST="${2:?Usage: $0 SOURCE DEST}"
TIMESTAMP=$(date '+%Y%m%d_%H%M%S')
BACKUP_NAME="backup_${TIMESTAMP}.tar.gz"
LOG="/var/log/backup.log"

log() { echo "[$(date '+%F %T')] $*" | tee -a "$LOG"; }
die() { log "ERROR: $*"; exit 1; }

[[ -d "$BACKUP_SRC" ]]  || die "Source not a directory: $BACKUP_SRC"
[[ -d "$BACKUP_DEST" ]] || mkdir -p "$BACKUP_DEST"

log "Starting backup of $BACKUP_SRC → $BACKUP_DEST/$BACKUP_NAME"
tar czf "${BACKUP_DEST}/${BACKUP_NAME}" -C "$(dirname "$BACKUP_SRC")" "$(basename "$BACKUP_SRC")" \
    || die "tar failed"

SIZE=$(du -sh "${BACKUP_DEST}/${BACKUP_NAME}" | cut -f1)
log "Backup complete: $BACKUP_NAME ($SIZE)"

# Prune backups older than 30 days
find "$BACKUP_DEST" -name "backup_*.tar.gz" -mtime +30 -delete
log "Old backups pruned."
```

### 17.2 Log Analyser

```bash
#!/usr/bin/env bash
set -euo pipefail

LOGFILE="${1:?Provide a log file}"
[[ -f "$LOGFILE" ]] || { echo "File not found: $LOGFILE"; exit 1; }

echo "=== Log Analysis: $LOGFILE ==="
echo "Total lines:   $(wc -l < "$LOGFILE")"
echo "Error count:   $(grep -ci "error" "$LOGFILE" || echo 0)"
echo "Warning count: $(grep -ci "warn"  "$LOGFILE" || echo 0)"
echo ""

echo "--- Top 10 IP Addresses ---"
grep -oE '[0-9]{1,3}(\.[0-9]{1,3}){3}' "$LOGFILE" \
    | sort | uniq -c | sort -rn | head -10

echo ""
echo "--- Errors by Hour ---"
grep -i "error" "$LOGFILE" \
    | awk '{print $1, substr($2,1,2)}' \
    | sort | uniq -c | sort -rn | head -10

echo ""
echo "--- HTTP Status Codes ---"
awk '{print $9}' "$LOGFILE" 2>/dev/null \
    | grep -E '^[0-9]{3}$' \
    | sort | uniq -c | sort -rn
```

### 17.3 Interactive Menu

```bash
#!/usr/bin/env bash
show_menu() {
    echo ""
    echo "================================"
    echo "   System Administration Menu"
    echo "================================"
    echo "  1. Show disk usage"
    echo "  2. Show memory usage"
    echo "  3. Show running processes"
    echo "  4. Show network connections"
    echo "  5. Show system uptime"
    echo "  q. Quit"
    echo "================================"
    printf "Select option: "
}

while true; do
    show_menu
    read -r choice
    case "$choice" in
        1) df -h ;;
        2) free -h ;;
        3) ps aux | head -20 ;;
        4) ss -tlnp ;;
        5) uptime ;;
        q|Q) echo "Goodbye!"; exit 0 ;;
        *) echo "Invalid option: $choice" ;;
    esac
    echo ""
    read -rp "Press Enter to continue..."
done
```

---

## 18. Quick Reference Cheat Sheet

### Process and Subshell

| Syntax | Meaning |
|---|---|
| `cmd &` | Run in background |
| `$!` | PID of last background job |
| `wait $pid` | Wait for PID to finish |
| `( cmds )` | Subshell — isolated scope |
| `{ cmds; }` | Group — current shell |
| `<(cmd)` | Process substitution (as input file) |
| `>(cmd)` | Process substitution (as output file) |

### Regex

| Pattern | Matches |
|---|---|
| `^` | Start of string |
| `$` | End of string |
| `.` | Any character |
| `[0-9]` | Any digit |
| `[a-zA-Z]` | Any letter |
| `+` | One or more |
| `*` | Zero or more |
| `?` | Zero or one |
| `{n,m}` | Between n and m |
| `(a\|b)` | Alternation |

### Advanced I/O

| Syntax | Meaning |
|---|---|
| `exec 3< file` | Open fd 3 for reading |
| `exec 4> file` | Open fd 4 for writing |
| `read -u3 line` | Read from fd 3 |
| `echo "x" >&4` | Write to fd 4 |
| `exec 3>&-` | Close fd 3 |
| `mapfile -t arr < file` | Read file into array |
| `$(<file)` | Read file into variable |

### Traps

| Trap | When it fires |
|---|---|
| `EXIT` | Any exit |
| `ERR` | Command fails |
| `INT` | Ctrl+C |
| `TERM` | `kill` signal |
| `HUP` | Terminal close |
| `DEBUG` | Before every command |

### Useful One-Liners

```bash
# Retry a command up to N times
retry() { local n=$1; shift; for ((i=0;i<n;i++)); do "$@" && return; sleep 1; done; return 1; }

# Run only if not already running
run_once() { pgrep -f "$1" &>/dev/null || "$@" & }

# Confirm before proceeding
confirm() { read -rp "${1:-Are you sure?} [y/N] " r; [[ "$r" =~ ^[Yy] ]]; }

# Colour output
red()   { echo -e "\e[31m$*\e[0m"; }
green() { echo -e "\e[32m$*\e[0m"; }
yellow(){ echo -e "\e[33m$*\e[0m"; }
bold()  { echo -e "\e[1m$*\e[0m"; }

# Absolute path of a file
abspath() { echo "$(cd "$(dirname "$1")" && pwd)/$(basename "$1")"; }

# Check if running as root
is_root() { [[ $EUID -eq 0 ]]; }
is_root || { echo "Must run as root"; exit 1; }
```