# 📁 File Operations

> In Linux, **everything is a file** — regular files, directories, devices, sockets, and pipes are all represented as files. Mastering file operations is fundamental to working effectively on any Linux system.

---

## 1. The Linux Filesystem Hierarchy

### 1.1 Filesystem Hierarchy Standard (FHS)

The FHS defines the directory structure and contents of Linux distributions.

```
/
├── bin/        → Essential user binaries (ls, cp, mv)
├── boot/       → Boot loader files, kernel
├── dev/        → Device files (sda, tty, null)
├── etc/        → System-wide configuration files
├── home/       → User home directories
├── lib/        → Shared libraries for /bin and /sbin
├── lib64/      → 64-bit shared libraries
├── media/      → Mount points for removable media
├── mnt/        → Temporary mount points
├── opt/        → Optional/third-party software
├── proc/       → Virtual filesystem for process/kernel info
├── root/       → Root user's home directory
├── run/        → Runtime data (PIDs, sockets) — cleared on boot
├── sbin/       → System binaries (only root)
├── srv/        → Data for services (web, ftp)
├── sys/        → Virtual filesystem for kernel/device info
├── tmp/        → Temporary files (cleared on boot/reboot)
├── usr/        → Secondary hierarchy (user programs, libraries)
│   ├── bin/    → Most user commands
│   ├── lib/    → Libraries for /usr/bin
│   ├── local/  → Locally installed software
│   └── share/  → Architecture-independent data
└── var/        → Variable data (logs, spools, caches)
    ├── log/    → Log files
    ├── mail/   → User mailboxes
    └── spool/  → Print/cron queues
```

### 1.2 Key Paths

| Path | Purpose |
|---|---|
| `/etc/passwd` | User account info |
| `/etc/fstab` | Filesystem mount table |
| `/etc/hosts` | Static hostname resolution |
| `/var/log/syslog` | System logs |
| `/tmp` | Temporary files (world-writable) |
| `/dev/null` | Discard output |
| `/dev/zero` | Source of null bytes |
| `/dev/random` | Random data source |
| `/proc/cpuinfo` | CPU information |
| `/proc/meminfo` | Memory information |

---

## 2. File Types

### 2.1 Linux File Types

Linux has 7 file types, identifiable by the first character of `ls -l` output.

| Symbol | Type | Example |
|---|---|---|
| `-` | Regular file | `/etc/passwd`, `/bin/ls` |
| `d` | Directory | `/home/`, `/etc/` |
| `l` | Symbolic link | `/usr/bin/python → python3` |
| `c` | Character device | `/dev/tty`, `/dev/null` |
| `b` | Block device | `/dev/sda`, `/dev/sdb1` |
| `p` | Named pipe (FIFO) | `/run/systemd/initctl/fifo` |
| `s` | Socket | `/run/docker.sock` |

```bash
ls -la /dev/ | head -20    # See multiple file types
file /bin/ls               # Identify type of any file
stat /etc/passwd           # Detailed file metadata
```

### 2.2 Identifying File Types

```bash
file filename              # Detect type by content (magic bytes)
file *                     # Detect type of all files in directory
file -i filename           # Show MIME type
ls -l filename             # First char shows type
```

---

## 3. Navigating the Filesystem

### 3.1 Essential Navigation Commands

```bash
pwd                        # Print Working Directory
cd /etc                    # Change to absolute path
cd ~                       # Go to home directory
cd                         # Also goes to home directory
cd -                       # Go to previous directory
cd ..                      # Go up one level
cd ../..                   # Go up two levels
cd ../../etc               # Relative path navigation
```

### 3.2 Absolute vs Relative Paths

| Type | Format | Example |
|---|---|---|
| **Absolute** | Starts with `/` | `/home/alice/docs/report.txt` |
| **Relative** | Starts from current dir | `docs/report.txt` or `../alice/docs/` |

### 3.3 Special Path Symbols

