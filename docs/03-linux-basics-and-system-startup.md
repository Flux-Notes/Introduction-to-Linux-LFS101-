# 🚀 Linux Basics and System Startup

> Understanding how Linux boots and starts up is fundamental to system administration. This covers the full boot sequence from power-on to login prompt, init systems, partition layouts, and essential filesystem concepts.

---

## 1. The Boot Process Overview

### 1.1 Boot Sequence at a Glance

When you power on a Linux machine, it goes through a precise sequence of steps before handing control to the user.

```
Power On
   ↓
BIOS / UEFI (Firmware)
   ↓
Bootloader (GRUB2)
   ↓
Linux Kernel
   ↓
initramfs / initrd
   ↓
Init System (systemd / SysVinit / Upstart)
   ↓
Login Prompt / Desktop
```

### 1.2 Boot Stages Summary

| Stage | Component | Role |
|-------|-----------|------|
| 1 | BIOS / UEFI | POST, finds bootable device |
| 2 | Bootloader | Loads the kernel into memory |
| 3 | Kernel | Initializes hardware, mounts root fs |
| 4 | initramfs | Temporary root fs for early boot tasks |
| 5 | Init System | Starts all services and processes |
| 6 | Shell / Display Manager | Presents login prompt or GUI |

---

## 2. BIOS and UEFI

### 2.1 BIOS (Basic Input/Output System)

**BIOS** is the legacy firmware found on older hardware. It runs immediately on power-on before any OS code.

- Performs **POST** (Power-On Self-Test) — checks hardware
- Reads the **MBR** (Master Boot Record) from the first 512 bytes of a disk
- Hands off to the bootloader found in the MBR

!!! info
    BIOS is limited to **2 TB** disk sizes and **4 primary** partitions with MBR. It is being replaced by UEFI on modern hardware.

### 2.2 UEFI (Unified Extensible Firmware Interface)

**UEFI** is the modern replacement for BIOS with far more features.

| Feature | BIOS | UEFI |
|---------|------|------|
| Max disk size | 2 TB (MBR) | 9.4 ZB (GPT) |
| Partition scheme | MBR | GPT |
| Boot mode | 16-bit real mode | 32/64-bit protected mode |
| Secure Boot | ❌ | ✅ |
| Boot speed | Slower | Faster |
| GUI setup | ❌ (text only) | ✅ (often graphical) |
| Network boot | Limited | Full PXE support |

### 2.3 Secure Boot

**Secure Boot** is a UEFI feature that ensures only signed bootloaders and kernels can run. Linux distributions ship with **shim** — a small signed bootloader that can chain-load GRUB.

!!! warning
    Secure Boot can cause issues when installing custom kernels or unsigned drivers. It can be disabled in UEFI settings if needed.

---

## 3. Disk Partitioning

### 3.1 MBR vs GPT

| Feature | MBR | GPT |
|---------|-----|-----|
| Max partitions | 4 primary (or 3 + extended) | 128 primary |
| Max disk size | 2 TB | 9.4 ZB |
| Partition table location | Sector 0 | Sector 1 (with backup at end) |
| Redundancy | None | Backup table at end of disk |
| Required for UEFI | ❌ | ✅ (recommended) |

### 3.2 Recommended Partition Layout

#### For a typical Linux desktop (UEFI + GPT):

| Partition | Mount Point | Size | Filesystem | Purpose |
|-----------|-------------|------|------------|---------|
| EFI System | `/boot/efi` | 512 MB – 1 GB | FAT32 | UEFI bootloader files |
| Boot | `/boot` | 1 GB | ext4 | Kernel and initramfs |
| Root | `/` | 20–50 GB | ext4 / btrfs | OS and installed packages |
| Home | `/home` | Remaining | ext4 / btrfs | User data and configs |
| Swap | `[swap]` | 1× or 2× RAM | swap | Virtual memory overflow |

#### For a server (minimal):

| Partition | Mount | Size | Notes |
|-----------|-------|------|-------|
| EFI | `/boot/efi` | 512 MB | UEFI only |
| Root | `/` | 20+ GB | All data |
| Swap | `[swap]` | = RAM | Hibernation support |

!!! tip
    On systems with 16+ GB RAM, a swap partition of **4–8 GB** is usually sufficient unless you need hibernation (which requires swap ≥ RAM size).

