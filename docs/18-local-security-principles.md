# 🔐 Local Security Principles

> **Linux local security** is the practice of hardening a system from the inside out — controlling who can log in, what they can do, how files are protected, and how the system audits and responds to threats. A secure Linux system is built on the principle of **least privilege**: every user, process, and service gets only the access it absolutely needs.

---

## 1. Core Security Principles

| Principle | Description |
|---|---|
| **Least Privilege** | Grant the minimum permissions necessary to perform a task |
| **Defence in Depth** | Layer multiple security controls — no single point of failure |
| **Separation of Privilege** | Split powerful functions across multiple accounts/processes |
| **Fail Securely** | When something breaks, default to a secure state |
| **Minimal Attack Surface** | Disable/remove everything not needed |
| **Accountability** | Log actions so they can be traced to specific users |
| **Zero Trust** | Never assume a process or user is trustworthy by default |

---

## 2. User and Account Security

### 2.1 The `/etc/passwd` File

```bash
cat /etc/passwd
# Format: username:x:UID:GID:GECOS:home:shell
# root:x:0:0:root:/root:/bin/bash
# alice:x:1001:1001:Alice Smith:/home/alice:/bin/bash
# daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
```

| Field | Meaning |
|---|---|
| `username` | Login name |
| `x` | Password stored in `/etc/shadow` |
| `UID` | User ID (0 = root, 1–999 = system, 1000+ = regular) |
| `GID` | Primary group ID |
| `GECOS` | Full name / description |
| `home` | Home directory |
| `shell` | Login shell (`/sbin/nologin` = account cannot log in) |

```bash
# List all users
cut -d: -f1 /etc/passwd

# List all system accounts (UID < 1000)
awk -F: '$3 < 1000 {print $1, $3}' /etc/passwd

# List all regular users (UID >= 1000)
awk -F: '$3 >= 1000 {print $1, $3}' /etc/passwd

# Check if user exists
id alice
getent passwd alice
```

### 2.2 The `/etc/shadow` File

Shadow passwords keep the hashed passwords away from world-readable `/etc/passwd`.

```bash
# Format: username:hashed_pw:lastchange:min:max:warn:inactive:expire:reserved
# alice:$6$salt$hash...:19000:0:99999:7:::
```

| Field | Meaning |
|---|---|
| `hashed_pw` | Hashed password (`!` = locked, `*` = no login) |
| `lastchange` | Days since epoch of last password change |
| `min` | Min days before password can be changed |
| `max` | Max days before password must be changed |
| `warn` | Days before expiry to warn user |
| `inactive` | Days after expiry before account is disabled |
| `expire` | Absolute expiry date (days since epoch) |

```bash
# View shadow (root only)
sudo cat /etc/shadow

# Check password status for a user
sudo passwd -S alice
# alice P 2024-01-15 0 99999 7 -1 (P=has password, L=locked, NP=no password)

# Lock / Unlock account
sudo passwd -l alice            # lock (prepends ! to hash)
sudo passwd -u alice            # unlock

# Force password change on next login
sudo passwd -e alice            # expire password immediately
sudo chage -d 0 alice           # alternative
```

### 2.3 Password Aging with `chage`

```bash
chage -l alice                  # list password aging info
sudo chage -M 90 alice          # max 90 days before password must change
sudo chage -m 7 alice           # min 7 days before password can change
sudo chage -W 14 alice          # warn 14 days before expiry
sudo chage -I 30 alice          # disable account 30 days after expiry
sudo chage -E 2025-12-31 alice  # account expires on date
sudo chage -E -1 alice          # remove expiry date
sudo chage -d 0 alice           # force change at next login
```

### 2.4 PAM — Pluggable Authentication Modules

PAM provides a flexible mechanism for authenticating users through stacked modules.

```bash
# PAM config files
ls /etc/pam.d/                  # per-service PAM configs
cat /etc/pam.d/common-auth      # shared auth stack (Debian/Ubuntu)
cat /etc/pam.d/system-auth      # shared auth stack (RHEL/Fedora)
cat /etc/pam.d/sshd             # SSH-specific PAM config
cat /etc/pam.d/sudo             # sudo-specific PAM config
```

**PAM module types:**