| Symbol | Meaning |
|---|---|
| `.` | Current directory |
| `..` | Parent directory |
| `~` | Home directory of current user |
| `~alice` | Home directory of user alice |
| `-` | Previous working directory (in `cd`) |

---

## 4. Listing Files

### 4.1 `ls` Command

```bash
ls                         # List files in current directory
ls /etc                    # List files in /etc
ls -l                      # Long format (permissions, size, date)
ls -a                      # Show hidden files (starting with .)
ls -la                     # Long format + hidden files
ls -lh                     # Human-readable file sizes
ls -lt                     # Sort by modification time (newest first)
ls -ltr                    # Sort by modification time (oldest first)
ls -lS                     # Sort by file size (largest first)
ls -R                      # Recursive listing
ls -d */                   # List only directories
ls -1                      # One file per line
ls --color=auto            # Colorized output
```

### 4.2 Understanding `ls -l` Output

```
-rw-r--r-- 1 alice devs 4096 May 10 14:32 report.txt
│          │ │     │    │    │             │
│          │ │     │    │    │             └── Filename
│          │ │     │    │    └── Modification date/time
│          │ │     │    └── File size in bytes
│          │ │     └── Group
│          │ └── Owner
│          └── Hard link count
└── Permissions (type + user + group + others)
```

### 4.3 `tree` — Directory Tree View

```bash
tree                       # Tree of current directory
tree /etc                  # Tree of /etc
tree -L 2                  # Limit depth to 2 levels
tree -a                    # Include hidden files
tree -d                    # Directories only
tree -h                    # Human-readable sizes
tree -p                    # Show permissions
tree -f                    # Full path for each file
```

!!! tip "Install tree"
    ```bash
    sudo apt install tree      # Debian/Ubuntu
    sudo dnf install tree      # Fedora/RHEL
    ```

---

## 5. Creating Files and Directories

### 5.1 Creating Files

```bash
touch filename             # Create empty file (or update timestamp)
touch file1 file2 file3    # Create multiple files
touch -m filename          # Update only modification time
touch -t 202501011200 file # Set specific timestamp

> filename                 # Create empty file via redirection
echo "Hello" > file.txt    # Create file with content
echo "more" >> file.txt    # Append to file
cat > file.txt             # Interactive — type content, Ctrl+D to save
```

### 5.2 Creating Directories

```bash
mkdir mydir                # Create a directory
mkdir -p a/b/c             # Create nested directories (no error if exists)
mkdir dir1 dir2 dir3       # Create multiple directories
mkdir -m 750 secure_dir    # Create with specific permissions
```

### 5.3 Creating Files with Content

```bash
# Heredoc
cat > config.txt << EOF
server=localhost
port=8080
debug=true
EOF

# printf
printf "line1\nline2\nline3\n" > output.txt

# tee — write to file and stdout simultaneously
echo "Hello" | tee file.txt
echo "Hello" | tee -a file.txt    # Append mode
echo "Hello" | tee file1.txt file2.txt   # Write to multiple files
```

---

## 6. Copying, Moving, and Renaming

### 6.1 `cp` — Copy Files and Directories

```bash
cp source dest             # Copy file
cp file1 file2 dir/        # Copy multiple files to directory
cp -r srcdir/ destdir/     # Copy directory recursively
cp -p file dest            # Preserve permissions, timestamps, owner
cp -a srcdir/ destdir/     # Archive mode (recursive + preserve all)
cp -i file dest            # Prompt before overwrite
cp -n file dest            # Never overwrite existing file
cp -u file dest            # Copy only if source is newer
cp -v file dest            # Verbose output
cp -l file dest            # Create hard link instead of copy
cp -s file dest            # Create symbolic link instead of copy
```

### 6.2 `mv` — Move and Rename

```bash
mv oldname newname         # Rename a file
mv file /path/to/dir/      # Move file to directory
mv dir1/ dir2/             # Move/rename directory
mv file1 file2 dir/        # Move multiple files
mv -i file dest            # Prompt before overwrite
mv -n file dest            # Never overwrite
mv -u file dest            # Move only if source is newer
mv -v file dest            # Verbose
```

