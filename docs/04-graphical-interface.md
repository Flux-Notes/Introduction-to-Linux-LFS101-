# 🖥️ Graphical Interface

> Linux offers a fully modular graphical stack — unlike Windows or macOS, every component (display server, window manager, desktop environment) is swappable. This covers the X Window System, Wayland, desktop environments, display managers, and essential GUI tools.

---

## 1. The Linux Graphical Stack

### 1.1 How the GUI Works

The Linux graphical interface is built from several independent, layered components — each replaceable.

```
User Applications (Firefox, Thunar, Terminal)
              ↕
    Desktop Environment (GNOME, KDE, XFCE)
              ↕
      Window Manager (Mutter, KWin, Openbox)
              ↕
    Display Server (X11 / Wayland)
              ↕
     Display Driver (Mesa, NVIDIA, AMD)
              ↕
            GPU Hardware
```

### 1.2 Component Roles

| Component | Role | Examples |
|-----------|------|---------|
| **Display Server** | Handles drawing to screen, input events | X.Org, Wayland |
| **Window Manager** | Controls window placement, borders, focus | Mutter, KWin, Openbox |
| **Desktop Environment** | Full GUI suite (panels, file manager, settings) | GNOME, KDE, XFCE |
| **Display Manager** | Login screen / session selector | GDM, SDDM, LightDM |
| **Compositor** | Renders visual effects, transparency | Mutter, KWin, Picom |

---

## 2. X Window System (X11)

### 2.1 What is X11?

The **X Window System** (X11, or simply "X") is the traditional display server protocol for Linux and Unix. It has been the foundation of Linux GUIs since the 1980s.

- Developed at MIT in 1984
- Current implementation: **X.Org Server**
- Uses a **client-server model** — applications are X clients, the X server handles rendering
- Network-transparent: applications can display on a remote screen over a network

### 2.2 X11 Architecture

```
X Client (Application)
        ↕  (X11 Protocol over socket or TCP)
X Server (X.Org)
        ↕
Display Hardware
```

!!! info
    Despite the name, the **X server** runs locally on your machine and the **X client** is the application. This is the reverse of how "server/client" is usually thought of.

### 2.3 Key X11 Files and Directories

| Path | Purpose |
|------|---------|
| `/etc/X11/` | X configuration directory |
| `/etc/X11/xorg.conf` | Main X server config (often auto-detected now) |
| `/etc/X11/xorg.conf.d/` | Modular config snippets |
| `~/.xinitrc` | X startup script for user session |
| `~/.Xauthority` | X authentication cookie file |
| `/var/log/Xorg.0.log` | X server log file |

```bash
# Start X manually (if not using a display manager)
startx

# Display current X session display number
echo $DISPLAY

# Check X server log for errors
cat /var/log/Xorg.0.log | grep EE
```

### 2.4 Limitations of X11

- Originally designed in the 1980s — carries significant legacy complexity
- Security issues: any X client can read input/screen of other X clients
- Performance limitations with modern GPU compositing
- Complex codebase difficult to maintain and extend

---

## 3. Wayland

### 3.1 What is Wayland?

**Wayland** is the modern replacement for X11. It is a display server **protocol** (not a server itself) designed to be simpler, faster, and more secure.

- Each application renders directly to a buffer; the compositor combines them
- No network transparency by default (unlike X11)
- Better security: apps cannot spy on other apps' input/display
- Smoother rendering with less tearing

### 3.2 X11 vs Wayland

| Feature | X11 | Wayland |
|---------|-----|---------|
| Age | 1984 | 2008 |
| Architecture | Client-server (X.Org) | Protocol + compositor |
| Security | Low (apps share input/screen) | High (app isolation) |
| Network transparency | ✅ Native | ❌ (via XWayland for X apps) |
| Fractional scaling | Limited | ✅ |
| HDR support | Limited | ✅ (growing) |
| App compatibility | Universal | Requires Wayland-native or XWayland |
| Default on GNOME | Since GNOME 40 | ✅ |
| Default on KDE | KDE 6 | ✅ |

### 3.3 XWayland

**XWayland** is a compatibility layer that runs inside a Wayland session and allows legacy X11 applications to work without modification.

