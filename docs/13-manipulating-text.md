# 📝 Manipulating Text

> **Text manipulation** is the backbone of Linux administration — from filtering logs and transforming data to automating reports, mastering these tools turns raw text into actionable information without ever opening a GUI.

---

## 1. Overview of Text Manipulation in Linux

Linux treats almost everything as a file, and most configuration, log, and data files are plain text. The shell provides a rich ecosystem of utilities to view, search, sort, transform, and process text — individually or chained together via pipes.

| Category | Key Tools |
|---|---|
| Viewing | `cat`, `tac`, `head`, `tail`, `less`, `more` |
| Searching | `grep`, `egrep`, `fgrep` |
| Sorting & Deduplication | `sort`, `uniq` |
| Counting | `wc` |
| Cutting & Joining | `cut`, `paste`, `join` |
| Replacing & Translating | `tr`, `sed` |
| Field Processing | `awk` |
| Encoding | `base64`, `xxd` |
| Compression-aware | `zcat`, `zgrep`, `bzcat` |
| Stream Editing | `sed`, `awk` |

---

## 2. Viewing File Content

### 2.1 `cat` — Concatenate and Print

`cat` reads one or more files and writes them to standard output.

```bash
cat file.txt                   # print file
cat file1.txt file2.txt        # concatenate two files
cat -n file.txt                # number all output lines
cat -b file.txt                # number non-blank lines only
cat -A file.txt                # show tabs (^I) and line endings ($)
cat -s file.txt                # squeeze multiple blank lines into one
cat > newfile.txt              # create file interactively (Ctrl+D to save)
cat >> file.txt                # append interactively
```

### 2.2 `tac` — Reverse of `cat`

`tac` prints lines in reverse order (last line first).

```bash
tac file.txt                   # print file reversed line by line
tac -s ':' file.txt            # use ':' as record separator
```

### 2.3 `head` — View Beginning of File

```bash
head file.txt                  # first 10 lines (default)
head -n 20 file.txt            # first 20 lines
head -c 100 file.txt           # first 100 bytes
head -n -5 file.txt            # all lines EXCEPT the last 5
```

### 2.4 `tail` — View End of File

```bash
tail file.txt                  # last 10 lines (default)
tail -n 20 file.txt            # last 20 lines
tail -c 200 file.txt           # last 200 bytes
tail -n +5 file.txt            # all lines starting from line 5
tail -f /var/log/syslog        # follow file in real time (live log monitoring)
tail -F /var/log/app.log       # follow even if file is recreated (log rotation safe)
```

!!! tip "Live Log Monitoring"
    `tail -f` is essential for watching logs in real time. Use `Ctrl+C` to stop. Combine with `grep` for filtered live output: `tail -f app.log | grep ERROR`

### 2.5 `less` and `more` — Pagers

```bash
less file.txt                  # scroll up/down freely
more file.txt                  # scroll forward only
```

| `less` Navigation Key | Action |
|---|---|
| `Space` / `f` | Forward one page |
| `b` | Back one page |
| `g` | Go to beginning |
| `G` | Go to end |
| `/pattern` | Search forward |
| `?pattern` | Search backward |
| `n` / `N` | Next / previous match |
| `q` | Quit |

---

## 3. Searching Text with `grep`

`grep` (Global Regular Expression Print) searches for patterns in files or stdin.

### 3.1 Basic Syntax

```bash
grep pattern file              # basic search
grep "hello world" file.txt    # phrase with spaces (quote it)
grep -i "error" file.txt       # case-insensitive
grep -v "debug" file.txt       # invert match (lines NOT matching)
grep -n "warning" file.txt     # show line numbers
grep -c "fail" file.txt        # count matching lines
grep -l "TODO" *.py            # list files that match
grep -L "TODO" *.py            # list files that do NOT match
grep -r "config" /etc/         # recursive search
grep -R "password" /home/      # recursive, follows symlinks
```

### 3.2 Extended and Fixed-String Variants