| Type | Purpose |
|---|---|
| `auth` | Authenticate user (verify identity) |
| `account` | Check account validity (expiry, restrictions) |
| `password` | Update authentication credentials |
| `session` | Set up/tear down session environment |

**PAM control flags:**

| Flag | Meaning |
|---|---|
| `required` | Must succeed; failure is noted but stack continues |
| `requisite` | Must succeed; failure immediately stops the stack |
| `sufficient` | Success is enough; stack stops if prior required passed |
| `optional` | Not required; result mostly ignored |

```bash
# Example: enforce strong passwords with pam_pwquality
# /etc/pam.d/common-password:
# password requisite pam_pwquality.so retry=3 minlen=12 ucredit=-1 lcredit=-1 dcredit=-1 ocredit=-1

# Example: limit login attempts with pam_tally2 / pam_faillock
# /etc/pam.d/common-auth:
# auth required pam_faillock.so preauth silent audit deny=5 unlock_time=900
# auth [default=die] pam_faillock.so authfail audit deny=5 unlock_time=900

# View failed login attempts
sudo faillock --user alice

# Reset lockout
sudo faillock --user alice --reset
```

### 2.5 `/etc/login.defs` — Login Defaults

```bash
cat /etc/login.defs
```

Key settings:

```bash
PASS_MAX_DAYS   99999       # maximum password age
PASS_MIN_DAYS   0           # minimum password age
PASS_MIN_LEN    8           # minimum password length (PAM overrides this)
PASS_WARN_AGE   7           # days to warn before expiry
UID_MIN         1000        # minimum UID for regular users
UID_MAX         60000       # maximum UID for regular users
GID_MIN         1000        # minimum GID for regular groups
GID_MAX         60000       # maximum GID for regular groups
LOGIN_RETRIES   5           # number of login retries
LOGIN_TIMEOUT   60          # seconds before login times out
UMASK           022         # default umask for new files
CREATE_HOME     yes         # create home dir for new users
ENCRYPT_METHOD  SHA512      # password hashing algorithm
```

---

## 3. File and Directory Permissions

### 3.1 Permission Basics

```bash
ls -l file.txt
# -rwxr-xr-- 1 alice developers 4096 Jan 15 10:00 file.txt
#  │││││││││
#  ││││││││└── other: read only (r--)
#  │││││└───── group: read+exec (r-x)
#  ││└───────── owner: read+write+exec (rwx)
#  │└─────────── file type (- = regular, d = dir, l = symlink)
```

| Permission | File Effect | Directory Effect |
|---|---|---|
| `r` (4) | Read file contents | List directory contents |
| `w` (2) | Modify file contents | Create/delete files inside |
| `x` (1) | Execute file | Enter (cd into) directory |

```bash
chmod 755 file             # rwxr-xr-x
chmod 644 file             # rw-r--r--
chmod 600 file             # rw------- (private file)
chmod 700 dir              # rwx------ (private directory)
chmod u+x file             # add execute for owner
chmod g-w file             # remove write for group
chmod o= file              # remove all permissions for others
chmod a+r file             # add read for all
chmod -R 755 dir/          # recursive
```

### 3.2 Special Permission Bits

```bash
# Setuid (SUID) — file runs with owner's privileges
chmod u+s /usr/bin/passwd       # numeric: 4xxx
ls -l /usr/bin/passwd
# -rwsr-xr-x root root ...      # 's' in owner execute position

# Setgid (SGID) — file runs with group's privileges
# On directories: new files inherit group
chmod g+s /shared/dir           # numeric: 2xxx
ls -l /shared/dir
# drwxrwsr-x group group ...    # 's' in group execute position

# Sticky bit — only owner can delete their files (used on /tmp)
chmod +t /tmp                   # numeric: 1xxx
ls -ld /tmp
# drwxrwxrwt root root ...      # 't' in other execute position

# Combined: setuid + sticky on directory
chmod 1777 /tmp
chmod 3775 /shared/dir          # setgid + sticky + rwxrwxr-x
```

!!! warning "SUID Security Risk"
    SUID on executables is a privilege escalation risk. Audit all SUID/SGID binaries regularly:
    ```bash
    find / -perm -4000 -type f 2>/dev/null    # all SUID files
    find / -perm -2000 -type f 2>/dev/null    # all SGID files
    find / -perm -6000 -type f 2>/dev/null    # SUID + SGID
    ```