!!! info "mv vs cp"
    `mv` on the same filesystem is instant (just updates the directory entry). `mv` across filesystems copies then deletes. `cp` always creates a new copy.

---

## 7. Deleting Files and Directories

### 7.1 `rm` — Remove Files

```bash
rm filename                # Delete a file
rm file1 file2 file3       # Delete multiple files
rm -i file                 # Prompt for confirmation
rm -f file                 # Force delete (no prompt, no error if missing)
rm -r directory/           # Delete directory recursively
rm -rf directory/          # Force recursive delete (no prompts)
rm -v file                 # Verbose
rm -- -filename            # Delete file starting with dash
```

!!! warning "rm is permanent"
    Linux has no recycle bin by default. `rm -rf` on the wrong directory can be catastrophic. Always double-check. Consider using `trash-cli` for safer deletion.

```bash
# Safer alternatives
sudo apt install trash-cli
trash-put filename         # Move to trash instead of deleting
trash-list                 # View trashed files
trash-restore              # Restore files
trash-empty                # Permanently delete trash
```

### 7.2 `rmdir` — Remove Empty Directories

```bash
rmdir emptydir             # Only works if directory is empty
rmdir -p a/b/c             # Remove nested empty directories
```

---

## 8. Searching for Files

### 8.1 `find` — Powerful File Search

```bash
# Basic syntax: find [path] [options] [expression]

find /home -name "*.txt"           # Find .txt files under /home
find . -name "report*"             # Wildcard in current dir
find / -name "passwd" 2>/dev/null  # Suppress permission errors
find . -iname "*.TXT"              # Case-insensitive name match
find . -type f                     # Only regular files
find . -type d                     # Only directories
find . -type l                     # Only symbolic links
find . -empty                      # Empty files or directories

# By size
find / -size +100M                 # Files larger than 100 MB
find / -size -1k                   # Files smaller than 1 KB
find / -size +1G                   # Files larger than 1 GB

# By time
find . -mtime -7                   # Modified within last 7 days
find . -mtime +30                  # Modified more than 30 days ago
find . -atime -1                   # Accessed within last 24 hours
find . -newer /etc/passwd          # Newer than /etc/passwd

# By permissions
find / -perm 777                   # Exactly 777
find / -perm -o+w                  # World-writable files
find / -perm /u+s                  # SUID files

# By owner
find / -user alice                 # Files owned by alice
find / -group developers           # Files owned by group
find / -nouser                     # Files with no valid user (orphaned)

# Execute actions
find . -name "*.log" -delete       # Delete found files
find . -name "*.sh" -exec chmod +x {} \;    # chmod each found file
find . -name "*.txt" -exec cat {} \;        # cat each found file
find . -type f -exec ls -lh {} +            # Batch exec (faster)
find . -name "*.bak" -ok rm {} \;           # Interactive exec (asks each)

# Combining conditions
find . -name "*.log" -size +1M     # AND (default)
find . -name "*.log" -o -name "*.txt"  # OR
find . ! -name "*.txt"             # NOT
find . -name "*.log" -size +1M -newer /etc/passwd  # Multiple AND
```

### 8.2 `locate` — Fast Database Search

```bash
locate filename            # Search pre-built database (very fast)
locate "*.conf"            # Wildcard search
locate -i filename         # Case-insensitive
locate -c filename         # Count matches only
locate -n 10 filename      # Show only first 10 results
locate -e filename         # Verify files actually exist

sudo updatedb              # Rebuild the database (run as root)
```

!!! info "find vs locate"
    `locate` is much faster (uses a database) but may be outdated. `find` searches in real-time and is always current. Run `sudo updatedb` to refresh `locate`'s database.

### 8.3 `which`, `whereis`, `type`

```bash
which python3              # Location of executable in PATH
which -a python3           # All locations in PATH
whereis python3            # Binary, source, and man page locations
type ls                    # Is it a builtin, alias, or external command?
type -a ls                 # All locations and type info
```