```bash
egrep "cat|dog" file.txt       # extended regex (same as grep -E)
grep -E "^[0-9]{3}-" file.txt  # ERE: lines starting with 3 digits and dash
fgrep "192.168.1.1" file.txt   # fixed string, no regex (fastest)
grep -F "user.name" file.txt   # same as fgrep
```

### 3.3 Output Control

```bash
grep -o "pattern" file.txt     # print only the matched part
grep -A 3 "ERROR" file.txt     # 3 lines After match
grep -B 3 "ERROR" file.txt     # 3 lines Before match
grep -C 3 "ERROR" file.txt     # 3 lines Context (before and after)
grep --color=auto "root" /etc/passwd   # highlight matches
grep -m 5 "warning" file.txt   # stop after 5 matches
grep -w "root" /etc/passwd     # match whole words only
grep -x "exact line" file.txt  # match whole lines only
```

### 3.4 Common Regex Anchors and Metacharacters

| Pattern | Meaning |
|---|---|
| `^` | Start of line |
| `$` | End of line |
| `.` | Any single character |
| `*` | Zero or more of previous |
| `+` | One or more (ERE) |
| `?` | Zero or one (ERE) |
| `[abc]` | Character class |
| `[^abc]` | Negated class |
| `\b` | Word boundary |
| `(a|b)` | Alternation (ERE) |

```bash
grep "^root" /etc/passwd       # lines starting with "root"
grep "bash$" /etc/passwd       # lines ending with "bash"
grep "^$" file.txt             # empty lines
grep "^[[:space:]]*$" file.txt # blank lines (spaces/tabs only)
```

---

## 4. Sorting Text with `sort`

### 4.1 Basic Sorting

```bash
sort file.txt                  # alphabetical sort
sort -r file.txt               # reverse order
sort -n file.txt               # numeric sort
sort -h file.txt               # human-readable numbers (1K, 2M)
sort -u file.txt               # sort and remove duplicates
sort -f file.txt               # ignore case
sort -R file.txt               # random shuffle
```

### 4.2 Field-Based Sorting

```bash
sort -t: -k3 -n /etc/passwd    # sort by 3rd field (UID), colon-delimited
sort -t, -k2,2 data.csv        # sort by 2nd field only
sort -t: -k1,1 -k3,3n file    # primary key field 1, secondary field 3 numeric
sort -k2 -k1 file.txt          # sort by col 2 then col 1
```

### 4.3 Sorting Files by Size

```bash
ls -l | sort -k5 -n            # sort ls output by file size
du -sh * | sort -h             # sort disk usage human-readable
```

---

## 5. Removing Duplicates with `uniq`

`uniq` works on **adjacent** duplicate lines — always sort first.

```bash
uniq file.txt                  # remove adjacent duplicates
sort file.txt | uniq           # sort then remove all duplicates
sort file.txt | uniq -d        # show only duplicate lines
sort file.txt | uniq -u        # show only unique lines
sort file.txt | uniq -c        # count occurrences
sort file.txt | uniq -c | sort -rn   # frequency table, most common first
uniq -i file.txt               # case-insensitive comparison
uniq -f 2 file.txt             # skip first 2 fields when comparing
uniq -s 4 file.txt             # skip first 4 characters
```

---

## 6. Counting with `wc`

```bash
wc file.txt                    # lines, words, bytes
wc -l file.txt                 # line count only
wc -w file.txt                 # word count only
wc -c file.txt                 # byte count
wc -m file.txt                 # character count (differs from bytes for UTF-8)
wc -L file.txt                 # length of longest line
wc -l *.py                     # count lines in all .py files
cat file.txt | wc -l           # pipe usage
```

---

## 7. Cutting Fields with `cut`

### 7.1 By Delimiter (Fields)

```bash
cut -d: -f1 /etc/passwd        # first field, colon-delimited (usernames)
cut -d: -f1,3 /etc/passwd      # fields 1 and 3
cut -d: -f1-3 /etc/passwd      # fields 1 through 3
cut -d, -f2- data.csv          # field 2 to end
cut -d$'\t' -f1 file.tsv       # tab-delimited
```