### 3.3 Umask — Default Permission Mask

`umask` defines which permission bits are **removed** from newly created files.

```bash
umask                           # show current umask (e.g., 0022)
umask 027                       # set umask: files=640, dirs=750
umask -S                        # symbolic form: u=rwx,g=rx,o=

# How umask works:
# New files base:      666 (rw-rw-rw-)
# New dirs base:       777 (rwxrwxrwx)
# umask 022:
#   666 - 022 = 644 (rw-r--r--)
#   777 - 022 = 755 (rwxr-xr-x)

# Common umask values
# 022 — default: owner=rw, group=r, other=r (files); owner=rwx, group=rx, other=rx (dirs)
# 027 — moderate: owner=rw, group=r, other=none
# 077 — strict: owner=rw, group=none, other=none
# 002 — collaborative: owner=rw, group=rw, other=r

# Set umask in ~/.bashrc for persistence
echo "umask 027" >> ~/.bashrc
```

### 3.4 Access Control Lists (ACLs)

ACLs extend standard permissions to allow fine-grained control per user/group.

```bash
# View ACL
getfacl file.txt
getfacl -R directory/           # recursive

# Set ACL for a specific user
setfacl -m u:bob:rw file.txt    # give bob read+write
setfacl -m u:carol:r file.txt   # give carol read only
setfacl -m g:devs:rx dir/       # give devs group read+exec on dir

# Set default ACL (inherited by new files in directory)
setfacl -d -m u:bob:rw shared/ # bob gets rw on all new files

# Remove ACL entry
setfacl -x u:bob file.txt       # remove bob's entry
setfacl -b file.txt             # remove ALL ACL entries

# Recursive ACL
setfacl -R -m u:bob:rwx project/

# Copy ACL from one file to another
getfacl source.txt | setfacl --set-file=- dest.txt
```

!!! info "ACL Prerequisites"
    The filesystem must be mounted with ACL support. Most modern Linux filesystems (ext4, xfs, btrfs) support ACLs by default. Check with `tune2fs -l /dev/sda1 | grep "Default mount"` or look for `acl` in `/proc/mounts`.

### 3.5 Extended Attributes and `chattr`

```bash
# Make file immutable (even root cannot modify)
sudo chattr +i /etc/resolv.conf
sudo chattr -i /etc/resolv.conf     # remove immutable flag

# Append-only (can add data, cannot overwrite/delete)
sudo chattr +a /var/log/app.log

# List attributes
lsattr /etc/resolv.conf
lsattr -R /etc/             # recursive

# Common attributes
# i = immutable
# a = append only
# s = secure deletion (zero on delete)
# u = undeletable
# e = extent format (usually set by default on ext4)
```

---

## 4. `sudo` — Controlled Privilege Escalation

### 4.1 Using `sudo`

```bash
sudo command                    # run command as root
sudo -u bob command             # run as user bob
sudo -i                         # interactive root shell (login shell)
sudo -s                         # root shell (non-login)
sudo !!                         # re-run last command as root
sudo -l                         # list what current user can sudo
sudo -l -U alice                # list what alice can sudo (root only)
sudo -v                         # extend sudo timestamp (re-auth)
sudo -k                         # invalidate sudo timestamp immediately
```

### 4.2 The `sudoers` File

```bash
# Always edit with visudo (syntax-checks before saving)
sudo visudo                     # edit /etc/sudoers
sudo visudo -f /etc/sudoers.d/alice  # edit a drop-in file

# /etc/sudoers syntax:
# user  host=(runas)  commands

# Allow alice to run all commands
alice   ALL=(ALL:ALL) ALL

# Allow with no password prompt
bob     ALL=(ALL) NOPASSWD: ALL

# Allow specific commands only
carol   ALL=(ALL) /usr/bin/systemctl restart nginx, /usr/bin/journalctl

# Allow a group
%wheel  ALL=(ALL:ALL) ALL        # % prefix = group
%sudo   ALL=(ALL:ALL) ALL

# Run as specific user only
deploy  ALL=(www-data) /usr/bin/php

# With no password for specific commands
ops     ALL=(ALL) NOPASSWD: /usr/sbin/reboot, /usr/sbin/shutdown

# Aliases to keep sudoers clean
User_Alias  ADMINS = alice, bob, carol
Cmnd_Alias  SERVICES = /bin/systemctl, /usr/sbin/service
Host_Alias  SERVERS = server1, server2

ADMINS  SERVERS=(ALL) SERVICES
```

