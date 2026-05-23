# 🏛️ The Linux Foundation

> *"Linux is not just an operating system — it's a movement, a philosophy, and the backbone of the modern digital world."*

This document covers the **A to Z of The Linux Foundation** — its history, mission, governance, projects, and why it matters — as part of the LFS101 Introduction to Linux course.

---

## 1. What is the Linux Foundation?

The **Linux Foundation** is a non-profit consortium founded in **2000** by merging two organizations:
- **Open Source Development Labs (OSDL)**
- **Free Standards Group (FSG)**

Its primary mission is to **promote, protect, and advance Linux and collaborative open source development** by providing a neutral home for the world's most critical open source projects.

### 1.1 Mission Statement

> *"The Linux Foundation's mission is to promote, protect, and advance Linux and to support the growth of Linux and open source communities."*

In simple terms, it:
- Funds and sponsors Linux kernel development
- Provides legal protection for Linux developers
- Organizes events, training, and certification programs
- Hosts thousands of collaborative open source projects

### 1.2 Why It Matters

| Without Linux Foundation | With Linux Foundation |
|--------------------------|----------------------|
| Fragmented kernel development | Unified, coordinated development |
| No legal protection for contributors | IP protection and legal support |
- No standard certifications | Industry-recognized certifications |
| Slower adoption in enterprise | Broad corporate and community adoption |

!!! info "Did you know?"
    The Linux Foundation currently hosts over **900 open source projects** and has members including Google, Microsoft, IBM, Intel, Amazon, and thousands of individual developers worldwide.

---

## 2. History of Linux and the Foundation

### 2.1 The Birth of Linux (1991)

In **August 1991**, a 21-year-old Finnish computer science student named **Linus Torvalds** posted a now-famous message to a Usenet newsgroup:

```
"Hello everybody out there using minix —
I'm doing a (free) operating system (just a hobby, won't be big and
professional like gnu) for 386(486) AT clones."
                                    — Linus Torvalds, August 25, 1991
```

- The first version, **Linux 0.01**, was released in **September 1991**
- It was inspired by **MINIX**, a small Unix-like OS used for teaching
- Linus released it under the **GNU General Public License (GPL)** in 1992
- This decision made Linux **free and open source** forever

### 2.2 Timeline of Key Events

| Year | Event |
|------|-------|
| **1969** | Unix created at AT&T Bell Labs by Ken Thompson and Dennis Ritchie |
| **1983** | Richard Stallman announces the GNU Project |
| **1987** | MINIX released by Andrew Tanenbaum |
| **1991** | Linus Torvalds releases Linux 0.01 |
| **1992** | Linux released under GNU GPL — truly free and open |
| **1993** | Slackware and Debian, the first Linux distributions, released |
| **1994** | Linux kernel 1.0 released |
| **1996** | Tux (the penguin) becomes Linux's mascot |
| **1998** | IBM, Intel, Oracle announce major Linux support |
| **2000** | Linux Foundation's predecessor organizations formed |
| **2007** | Linux Foundation officially founded |
| **2011** | Linux kernel 3.0 released — 20th anniversary |
| **2015** | Linux Foundation launches Let's Encrypt |
| **2022** | Linux kernel 6.0 released |

### 2.3 The GNU Connection

Linux didn't grow in isolation — it was paired with the **GNU Project tools** created by **Richard Stallman** and the Free Software Foundation (FSF):

```
GNU Project + Linux Kernel = GNU/Linux Operating System

GNU provided:  gcc (compiler), bash (shell), glibc (C library),
               coreutils (ls, cp, mv...), make, gdb, and much more

Linux provided: The kernel (hardware management, process scheduling,
                memory management, device drivers)
```

!!! tip "GNU/Linux vs Linux"
    Technically, what most people call "Linux" is **GNU/Linux** — the Linux kernel combined with GNU tools. Richard Stallman prefers the term GNU/Linux to acknowledge both contributions. In everyday usage, "Linux" refers to the full system.

---

## 3. Linux Kernel

### 3.1 What is a Kernel?

The **kernel** is the core of the operating system — it is the bridge between software (applications) and hardware (CPU, memory, disk, network).