```bash
# Check if running X11 or Wayland
echo $XDG_SESSION_TYPE     # outputs: x11 or wayland

# Check if XWayland is running
ps aux | grep Xwayland
```

!!! tip
    Most applications work fine under XWayland. Native Wayland support has become standard in modern GTK and Qt applications.

---

## 4. Desktop Environments

### 4.1 What is a Desktop Environment?

A **Desktop Environment (DE)** is a complete graphical user interface suite including:

- Window manager
- File manager
- Panel / taskbar
- Application launcher
- System settings
- Default applications (text editor, terminal, image viewer)

### 4.2 Major Desktop Environments

| DE | Toolkit | Style | RAM Usage | Default On |
|----|---------|-------|-----------|-----------|
| **GNOME** | GTK4 | Modern, minimal | ~800 MB | Ubuntu, Fedora |
| **KDE Plasma** | Qt6 | Feature-rich, Windows-like | ~600 MB | Kubuntu, openSUSE |
| **XFCE** | GTK3 | Lightweight, traditional | ~300 MB | Xubuntu, Manjaro XFCE |
| **LXQt** | Qt5/6 | Very lightweight | ~200 MB | Lubuntu |
| **LXDE** | GTK2 | Minimal, older | ~150 MB | Legacy Lubuntu |
| **Cinnamon** | GTK3 | Traditional, Windows-like | ~500 MB | Linux Mint |
| **MATE** | GTK3 | GNOME 2 continuation | ~350 MB | Ubuntu MATE |
| **Budgie** | GTK4 | Modern, elegant | ~500 MB | Solus, Ubuntu Budgie |
| **Pantheon** | GTK | macOS-inspired | ~500 MB | elementaryOS |

### 4.3 GNOME

**GNOME** (GNU Network Object Model Environment) is the most widely used Linux DE, default on Ubuntu, Fedora, Debian, and many others.

**Key features:**

- Activities Overview (Super key) — shows open windows and app launcher
- Extensions system — customizable via GNOME Extensions
- GNOME Shell — the top bar + Activities panel
- Nautilus — file manager
- Settings app for system configuration

```bash
# Check GNOME version
gnome-shell --version

# Restart GNOME Shell (X11 only)
Alt + F2 → type "r" → Enter

# List installed GNOME extensions
gnome-extensions list

# Enable/disable an extension
gnome-extensions enable <uuid>
gnome-extensions disable <uuid>
```

### 4.4 KDE Plasma

**KDE Plasma** is a powerful, highly customizable DE with a more traditional desktop layout.

**Key features:**

- System Settings — comprehensive control panel
- Dolphin — feature-rich file manager
- KRunner — quick launcher (Alt+F2)
- Widgets (Plasmoids) on desktop and panel
- Virtual Desktops and Activities system

```bash
# Check KDE Plasma version
plasmashell --version

# Restart Plasma Shell
kquitapp5 plasmashell && kstart5 plasmashell
```

### 4.5 XFCE

**XFCE** is a lightweight, fast desktop environment that is ideal for older hardware or users who prefer a traditional workflow.

**Key features:**

- Thunar — fast file manager
- Xfwm4 — window manager with basic compositing
- Xfce Panel — highly configurable panel
- Low resource usage — runs well on 512 MB RAM

---

## 5. Window Managers

### 5.1 What is a Window Manager?

A **Window Manager (WM)** controls how windows are displayed, moved, resized, and focused. Unlike a full DE, a standalone WM provides a minimal environment.

### 5.2 Types of Window Managers

| Type | Description | Examples |
|------|-------------|---------|
| **Stacking** | Windows overlap like paper on a desk | Openbox, Fluxbox |
| **Tiling** | Windows auto-tile to fill screen | i3, Sway, Hyprland, dwm |
| **Compositing** | Stacking + visual effects | Mutter (GNOME), KWin (KDE) |
| **Dynamic** | Switch between tiling and floating | Awesome, Qtile, xmonad |

### 5.3 Popular Standalone Window Managers

| WM | Type | Display Server | Notes |
|----|------|---------------|-------|
| **i3** | Tiling | X11 | Keyboard-driven, config file |
| **Sway** | Tiling | Wayland | i3-compatible for Wayland |
| **Hyprland** | Dynamic tiling | Wayland | Animations, modern effects |
| **Openbox** | Stacking | X11 | Lightweight, right-click menu |
| **dwm** | Tiling | X11 | Minimalist, patched in C |
| **Awesome** | Dynamic | X11 | Lua-configured |
| **bspwm** | Tiling | X11 | Controlled via bspc commands |

