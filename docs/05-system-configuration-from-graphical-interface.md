# ⚙️ System Configuration from the Graphical Interface

> Linux provides rich graphical tools for configuring nearly every aspect of the system — from network settings and user accounts to display, sound, and hardware — without ever touching the terminal. This covers the key GUI configuration tools across GNOME, KDE, and distro-neutral utilities.

---

## 1. System Settings Overview

### 1.1 Where to Find Settings

Every major desktop environment ships a central **System Settings** application that consolidates configuration panels in one place.

| DE | Settings App | How to Open |
|----|-------------|-------------|
| **GNOME** | GNOME Settings | Super → "Settings" / `gnome-control-center` |
| **KDE Plasma** | System Settings | App Menu → System Settings / `systemsettings5` |
| **XFCE** | XFCE Settings Manager | App Menu → Settings / `xfce4-settings-manager` |
| **Cinnamon** | System Settings | App Menu → System Settings / `cinnamon-settings` |
| **MATE** | MATE Control Center | App Menu → System / `mate-control-center` |

```bash
# Open settings from terminal
gnome-control-center          # GNOME
systemsettings5               # KDE Plasma
xfce4-settings-manager        # XFCE
cinnamon-settings             # Cinnamon
mate-control-center           # MATE
```

### 1.2 Settings Categories at a Glance

| Category | What You Configure |
|----------|--------------------|
| **Network** | Wi-Fi, Ethernet, VPN, proxy |
| **Display** | Resolution, refresh rate, night light, scaling |
| **Sound** | Output/input devices, volume, notifications |
| **Users** | Add/remove users, passwords, account types |
| **Date & Time** | Timezone, NTP sync, 24h/12h format |
| **Power** | Sleep, suspend, battery, screen blank |
| **Bluetooth** | Pair/unpair devices |
| **Printers** | Add/configure/test printers |
| **Keyboard** | Layout, shortcuts, input methods |
| **Mouse & Touchpad** | Speed, scrolling, tap-to-click |
| **Accessibility** | Screen reader, high contrast, zoom |
| **Startup Apps** | Programs that launch at login |

---

## 2. Network Configuration

### 2.1 NetworkManager

**NetworkManager** is the standard network management daemon used by most desktop Linux distributions. It handles wired, wireless, VPN, mobile broadband, and more — with both GUI and CLI interfaces.

```bash
# Check NetworkManager status
systemctl status NetworkManager

# GUI front-ends for NetworkManager
nm-connection-editor          # Standalone connection editor
nmtui                         # Terminal UI (TUI) for NetworkManager
nmcli                         # Full CLI interface
```

### 2.2 GNOME Network Settings

In GNOME Settings → **Network**:

- **Wired** — view connection details, configure IP (DHCP or static), DNS, routes
- **Wi-Fi** — scan and connect to networks, manage saved networks, configure hotspot
- **VPN** — add VPN connections (OpenVPN, WireGuard, IPsec)
- **Network Proxy** — configure HTTP/HTTPS/SOCKS proxy or auto-detect

### 2.3 Configuring a Static IP (GUI)

In GNOME → Network → select connection → ⚙️ icon → **IPv4**:

| Field | Example |
|-------|---------|
| Method | Manual |
| Address | `192.168.1.100` |
| Netmask | `255.255.255.0` or `/24` |
| Gateway | `192.168.1.1` |
| DNS | `8.8.8.8, 8.8.4.4` |

!!! tip
    After saving a static IP in GNOME, toggle the connection off and on (or disconnect and reconnect) to apply the new settings immediately.

### 2.4 VPN Configuration (GUI)

GNOME and KDE support VPN connections natively. Common plugins:

| VPN Type | Package to Install |
|----------|-------------------|
| OpenVPN | `network-manager-openvpn-gnome` |
| WireGuard | `network-manager-wireguard` (or import `.conf`) |
| L2TP/IPsec | `network-manager-l2tp-gnome` |
| PPTP | `network-manager-pptp-gnome` |

```bash
# Install OpenVPN plugin (Ubuntu/Debian)
sudo apt install network-manager-openvpn-gnome

# Import an OpenVPN config file via CLI
nmcli connection import type openvpn file ~/client.ovpn
```

### 2.5 Wi-Fi Hotspot

GNOME Settings → Network → Wi-Fi → **Turn On Wi-Fi Hotspot**:

- Creates an access point your other devices can join
- Shares the active wired or mobile data connection
- SSID and password are auto-generated (editable)