### 3.3 Common Filesystems

| Filesystem | Distro Default | Features |
|------------|---------------|----------|
| **ext4** | Ubuntu, Debian | Stable, journaling, widely supported |
| **xfs** | RHEL, Fedora | High performance, large files |
| **btrfs** | openSUSE, Fedora | Snapshots, RAID, COW |
| **f2fs** | Some embedded | Flash-optimized |
| **FAT32** | EFI partition | Required for UEFI ESP |
| **tmpfs** | `/tmp`, `/run` | RAM-based, cleared on reboot |

---

## 4. The Bootloader (GRUB2)

### 4.1 What is GRUB?

**GRUB2** (GRand Unified Bootloader version 2) is the standard Linux bootloader. It:

- Presents a boot menu (if multiple OS/kernels are installed)
- Loads the selected kernel into RAM
- Passes kernel parameters (options) to the kernel
- Hands off execution to the kernel

### 4.2 GRUB Boot Menu

At startup, GRUB may display a menu where you can:

- Select which kernel to boot
- Boot into **recovery mode**
- Edit kernel parameters temporarily
- Boot a different OS (dual-boot)

!!! tip
    Hold **`Shift`** (BIOS) or **`Esc`** (UEFI) during boot to force the GRUB menu to appear if it is hidden.

### 4.3 Key GRUB Files

| File / Directory | Purpose |
|-----------------|---------|
| `/boot/grub/grub.cfg` | Main GRUB configuration (auto-generated, do not edit) |
| `/etc/default/grub` | User-editable GRUB settings |
| `/etc/grub.d/` | Scripts that generate `grub.cfg` |
| `/boot/efi/EFI/` | UEFI bootloader files |

### 4.4 Editing GRUB Settings

```bash
# Edit user settings
sudo nano /etc/default/grub

# Regenerate grub.cfg after editing
sudo update-grub           # Debian/Ubuntu
sudo grub2-mkconfig -o /boot/grub2/grub.cfg  # RHEL/Fedora
```

### 4.5 Common GRUB Parameters (`/etc/default/grub`)

```bash
GRUB_DEFAULT=0                  # Default menu entry (0 = first)
GRUB_TIMEOUT=5                  # Seconds to show menu before auto-boot
GRUB_TIMEOUT_STYLE=menu         # hidden, countdown, or menu
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"  # Kernel params for normal boot
GRUB_CMDLINE_LINUX=""           # Extra kernel params for all boots
```

### 4.6 Useful Kernel Parameters

| Parameter | Effect |
|-----------|--------|
| `quiet` | Suppress most boot messages |
| `splash` | Show splash screen |
| `nomodeset` | Disable GPU mode-setting (fix display issues) |
| `single` / `1` | Boot into single-user (rescue) mode |
| `ro` | Mount root filesystem read-only initially |
| `rw` | Mount root filesystem read-write from start |
| `init=/bin/bash` | Boot directly to shell (recovery) |

---

## 5. The Linux Kernel Initialization

### 5.1 What the Kernel Does at Boot

Once GRUB loads the kernel image (`vmlinuz`) into memory and executes it:

1. Kernel decompresses itself
2. Detects and initializes CPU, memory, and hardware
3. Sets up memory management (MMU)
4. Mounts **initramfs** as a temporary root filesystem
5. Executes `/init` inside initramfs
6. Performs final hardware setup (drivers, modules)
7. Mounts the real root filesystem (`/`)
8. Executes the **init system** (PID 1)

### 5.2 initramfs / initrd

**initramfs** (initial RAM filesystem) is a compressed archive containing a minimal temporary root filesystem used during early boot.

```
GRUB loads:
  ├── vmlinuz      ← compressed kernel image
  └── initramfs    ← temporary root filesystem (cpio archive)
```

**Purpose of initramfs:**

- Load kernel modules needed to mount the real root (e.g., disk drivers)
- Handle encrypted disks (LUKS), LVM, software RAID
- Mount the real root filesystem
- Hand control to the real init system

!!! info
    On Debian/Ubuntu: `initramfs-tools` manages initramfs. On RHEL/Fedora: `dracut` is used instead.

```bash
# Rebuild initramfs (Ubuntu/Debian)
sudo update-initramfs -u

# Rebuild initramfs (RHEL/Fedora)
sudo dracut --force
```

### 5.3 Kernel Image Files