### 7.2 By Character Position

```bash
cut -c1-5 file.txt             # characters 1 to 5
cut -c-10 file.txt             # first 10 characters
cut -c5- file.txt              # from character 5 to end
cut -c1,5,10 file.txt          # characters 1, 5, and 10
```

### 7.3 By Bytes

```bash
cut -b1-4 file.txt             # bytes 1 to 4
```

---

## 8. Pasting and Joining Files

### 8.1 `paste` — Merge Lines Side by Side

```bash
paste file1.txt file2.txt      # merge line by line with tab separator
paste -d, file1.txt file2.txt  # use comma as delimiter
paste -d'|' file1 file2 file3  # three files, pipe delimiter
paste -s file.txt              # serial: all lines on one line
paste -s -d, file.txt          # serial with comma separator
```

### 8.2 `join` — Join on a Common Field

`join` works like a database JOIN — both files must be sorted on the join field.

```bash
join file1.txt file2.txt       # join on first field (default)
join -1 2 -2 1 file1 file2     # join file1 field 2 with file2 field 1
join -t: file1 file2           # colon delimiter
join -a 1 file1 file2          # left outer join (keep unmatched lines from file1)
join -a 2 file1 file2          # right outer join
join -v 1 file1 file2          # show only unmatched lines from file1
```

---

## 9. Translating Characters with `tr`

`tr` reads from stdin and translates or deletes characters. It does **not** accept filenames directly.

### 9.1 Basic Translation

```bash
echo "hello" | tr 'a-z' 'A-Z'             # lowercase to uppercase
echo "Hello World" | tr 'A-Z' 'a-z'       # uppercase to lowercase
echo "hello 123" | tr '0-9' 'X'           # replace digits with X
cat file.txt | tr ' ' '_'                  # replace spaces with underscores
echo "abc" | tr 'abc' 'xyz'               # character mapping
```

### 9.2 Deleting and Squeezing

```bash
echo "hello world" | tr -d ' '            # delete all spaces
echo "hello   world" | tr -s ' '          # squeeze multiple spaces to one
echo "abc123" | tr -d '0-9'              # delete all digits
echo "abc\ndef" | tr -d '\n'             # remove newlines (join lines)
cat file.txt | tr -s '\n'                 # squeeze multiple blank lines
```

### 9.3 Complement

```bash
echo "hello 123!" | tr -dc '0-9\n'        # delete everything except digits and newlines
echo "abc xyz" | tr -dc 'a-zA-Z\n'        # keep only letters
```

### 9.4 POSIX Character Classes in `tr`

```bash
tr '[:lower:]' '[:upper:]'    # portable uppercase conversion
tr '[:space:]' '\n'           # split words to lines
tr -d '[:punct:]'             # remove punctuation
```

| Class | Matches |
|---|---|
| `[:alpha:]` | Letters |
| `[:digit:]` | Digits |
| `[:alnum:]` | Letters and digits |
| `[:space:]` | Whitespace |
| `[:upper:]` | Uppercase letters |
| `[:lower:]` | Lowercase letters |
| `[:punct:]` | Punctuation |
| `[:print:]` | Printable characters |

---

## 10. Stream Editing with `sed`

`sed` (Stream EDitor) performs text transformations on input line by line.

### 10.1 Substitution (Most Common)

```bash
sed 's/old/new/' file.txt             # replace first occurrence per line
sed 's/old/new/g' file.txt            # replace all occurrences
sed 's/old/new/2' file.txt            # replace 2nd occurrence per line
sed 's/old/new/gi' file.txt           # global, case-insensitive
sed 's/old/new/gp' file.txt           # global, print matched lines
sed -i 's/old/new/g' file.txt         # edit file in-place
sed -i.bak 's/old/new/g' file.txt     # in-place with .bak backup
sed 's/^/PREFIX: /' file.txt          # add prefix to every line
sed 's/$/ SUFFIX/' file.txt           # add suffix to every line
sed 's/[[:space:]]*$//' file.txt      # strip trailing whitespace
```