### 4.3 Drop-in Files (`/etc/sudoers.d/`)

```bash
# Create per-user or per-group sudo rules
sudo visudo -f /etc/sudoers.d/deploy-user

# /etc/sudoers.d/deploy-user:
# deploy ALL=(www-data) NOPASSWD: /usr/bin/php, /usr/bin/composer

# Permissions must be 440
sudo chmod 440 /etc/sudoers.d/deploy-user

# List all sudoers configs
ls /etc/sudoers.d/
```

!!! warning "Never Edit sudoers Directly"
    Always use `visudo` — it validates syntax before saving. A syntax error in `/etc/sudoers` can lock everyone out of `sudo` permanently (requiring recovery mode to fix).

---

## 5. Process Security

### 5.1 Process Isolation Concepts

```bash
# View process tree with user context
ps auxf                         # forest view with users
ps -eo pid,user,group,comm,args # custom format
pstree -u                       # tree with usernames

# Check what user a process runs as
ps -p $(pgrep nginx) -o user,pid,comm

# Identify processes running as root
ps aux | awk '$1 == "root"'
```

### 5.2 Running Services as Non-Root

Services should never run as root if avoidable. Create dedicated service users:

```bash
# Create system user (no login, no home)
sudo useradd -r -s /sbin/nologin -d /nonexistent nginx-user

# Create with home dir for service data
sudo useradd -r -s /sbin/nologin -d /var/lib/myapp -m myapp

# Run a command as another user
sudo -u nginx-user command
su -s /bin/bash -c "command" nginx-user

# In systemd service file
# [Service]
# User=myapp
# Group=myapp
# ...
```

### 5.3 `ulimit` — Resource Limits

Limits prevent runaway processes from consuming all system resources.

```bash
ulimit -a                       # show all limits for current shell
ulimit -n 65536                 # set max open file descriptors
ulimit -u 200                   # max user processes
ulimit -m 512000                # max memory (KB)
ulimit -f 102400                # max file size (KB)
ulimit -c 0                     # disable core dumps (security)
ulimit -c unlimited             # enable core dumps (debugging)

# Hard vs soft limits
ulimit -Hn                      # show hard limit for open files
ulimit -Sn                      # show soft limit for open files
ulimit -Hn 65536                # set hard limit (root only)
ulimit -Sn 65536                # set soft limit

# Persistent limits: /etc/security/limits.conf
# username  type  resource  value
# alice     soft  nofile    4096
# alice     hard  nofile    65536
# @devs     soft  nproc     200
# *         hard  core      0           # no core dumps system-wide
```

### 5.4 Capabilities — Fine-Grained Root Powers

Instead of giving a process full root, assign only the capabilities it needs.

```bash
# View capabilities on a file
getcap /usr/bin/ping

# Set capability
sudo setcap cap_net_raw+ep /usr/bin/ping        # ping without SUID
sudo setcap cap_net_bind_service+ep /usr/bin/node  # bind port <1024

# Remove capabilities
sudo setcap -r /usr/bin/ping

# View process capabilities
cat /proc/$$/status | grep Cap
capsh --decode=0000000000003000    # decode hex capability set

# Common capabilities
# cap_net_raw        Raw network access (ping)
# cap_net_bind_service  Bind to ports < 1024
# cap_sys_admin      Wide-ranging admin operations
# cap_dac_override   Bypass file permission checks
# cap_setuid/setgid  Change UID/GID
# cap_kill           Send signals to any process
# cap_sys_ptrace     Trace processes (strace)
```

---

## 6. SSH Security

### 6.1 Hardening `sshd_config`

```bash
sudo nano /etc/ssh/sshd_config
```