| File | Description |
|------|-------------|
| `vmlinux` | Uncompressed kernel ELF binary |
| `vmlinuz` | Compressed kernel image (z = zlib) |
| `bzImage` | Big zImage — most common on x86 |
| `uImage` | U-Boot wrapped kernel (embedded) |

```bash
# View kernel images in /boot
ls -lh /boot/vmlinuz*
ls -lh /boot/initrd*
```

---

## 6. Init Systems

### 6.1 What is an Init System?

The **init system** is the first process started by the kernel (PID 1). It is responsible for:

- Starting all system services and daemons
- Managing system runlevels / targets
- Handling service dependencies
- Rebooting or shutting down the system

### 6.2 Init System Comparison

| Init System | Used By | Configuration |
|------------|---------|---------------|
| **systemd** | Ubuntu, Fedora, RHEL, Debian (modern) | Unit files in `/etc/systemd/system/` |
| **SysVinit** | Legacy RHEL, older Debian | Shell scripts in `/etc/init.d/` |
| **Upstart** | Older Ubuntu (pre-15.04) | Jobs in `/etc/init/` |
| **OpenRC** | Gentoo, Alpine | Scripts in `/etc/init.d/` |
| **runit** | Void Linux | Service dirs in `/etc/sv/` |

!!! info
    **systemd** is now the default init system on nearly all major Linux distributions.

### 6.3 systemd

**systemd** is a system and service manager that handles parallel service startup, dependency resolution, logging (journald), and more.

#### Key systemd Concepts

| Concept | Description |
|---------|-------------|
| **Unit** | A configuration object (service, socket, mount, etc.) |
| **Target** | Group of units (replaces SysV runlevels) |
| **Journal** | Binary logging system (`journald`) |
| **Socket activation** | Start services on-demand when a socket receives a connection |
| **Cgroups** | Resource isolation per service |

#### systemd Unit Types

| Unit Type | Extension | Purpose |
|-----------|-----------|---------|
| Service | `.service` | Background daemons |
| Target | `.target` | Grouping / runlevel equivalent |
| Socket | `.socket` | Socket-based activation |
| Mount | `.mount` | Filesystem mount points |
| Timer | `.timer` | Scheduled tasks (cron replacement) |
| Path | `.path` | Trigger on filesystem path change |
| Device | `.device` | Hardware device units |

#### systemd Targets (Runlevel Equivalents)

| systemd Target | SysV Runlevel | Description |
|----------------|--------------|-------------|
| `poweroff.target` | 0 | Halt / power off |
| `rescue.target` | 1 | Single-user / rescue mode |
| `multi-user.target` | 3 | Multi-user, no GUI |
| `graphical.target` | 5 | Multi-user with GUI |
| `reboot.target` | 6 | Reboot |
| `emergency.target` | — | Minimal emergency shell |

```bash
# Check current target
systemctl get-default

# Set default target
sudo systemctl set-default multi-user.target

# Switch to a target immediately
sudo systemctl isolate rescue.target
```

---

## 7. Managing Services with systemctl

### 7.1 Basic Service Commands

```bash
# Start a service
sudo systemctl start <service>

# Stop a service
sudo systemctl stop <service>

# Restart a service
sudo systemctl restart <service>

# Reload config without restarting
sudo systemctl reload <service>

# Enable service at boot
sudo systemctl enable <service>

# Disable service at boot
sudo systemctl disable <service>

# Check service status
systemctl status <service>

# Check if service is active
systemctl is-active <service>

# Check if service is enabled
systemctl is-enabled <service>
```

### 7.2 Viewing System State

```bash
# List all running services
systemctl list-units --type=service --state=running

# List all failed units
systemctl --failed

# List all unit files
systemctl list-unit-files

# Show dependencies of a unit
systemctl list-dependencies <service>
```

### 7.3 Example: nginx Service Unit

```ini
# /etc/systemd/system/nginx.service
[Unit]
Description=The NGINX HTTP Server
After=network.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID

[Install]
WantedBy=multi-user.target
```

!!! tip
    After creating or editing a unit file, run `sudo systemctl daemon-reload` before starting the service.

---

## 8. System Logging

### 8.1 journald (systemd Journal)

**journald** collects logs from the kernel, services, and applications into a structured binary log.