---

## 9. File Permissions

### 9.1 Permission Basics

Every file has three sets of permissions for three categories:

```
-rwxr-xr--  1  alice  devs  4096  May 10  file.sh
 │││││││││
 ││││││││└── Others: r-- (read only)
 │││││││
 │││││└───── Group: r-x (read + execute)
 │││││
 ││└───────── User/Owner: rwx (read + write + execute)
 ││
 │└─────────── File type (- = regular file)
```

| Permission | File | Directory |
|---|---|---|
| `r` (4) | Read file contents | List directory contents |
| `w` (2) | Modify file | Create/delete files in directory |
| `x` (1) | Execute file | Enter directory (cd into it) |

### 9.2 `chmod` — Change Permissions

**Symbolic mode:**

```bash
chmod u+x file             # Add execute for user/owner
chmod g-w file             # Remove write for group
chmod o=r file             # Set others to read only
chmod a+x file             # Add execute for all (user, group, others)
chmod u+x,g-w file         # Multiple changes at once
chmod -R 755 directory/    # Recursive
```

**Octal mode:**

```bash
chmod 755 file             # rwxr-xr-x (common for executables)
chmod 644 file             # rw-r--r-- (common for regular files)
chmod 700 file             # rwx------ (private executable)
chmod 600 file             # rw------- (private file, e.g., SSH keys)
chmod 777 file             # rwxrwxrwx (world-writable — avoid!)
chmod 000 file             # ---------- (no permissions)
chmod -R 750 directory/    # Recursive directory permissions
```

**Octal reference table:**

| Octal | Binary | Permissions |
|---|---|---|
| 0 | 000 | `---` |
| 1 | 001 | `--x` |
| 2 | 010 | `-w-` |
| 3 | 011 | `-wx` |
| 4 | 100 | `r--` |
| 5 | 101 | `r-x` |
| 6 | 110 | `rw-` |
| 7 | 111 | `rwx` |

### 9.3 `chown` — Change Owner

```bash
chown alice file           # Change owner to alice
chown alice:devs file      # Change owner and group
chown :devs file           # Change group only (same as chgrp)
chown -R alice:devs dir/   # Recursive
chown --reference=ref file # Copy owner from reference file
```

### 9.4 `chgrp` — Change Group

```bash
chgrp devs file            # Change group to devs
chgrp -R devs directory/   # Recursive
```

### 9.5 Special Permission Bits

| Bit | Name | Effect on Files | Effect on Directories |
|---|---|---|---|
| `4000` | **SUID** | Runs as file owner's UID | (no effect) |
| `2000` | **SGID** | Runs as file owner's GID | New files inherit directory's group |
| `1000` | **Sticky Bit** | (no effect) | Only owner can delete their files |

```bash
chmod u+s file             # Set SUID
chmod g+s directory/       # Set SGID on directory
chmod +t /tmp              # Set sticky bit (like /tmp)
chmod 4755 file            # SUID + rwxr-xr-x
chmod 2775 directory/      # SGID + rwxrwxr-x
chmod 1777 /tmp            # Sticky + rwxrwxrwx

ls -l /usr/bin/passwd      # -rwsr-xr-x (SUID example)
ls -ld /tmp                # drwxrwxrwt (sticky bit example — 't')
```

### 9.6 `umask` — Default Permission Mask

`umask` defines which permissions are **removed** from newly created files/directories.

```bash
umask                      # Show current umask (e.g., 0022)
umask 027                  # Set umask (removes group write, all others)
umask -S                   # Show in symbolic format
```

| umask | New File (666 - umask) | New Dir (777 - umask) |
|---|---|---|
| `022` | `644` (rw-r--r--) | `755` (rwxr-xr-x) |
| `027` | `640` (rw-r-----) | `750` (rwxr-x---) |
| `077` | `600` (rw-------) | `700` (rwx------) |

To make umask permanent, add it to `~/.bashrc` or `~/.profile`.

### 9.7 Access Control Lists (ACLs)