### 10.2 Deleting Lines

```bash
sed '3d' file.txt                     # delete line 3
sed '3,7d' file.txt                   # delete lines 3 to 7
sed '/pattern/d' file.txt             # delete lines matching pattern
sed '/^$/d' file.txt                  # delete empty lines
sed '/^#/d' file.txt                  # delete comment lines
sed '1d' file.txt                     # delete first line (header)
sed '$d' file.txt                     # delete last line
```

### 10.3 Printing Lines

```bash
sed -n '5p' file.txt                  # print only line 5
sed -n '5,10p' file.txt               # print lines 5 to 10
sed -n '/pattern/p' file.txt          # print lines matching pattern
sed -n '/start/,/end/p' file.txt      # print between two patterns
```

### 10.4 Inserting and Appending

```bash
sed '2i\New line before' file.txt     # insert before line 2
sed '2a\New line after' file.txt      # append after line 2
sed '/pattern/a\Added after' file.txt # append after matching line
sed '1i\#!/bin/bash' script.sh        # prepend shebang line
```

### 10.5 Multiple Expressions

```bash
sed -e 's/foo/bar/g' -e 's/baz/qux/g' file.txt   # two substitutions
sed -f commands.sed file.txt                        # read commands from file
```

### 10.6 Address Ranges

```bash
sed '2,4s/old/new/' file.txt          # substitute only in lines 2-4
sed '/start/,/end/s/old/new/' file.txt # substitute between patterns
sed '0,/pattern/d' file.txt           # delete up to and including first match
```

!!! warning "In-Place Editing"
    Always use `sed -i.bak` to create a backup before in-place edits. Test your sed command without `-i` first to verify the output.

---

## 11. Field Processing with `awk`

`awk` is a full text-processing language. It reads input line by line, splits fields, and executes programs.

### 11.1 Basics and Field Variables

```bash
awk '{print}' file.txt                 # print all lines (like cat)
awk '{print $1}' file.txt             # print first field
awk '{print $1, $3}' file.txt         # print fields 1 and 3
awk '{print $NF}' file.txt            # print last field
awk '{print $(NF-1)}' file.txt        # print second-to-last field
awk '{print NR, $0}' file.txt         # print line number and full line
awk 'NR==5' file.txt                  # print line 5
awk 'NR>=5 && NR<=10' file.txt        # print lines 5 to 10
```

| awk Variable | Meaning |
|---|---|
| `$0` | Entire current line |
| `$1, $2...` | Field 1, 2… |
| `NF` | Number of fields in current line |
| `NR` | Current line (record) number |
| `FS` | Field separator (default: whitespace) |
| `OFS` | Output field separator |
| `RS` | Record separator (default: newline) |
| `ORS` | Output record separator |
| `FILENAME` | Current filename |

### 11.2 Custom Field Separators

```bash
awk -F: '{print $1}' /etc/passwd          # colon separator
awk -F, '{print $2}' data.csv             # CSV
awk -F'\t' '{print $3}' file.tsv          # tab separator
awk 'BEGIN{FS=":"} {print $1}' /etc/passwd  # set FS in BEGIN block
```

### 11.3 BEGIN and END Blocks

```bash
awk 'BEGIN{print "Start"} {print} END{print "Done"}' file.txt
awk 'BEGIN{count=0} /error/{count++} END{print count " errors"}' file.txt
awk 'BEGIN{FS=":"; OFS="\t"} {print $1,$3}' /etc/passwd
```

### 11.4 Conditions and Patterns

```bash
awk '/pattern/' file.txt               # print lines matching pattern
awk '!/pattern/' file.txt              # lines NOT matching
awk '$3 > 100' file.txt                # lines where field 3 > 100
awk '$1 == "root"' /etc/passwd         # field 1 equals "root"
awk 'NF > 3' file.txt                  # lines with more than 3 fields
awk 'length($0) > 80' file.txt         # lines longer than 80 chars
```

