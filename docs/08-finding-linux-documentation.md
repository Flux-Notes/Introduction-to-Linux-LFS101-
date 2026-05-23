# 📚 Finding Linux Documentation

> Linux ships with an extensive built-in documentation system — from manual pages and info documents to online wikis and community forums. Knowing where and how to look is a core skill for any Linux user or administrator.

---

## 1. The man Command

### 1.1 What are Man Pages?

**Man pages** (manual pages) are the primary built-in documentation system for Linux. Almost every command, system call, config file, and library function has a man page.

```bash
# Open a man page
man ls
man grep
man bash
man 5 passwd           # Section 5 (file formats) for passwd
```

### 1.2 Man Page Sections

| Section | Contents |
|---------|---------|
| **1** | User commands (`ls`, `grep`, `cp`) |
| **2** | System calls (`open`, `read`, `fork`) |
| **3** | C library functions (`printf`, `malloc`) |
| **4** | Device files (`/dev/null`, `/dev/sda`) |
| **5** | File formats and conventions (`/etc/passwd`, `fstab`) |
| **6** | Games and screensavers |
| **7** | Miscellaneous (standards, conventions, macros) |
| **8** | System admin commands (`mount`, `fdisk`, `systemctl`) |

```bash
# Specify section explicitly
man 1 printf           # printf shell command
man 3 printf           # printf C library function
man 5 crontab          # crontab file format
man 8 useradd          # useradd admin command

# Search all sections by keyword
man -k keyword
man -k "copy file"
man -k passwd          # All pages mentioning passwd

# One-line descriptions of all matches
whatis ls
whatis -w "ls*"        # Wildcard
```

### 1.3 Navigating Man Pages

Man pages open in `less` by default:

| Key | Action |
|-----|--------|
| `Space` / `PgDn` | Scroll down one page |
| `b` / `PgUp` | Scroll up one page |
| `↑` / `↓` | Scroll one line |
| `/pattern` | Search forward |
| `?pattern` | Search backward |
| `n` | Next search match |
| `N` | Previous search match |
| `g` | Go to beginning |
| `G` | Go to end |
| `q` | Quit |

### 1.4 Man Page Structure

A typical man page contains these sections in order:

| Section | Description |
|---------|-------------|
| **NAME** | Command name and one-line description |
| **SYNOPSIS** | Usage syntax with options |
| **DESCRIPTION** | Full explanation of what the command does |
| **OPTIONS** | All flags and their meanings |
| **EXAMPLES** | Usage examples |
| **FILES** | Related configuration files |
| **SEE ALSO** | Related commands and pages |
| **BUGS** | Known issues |
| **AUTHOR** | Who wrote it |

```bash
# Jump directly to a section while in man
# Press / then type the section name
/EXAMPLES
/OPTIONS
```

---

## 2. The info Command

### 2.1 What is info?

**info** is the GNU documentation system, offering more detailed and structured documentation than man pages for GNU tools. Pages are hyperlinked nodes rather than flat text.

```bash
# Open info page for a command
info ls
info bash
info coreutils

# Open info at a specific node
info coreutils "ls invocation"
```

### 2.2 Navigating info Pages

| Key | Action |
|-----|--------|
| `Space` / `PgDn` | Scroll down |
| `b` / `PgUp` | Scroll up |
| `Tab` | Move to next hyperlink |
| `Enter` | Follow a hyperlink (node) |
| `l` | Go back to previous node |
| `n` | Next node at same level |
| `p` | Previous node at same level |
| `u` | Up to parent node |
| `t` | Go to top (table of contents) |
| `/pattern` | Search forward |
| `q` | Quit |

!!! tip
    If a command has both a `man` page and an `info` page, the `info` page is usually more detailed, especially for GNU coreutils like `ls`, `cp`, `grep`, and `awk`.

---

## 3. Built-in Help

### 3.1 --help Flag

Nearly every command accepts `--help` or `-h` for a concise usage summary — faster than opening a full man page.

```bash
ls --help
grep --help
tar --help
systemctl --help
python3 --help

# Pipe through less if output is long
grep --help | less
tar --help 2>&1 | less    # Some commands write help to stderr
```

### 3.2 Shell Built-in help

For **bash built-in commands** (not external programs), use the `help` command instead of `man`:

```bash
# List all bash built-ins
help

# Help for a specific built-in
help cd
help echo
help export
help alias
help if
help for
help while
```

!!! info
    Built-ins like `cd`, `echo`, `export`, `alias`, `source`, `read`, and `test` live inside the shell itself — they have no separate man page. `man echo` shows the external `/bin/echo`, not the built-in.

### 3.3 type — Identify What a Command Is