ACLs extend the basic permission model to allow per-user or per-group permissions beyond owner/group/others.

```bash
getfacl file               # View ACL of a file
setfacl -m u:alice:rw file # Give alice read+write
setfacl -m g:devs:r file   # Give group devs read
setfacl -x u:alice file    # Remove alice's ACL entry
setfacl -b file            # Remove all ACL entries
setfacl -R -m u:alice:rx dir/  # Recursive ACL
setfacl -d -m g:devs:rw dir/  # Set default ACL for new files
```

!!! info "ACL Indicator"
    A `+` at the end of `ls -l` permissions (e.g., `-rw-rw-r--+`) means an ACL is set on that file.

---

## 10. Links

### 10.1 Hard Links

A hard link is another directory entry pointing to the **same inode** (same data on disk).

```bash
ln source.txt hardlink.txt        # Create hard link
ls -li source.txt hardlink.txt    # Same inode number
```

**Properties:**
- Same inode number as original
- Deleting one doesn't delete the data (until all links removed)
- Cannot span filesystems
- Cannot link to directories (usually)
- No "original" — all links are equal

### 10.2 Symbolic (Soft) Links

A symlink is a special file that contains a **path** pointing to another file.

```bash
ln -s /path/to/target linkname    # Create symbolic link
ln -s ../lib/module.py ./module.py  # Relative symlink
ln -sf target linkname            # Force overwrite existing link
ls -la                            # Shows: linkname -> target
readlink linkname                 # Print target of symlink
readlink -f linkname              # Resolve full absolute path
```

**Properties:**
- Different inode from target
- Can span filesystems
- Can link to directories
- Breaks if target is deleted (dangling symlink)
- Shows as `l` type in `ls -l`

### 10.3 Hard vs Symbolic Links

| Feature | Hard Link | Symbolic Link |
|---|---|---|
| Inode | Same as original | Different (its own) |
| Cross-filesystem | ❌ No | ✅ Yes |
| Link to directory | ❌ No (usually) | ✅ Yes |
| Survives target deletion | ✅ Yes | ❌ No (dangling) |
| `ls -l` type | `-` (same as file) | `l` |
| Size shown | Full file size | Length of path string |

```bash
# Find dangling symlinks
find /path -xtype l            # Symbolic links pointing to nonexistent files

# Find all hard links to a file
find / -inum $(stat -c %i file.txt) 2>/dev/null
```

---

## 11. Viewing File Contents

### 11.1 `cat` — Concatenate and Display

```bash
cat file.txt               # Display entire file
cat file1 file2            # Concatenate two files
cat -n file.txt            # Show line numbers
cat -A file.txt            # Show special characters ($ for newlines, ^I for tabs)
cat -s file.txt            # Suppress consecutive blank lines
cat > file.txt             # Write to file (Ctrl+D to finish)
cat >> file.txt            # Append to file
```

### 11.2 `less` and `more` — Paging

```bash
less file.txt              # Page through file (preferred)
more file.txt              # Older pager (forward only)
```

**`less` navigation:**

| Key | Action |
|---|---|
| `Space` / `f` | Next page |
| `b` | Previous page |
| `g` | Go to beginning |
| `G` | Go to end |
| `/pattern` | Search forward |
| `?pattern` | Search backward |
| `n` | Next search match |
| `N` | Previous search match |
| `q` | Quit |
| `h` | Help |

### 11.3 `head` and `tail`

```bash
head file.txt              # First 10 lines (default)
head -n 20 file.txt        # First 20 lines
head -c 100 file.txt       # First 100 bytes

tail file.txt              # Last 10 lines (default)
tail -n 20 file.txt        # Last 20 lines
tail -c 100 file.txt       # Last 100 bytes
tail -f file.txt           # Follow file in real-time (live logs)
tail -F file.txt           # Follow + reopen if file rotates
```

### 11.4 `od` and `xxd` — Binary Viewing