```
┌─────────────────────────────────┐
│         User Applications       │  ← What you run (browser, editor, etc.)
├─────────────────────────────────┤
│         System Libraries        │  ← glibc, system calls
├─────────────────────────────────┤
│         Linux Kernel            │  ← Process mgmt, memory, drivers, FS
├─────────────────────────────────┤
│         Hardware                │  ← CPU, RAM, Disk, Network, GPU
└─────────────────────────────────┘
```

### 3.2 What the Kernel Does

| Responsibility | Description |
|----------------|-------------|
| **Process Management** | Creates, schedules, and terminates processes |
| **Memory Management** | Allocates and frees RAM; manages virtual memory |
| **Device Drivers** | Communicates with hardware (disk, GPU, network card) |
| **File System Management** | Reads/writes files on ext4, xfs, btrfs, etc. |
| **Networking** | Manages TCP/IP stack, sockets, network interfaces |
| **Security** | Enforces permissions, namespaces, capabilities |
| **System Calls** | Provides an API for programs to talk to the kernel |

### 3.3 Kernel Versions

Linux kernel versions follow the format: `major.minor.patch`

```bash
# Check your kernel version
uname -r
# Example output: 6.5.0-45-generic

# Detailed kernel and system info
uname -a
# Example: Linux hostname 6.5.0-45-generic #45-Ubuntu SMP x86_64 GNU/Linux
```

```
Kernel version anatomy:
6  .  5  .  0  -  45  -  generic
│     │     │     │       └── Flavor (generic, lowlatency, etc.)
│     │     │     └────────── ABI (Application Binary Interface) number
│     │     └──────────────── Patch level
│     └────────────────────── Minor version
└──────────────────────────── Major version
```

### 3.4 Kernel Development Process

The Linux kernel is one of the largest collaborative software projects in history:

- **~20 million lines of code** in the kernel
- **~4,000–5,000 contributors** per release cycle
- A new kernel version is released approximately every **2–3 months**
- **Linus Torvalds** still maintains final merge authority
- Development happens publicly on the **Linux Kernel Mailing List (LKML)**

```
Kernel development flow:
Developer writes patch → Submits to subsystem maintainer
→ Subsystem maintainer reviews → Merges into linux-next
→ Linus pulls into mainline → Released as stable version
→ Stable team backports fixes → Long-Term Support (LTS) maintained
```

### 3.5 Long-Term Support (LTS) Kernels

| Type | Support Duration | Use Case |
|------|-----------------|----------|
| **Mainline** | Until next release (~2–3 months) | Latest features |
| **Stable** | A few months | General use |
| **LTS (Long-Term Support)** | 2–6 years | Production servers, embedded |
| **EOL (End of Life)** | No longer supported | Avoid |

!!! warning "Keep your kernel updated"
    Running an outdated or EOL kernel exposes you to unpatched security vulnerabilities. Always use a supported kernel version, especially on servers.

---

## 4. Linux Distributions

### 4.1 What is a Distribution?

A **Linux distribution** (or "distro") is the Linux kernel + a collection of software bundled together as a complete operating system:

```
Linux Distribution = Linux Kernel
                   + GNU Tools (bash, gcc, coreutils)
                   + Package Manager (apt, dnf, pacman)
                   + Desktop Environment (GNOME, KDE, XFCE)
                   + System Software (systemd, NetworkManager)
                   + Default Applications
                   + Configuration and Branding
```

### 4.2 Major Distribution Families

```
                        Linux Kernel
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
        Debian           Red Hat            Arch
          │                 │                 │
    ┌─────┴─────┐      ┌────┴────┐        ┌───┴───┐
  Ubuntu     Debian   RHEL     Fedora   Arch   Manjaro
    │         Mint      │         │
  Kubuntu    Kali    CentOS   AlmaLinux
  Lubuntu    Parrot  Rocky
  Pop!_OS
```

### 4.3 Choosing a Distribution