```bash
# --- Authentication ---
PermitRootLogin no                  # never allow direct root login
PasswordAuthentication no           # disable password auth (key-only)
PubkeyAuthentication yes            # require public key
AuthorizedKeysFile .ssh/authorized_keys
PermitEmptyPasswords no             # never allow blank passwords
ChallengeResponseAuthentication no  # disable keyboard-interactive

# --- Access Control ---
AllowUsers alice bob deploy         # whitelist users
AllowGroups sshusers                # whitelist groups
DenyUsers root guest                # blacklist users
MaxAuthTries 3                      # brute-force protection
MaxSessions 4                       # max concurrent sessions per connection
LoginGraceTime 30                   # seconds to authenticate

# --- Protocol & Crypto ---
Protocol 2                          # SSHv2 only (SSHv1 is broken)
Port 22                             # consider changing to non-standard
AddressFamily inet                  # IPv4 only if IPv6 not needed

# --- Session Security ---
ClientAliveInterval 300             # send keepalive every 5 min
ClientAliveCountMax 2               # disconnect after 2 missed keepalives
TCPKeepAlive no                     # use SSH-level keepalive instead
Compression delayed                 # compress only after auth

# --- Features to Disable ---
X11Forwarding no                    # no GUI forwarding if not needed
AllowAgentForwarding no             # disable agent forwarding
AllowTcpForwarding no               # disable port forwarding if not needed
PermitTunnel no                     # no VPN tunnelling
PrintMotd no                        # suppress MOTD
PrintLastLog yes                    # show last login info

# --- Apply changes ---
# sudo sshd -t                      # test config syntax
# sudo systemctl restart sshd
```

### 6.2 SSH Key Best Practices

```bash
# Generate strong key
ssh-keygen -t ed25519 -C "user@host-$(date +%Y%m%d)"

# Or RSA (if ed25519 not supported)
ssh-keygen -t rsa -b 4096 -C "user@host"

# Protect private key
chmod 600 ~/.ssh/id_ed25519         # private key: owner read only
chmod 644 ~/.ssh/id_ed25519.pub     # public key: readable

# Restrict key in authorized_keys
# (prepend options before the key)
no-port-forwarding,no-X11-forwarding,no-agent-forwarding,
from="192.168.1.0/24" ssh-ed25519 AAAA... user@host

# Time-limit a key
from="192.168.1.5",expiry-time="20251231000000" ssh-ed25519 ...

# Protect ~/.ssh directory
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 644 ~/.ssh/known_hosts
```

---

## 7. Firewalls and Network Security

### 7.1 `ufw` — Quick Setup

```bash
sudo ufw default deny incoming      # block all inbound by default
sudo ufw default allow outgoing     # allow all outbound
sudo ufw allow ssh                  # allow SSH
sudo ufw allow 80/tcp               # allow HTTP
sudo ufw allow 443/tcp              # allow HTTPS
sudo ufw enable                     # activate firewall
sudo ufw status verbose             # verify rules
```

### 7.2 `iptables` — Core Rules

```bash
# Drop invalid packets
iptables -A INPUT -m state --state INVALID -j DROP

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow loopback
iptables -A INPUT -i lo -j ACCEPT

# Allow SSH from specific subnet only
iptables -A INPUT -p tcp --dport 22 -s 192.168.1.0/24 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP

# Rate-limit SSH (brute-force protection)
iptables -A INPUT -p tcp --dport 22 -m state --state NEW \
    -m recent --set --name SSH
iptables -A INPUT -p tcp --dport 22 -m state --state NEW \
    -m recent --update --seconds 60 --hitcount 4 --name SSH -j DROP

# Default deny all
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT
```

---

## 8. Logging and Auditing

### 8.1 System Logs

```bash
# Key log files
/var/log/auth.log           # authentication, sudo, SSH (Debian/Ubuntu)
/var/log/secure             # same on RHEL/Fedora
/var/log/syslog             # general system messages (Debian)
/var/log/messages           # general system messages (RHEL)
/var/log/faillog            # failed login attempts
/var/log/lastlog            # last login for every user
/var/log/wtmp               # login/logout history (binary)
/var/log/btmp               # failed login attempts (binary)

# Read binary logs
last                        # successful logins (from wtmp)
last -n 20                  # last 20 logins
lastb                       # failed logins (from btmp, root only)
lastlog                     # last login for all users
lastlog -u alice            # last login for alice

# Monitor auth log
tail -f /var/log/auth.log
grep "Failed password" /var/log/auth.log | tail -20
grep "Accepted publickey" /var/log/auth.log

# Journald (systemd)
journalctl -u sshd                      # SSH service logs
journalctl _COMM=sudo                   # all sudo activity
journalctl -p err                       # errors and above
journalctl --since "1 hour ago"         # recent entries
journalctl -f                           # follow live
```