```bash
od -c file.bin             # Octal dump with character representation
od -x file.bin             # Hexadecimal dump
xxd file.bin               # Hex + ASCII side by side
xxd -l 64 file.bin         # First 64 bytes only
strings file.bin           # Extract readable strings from binary
```

---

## 12. File Comparison

### 12.1 `diff` — Compare Files

```bash
diff file1.txt file2.txt          # Line-by-line differences
diff -u file1 file2               # Unified format (used in patches)
diff -y file1 file2               # Side-by-side comparison
diff -i file1 file2               # Case-insensitive
diff -w file1 file2               # Ignore whitespace differences
diff -r dir1/ dir2/               # Recursive directory comparison
diff --color file1 file2          # Colorized output
```

**Reading diff output:**
```
< line only in file1
> line only in file2
---  separator
```

### 12.2 `patch` — Apply Differences

```bash
diff -u original.c modified.c > changes.patch   # Create patch
patch original.c < changes.patch                # Apply patch
patch -R original.c < changes.patch             # Reverse/undo patch
```

### 12.3 `cmp` — Byte-by-Byte Comparison

```bash
cmp file1 file2            # Reports first difference
cmp -l file1 file2         # List all differing bytes
cmp -s file1 file2         # Silent — exit code only (0=same, 1=differ)
```

### 12.4 `md5sum` / `sha256sum` — Verify Integrity

```bash
md5sum file.iso            # Generate MD5 checksum
sha256sum file.iso         # Generate SHA-256 checksum (preferred)
sha256sum -c checksums.txt # Verify files against checksum file
md5sum * > MD5SUMS         # Checksum all files in directory
```

---

## 13. Archiving and Compression

### 13.1 `tar` — Tape Archive

```bash
# Create archives
tar -cvf archive.tar files/        # Create tar (verbose)
tar -czvf archive.tar.gz files/    # Create + gzip compress
tar -cjvf archive.tar.bz2 files/   # Create + bzip2 compress
tar -cJvf archive.tar.xz files/    # Create + xz compress

# Extract archives
tar -xvf archive.tar               # Extract tar
tar -xzvf archive.tar.gz           # Extract gzip tar
tar -xjvf archive.tar.bz2          # Extract bzip2 tar
tar -xJvf archive.tar.xz           # Extract xz tar
tar -xvf archive.tar -C /dest/     # Extract to specific directory

# List contents without extracting
tar -tvf archive.tar               # List tar contents
tar -tzvf archive.tar.gz           # List gzip tar contents

# Extract specific files
tar -xvf archive.tar file.txt      # Extract only file.txt
```

**tar flags reference:**

| Flag | Meaning |
|---|---|
| `c` | Create archive |
| `x` | Extract archive |
| `t` | List contents |
| `v` | Verbose output |
| `f` | Specify filename |
| `z` | gzip compression |
| `j` | bzip2 compression |
| `J` | xz compression |
| `C` | Change to directory |
| `p` | Preserve permissions |
| `--exclude` | Exclude pattern |

### 13.2 Compression Tools

```bash
# gzip
gzip file.txt              # Compress (replaces original with .gz)
gzip -k file.txt           # Keep original file
gzip -d file.txt.gz        # Decompress (or gunzip)
gunzip file.txt.gz         # Decompress
gzip -9 file.txt           # Maximum compression
gzip -l file.txt.gz        # Show compression ratio
zcat file.txt.gz           # View without decompressing

# bzip2
bzip2 file.txt             # Compress to .bz2
bzip2 -d file.txt.bz2      # Decompress (or bunzip2)
bunzip2 file.txt.bz2       # Decompress
bzcat file.txt.bz2         # View without decompressing

# xz (best compression ratio)
xz file.txt                # Compress to .xz
xz -d file.txt.xz          # Decompress
unxz file.txt.xz           # Decompress
xzcat file.txt.xz          # View without decompressing
xz -k file.txt             # Keep original

# zip/unzip
zip archive.zip file1 file2       # Create zip
zip -r archive.zip directory/     # Recursive zip
unzip archive.zip                 # Extract zip
unzip -l archive.zip              # List contents
unzip archive.zip -d /dest/       # Extract to directory
```