| Use Case | Recommended Distro |
|----------|--------------------|
| **Beginners / Desktop** | Ubuntu, Linux Mint, Pop!_OS |
| **Enterprise Server** | RHEL, Ubuntu Server, SLES |
| **Security / Penetration Testing** | Kali Linux, Parrot OS |
| **Privacy-focused** | Tails, Whonix |
| **Advanced / Customizable** | Arch Linux, Gentoo |
| **Minimal / Embedded** | Alpine Linux, Buildroot |
| **Scientific Computing** | Fedora, Scientific Linux |
| **Containers** | Alpine Linux, Flatcar |

### 4.4 Package Managers by Family

| Family | Package Manager | Install Command | Example |
|--------|----------------|-----------------|---------|
| **Debian/Ubuntu** | APT | `apt install` | `apt install vim` |
| **Red Hat/Fedora** | DNF/YUM | `dnf install` | `dnf install vim` |
| **Arch** | Pacman | `pacman -S` | `pacman -S vim` |
| **SUSE** | Zypper | `zypper install` | `zypper install vim` |
| **Alpine** | APK | `apk add` | `apk add vim` |

```bash
# Debian/Ubuntu — update and install
sudo apt update && sudo apt install vim -y

# Red Hat/Fedora — update and install
sudo dnf update && sudo dnf install vim -y

# Arch Linux — sync and install
sudo pacman -Syu vim
```

---

## 5. The Linux Foundation — Organization & Structure

### 5.1 How It's Organized

The Linux Foundation operates as a **neutral, non-profit hub** for open source collaboration:

```
Linux Foundation Structure:
├── Board of Directors (elected corporate + individual members)
├── Technical Advisory Board (TAB)
├── Linux Kernel Organization (kernel.org)
├── Workgroups and Special Interest Groups
└── Hosted Projects (900+ projects)
```

### 5.2 Membership Tiers

| Tier | Members | Role |
|------|---------|------|
| **Platinum** | Google, Microsoft, IBM, Intel, Meta, etc. | Strategic governance |
| **Gold** | Mid-to-large tech companies | Active participation |
| **Silver** | Startups and smaller companies | Community support |
| **Associate** | Non-profits, government, academia | Outreach and education |
| **Individual** | Independent developers | Community voice |

!!! info "Microsoft and Linux?"
    Yes — Microsoft is a Platinum member of the Linux Foundation. Microsoft Azure runs more Linux VMs than Windows VMs, and Microsoft contributes significantly to the Linux kernel (especially for Hyper-V and Azure support).

### 5.3 How the Foundation is Funded

- **Membership fees** from corporate and individual members
- **Training and certification revenue** (LF training courses, LFCS, LFCE, CKA, etc.)
- **Event revenue** (Linux Open Source Summit, KubeCon, etc.)
- **Project-specific funding** from sponsors

---

## 6. Major Linux Foundation Projects

The Linux Foundation hosts some of the most critical open source projects in the world:

### 6.1 Core Projects

| Project | Description |
|---------|-------------|
| **Linux Kernel** | The kernel itself — the foundation of everything |
| **Let's Encrypt** | Free SSL/TLS certificates for the web |
| **Node.js** | JavaScript runtime built on Chrome's V8 engine |
| **OpenSSF** | Open Source Security Foundation |
| **SPDX** | Software Package Data Exchange — software licensing standard |

### 6.2 Cloud & Container Projects

| Project | Description |
|---------|-------------|
| **Kubernetes** | Container orchestration (via CNCF) |
| **Prometheus** | Monitoring and alerting system |
| **Envoy** | Cloud-native proxy |
| **containerd** | Industry-standard container runtime |
| **Argo** | GitOps and workflow automation |
| **Helm** | Kubernetes package manager |

### 6.3 Networking Projects

| Project | Description |
|---------|-------------|
| **ONAP** | Open Network Automation Platform |
| **fd.io** | Fast data I/O for networking |
| **Akraino** | Edge computing infrastructure |
| **OpenDaylight** | Software-defined networking (SDN) |

### 6.4 Security Projects

| Project | Description |
|---------|-------------|
| **OpenSSF** | Cross-industry security collaboration |
| **Sigstore** | Software signing and verification |
| **SLSA** | Supply chain security framework |
| **Zowe** | Open source framework for IBM Z |