### 8.2 `auditd` — The Linux Audit System

The audit daemon provides kernel-level event logging for security-relevant actions.

```bash
# Install and start
sudo apt install auditd audispd-plugins    # Debian/Ubuntu
sudo dnf install audit                      # RHEL/Fedora
sudo systemctl enable --now auditd

# Key files
/etc/audit/auditd.conf          # audit daemon config
/etc/audit/rules.d/audit.rules  # persistent audit rules
/var/log/audit/audit.log        # audit log file

# Manage rules
sudo auditctl -l                # list active rules
sudo auditctl -s                # show audit status
sudo auditctl -D                # delete all rules (temporary)

# Watch a file (log all access)
sudo auditctl -w /etc/passwd -p rwxa -k passwd_changes
sudo auditctl -w /etc/sudoers -p rwa -k sudoers_changes
sudo auditctl -w /etc/shadow -p rwa -k shadow_changes

# Watch a directory
sudo auditctl -w /etc/ssh/ -p rwxa -k ssh_config

# Audit syscalls
sudo auditctl -a always,exit -F arch=b64 -S execve -k exec_tracking
sudo auditctl -a always,exit -F arch=b64 -S unlink -k file_deletion

# Search audit log
sudo ausearch -k passwd_changes                 # by key
sudo ausearch -ua alice                         # by user
sudo ausearch -ts today                         # today's events
sudo ausearch -ts recent                        # last 10 minutes
sudo ausearch -m LOGIN                          # login events
sudo ausearch -m USER_AUTH -sv no              # failed auth

# Generate reports
sudo aureport                                   # general summary
sudo aureport --auth                            # authentication report
sudo aureport --failed                          # failed events
sudo aureport --login                           # login report
sudo aureport -x --summary                      # executable summary
```

**Persistent audit rules (`/etc/audit/rules.d/50-security.rules`):**

```bash
# Delete all rules first
-D

# Increase buffer size
-b 8192

# Failure mode: 1=print, 2=panic
-f 1

# Watch critical files
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/group -p wa -k identity
-w /etc/sudoers -p wa -k sudoers
-w /etc/sudoers.d/ -p wa -k sudoers
-w /etc/ssh/sshd_config -p wa -k sshd_config

# Monitor privileged commands
-a always,exit -F path=/usr/bin/sudo -F perm=x -k priv_cmd
-a always,exit -F path=/usr/bin/su -F perm=x -k priv_cmd
-a always,exit -F path=/usr/bin/passwd -F perm=x -k priv_cmd

# Monitor network config changes
-w /etc/hosts -p wa -k network_config
-w /etc/resolv.conf -p wa -k network_config

# Track use of SUID/SGID
-a always,exit -F arch=b64 -S execve -F euid=0 -F uid!=0 -k setuid_exec

# Make rules immutable (require reboot to change)
-e 2
```

---

## 9. Intrusion Detection and Monitoring

### 9.1 File Integrity Monitoring with `aide`

```bash
# Install
sudo apt install aide       # Debian/Ubuntu
sudo dnf install aide       # RHEL/Fedora

# Initialise database (do this on a known-clean system)
sudo aide --init
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Check for changes
sudo aide --check

# Update database (after legitimate changes)
sudo aide --update
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Schedule daily checks
echo "0 2 * * * root /usr/bin/aide --check | mail -s 'AIDE Report' admin@example.com" \
    | sudo tee /etc/cron.d/aide-check

# Config: /etc/aide/aide.conf or /etc/aide.conf
```

### 9.2 `fail2ban` — Automated Brute-Force Protection

```bash
# Install
sudo apt install fail2ban       # Debian/Ubuntu
sudo dnf install fail2ban       # RHEL/Fedora
sudo systemctl enable --now fail2ban

# Status
sudo fail2ban-client status                     # list all jails
sudo fail2ban-client status sshd                # SSH jail status
sudo fail2ban-client status nginx-http-auth     # nginx jail

# Manually ban / unban an IP
sudo fail2ban-client set sshd banip 1.2.3.4
sudo fail2ban-client set sshd unbanip 1.2.3.4

# Config files
/etc/fail2ban/jail.conf         # default config (do not edit)
/etc/fail2ban/jail.local        # override file (create this)
/etc/fail2ban/jail.d/           # drop-in directory
```