!!! tip
    Tiling window managers are popular among power users and developers for efficient keyboard-driven workflows with no mouse required.

---

## 6. Display Managers

### 6.1 What is a Display Manager?

A **Display Manager (DM)** — also called a **Login Manager** — is the graphical login screen. It:

- Displays a login prompt
- Authenticates the user
- Launches the selected desktop session

### 6.2 Common Display Managers

| Display Manager | DE | Notes |
|----------------|-----|-------|
| **GDM** (GNOME Display Manager) | GNOME | Default on Ubuntu, Fedora |
| **SDDM** (Simple Desktop Display Manager) | KDE Plasma | QML-based, modern |
| **LightDM** | XFCE, MATE, others | Lightweight, theme-friendly |
| **LXDM** | LXDE | Minimal |
| **XDM** | Any | Legacy, minimal |
| **ly** | Any | TUI (terminal-based) login manager |

```bash
# Check which display manager is active
systemctl status display-manager

# Switch display manager (Debian/Ubuntu)
sudo dpkg-reconfigure lightdm    # or gdm3, sddm

# Enable/disable display manager
sudo systemctl enable gdm
sudo systemctl disable gdm
```

---

## 7. Remote Desktop and GUI Over Network

### 7.1 X11 Forwarding (SSH)

X11 allows running GUI apps on a remote server and displaying them locally over SSH.

```bash
# Connect with X11 forwarding enabled
ssh -X user@remote-host

# Trusted forwarding (less security restrictions)
ssh -Y user@remote-host

# Once connected, launch a GUI app
firefox &
gedit &
```

!!! warning
    `-X` (untrusted) is safer than `-Y` (trusted). Use `-Y` only on trusted networks as it grants the remote host more access to your local display.

### 7.2 VNC (Virtual Network Computing)

VNC shares a full desktop session over the network.

```bash
# Install TigerVNC server
sudo apt install tigervnc-standalone-server  # Debian/Ubuntu

# Start a VNC server on display :1
vncserver :1

# Set VNC password
vncpasswd

# Stop VNC server
vncserver -kill :1

# Connect from client (Linux)
vncviewer remote-host:5901
```

### 7.3 RDP (Remote Desktop Protocol)

**xrdp** allows connecting to a Linux machine using Windows Remote Desktop or any RDP client.

```bash
# Install xrdp
sudo apt install xrdp

# Enable and start xrdp
sudo systemctl enable --now xrdp

# Connect from Windows: mstsc.exe → enter Linux IP
# Connect from Linux:
rdesktop remote-host
# or
xfreerdp /u:username /v:remote-host
```

### 7.4 Comparison

| Method | Speed | Setup | Use Case |
|--------|-------|-------|---------|
| X11 Forwarding | Slow (latency) | Easy (SSH) | Single remote GUI app |
| VNC | Medium | Moderate | Full desktop, cross-platform |
| RDP (xrdp) | Fast | Moderate | Full desktop, Windows clients |
| SPICE | Fast | Complex | Virtual machines (KVM/QEMU) |

---

## 8. Useful GUI Tools and Applications

### 8.1 System Tools

| Tool | Category | DE |
|------|----------|----|
| **GNOME System Monitor** | Process/resource monitor | GNOME |
| **KSysGuard / System Monitor** | Process/resource monitor | KDE |
| **Baobab** | Disk usage analyzer | GNOME |
| **GParted** | Partition editor | Any |
| **GNOME Disks** | Disk management | GNOME |
| **Timeshift** | System snapshot/restore | Any |
| **Synaptic** | Graphical package manager | Any (GTK) |
| **Discover** | Software center | KDE |
| **GNOME Software** | Software center | GNOME |

### 8.2 Productivity Applications

| Application | Category | Notes |
|-------------|----------|-------|
| **LibreOffice** | Office suite | Word, Calc, Impress |
| **Thunderbird** | Email client | Cross-platform |
| **GIMP** | Image editor | Photoshop alternative |
| **Inkscape** | Vector graphics | Illustrator alternative |
| **VLC** | Media player | Universal codec support |
| **Evince / Okular** | PDF viewer | GNOME / KDE |
| **Nautilus / Dolphin** | File manager | GNOME / KDE |