### 6.5 The Cloud Native Computing Foundation (CNCF)

The **CNCF** is one of the Linux Foundation's most impactful sub-foundations:

```
CNCF Mission: Make cloud-native computing ubiquitous

CNCF Project Maturity Levels:
Sandbox → Incubating → Graduated

Graduated Projects (battle-tested):
- Kubernetes   (container orchestration)
- Prometheus   (monitoring)
- Envoy        (service proxy)
- CoreDNS      (DNS)
- containerd   (container runtime)
- Fluentd      (logging)
- Jaeger       (distributed tracing)
- Vitess       (database scaling)
- Argo         (workflow engine)
- Flux         (GitOps)
```

---

## 7. Linux Kernel Development & Governance

### 7.1 Who Controls Linux?

Linux development is meritocratic and open:

```
Governance Hierarchy:
Linus Torvalds (BDFL — Benevolent Dictator For Life)
    │
    ├── Greg Kroah-Hartman (stable kernel maintainer)
    ├── Subsystem Maintainers (~100+ maintainers)
    │       ├── Networking subsystem maintainer
    │       ├── Filesystem subsystem maintainer
    │       ├── ARM architecture maintainer
    │       └── ... (many more)
    │
    └── Individual Contributors (thousands)
```

### 7.2 The Kernel Development Model

```bash
# How a kernel patch is submitted:
# 1. Write code and test locally
# 2. Generate patch
git format-patch HEAD~1

# 3. Check patch for style issues
./scripts/checkpatch.pl 0001-my-fix.patch

# 4. Send to the appropriate mailing list
git send-email --to=netdev@vger.kernel.org 0001-my-fix.patch

# 5. Respond to review feedback, revise, resubmit
# 6. Maintainer merges into their tree
# 7. Linus pulls from maintainer trees
```

### 7.3 Kernel Release Cycle

```
Week 1–2: Merge Window (new features accepted into mainline)
Week 3+:  RC Phase (Release Candidates — bug fixes only)
          rc1 → rc2 → rc3 → ... → rc7/rc8
Final:    Stable release (e.g., 6.6)
Then:     Stable team maintains 6.6.1, 6.6.2... (bug fixes)
LTS:      Some versions maintained for 2–6 years (e.g., 6.1 LTS)
```

### 7.4 Contributor Statistics

The Linux kernel is one of the most active open source projects:

```
Per kernel release cycle:
- ~10,000–15,000 patches merged
- ~4,000–5,000 unique contributors
- Changes submitted every few minutes on average

Top corporate contributors include:
- Intel, AMD (hardware support)
- Google (Android, security)
- Red Hat/IBM (enterprise features)
- Samsung (ARM, embedded)
- Meta (networking, storage)
- Microsoft (Hyper-V, Azure drivers)
```

---

## 8. Open Source Philosophy & Licensing

### 8.1 What is Open Source?

**Open source** software is software where the source code is publicly available and can be:
- **Viewed** — anyone can read the code
- **Modified** — anyone can change it
- **Distributed** — anyone can share it
- **Used** — anyone can run it (subject to license terms)

### 8.2 Free Software vs Open Source

These terms are related but philosophically different:

| | Free Software (FSF) | Open Source (OSI) |
|--|--------------------|--------------------|
| **Champion** | Richard Stallman | Eric Raymond, Bruce Perens |
| **Focus** | Freedom as a moral right | Practical development benefits |
| **Key document** | GNU Manifesto | Open Source Definition |
| **License example** | GPL | MIT, Apache, BSD, GPL |

!!! tip "The Four Freedoms (GNU)"
    Richard Stallman defined the **Four Essential Freedoms** of free software:

    - **Freedom 0**: Run the program for any purpose
    - **Freedom 1**: Study and modify the source code
    - **Freedom 2**: Redistribute copies
    - **Freedom 3**: Distribute modified versions

### 8.3 Common Open Source Licenses