**Example `/etc/fail2ban/jail.local`:**

```ini
[DEFAULT]
bantime  = 3600         ; ban for 1 hour
findtime = 600          ; scan last 10 minutes
maxretry = 5            ; 5 failures before ban
banaction = iptables-multiport

[sshd]
enabled  = true
port     = ssh
logpath  = /var/log/auth.log
maxretry = 3
bantime  = 86400        ; ban SSH brute-forcers for 24 hours

[nginx-http-auth]
enabled  = true
port     = http,https
logpath  = /var/log/nginx/error.log
```

### 9.3 Checking for Rootkits with `rkhunter` / `chkrootkit`

```bash
# rkhunter
sudo apt install rkhunter
sudo rkhunter --update              # update signature database
sudo rkhunter --propupd             # update file property database
sudo rkhunter --check               # run full system check
sudo rkhunter --check --sk          # skip keypress prompts
tail /var/log/rkhunter.log          # view results

# chkrootkit
sudo apt install chkrootkit
sudo chkrootkit                     # run scan
sudo chkrootkit -q                  # quiet: only show problems
```

---

## 10. SELinux and AppArmor — Mandatory Access Control

### 10.1 SELinux (RHEL/Fedora/CentOS)

SELinux enforces Mandatory Access Control (MAC) policies at the kernel level.

```bash
# Check status
getenforce                          # Enforcing / Permissive / Disabled
sestatus                            # detailed status

# Set mode
sudo setenforce 0                   # Permissive (logs but doesn't block)
sudo setenforce 1                   # Enforcing (blocks violations)

# Persistent mode: /etc/selinux/config
# SELINUX=enforcing    # enforcing, permissive, or disabled
# SELINUXTYPE=targeted # targeted, minimum, or mls

# View SELinux context
ls -Z file.txt                      # file context
ps auxZ | grep nginx                # process context
id -Z                               # current user context

# Manage file contexts
chcon -t httpd_sys_content_t /var/www/html/index.html   # temporary
sudo restorecon -Rv /var/www/html/                       # restore defaults
semanage fcontext -a -t httpd_sys_content_t "/mydir(/.*)?"  # permanent

# View and search denials
ausearch -m avc -ts today           # SELinux denial audit events
grep "avc: denied" /var/log/audit/audit.log
sealert -a /var/log/audit/audit.log # analyse and suggest fixes

# Manage booleans (toggleable policy switches)
getsebool -a                        # list all booleans
getsebool httpd_can_network_connect # check one boolean
setsebool -P httpd_can_network_connect on  # set permanently

# Troubleshoot with audit2allow
audit2allow -a                      # suggest policy from recent denials
audit2allow -i /var/log/audit/audit.log -M mypolicy  # create module
semodule -i mypolicy.pp             # install custom module
```

### 10.2 AppArmor (Debian/Ubuntu)

AppArmor confines programs using path-based profiles.

```bash
# Check status
sudo apparmor_status
sudo aa-status                      # same

# Modes
# enforce  — blocks violations
# complain — logs violations, doesn't block
# disabled — profile inactive

# Set profile mode
sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx      # enforce
sudo aa-complain /etc/apparmor.d/usr.sbin.nginx     # complain
sudo aa-disable /etc/apparmor.d/usr.sbin.nginx      # disable

# Reload a profile
sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.nginx

# View denials
sudo dmesg | grep DENIED
sudo journalctl -k | grep DENIED
sudo tail -f /var/log/syslog | grep apparmor

# Profile location
ls /etc/apparmor.d/

# Generate a profile for a program
sudo aa-genprof /usr/bin/myapp      # interactive profile generator
sudo aa-logprof                     # update profile from recent logs
```

---

## 11. Security Hardening Checklist

### 11.1 Account Security

```bash
# Remove unused accounts
sudo userdel -r olduser

# Lock system accounts
awk -F: '$3 < 1000 {print $1}' /etc/passwd | while read u; do
    sudo passwd -l "$u" 2>/dev/null
done

# Check for accounts with empty passwords
sudo awk -F: '($2 == "" || $2 == "!!" ) {print $1}' /etc/shadow

# Check for UID 0 accounts (should be root only)
awk -F: '$3 == 0 {print $1}' /etc/passwd

# Check for accounts with no password field
awk -F: 'NF < 7 {print $1}' /etc/passwd
```