```bash
# Show what type a command is
type ls           # ls is /usr/bin/ls (external)
type cd           # cd is a shell builtin
type echo         # echo is a shell builtin
type ll           # ll is aliased to 'ls -lh ...'
type python3      # python3 is /usr/bin/python3

# Show all occurrences
type -a echo      # Built-in AND /bin/echo
```

---

## 4. The /usr/share/doc Directory

### 4.1 Package Documentation

Most installed packages include documentation in `/usr/share/doc/<package-name>/`:

```bash
# List documentation for installed packages
ls /usr/share/doc/

# View docs for a specific package
ls /usr/share/doc/bash/
ls /usr/share/doc/nginx/

# Common files found there
# README       — general introduction
# changelog    — version history
# copyright    — license information
# examples/    — sample config files
```

```bash
# Read a compressed changelog
zcat /usr/share/doc/bash/changelog.Debian.gz | less

# Read plain text docs
cat /usr/share/doc/nginx/README
less /usr/share/doc/openssh-server/README.Debian
```

---

## 5. Online Documentation Resources

### 5.1 Official Distribution Wikis

| Distro | Wiki / Docs URL |
|--------|----------------|
| **Arch Linux** | [wiki.archlinux.org](https://wiki.archlinux.org) |
| **Ubuntu** | [help.ubuntu.com](https://help.ubuntu.com) |
| **Debian** | [wiki.debian.org](https://wiki.debian.org) |
| **Fedora** | [docs.fedoraproject.org](https://docs.fedoraproject.org) |
| **openSUSE** | [doc.opensuse.org](https://doc.opensuse.org) |
| **Gentoo** | [wiki.gentoo.org](https://wiki.gentoo.org) |

!!! tip
    The **Arch Wiki** is the most comprehensive Linux reference available — useful even on non-Arch systems. It covers hardware, software, configuration, and troubleshooting in exhaustive detail.

### 5.2 General Linux Resources

| Resource | URL | Best For |
|----------|-----|---------|
| **The Linux Documentation Project (TLDP)** | tldp.org | HOWTOs, guides, man pages |
| **Linux man pages online** | man7.org | Browse man pages in browser |
| **GNU Manuals** | gnu.org/manual | GNU tool documentation |
| **Stack Exchange (Unix & Linux)** | unix.stackexchange.com | Q&A for specific problems |
| **Stack Overflow** | stackoverflow.com | Scripting and programming |
| **Reddit r/linux** | reddit.com/r/linux | Community discussion |
| **Reddit r/linuxquestions** | reddit.com/r/linuxquestions | Beginner help |

### 5.3 Package-Specific Docs

```bash
# Find the homepage/docs for an installed package (Debian/Ubuntu)
apt show package-name | grep Homepage

# Example
apt show nginx | grep Homepage
apt show python3 | grep Homepage
```

---

## 6. Searching for Documentation

### 6.1 apropos — Search Man Page Descriptions

`apropos` searches man page **names and short descriptions** for a keyword — useful when you don't know the exact command name.

```bash
# Search for commands related to a topic
apropos compress
apropos network
apropos "disk usage"
apropos partition

# Same as man -k
man -k compress
```

### 6.2 Finding Config File Documentation

```bash
# Most config files have a man page in section 5
man 5 fstab
man 5 crontab
man 5 sudoers
man 5 sshd_config
man 5 passwd
man 5 resolv.conf

# Check if a config file has inline comments
grep "^#" /etc/ssh/sshd_config | less
```

### 6.3 journalctl for Error Lookup

```bash
# Find errors in system logs to research
journalctl -p err -b | head -30

# Search logs for a specific service
journalctl -u nginx --since "1 hour ago"
```

---

## 📋 Quick Reference Cheat Sheet

```bash
# --- man pages ---
man command              # Open manual page
man 5 passwd             # Section 5 (file formats)
man -k keyword           # Search all man pages by keyword
whatis command           # One-line description

# --- info pages ---
info command             # Open GNU info page

# --- Quick help ---
command --help           # Short usage summary
help cd                  # Help for bash built-ins
type command             # Identify command type

# --- Package docs ---
ls /usr/share/doc/package-name/

# --- Search ---
apropos keyword          # Search man descriptions
man -k keyword           # Same as apropos
```

| Command | Description |
|---------|-------------|
| `man ls` | Open man page for `ls` |
| `man -k keyword` | Search man pages by keyword |
| `man 5 passwd` | Man page for passwd file format |
| `whatis command` | One-line command description |
| `info command` | Open GNU info documentation |
| `command --help` | Quick usage summary |
| `help cd` | Help for bash built-in commands |
| `type command` | Check if command is built-in, alias, or binary |
| `apropos keyword` | Find commands related to a topic |
| `ls /usr/share/doc/` | Browse installed package documentation |