---

## 3. Display Configuration

### 3.1 GNOME Display Settings

GNOME Settings → **Displays**:

- **Resolution** — select from detected modes
- **Refresh Rate** — choose Hz (60, 120, 144, etc.)
- **Orientation** — normal, 90°, 180°, 270°
- **Scale** — 100%, 125%, 150%, 200% (fractional needs experimental flag)
- **Night Light** — reduce blue light on a schedule or sunset-to-sunrise
- **Multi-monitor** — mirror or extend, set primary display

### 3.2 KDE Display Settings

KDE System Settings → **Display and Monitor**:

- **Resolution & Refresh Rate** — per-display settings
- **Scale** — per-display fractional scaling (KDE supports this natively)
- **Night Color** — blue light filter
- **Display Layout** — drag-and-drop multi-monitor arrangement
- **Color Profile** — ICC profile management

### 3.3 Night Light / Blue Light Filter

| DE | Feature Name | Location |
|----|-------------|---------|
| GNOME | Night Light | Settings → Displays → Night Light |
| KDE | Night Color | System Settings → Display → Night Color |
| XFCE | Redshift | Install `redshift-gtk`, system tray |

```bash
# Install and run Redshift (any DE)
sudo apt install redshift-gtk
redshift -O 4000              # Set manual color temperature (K)
redshift -x                   # Reset to normal
```

---

## 4. User and Account Management

### 4.1 GNOME Users Settings

GNOME Settings → **Users**:

- View all user accounts
- Add new standard or administrator accounts
- Change password, profile picture, username
- Enable / disable auto-login
- Delete accounts

!!! warning
    You must click **Unlock** (enter your password) in the top-right before making changes to user accounts in GNOME Settings.

### 4.2 Account Types

| Type | Description | sudo Access |
|------|-------------|------------|
| **Standard** | Normal user, limited system changes | ❌ |
| **Administrator** | Can make system changes, install software | ✅ |
| **Root** | Full unrestricted access (uid=0) | N/A |

### 4.3 Adding a User (GUI)

**GNOME:**
Settings → Users → Unlock → **Add User** → choose type → set name and password → **Add**

**KDE:**
System Settings → Users → **+** button → fill in details → set account type

### 4.4 Parental Controls (GNOME)

GNOME includes a **Parental Controls** app (`gnome-parental-controls`) for managed/child accounts:

- Restrict app usage by age rating
- Block web content categories
- Set screen time limits
- Prevent account settings changes

```bash
# Open Parental Controls
gnome-parental-controls
```

---

## 5. Date, Time, and Timezone

### 5.1 GNOME Date & Time Settings

GNOME Settings → **Date & Time**:

- **Automatic Date & Time** — sync via NTP (recommended)
- **Automatic Time Zone** — detect timezone from location
- **Manual Time Zone** — search and select from map or list
- **Time Format** — 24-hour or AM/PM

### 5.2 NTP (Network Time Protocol)

**NTP** keeps the system clock accurate by syncing with internet time servers. `systemd-timesyncd` handles this on most modern systems.

```bash
# Check time sync status
timedatectl

# Check NTP sync detail
timedatectl show-timesync --all

# Enable NTP sync
sudo timedatectl set-ntp true

# Set timezone manually
sudo timedatectl set-timezone Asia/Kolkata

# List all available timezones
timedatectl list-timezones
timedatectl list-timezones | grep Asia
```

### 5.3 timedatectl Output Example

```
               Local time: Sat 2024-05-23 14:30:00 IST
           Universal time: Sat 2024-05-23 09:00:00 UTC
                 RTC time: Sat 2024-05-23 09:00:00
                Time zone: Asia/Kolkata (IST, +0530)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

---

## 6. Sound Configuration

### 6.1 PulseAudio and PipeWire

| System | Role | Notes |
|--------|------|-------|
| **PulseAudio** | Audio server (legacy) | Standard before ~2022 |
| **PipeWire** | Modern audio/video server | Replaces PulseAudio + JACK |
| **ALSA** | Kernel audio layer | Low-level, always present |
| **JACK** | Pro audio (low latency) | Now handled by PipeWire |

!!! info
    Most modern distributions (Ubuntu 22.04+, Fedora 34+) now use **PipeWire** as the default audio server. It is backward-compatible with PulseAudio applications.

### 6.2 GNOME Sound Settings

GNOME Settings → **Sound**:

- **Output** — select audio device, adjust volume
- **Input** — select microphone, adjust input level
- **System Sounds** — toggle event sounds on/off
- **Volume Levels** — per-application volume (via output section)

### 6.3 PaveControl (PulseAudio Volume Control)

**pavucontrol** is a powerful graphical mixer that works with both PulseAudio and PipeWire, providing more control than the basic settings panel.

```bash
# Install pavucontrol
sudo apt install pavucontrol