### 11.2 File System Security

```bash
# Find world-writable files (security risk)
find / -perm -0002 -not -path "/proc/*" -type f 2>/dev/null

# Find world-writable directories
find / -perm -0002 -not -path "/proc/*" -type d 2>/dev/null

# Find files with no owner
find / -nouser -o -nogroup 2>/dev/null

# Find SUID/SGID files
find / -perm -4000 -o -perm -2000 2>/dev/null | sort

# Secure /tmp
# In /etc/fstab:
# tmpfs /tmp tmpfs defaults,nosuid,noexec,nodev 0 0

# Check mount options
mount | grep " /tmp "
cat /proc/mounts | grep /tmp
```

### 11.3 Network Security

```bash
# Disable unused network services
ss -tlnp                            # identify listening services
sudo systemctl disable telnet       # disable Telnet
sudo systemctl disable rsh          # disable rsh
sudo systemctl disable rlogin       # disable rlogin
sudo systemctl disable ftp          # disable FTP

# Disable IP forwarding (if not a router)
sysctl net.ipv4.ip_forward          # check current value
sudo sysctl -w net.ipv4.ip_forward=0
echo "net.ipv4.ip_forward = 0" | sudo tee -a /etc/sysctl.d/99-security.conf

# Disable ICMP redirects
echo "net.ipv4.conf.all.accept_redirects = 0" | sudo tee -a /etc/sysctl.d/99-security.conf
echo "net.ipv4.conf.all.send_redirects = 0"   | sudo tee -a /etc/sysctl.d/99-security.conf

# Enable SYN cookies (SYN flood protection)
echo "net.ipv4.tcp_syncookies = 1" | sudo tee -a /etc/sysctl.d/99-security.conf

# Apply sysctl changes
sudo sysctl --system
```

### 11.4 Service Hardening

```bash
# List all enabled services
systemctl list-unit-files --state=enabled

# Disable unnecessary services
sudo systemctl disable --now avahi-daemon   # mDNS/zeroconf
sudo systemctl disable --now cups           # printing (if unneeded)
sudo systemctl disable --now bluetooth      # if no Bluetooth hardware
sudo systemctl disable --now rpcbind        # NFS prerequisite (if no NFS)

# Check for unnecessary open ports
ss -tlnp
nmap -sV localhost
```

---

## 12. Quick Reference Cheat Sheet

### Account Security

| Command | What it does |
|---|---|
| `passwd -l username` | Lock user account |
| `passwd -u username` | Unlock user account |
| `passwd -e username` | Force password change |
| `chage -l username` | View password aging |
| `chage -M 90 username` | Set 90-day max password age |
| `id username` | Show user UID, GID, groups |
| `faillock --user username` | View failed logins |
| `faillock --user username --reset` | Clear login lockout |

### File Permissions

| Command | What it does |
|---|---|
| `chmod 600 file` | Owner read/write only |
| `chmod 700 dir` | Owner access only |
| `chmod +t /tmp` | Set sticky bit |
| `chmod u+s file` | Set SUID |
| `chattr +i file` | Make immutable |
| `lsattr file` | List attributes |
| `getfacl file` | Show ACL |
| `setfacl -m u:bob:rw file` | Grant bob read/write via ACL |

### sudo

| Command | What it does |
|---|---|
| `sudo -l` | List sudo privileges |
| `sudo visudo` | Safely edit sudoers |
| `sudo -k` | Invalidate timestamp |
| `sudo -u bob cmd` | Run command as bob |

### Auditing

| Command | What it does |
|---|---|
| `last` | Recent successful logins |
| `lastb` | Recent failed logins |
| `lastlog` | Last login for all users |
| `ausearch -k keyname` | Search audit log by key |
| `aureport --auth` | Authentication report |
| `auditctl -l` | List active audit rules |
| `auditctl -w file -p rwa -k key` | Watch a file |

### Intrusion Detection

| Command | What it does |
|---|---|
| `sudo aide --check` | Check for file changes |
| `sudo rkhunter --check` | Scan for rootkits |
| `sudo fail2ban-client status sshd` | SSH jail status |
| `sudo chkrootkit` | Alternative rootkit scan |
| `getenforce` | SELinux enforcement mode |
| `sudo aa-status` | AppArmor profile status |