### 8.3 Terminal Emulators

| Terminal | DE | Features |
|----------|----|---------|
| **GNOME Terminal** | GNOME | Tabs, profiles |
| **Konsole** | KDE | Tabs, split view |
| **Alacritty** | Any | GPU-accelerated, fast |
| **Kitty** | Any | GPU-accelerated, extensible |
| **Tilix** | GNOME | Tiling terminal |
| **Terminator** | Any | Split panes |
| **xterm** | Any | Minimal, classic |

---

## 9. Configuring the Display

### 9.1 Screen Resolution and Monitors

```bash
# List connected displays and resolutions (X11)
xrandr

# Set resolution on a display
xrandr --output HDMI-1 --mode 1920x1080

# Set refresh rate
xrandr --output HDMI-1 --mode 1920x1080 --rate 144

# Mirror displays
xrandr --output HDMI-1 --same-as eDP-1

# Extend displays (HDMI-1 to the right of eDP-1)
xrandr --output HDMI-1 --right-of eDP-1

# Wayland: use display settings GUI or wlr-randr
wlr-randr                          # list outputs
wlr-randr --output HDMI-A-1 --mode 1920x1080
```

### 9.2 Display Variables

```bash
# Current display server type
echo $XDG_SESSION_TYPE    # x11 or wayland

# Current display number (X11)
echo $DISPLAY             # e.g., :0

# Current desktop environment
echo $XDG_CURRENT_DESKTOP # e.g., GNOME, KDE

# Current session desktop
echo $DESKTOP_SESSION
```

### 9.3 DPI and Scaling

```bash
# Check current DPI (X11)
xdpyinfo | grep resolution

# Set DPI via Xresources
echo "Xft.dpi: 144" >> ~/.Xresources
xrdb -merge ~/.Xresources

# GNOME scaling (Wayland/X11)
gsettings set org.gnome.desktop.interface scaling-factor 2

# GNOME fractional scaling (Wayland, experimental)
gsettings set org.gnome.mutter experimental-features "['scale-monitor-framebuffer']"
```

---

## 📋 Quick Reference Cheat Sheet

```bash
# --- Session Info ---
echo $XDG_SESSION_TYPE        # x11 or wayland
echo $XDG_CURRENT_DESKTOP     # GNOME, KDE, XFCE, etc.
echo $DISPLAY                 # X11 display number

# --- Display / Monitor (X11) ---
xrandr                        # List displays and resolutions
xrandr --output HDMI-1 --mode 1920x1080 --rate 60
xrandr --output HDMI-1 --right-of eDP-1

# --- Display Manager ---
systemctl status display-manager
sudo systemctl restart gdm    # Restart GDM (GNOME)
sudo systemctl restart sddm   # Restart SDDM (KDE)

# --- GNOME ---
gnome-shell --version
gnome-extensions list
Alt+F2 → "r"                  # Restart GNOME Shell (X11 only)

# --- KDE ---
plasmashell --version
kquitapp5 plasmashell && kstart5 plasmashell  # Restart Plasma

# --- X11 Forwarding ---
ssh -X user@host              # Enable X11 forwarding
ssh -Y user@host              # Trusted X11 forwarding

# --- VNC ---
vncserver :1                  # Start VNC on display :1
vncserver -kill :1            # Stop VNC
vncviewer host:5901           # Connect to VNC

# --- Logs ---
cat /var/log/Xorg.0.log | grep EE   # X11 errors
journalctl -b | grep -i display     # Display-related boot logs
```

| Command | Description |
|---------|-------------|
| `echo $XDG_SESSION_TYPE` | Check X11 or Wayland |
| `xrandr` | List/configure displays (X11) |
| `ssh -X user@host` | X11 forwarding over SSH |
| `systemctl status display-manager` | Check login manager |
| `gnome-shell --version` | GNOME version |
| `plasmashell --version` | KDE Plasma version |
| `vncserver :1` | Start VNC server |
| `cat /var/log/Xorg.0.log` | X server log |
| `echo $DISPLAY` | Current X11 display |
| `gsettings set ...` | Configure GNOME settings via CLI |