# Launch
pavucontrol
```

| Tab | Purpose |
|-----|---------|
| **Playback** | Per-app volume for playing audio |
| **Recording** | Per-app input source selection |
| **Output Devices** | Select and configure audio outputs |
| **Input Devices** | Select and configure microphones |
| **Configuration** | Select audio profiles (stereo, surround, HDMI) |

---

## 7. Power Management

### 7.1 GNOME Power Settings

GNOME Settings → **Power**:

- **Screen Blank** — time before screen turns off
- **Automatic Suspend** — suspend after idle time (on battery / plugged in)
- **Power Button Behavior** — suspend, power off, or nothing
- **Show Battery Percentage** — display in top bar
- **Performance Mode** — power-saver, balanced, performance (if supported)

### 7.2 KDE Power Management

KDE System Settings → **Power Management**:

- Separate profiles for **AC power**, **battery**, and **low battery**
- Per-profile: screen brightness, dim screen, suspend, screen lock
- **Energy Saving** — configure CPU frequency governor

### 7.3 TLP (Advanced Power Management — Laptops)

**TLP** provides automatic power saving for laptops beyond what the DE offers.

```bash
# Install TLP
sudo apt install tlp tlp-rdw

# Enable and start TLP
sudo systemctl enable --now tlp

# Check TLP status
sudo tlp-stat -s

# View full TLP status and settings
sudo tlp-stat
```

### 7.4 Suspend, Hibernate, and Hybrid Sleep

| Mode | RAM | Disk | Resume Speed | Power Use |
|------|-----|------|-------------|-----------|
| **Suspend** (sleep) | ✅ kept | ❌ | Fast (~2s) | Low |
| **Hibernate** | ❌ saved to disk | ✅ | Slow (~30s) | Zero |
| **Hybrid Sleep** | ✅ kept | ✅ backup | Fast (or slow) | Low |

```bash
# Suspend now
systemctl suspend

# Hibernate now (requires swap ≥ RAM)
systemctl hibernate

# Hybrid sleep
systemctl hybrid-sleep
```

---

## 8. Bluetooth

### 8.1 GNOME Bluetooth Settings

GNOME Settings → **Bluetooth**:

- Enable/disable Bluetooth radio
- Make device discoverable
- Pair new devices (headphones, keyboards, mice, phones)
- Trust / untrust paired devices
- Send/receive files via Bluetooth (OBEX)

### 8.2 Blueman (GTK Bluetooth Manager)

**Blueman** is a full-featured Bluetooth manager that works across all DEs.

```bash
# Install Blueman
sudo apt install blueman

# Launch Blueman
blueman-manager
```

### 8.3 bluetoothctl (CLI)

```bash
# Open interactive Bluetooth shell
bluetoothctl

# Inside bluetoothctl:
power on                      # Turn on Bluetooth
scan on                       # Start scanning for devices
devices                       # List discovered devices
pair XX:XX:XX:XX:XX:XX        # Pair with device MAC
connect XX:XX:XX:XX:XX:XX     # Connect to paired device
trust XX:XX:XX:XX:XX:XX       # Auto-connect in future
remove XX:XX:XX:XX:XX:XX      # Remove/unpair device
```

---

## 9. Printers and Scanners

### 9.1 CUPS (Common Unix Printing System)

Linux printing is handled by **CUPS**, which runs as a background service. GUI tools are front-ends to CUPS.

```bash
# Check CUPS status
systemctl status cups

# Open CUPS web interface
# Navigate to: http://localhost:631
```

### 9.2 Adding a Printer (GNOME)

GNOME Settings → **Printers** → **+** (Add Printer):

1. GNOME scans for network and USB printers automatically
2. Select discovered printer or enter IP manually
3. GNOME/CUPS fetches and installs the correct PPD driver
4. Print a test page to verify

### 9.3 Common Printer Driver Packages

```bash
# Generic PostScript / PDF driver
sudo apt install cups