### 11.5 Arithmetic and Aggregation

```bash
awk '{sum += $1} END{print sum}' file.txt             # sum a column
awk '{sum += $3; count++} END{print sum/count}' file  # average
awk 'NR>1 {print $2*$3}' prices.csv                   # multiply fields
awk '{if ($1>max) max=$1} END{print max}' file.txt    # maximum value
```

### 11.6 String Functions

```bash
awk '{print length($0)}' file.txt        # line length
awk '{print toupper($1)}' file.txt       # uppercase field 1
awk '{print tolower($0)}' file.txt       # lowercase entire line
awk '{gsub(/old/, "new"); print}' file   # global substitution
awk '{sub(/old/, "new"); print}' file    # first substitution per line
awk '{print substr($0, 1, 5)}' file      # substring: chars 1-5
awk '{n=split($0, arr, ":"); print n}' file  # split and count parts
awk '{printf "%-10s %5d\n", $1, $2}' file   # formatted output
```

### 11.7 Practical awk Examples

```bash
# Print unique values of field 1
awk '!seen[$1]++' file.txt

# Sum disk usage from du output
du -sh * | awk '{sum += $1} END{print sum}'

# Print lines between two patterns
awk '/START/,/END/' file.txt

# Reorder fields
awk '{print $3, $1, $2}' file.txt

# Print every other line
awk 'NR%2==0' file.txt

# Count occurrences of each value in field 1
awk '{count[$1]++} END{for (k in count) print count[k], k}' file.txt | sort -rn
```

---

## 12. Comparing Files

### 12.1 `diff` — Line-by-Line Comparison

```bash
diff file1.txt file2.txt           # standard diff output
diff -u file1.txt file2.txt        # unified format (easier to read)
diff -i file1.txt file2.txt        # ignore case
diff -b file1.txt file2.txt        # ignore whitespace changes
diff -w file1.txt file2.txt        # ignore all whitespace
diff -y file1.txt file2.txt        # side-by-side comparison
diff -r dir1/ dir2/                # recursive directory comparison
diff --brief dir1/ dir2/           # just report if files differ
```

!!! info "Reading diff Output"
    In standard diff: `<` means line from file1, `>` means line from file2. `c` = changed, `a` = added, `d` = deleted.
    In unified diff: `-` = removed, `+` = added, `@@` lines show location.

### 12.2 `patch` — Apply a diff

```bash
diff -u original.txt modified.txt > changes.patch    # create patch
patch original.txt < changes.patch                    # apply patch
patch -R original.txt < changes.patch                 # reverse (undo) patch
patch -p1 < patchfile                                  # strip 1 leading slash from paths
```

### 12.3 `comm` — Compare Sorted Files

```bash
comm file1.txt file2.txt           # 3 columns: only-in-1, only-in-2, in-both
comm -12 file1.txt file2.txt       # lines in BOTH files (intersection)
comm -23 file1.txt file2.txt       # lines only in file1
comm -13 file1.txt file2.txt       # lines only in file2
comm -3 file1.txt file2.txt        # lines NOT in common
```

!!! warning
    Both files must be sorted before using `comm`.

### 12.4 `cmp` — Byte-by-Byte Comparison

```bash
cmp file1 file2                    # report first difference
cmp -l file1 file2                 # list all differing bytes
cmp -s file1 file2                 # silent: exit code 0 if identical
```

---

## 13. Encoding and Hashing

### 13.1 `base64`

```bash
base64 file.txt                    # encode to base64
base64 -d encoded.txt              # decode base64
echo "hello" | base64              # encode string
echo "aGVsbG8=" | base64 -d       # decode string
```

### 13.2 Checksums and Hashing

```bash
md5sum file.txt                    # MD5 hash
sha1sum file.txt                   # SHA-1 hash
sha256sum file.txt                 # SHA-256 hash
sha256sum -c checksums.txt         # verify against checksum file
md5sum *.iso > MD5SUMS             # create checksum file for all ISOs
```