| License | Type | Key Characteristics |
|---------|------|---------------------|
| **GPL v2** | Copyleft | Used by Linux kernel; derivatives must also be GPL |
| **GPL v3** | Copyleft | Adds anti-tivoization clause |
| **LGPL** | Weak copyleft | Libraries; allows proprietary linking |
| **MIT** | Permissive | Very permissive; attribution required |
| **Apache 2.0** | Permissive | Patent grant; attribution required |
| **BSD 2/3-clause** | Permissive | Minimal restrictions |
| **MPL 2.0** | Weak copyleft | File-level copyleft |

```
Copyleft spectrum:
Strong ←────────────────────────────────→ Permissive
GPL v3    GPL v2    LGPL    MPL    Apache    MIT    BSD
```

### 8.4 Why Linux Uses GPL v2

The Linux kernel is licensed under **GPL v2 specifically** (not v3). This means:

- Anyone who distributes Linux must provide the source code
- Any modifications to the kernel must also be GPL v2
- Companies **cannot** make the kernel proprietary
- This is what ensures Linux remains forever free

```
"I want Linux to be free in the same sense that the sun is free —
not as in free beer, but as in freedom."
                                    — Linus Torvalds
```

---

## 9. Linux Foundation Training & Certification

### 9.1 Training Programs

The Linux Foundation offers free and paid courses:

| Course | Level | Focus |
|--------|-------|-------|
| **LFS101** | Beginner | Introduction to Linux (this course!) |
| **LFS151** | Beginner | Introduction to Cloud Infrastructure |
| **LFS201** | Intermediate | Essentials of Linux System Administration |
| **LFS211** | Intermediate | Linux Networking and Administration |
| **LFS261** | Advanced | DevOps and SRE Fundamentals |
| **LFS272** | Advanced | Hyperledger Fabric Administration |

### 9.2 Certifications

| Certification | Full Name | Focus |
|--------------|-----------|-------|
| **LFCS** | Linux Foundation Certified SysAdmin | Linux system administration |
| **LFCE** | Linux Foundation Certified Engineer | Advanced Linux engineering |
| **CKA** | Certified Kubernetes Administrator | Kubernetes administration |
| **CKAD** | Certified Kubernetes Application Developer | Kubernetes app development |
| **CKS** | Certified Kubernetes Security Specialist | Kubernetes security |
| **PCA** | Prometheus Certified Associate | Monitoring with Prometheus |

### 9.3 How to Get Certified

```
Certification exam format:
- Performance-based (hands-on terminal, not multiple choice)
- Exam duration: 2 hours (LFCS) / 2 hours (CKA)
- Remote proctored — take from home
- Score required: 66% (LFCS) / 66% (CKA)
- Two attempts included
- Certificate valid for 3 years

Preparation tips:
1. Complete the official LF training course
2. Practice in a real Linux environment daily
3. Use KillerCoda or killer.sh for exam simulators
4. Join the Linux Foundation Slack/forum communities
5. Time yourself — exam time management is critical
```

---

## 10. Linux in the Real World

### 10.1 Where Linux Runs

Linux is everywhere — far beyond the desktop:

```
Linux Powers:
├── 🌐 Internet Infrastructure
│     ├── ~96.4% of top 1 million web servers run Linux
│     ├── Most routers and switches run Linux
│     └── DNS servers, CDNs, load balancers
│
├── ☁️ Cloud Computing
│     ├── AWS, Google Cloud, Azure (Linux hosts)
│     ├── Docker and Kubernetes (Linux containers)
│     └── Serverless functions run on Linux
│
├── 📱 Mobile
│     ├── Android (Linux kernel)
│     ├── 3+ billion Android devices worldwide
│     └── Some feature phones
│
├── 🖥️ Supercomputers
│     ├── 100% of Top 500 supercomputers run Linux
│     └── Including Frontier (world's fastest in 2022)
│
├── 🔧 Embedded Systems
│     ├── Smart TVs (Tizen, WebOS)
│     ├── Routers (OpenWrt)
│     ├── Raspberry Pi
│     └── Industrial control systems
│
└── 🚀 Space & Science
      ├── International Space Station
      ├── James Webb Space Telescope
      ├── NASA Mars Ingenuity Helicopter (Linux!)
      └── CERN particle accelerators
```