# HP printers (HPLIP)
sudo apt install hplip hplip-gui

# Epson printers
sudo apt install printer-driver-escpr

# Canon printers
sudo apt install cnijfilter2   # or from Canon's website

# Samsung / Xerox
sudo apt install printer-driver-splix
```

### 9.4 Simple Scan (Scanner GUI)

```bash
# Install Simple Scan
sudo apt install simple-scan

# Launch
simple-scan
```

---

## 10. Keyboard and Input

### 10.1 Keyboard Layout

**GNOME:** Settings → **Keyboard** → Input Sources → **+** to add layout

**KDE:** System Settings → **Input Devices** → Keyboard → Layouts

```bash
# List available keyboard layouts
localectl list-keymaps

# Set keyboard layout system-wide
sudo localectl set-keymap us              # Console
sudo localectl set-x11-keymap us          # GUI (X11)

# Set layout with variant
sudo localectl set-x11-keymap gb intl     # UK international
```

### 10.2 Keyboard Shortcuts

**GNOME:** Settings → **Keyboard** → **View and Customize Shortcuts**

Categories include:
- Accessibility shortcuts
- Launchers (terminal, browser, files)
- Navigation (workspace switching)
- Screenshots
- Window management (minimize, maximize, move)
- Custom shortcuts (add your own commands)

```bash
# Add a custom shortcut via CLI (GNOME)
# Set name, command, and key binding
gsettings set org.gnome.settings-daemon.plugins.media-keys custom-keybindings \
  "['/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/']"

gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:\
/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ \
  name "Open Terminal"

gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:\
/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ \
  command "gnome-terminal"

gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:\
/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ \
  binding "<Super>t"
```

### 10.3 Input Methods (CJK and Other Scripts)

For typing in non-Latin scripts (Chinese, Japanese, Korean, Arabic, etc.), an **Input Method** framework is required.

| Framework | Description | Use Case |
|-----------|-------------|---------|
| **IBus** | Default on GNOME/Ubuntu | CJK, Indic, multilingual |
| **Fcitx5** | Fast, widely used | CJK, extensive language support |
| **SCIM** | Older alternative | Legacy |

```bash
# Install Fcitx5 with Chinese input (Ubuntu)
sudo apt install fcitx5 fcitx5-chinese-addons

# Set Fcitx5 as input method
im-config -n fcitx5
```

---

## 📋 Quick Reference Cheat Sheet

```bash
# --- Settings Apps ---
gnome-control-center          # GNOME System Settings
systemsettings5               # KDE System Settings
xfce4-settings-manager        # XFCE Settings Manager

# --- Network ---
nm-connection-editor          # NetworkManager GUI
nmtui                         # NetworkManager TUI
systemctl status NetworkManager

# --- Date & Time ---
timedatectl                   # Show time/date/NTP status
sudo timedatectl set-ntp true
sudo timedatectl set-timezone Asia/Kolkata
timedatectl list-timezones | grep India

# --- Users ---
# GUI: GNOME Settings → Users → Unlock → Add User

# --- Sound ---
pavucontrol                   # Advanced audio mixer GUI

# --- Power ---
systemctl suspend             # Suspend now
systemctl hibernate           # Hibernate now
sudo tlp-stat -s              # TLP power status (laptops)

# --- Bluetooth ---
bluetoothctl                  # Interactive Bluetooth shell
blueman-manager               # Bluetooth GUI (any DE)

# --- Printing ---
systemctl status cups
# Web UI: http://localhost:631
sudo apt install hplip-gui    # HP printer GUI

# --- Keyboard ---
sudo localectl set-x11-keymap us
localectl list-keymaps

# --- Display ---
# GNOME: Settings → Displays
# Fractional scaling (GNOME Wayland):
gsettings set org.gnome.mutter experimental-features \
  "['scale-monitor-framebuffer']"
```

| Command / Tool | Description |
|---------------|-------------|
| `gnome-control-center` | Open GNOME Settings |
| `systemsettings5` | Open KDE System Settings |
| `nm-connection-editor` | GUI network connection editor |
| `nmtui` | TUI network manager |
| `timedatectl` | View/set date, time, timezone, NTP |
| `pavucontrol` | Advanced audio mixer |
| `bluetoothctl` | CLI Bluetooth manager |
| `http://localhost:631` | CUPS printer web interface |
| `sudo localectl set-x11-keymap` | Set keyboard layout |
| `systemctl suspend` | Suspend system immediately |