---

## 14. Working with Compressed Text

Many log files are rotated and compressed. These tools let you work with them without decompressing manually.

```bash
zcat file.gz                       # cat a gzip file
zgrep "error" file.gz              # grep inside gzip
zless file.gz                      # page through gzip file
zdiff file1.gz file2.gz            # diff gzip files

bzcat file.bz2                     # cat bzip2 file
bzgrep "error" file.bz2            # grep inside bzip2

xzcat file.xz                      # cat xz file
lzcat file.lz                      # cat lzma file
```

---

## 15. `printf` and `echo` for Text Generation

```bash
echo "Hello World"                 # print with newline
echo -n "No newline"               # no trailing newline
echo -e "Line1\nLine2"             # interpret escape sequences
echo -e "\t Tabbed"                # tab character

printf "Name: %s\n" "Alice"        # formatted string
printf "%-10s %5d\n" "item" 42     # left-aligned, right-aligned
printf "%05d\n" 7                  # zero-padded number: 00007
printf "%.2f\n" 3.14159            # 2 decimal places: 3.14
printf "%x\n" 255                  # hex: ff
printf "%o\n" 8                    # octal: 10
```

---

## 16. `tee` — Read and Write Simultaneously

`tee` reads from stdin and writes to both stdout and a file — useful in pipelines when you want to save intermediate results.

```bash
ls -l | tee output.txt             # display AND save to file
ls -l | tee -a output.txt          # append instead of overwrite
ls -l | tee file1.txt file2.txt    # write to multiple files
make 2>&1 | tee build.log          # save both stdout and stderr
```

---

## 17. `xargs` — Build Commands from Input

`xargs` reads items from stdin and passes them as arguments to a command.

```bash
find . -name "*.txt" | xargs cat            # cat all found files
find . -name "*.tmp" | xargs rm             # delete all temp files
echo "a b c" | xargs -n1                    # one arg per line
echo "a b c" | xargs -n2                    # two args per invocation
find . -name "*.py" | xargs -I{} cp {} /backup/  # replace {} with each item
cat urls.txt | xargs -P4 wget               # parallel: 4 concurrent downloads
find . -print0 | xargs -0 grep "pattern"   # handle filenames with spaces
```

!!! tip "Handling Spaces in Filenames"
    Use `find -print0` with `xargs -0` to safely handle filenames containing spaces or special characters.

---

## 18. Text Manipulation in Scripts

### 18.1 Parameter Expansion (Bash Built-in)

```bash
var="Hello World"
echo ${var}                        # Hello World
echo ${#var}                       # 11 (length)
echo ${var:6}                      # World (from position 6)
echo ${var:6:3}                    # Wor (3 chars from position 6)
echo ${var/World/Linux}            # Hello Linux (replace first)
echo ${var//l/L}                   # HeLLo WorLd (replace all)
echo ${var,,}                      # hello world (lowercase)
echo ${var^^}                      # HELLO WORLD (uppercase)
echo ${var^}                       # Hello World (capitalize first)

file="report.tar.gz"
echo ${file%.gz}                   # report.tar (remove shortest suffix)
echo ${file%%.*}                   # report (remove longest suffix)
echo ${file#*.}                    # tar.gz (remove shortest prefix)
echo ${file##*.}                   # gz (extension only)
```

### 18.2 `read` — Read Input into Variables

```bash
read name                          # read into $name
read -p "Enter name: " name        # with prompt
read -s password                   # silent (for passwords)
read -t 10 answer                  # timeout after 10 seconds
read -a arr                        # read into array
IFS=: read user _ uid gid _ home shell <<< "$(grep root /etc/passwd)"
```

### 18.3 Here Documents and Here Strings

```bash
cat << EOF
Line 1
Line 2
EOF

cat <<- EOF          # strip leading tabs
    Line 1
    Line 2
EOF

grep "pattern" <<< "search in this string"    # here string
```