```bash
# View all logs (newest at bottom)
journalctl

# Follow new log entries in real time
journalctl -f

# View logs for a specific service
journalctl -u nginx.service

# View logs since last boot
journalctl -b

# View logs from previous boot
journalctl -b -1

# View kernel messages
journalctl -k

# Filter by priority (0=emerg ... 7=debug)
journalctl -p err

# Show logs since a time
journalctl --since "2024-01-01 10:00:00"
journalctl --since "1 hour ago"
```

### 8.2 Traditional Log Files

| File | Contents |
|------|---------|
| `/var/log/syslog` | General system messages (Debian/Ubuntu) |
| `/var/log/messages` | General system messages (RHEL/Fedora) |
| `/var/log/auth.log` | Authentication logs |
| `/var/log/kern.log` | Kernel messages |
| `/var/log/boot.log` | Boot process messages |
| `/var/log/dmesg` | Kernel ring buffer (hardware init) |
| `/var/log/apt/` | Package manager logs (Debian/Ubuntu) |
| `/var/log/dnf.log` | Package manager logs (Fedora/RHEL) |

```bash
# View kernel ring buffer
dmesg
dmesg | grep -i error
dmesg --level=err,warn
```

---

## 9. Shutdown and Reboot

### 9.1 systemctl Commands

```bash
# Shutdown immediately
sudo systemctl poweroff

# Reboot immediately
sudo systemctl reboot

# Suspend (sleep)
sudo systemctl suspend

# Hibernate
sudo systemctl hibernate
```

### 9.2 shutdown Command

```bash
# Shutdown now
sudo shutdown -h now

# Shutdown at a specific time
sudo shutdown -h 22:00

# Shutdown in N minutes
sudo shutdown -h +10

# Reboot now
sudo shutdown -r now

# Reboot in 5 minutes with a message
sudo shutdown -r +5 "System update in progress"

# Cancel a scheduled shutdown
sudo shutdown -c
```

### 9.3 Other Commands

```bash
# Reboot
sudo reboot

# Halt (stop CPU, don't power off)
sudo halt

# Power off
sudo poweroff
```

!!! warning
    Always use `shutdown` with a time delay on shared/production systems to warn logged-in users before rebooting.

---

## 📋 Quick Reference Cheat Sheet

```bash
# --- Boot & Kernel ---
uname -r                          # Kernel version
uname -a                          # All system info
cat /proc/cmdline                 # Kernel boot parameters
dmesg | head -50                  # Early kernel messages
ls /boot/                         # List boot files

# --- GRUB ---
sudo update-grub                  # Regenerate grub.cfg (Debian/Ubuntu)
sudo grub2-mkconfig -o /boot/grub2/grub.cfg  # (RHEL/Fedora)
cat /etc/default/grub             # View GRUB settings

# --- systemd / Services ---
systemctl status <service>        # Check service status
sudo systemctl start <service>    # Start service
sudo systemctl stop <service>     # Stop service
sudo systemctl restart <service>  # Restart service
sudo systemctl enable <service>   # Enable at boot
sudo systemctl disable <service>  # Disable at boot
systemctl list-units --failed     # Show failed units
systemctl get-default             # Show default target
sudo systemctl set-default graphical.target  # Set GUI boot

# --- Logging ---
journalctl -f                     # Follow live logs
journalctl -u <service>           # Logs for one service
journalctl -b                     # Logs since last boot
journalctl -p err                 # Error-level logs only
dmesg | grep -i error             # Kernel errors

# --- Shutdown / Reboot ---
sudo shutdown -h now              # Shutdown immediately
sudo shutdown -r now              # Reboot immediately
sudo shutdown -h +5 "Going down"  # Shutdown in 5 min with message
sudo shutdown -c                  # Cancel scheduled shutdown
sudo reboot                       # Quick reboot
```

| Command | Description |
|---------|-------------|
| `uname -r` | Show running kernel version |
| `dmesg` | Kernel ring buffer / boot messages |
| `systemctl status <svc>` | Service status |
| `systemctl enable <svc>` | Enable service at boot |
| `journalctl -f` | Live log tail |
| `journalctl -b` | Logs from current boot |
| `systemctl get-default` | Show current boot target |
| `sudo update-grub` | Regenerate GRUB config |
| `sudo shutdown -r now` | Reboot immediately |
| `sudo shutdown -h +10` | Shutdown in 10 minutes |