**Compression comparison:**

| Format | Speed | Ratio | Common Use |
|---|---|---|---|
| gzip | Fast | Medium | General purpose, web |
| bzip2 | Slow | Better | Source code distribution |
| xz | Slowest | Best | Linux package managers |
| zip | Fast | Medium | Cross-platform, Windows compat |

---

## 14. File Metadata and Information

### 14.1 `stat` — Detailed File Info

```bash
stat file.txt              # Full metadata
stat -c %s file.txt        # File size only (bytes)
stat -c %U file.txt        # Owner name
stat -c %a file.txt        # Permissions in octal
stat -c %i file.txt        # Inode number
stat -c "%n %s %U" *.txt   # Format multiple fields
```

**stat output fields:**

| Field | Meaning |
|---|---|
| File | Filename |
| Size | File size in bytes |
| Blocks | Disk blocks allocated |
| IO Block | Block size |
| File type | regular file, directory, etc. |
| Device | Device on which file resides |
| Inode | Inode number |
| Links | Hard link count |
| Access | Permissions in octal and symbolic |
| Uid/Gid | Numeric and name of owner/group |
| Access time | Last read time (atime) |
| Modify time | Last content change (mtime) |
| Change time | Last metadata change (ctime) |
| Birth time | Creation time (not always available) |

### 14.2 Timestamps

| Timestamp | Name | Updated when... |
|---|---|---|
| `atime` | Access time | File is read |
| `mtime` | Modification time | File content changes |
| `ctime` | Change time | Metadata changes (permissions, owner) |

```bash
# Update timestamps
touch file.txt             # Update atime and mtime to now
touch -a file.txt          # Update atime only
touch -m file.txt          # Update mtime only
touch -t 202501011200 file # Set specific timestamp

# Disable atime updates (performance optimization in /etc/fstab)
# noatime option in mount
```

### 14.3 `du` — Disk Usage

```bash
du file.txt                # Disk usage of file (in 512B blocks)
du -h file.txt             # Human-readable
du -h directory/           # Recursive directory usage
du -sh directory/          # Summary (total only)
du -sh *                   # Usage of all items in current dir
du -sh /* 2>/dev/null      # Disk usage of all top-level dirs
du --max-depth=1 -h /var   # One level deep
du -ah /etc | sort -rh | head -20   # Top 20 largest items
```

### 14.4 `df` — Disk Free Space

```bash
df                         # Disk usage of all mounted filesystems
df -h                      # Human-readable
df -hT                     # Include filesystem type
df /home                   # Usage for specific mount point
df -i                      # Inode usage instead of space
```

---

## 15. File Attributes

### 15.1 `chattr` — Change File Attributes (ext filesystems)

```bash
chattr +i file             # Immutable — cannot be modified, deleted, or renamed
chattr -i file             # Remove immutable attribute
chattr +a file             # Append-only — can only be appended to
chattr +u file             # Undeletable — marked for recovery
chattr -R +i directory/    # Recursive
```

### 15.2 `lsattr` — List File Attributes

```bash
lsattr file                # Show attributes
lsattr -R directory/       # Recursive
lsattr -a                  # Include hidden files
```

**Common attribute flags:**

| Flag | Meaning |
|---|---|
| `i` | Immutable |
| `a` | Append only |
| `d` | No dump |
| `s` | Secure deletion (zeros on delete) |
| `u` | Undeletable |
| `A` | No atime updates |
| `c` | Compressed |

---

## 16. Inodes

### 16.1 What is an Inode?

An **inode** (index node) is a data structure that stores metadata about a file — everything except the filename and file content.

| Stored in Inode | NOT in Inode |
|---|---|
| File type | Filename |
| Permissions | File content |
| Owner/Group | |
| Size | |
| Timestamps (a/m/c) | |
| Hard link count | |
| Pointers to data blocks | |

```bash
ls -i file.txt             # Show inode number
stat file.txt | grep Inode # Show inode number
df -i                      # Show inode usage per filesystem
```