---

## 19. Advanced: `perl` One-Liners

Perl is often available on Linux systems and handles complex text tasks concisely.

```bash
perl -pe 's/old/new/g' file.txt          # like sed substitute
perl -ne 'print if /pattern/' file.txt   # like grep
perl -i.bak -pe 's/old/new/g' file.txt  # in-place with backup
perl -lane 'print $F[0]' file.txt        # print first field (like awk)
perl -pe 'chomp; print scalar reverse $_, "\n"' file.txt  # reverse each line
```

---

## 20. Combining Tools — Pipelines and Practical Examples

The real power of text manipulation is in chaining tools with `|`.

```bash
# Top 10 most common words in a file
cat file.txt | tr -s '[:space:]' '\n' | sort | uniq -c | sort -rn | head -10

# Count lines, words, chars in multiple files
wc -lwc *.txt | sort -n

# Find all unique IP addresses in a log
grep -oE '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' access.log | sort -u

# Extract usernames from /etc/passwd (non-system users)
awk -F: '$3 >= 1000 {print $1}' /etc/passwd

# Count HTTP status codes in Apache log
awk '{print $9}' access.log | sort | uniq -c | sort -rn

# Find and replace in multiple files
grep -rl "oldtext" . | xargs sed -i 's/oldtext/newtext/g'

# Convert CSV to TSV
cat file.csv | tr ',' '\t' > file.tsv

# Remove blank lines and comment lines from a config
grep -v '^\s*#' config.conf | grep -v '^\s*$'

# Print lines between markers
sed -n '/BEGIN/,/END/p' file.txt

# Summarize disk usage, sorted largest first
du -sh * | sort -rh | head -20

# Extract email addresses
grep -oE '[a-zA-Z0-9._%+\-]+@[a-zA-Z0-9.\-]+\.[a-zA-Z]{2,}' file.txt | sort -u

# Monitor a log file for errors in real time
tail -f /var/log/syslog | grep --line-buffered -i "error\|fail\|crit"
```

---

## 21. Quick Reference Cheat Sheet

### Viewing

| Command | What it does |
|---|---|
| `cat -n file` | Print with line numbers |
| `tac file` | Print reversed |
| `head -n N file` | First N lines |
| `tail -n N file` | Last N lines |
| `tail -f file` | Follow file live |
| `less file` | Scrollable pager |

### Searching

| Command | What it does |
|---|---|
| `grep -i pattern file` | Case-insensitive search |
| `grep -r pattern dir/` | Recursive search |
| `grep -v pattern file` | Invert match |
| `grep -n pattern file` | Show line numbers |
| `grep -o pattern file` | Print match only |
| `grep -A/-B/-C N` | Context lines |

### Transforming

| Command | What it does |
|---|---|
| `sort -n file` | Numeric sort |
| `sort -t: -k3` | Sort by field |
| `uniq -c` | Count duplicates |
| `cut -d: -f1` | Extract field |
| `tr 'a-z' 'A-Z'` | Uppercase |
| `tr -d '\n'` | Remove chars |

### sed Quick Reference

| Expression | Effect |
|---|---|
| `s/old/new/g` | Global replace |
| `/pattern/d` | Delete matching lines |
| `^$/d` | Delete blank lines |
| `-n '5,10p'` | Print lines 5-10 |
| `-i.bak` | In-place with backup |

### awk Quick Reference

| Expression | Effect |
|---|---|
| `{print $1}` | First field |
| `{print $NF}` | Last field |
| `NR==5` | Line 5 only |
| `$3>100` | Conditional filter |
| `{sum+=$1}END{print sum}` | Sum a column |
| `-F:` | Set field separator |

### Comparison

| Command | What it does |
|---|---|
| `diff -u f1 f2` | Unified diff |
| `comm -12 f1 f2` | Lines in both |
| `cmp -s f1 f2` | Silent byte compare |
| `patch f < p` | Apply patch |