### 10.2 Linux Market Share

| Segment | Linux Market Share |
|---------|-------------------|
| Web Servers | ~96% |
| Cloud Instances | ~90%+ |
| Supercomputers | **100%** |
| Mobile (Android) | ~72% of smartphones globally |
| Desktop | ~3–4% |
| Embedded/IoT | Dominant |

!!! info "The Desktop Gap"
    Linux's desktop market share (~3–4%) is often cited as a weakness, but this is misleading. Most Linux users interact with it **through Android phones, cloud services, and web apps** — not traditional desktop environments.

### 10.3 Companies Built on Linux

```
These companies depend on Linux for their core business:
- Google     — Search, YouTube, Gmail, Android, GCP
- Amazon     — AWS, Alexa, Kindle, fulfillment centers
- Facebook/Meta — Social media infrastructure, AI/ML
- Netflix    — Streaming infrastructure
- Twitter/X  — Real-time social platform
- Wikipedia  — Encyclopedia infrastructure
- Tesla      — Infotainment and autopilot systems
- SpaceX     — Launch and spacecraft systems
```

---

## 11. Linux Community & Ecosystem

### 11.1 The Open Source Community Model

```
How Open Source Collaboration Works:
┌──────────────┐     contributes     ┌──────────────────┐
│   Developer  │ ──────────────────> │   Public Repo    │
│              │ <────────────────── │  (GitHub, etc.)  │
└──────────────┘     reviews/merges  └──────────────────┘
        ↑                                      │
        │ uses/improves                        │ available to
        │                                      ↓
┌──────────────┐                    ┌──────────────────┐
│   End User   │ <─────────────────  │    Everyone      │
└──────────────┘    free software   └──────────────────┘
```

### 11.2 Key Community Resources

| Resource | URL | Purpose |
|----------|-----|---------|
| **kernel.org** | kernel.org | Official kernel source and releases |
| **LKML** | lkml.org | Linux Kernel Mailing List archive |
| **Linux Foundation** | linuxfoundation.org | News, events, training |
| **DistroWatch** | distrowatch.com | Track Linux distributions |
| **The Linux Documentation Project** | tldp.org | HOWTOs and guides |
| **Arch Wiki** | wiki.archlinux.org | Best Linux reference wiki |
| **Stack Overflow** | stackoverflow.com | Q&A for Linux questions |
| **Reddit** | r/linux, r/linuxquestions | Community discussion |

### 11.3 Annual Linux Foundation Events

| Event | Focus |
|-------|-------|
| **Open Source Summit (North America/Europe/Japan)** | Broad open source topics |
| **KubeCon + CloudNativeCon** | Kubernetes and CNCF projects |
| **Linux Security Summit** | Kernel and Linux security |
| **Embedded Linux Conference** | IoT and embedded Linux |
| **Open Networking Summit** | Networking and SDN |
| **Linux Plumbers Conference** | Deep kernel and system-level work |

---

## 12. Getting Started with Linux

### 12.1 Ways to Try Linux

```
Option 1 — Live USB (no installation required)
  → Download Ubuntu ISO
  → Flash to USB with Rufus (Windows) or Etcher (cross-platform)
  → Boot from USB and try Linux without touching your hard drive

Option 2 — Virtual Machine (safest for beginners)
  → Install VirtualBox or VMware Workstation Player (free)
  → Create a new VM and install your chosen distro
  → Keeps Linux isolated from your main OS

Option 3 — WSL2 (Windows Subsystem for Linux)
  → Windows 10/11 users can run a Linux terminal natively
  → Run: wsl --install in PowerShell (installs Ubuntu by default)
  → Great for development without dual booting

Option 4 — Dual Boot
  → Install Linux alongside Windows/macOS
  → Choose OS at startup
  → Full hardware access, real performance
  → Risk: requires careful partitioning

Option 5 — Dedicated Machine
  → Wipe a spare laptop or PC and install Linux fully
  → Best learning experience — forces you to solve real problems
```

### 12.2 Installing Ubuntu (Most Beginner-Friendly)