!!! info "Inode Exhaustion"
    A filesystem can run out of inodes even if disk space remains. When this happens, no new files can be created. Check with `df -i`.

---

## 17. Working with Special Files

### 17.1 `/dev/null`, `/dev/zero`, `/dev/random`

```bash
# /dev/null — discard output
command > /dev/null        # Discard stdout
command 2>/dev/null        # Discard stderr
command &>/dev/null        # Discard both

# /dev/zero — stream of null bytes
dd if=/dev/zero of=emptyfile bs=1M count=10   # Create 10MB zero-filled file

# /dev/random — random bytes
dd if=/dev/random of=key.bin bs=32 count=1    # 32 bytes of random data
od -An -tx1 /dev/random | head -1             # View random hex bytes
```

### 17.2 Named Pipes (FIFOs)

```bash
mkfifo mypipe              # Create a named pipe
ls -l mypipe               # Shows: prw-r--r-- (p = pipe)

# Terminal 1 — write to pipe
echo "Hello" > mypipe

# Terminal 2 — read from pipe
cat < mypipe
```

---

## 18. Quick Reference Cheat Sheet

```bash
# ─── Navigation ──────────────────────────────────────────────────────
pwd                        # Print current directory
cd /path                   # Change directory (absolute)
cd ..                      # Go up one level
cd -                       # Go to previous directory
ls -la                     # List all files (long format)
tree -L 2                  # Directory tree, 2 levels deep

# ─── Creating ────────────────────────────────────────────────────────
touch file.txt             # Create empty file
mkdir -p a/b/c             # Create nested directories
echo "text" > file.txt     # Create file with content
echo "text" >> file.txt    # Append to file

# ─── Copying, Moving, Deleting ───────────────────────────────────────
cp -a src/ dest/           # Copy directory (preserve all)
cp -i file dest            # Copy with overwrite prompt
mv oldname newname         # Rename/move
rm -rf directory/          # Force delete directory
rmdir emptydir             # Remove empty directory

# ─── Searching ───────────────────────────────────────────────────────
find . -name "*.txt"       # Find by name
find / -size +100M         # Find files > 100MB
find / -mtime -7           # Modified in last 7 days
find . -perm /u+s          # Find SUID files
locate filename            # Fast database search
which python3              # Locate executable in PATH

# ─── Permissions ─────────────────────────────────────────────────────
chmod 755 file             # rwxr-xr-x
chmod 644 file             # rw-r--r--
chmod -R 750 dir/          # Recursive permissions
chown alice:devs file      # Change owner and group
umask 022                  # Default permissions mask
getfacl file               # View ACL
setfacl -m u:alice:rw file # Set ACL

# ─── Links ───────────────────────────────────────────────────────────
ln file hardlink           # Hard link
ln -s /path/target link    # Symbolic link
readlink -f link           # Resolve symlink to full path
find . -xtype l            # Find dangling symlinks

# ─── Viewing ─────────────────────────────────────────────────────────
cat file.txt               # Display file
less file.txt              # Page through file
head -n 20 file.txt        # First 20 lines
tail -n 20 file.txt        # Last 20 lines
tail -f /var/log/syslog    # Follow log in real-time
xxd file.bin               # Hex dump

# ─── Archiving ───────────────────────────────────────────────────────
tar -czvf archive.tar.gz dir/    # Create gzip archive
tar -xzvf archive.tar.gz         # Extract gzip archive
tar -tzvf archive.tar.gz         # List archive contents
gzip -k file.txt                 # Compress, keep original
zip -r archive.zip dir/          # Create zip

# ─── Metadata ────────────────────────────────────────────────────────
stat file.txt              # Full file metadata
du -sh directory/          # Directory disk usage
df -h                      # Filesystem disk space
ls -i file.txt             # Show inode number
md5sum file.iso            # Generate checksum
sha256sum file.iso         # Generate SHA-256 checksum
diff -u file1 file2        # Compare files (unified format)
```