```bash
# After booting from USB and selecting "Install Ubuntu":
# 1. Choose language and keyboard
# 2. Select "Normal Installation"
# 3. Choose "Erase disk and install Ubuntu" (or alongside Windows)
# 4. Set your timezone, username, and password
# 5. Wait for installation (~10–20 minutes)
# 6. Reboot and remove USB when prompted

# First things to do after installation:
sudo apt update && sudo apt upgrade -y      # Update everything
sudo apt install vim curl git wget -y       # Install useful tools
sudo apt install build-essential -y         # Install dev tools
```

### 12.3 Essential First Commands

```bash
# Who am I and where am I?
whoami              # Your username
hostname            # Your machine's name
pwd                 # Current directory (Print Working Directory)
uname -a            # System and kernel information
lsb_release -a      # Linux distribution info

# Navigate the filesystem
ls -la              # List all files with details
cd /home            # Change directory
cd ~                # Go to home directory
cd ..               # Go up one level

# Get help
man ls              # Manual page for 'ls'
ls --help           # Quick help for 'ls'
info bash           # Info pages (more detailed than man)

# System information
df -h               # Disk usage (human-readable)
free -h             # Memory usage (human-readable)
top                 # Running processes (interactive)
htop                # Better version of top (install: apt install htop)
```

---

## Quick Reference: Linux Foundation Cheat Sheet

### Key Facts to Remember

```
Linux Kernel Facts:
□ Created by Linus Torvalds in 1991
□ Licensed under GPL v2
□ ~20 million lines of code
□ New version released every 2–3 months
□ 100% of Top 500 supercomputers run Linux
□ Android is built on the Linux kernel

Linux Foundation Facts:
□ Founded in 2000 (officially 2007 after merger)
□ Non-profit consortium
□ Hosts 900+ open source projects
□ Offers LFCS, LFCE, CKA, CKAD, CKS certifications
□ Members include Google, Microsoft, IBM, Intel, Amazon

Open Source Licensing:
□ GPL v2 — Linux kernel license (copyleft)
□ MIT — most permissive, widely used
□ Apache 2.0 — permissive with patent grant
□ LGPL — for libraries; allows proprietary use
```

### Check Your System Info

```bash
# Kernel and OS info
uname -r                    # Kernel version
uname -a                    # Full kernel info
cat /etc/os-release         # Distribution info
lsb_release -a              # Distro name and version
hostnamectl                 # Hostname, kernel, OS details

# Hardware info
lscpu                       # CPU details
free -h                     # Memory usage
df -h                       # Disk usage
lsblk                       # Block devices (disks, partitions)
lspci                       # PCI devices (GPU, network card, etc.)
lsusb                       # USB devices

# Who's responsible for your kernel?
cat /proc/version           # Kernel build info
zcat /proc/config.gz 2>/dev/null | grep CONFIG_LOCALVERSION
```

### Key Directories to Know

```bash
/           # Root — top of the entire filesystem
/boot       # Kernel images, bootloader (GRUB) files
/etc        # System configuration files
/home       # User home directories (/home/username)
/root       # Root user's home directory
/bin        # Essential binaries (ls, cp, mv, bash)
/sbin       # System binaries (only root should run)
/usr        # User programs and data
/var        # Variable data (logs, databases, mail)
/tmp        # Temporary files (cleared on reboot)
/dev        # Device files (disks, terminals, etc.)
/proc       # Virtual filesystem — kernel and process info
/sys        # Virtual filesystem — hardware and kernel info
/lib        # Shared libraries for /bin and /sbin
/opt        # Optional third-party software
/mnt        # Mount points for temporary mounts
/media      # Mount points for removable media (USB, CD)
```

---

!!! tip "Continue Your Journey"
    Now that you understand **The Linux Foundation** and Linux's history, move on to **Linux Philosophy and Concepts** to learn the principles that make Linux tick — and why understanding them makes you a better Linux user.

---

*Part of the [Introduction to Linux — LFS101](https://github.com/Flux-Notes/Introduction-to-Linux-LFS101-) notes series.*
*Next: [02 — Linux Philosophy and Concepts](./02-linux-philosophy-and-concepts.md)*