# 📦 Common Applications

> Linux offers a rich ecosystem of applications for every task — from web browsing and office work to multimedia, development, and communication. This covers the most widely used applications across categories, their alternatives, and how to install and manage them.

---

## 1. Internet and Web Browsers

### 1.1 Web Browsers

| Browser | Engine | Notes |
|---------|--------|-------|
| **Firefox** | Gecko | Default on most distros, open source, strong privacy |
| **Chromium** | Blink | Open-source base of Chrome, no Google services |
| **Google Chrome** | Blink | Proprietary, full Google integration |
| **Brave** | Blink | Privacy-focused, built-in ad blocker |
| **LibreWolf** | Gecko | Firefox fork, hardened privacy defaults |
| **Falkon** | QtWebEngine | Lightweight, KDE-native |
| **GNOME Web (Epiphany)** | WebKitGTK | Minimal, GNOME-native |
| **Vivaldi** | Blink | Highly customizable, power-user features |

```bash
# Install Firefox (Ubuntu/Debian)
sudo apt install firefox

# Install Chromium
sudo apt install chromium-browser    # Ubuntu/Debian
sudo dnf install chromium            # Fedora

# Install Brave
curl -fsSL https://brave-browser-apt-release.s3.brave.com/brave-browser-archive-keyring.gpg \
  | sudo gpg --dearmor -o /usr/share/keyrings/brave-browser-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/brave-browser-archive-keyring.gpg] \
  https://brave-browser-apt-release.s3.brave.com/ stable main" \
  | sudo tee /etc/apt/sources.list.d/brave-browser.list
sudo apt update && sudo apt install brave-browser
```

### 1.2 Email Clients

| Client | Toolkit | Notes |
|--------|---------|-------|
| **Thunderbird** | GTK | Most popular, full-featured, open source |
| **Evolution** | GTK | GNOME-native, Exchange/CalDAV support |
| **KMail** | Qt | KDE-native, integrates with Kontact suite |
| **Geary** | GTK | Minimal, GNOME-style UI |
| **Claws Mail** | GTK | Lightweight, plugin-based |
| **Mutt / NeoMutt** | TUI | Terminal-based, extremely powerful |

```bash
# Install Thunderbird
sudo apt install thunderbird

# Install Evolution
sudo apt install evolution
```

### 1.3 Download Managers

| Tool | Type | Notes |
|------|------|-------|
| **uGet** | GUI | Multi-protocol, lightweight |
| **JDownloader2** | GUI | Handles hosters, video sites |
| **aria2** | CLI | Multi-source, BitTorrent, Metalink |
| **wget** | CLI | Simple HTTP/FTP downloads |
| **curl** | CLI | Transfers with many protocols |

```bash
# Download a file with wget
wget https://example.com/file.tar.gz

# Resume a broken download
wget -c https://example.com/file.tar.gz

# Download with aria2 (4 connections)
aria2c -x4 https://example.com/file.tar.gz
```

---

## 2. Office and Productivity

### 2.1 Office Suites

| Suite | Components | Notes |
|-------|-----------|-------|
| **LibreOffice** | Writer, Calc, Impress, Draw, Base, Math | Default on most distros, best MS Office compat |
| **OnlyOffice** | Docs, Sheets, Slides | High MS Office fidelity, collaborative |
| **WPS Office** | Writer, Spreadsheets, Presentation | Proprietary, close MS Office look |
| **Calligra Suite** | Words, Sheets, Stage | KDE-native |
| **Google Docs** | Browser-based | Cloud, via any web browser |

```bash
# Install LibreOffice
sudo apt install libreoffice

# Install specific LibreOffice components
sudo apt install libreoffice-writer
sudo apt install libreoffice-calc
sudo apt install libreoffice-impress

# Install OnlyOffice Desktop (Flatpak)
flatpak install flathub org.onlyoffice.desktopeditors
```

### 2.2 LibreOffice Components

| Application | Equivalent | Purpose |
|-------------|-----------|---------|
| **Writer** | Microsoft Word | Word processor |
| **Calc** | Microsoft Excel | Spreadsheet |
| **Impress** | PowerPoint | Presentation |
| **Draw** | Visio | Diagrams and vector drawing |
| **Base** | Access | Database front-end |
| **Math** | Equation Editor | Mathematical formula editor |

!!! tip
    To improve Microsoft Office compatibility in LibreOffice, go to **Tools → Options → Load/Save → General** and set default file formats to `.docx`, `.xlsx`, `.pptx`.

### 2.3 PDF Tools

| Tool | Type | Purpose |
|------|------|---------|
| **Evince** | GUI viewer | Default GNOME PDF viewer |
| **Okular** | GUI viewer | Default KDE PDF viewer, annotate/sign |
| **Zathura** | GUI viewer | Minimal, keyboard-driven |
| **GNOME Document Viewer** | GUI viewer | Alias for Evince |
| **PDFArranger** | GUI editor | Reorder, merge, split PDF pages |
| **LibreOffice Draw** | GUI editor | Edit PDF content |
| **pdftk** | CLI | Merge, split, rotate, stamp PDFs |
| **ghostscript** | CLI | Convert, compress, manipulate PDFs |
| **pdfgrep** | CLI | Search text inside PDFs |

```bash
# Merge PDFs with pdftk
pdftk file1.pdf file2.pdf cat output merged.pdf

# Compress a PDF with ghostscript
gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 \
   -dPDFSETTINGS=/ebook -dNOPAUSE -dQUIET -dBATCH \
   -sOutputFile=output.pdf input.pdf

# Split pages 1-3 from a PDF
pdftk input.pdf cat 1-3 output pages1to3.pdf
```

### 2.4 Note-taking and Personal Knowledge

| Application | Type | Notes |
|-------------|------|-------|
| **Obsidian** | Markdown vault | Links, graph view, plugins |
| **Joplin** | Markdown notes | Open source, sync, E2E encryption |
| **Logseq** | Outliner / graph | Open source Roam alternative |
| **Cherrytree** | Hierarchical notes | Rich text + code blocks |
| **Zim** | Wiki-style notes | Desktop wiki, plain text |
| **Standard Notes** | Encrypted notes | End-to-end encrypted, cross-platform |
| **GNOME Notes (Bijiben)** | Simple notes | Minimal, GNOME-native |

---

## 3. Multimedia

### 3.1 Media Players (Video)

| Player | Notes |
|--------|-------|
| **VLC** | Universal, plays almost any format, no codecs needed |
| **MPV** | Lightweight, CLI/GUI, hardware decoding, scriptable |
| **Celluloid** | GTK front-end for MPV |
| **GNOME Videos (Totem)** | GNOME-native, simple |
| **Kodi** | Full media center, home theater |
| **Haruna** | QML-based MPV front-end, KDE-native |

```bash
# Install VLC
sudo apt install vlc

# Install MPV
sudo apt install mpv

# Play video from terminal
vlc /path/to/video.mp4
mpv /path/to/video.mp4

# MPV with hardware decoding
mpv --hwdec=auto /path/to/video.mp4
```

### 3.2 Music Players

| Player | Style | Notes |
|--------|-------|-------|
| **Rhythmbox** | iTunes-like | GNOME-native, library management |
| **Amarok** | Feature-rich | KDE-native, scripts, covers |
| **Clementine** | iTunes-like | Spotify/Last.fm integration |
| **Strawberry** | Clementine fork | Active development |
| **Lollypop** | Modern | GNOME-style, album art focus |
| **Audacious** | Winamp-like | Lightweight, playlist-focused |
| **cmus** | TUI | Terminal-based, keyboard-driven |
| **ncmpcpp** | TUI | MPD client, powerful |

### 3.3 Audio Editing

| Application | Purpose |
|-------------|---------|
| **Audacity** | Multi-track audio editor, noise reduction, effects |
| **Ardour** | Professional DAW (Digital Audio Workstation) |
| **LMMS** | Beat/music production, FL Studio alternative |
| **Tenacity** | Audacity fork (no telemetry) |
| **SoX** | CLI audio conversion and processing |

```bash
# Install Audacity
sudo apt install audacity

# Convert audio with SoX (mp3 → wav)
sox input.mp3 output.wav

# Convert and resample
sox input.wav -r 44100 output.wav
```

### 3.4 Video Editing

| Application | Level | Notes |
|-------------|-------|-------|
| **Kdenlive** | Intermediate | KDE-native, full NLE, free |
| **Shotcut** | Beginner–Intermediate | Cross-platform, MLT framework |
| **OpenShot** | Beginner | Simple, drag-and-drop |
| **DaVinci Resolve** | Professional | Proprietary, free tier available |
| **Olive** | Intermediate | Modern UI, GPU-accelerated |
| **Blender (Video Editor)** | Advanced | Built-in NLE + 3D |

```bash
# Install Kdenlive
sudo apt install kdenlive

# Install Shotcut (Flatpak)
flatpak install flathub org.shotcut.Shotcut
```

### 3.5 Image Viewers

| Viewer | DE | Notes |
|--------|----|-------|
| **Eye of GNOME (eog)** | GNOME | Simple, fast |
| **gThumb** | GNOME | Viewer + basic editor |
| **Gwenview** | KDE | Viewer + basic editing |
| **Feh** | Any | Minimal, CLI/wallpaper setter |
| **Nomacs** | Any | Cross-platform, RAW support |
| **XnViewMP** | Any | Extensive format support |

---

## 4. Graphics and Image Editing

### 4.1 Raster Graphics (Bitmap)

| Application | Equivalent | Notes |
|-------------|-----------|-------|
| **GIMP** | Photoshop | Full-featured, scripting, plugins |
| **Krita** | Photoshop / Painter | Digital painting, illustration |
| **Pinta** | MS Paint / Paint.NET | Simple, beginner-friendly |
| **Darktable** | Lightroom | RAW photo processing, non-destructive |
| **RawTherapee** | Lightroom | RAW processor, open source |
| **digiKam** | Lightroom | Photo management + editing |

```bash
# Install GIMP
sudo apt install gimp

# Install Krita
sudo apt install krita

# Install Darktable
sudo apt install darktable
```

### 4.2 Vector Graphics

| Application | Equivalent | Notes |
|-------------|-----------|-------|
| **Inkscape** | Illustrator | Full SVG editor, open source |
| **Karbon** | Illustrator | KDE Calligra suite |
| **LibreOffice Draw** | Visio / Illustrator | Diagrams, basic vector |

### 4.3 3D Graphics

| Application | Purpose |
|-------------|---------|
| **Blender** | 3D modeling, animation, rendering, VFX |
| **FreeCAD** | Parametric CAD / engineering design |
| **OpenSCAD** | Script-based 3D CAD |
| **LibreCAD** | 2D CAD, AutoCAD alternative |

```bash
# Install Blender
sudo apt install blender

# Install FreeCAD
sudo apt install freecad
```

### 4.4 Screenshot and Screen Recording

| Tool | Type | Notes |
|------|------|-------|
| **GNOME Screenshot** | Screenshot | Built-in, `PrintScreen` key |
| **Flameshot** | Screenshot | Annotation, region select, upload |
| **Spectacle** | Screenshot | KDE default, full-featured |
| **OBS Studio** | Screen record/stream | Industry standard, streaming |
| **SimpleScreenRecorder** | Screen record | Easy setup, good quality |
| **Kazam** | Screen record | Simple, GIF export |
| **peek** | GIF recorder | Records region as animated GIF |

```bash
# Install Flameshot
sudo apt install flameshot

# Take screenshot with Flameshot
flameshot gui           # Interactive region capture
flameshot full          # Full screen capture
flameshot screen -p ~/  # Save to home directory

# Install OBS Studio
sudo apt install obs-studio
```

---

## 5. Development Tools

### 5.1 Text Editors

| Editor | Type | Notes |
|--------|------|-------|
| **VS Code** | GUI | Most popular, extensions, Git integration |
| **VSCodium** | GUI | VS Code without Microsoft telemetry |
| **Gedit** | GUI | GNOME default, simple |
| **Kate** | GUI | KDE default, multi-document, powerful |
| **Sublime Text** | GUI | Fast, proprietary, free to evaluate |
| **Geany** | GUI | Lightweight IDE-like editor |
| **Vim / Neovim** | TUI | Modal editor, highly extensible |
| **Emacs** | TUI/GUI | Extensible, Lisp-based, full environment |
| **Nano** | TUI | Beginner-friendly terminal editor |

```bash
# Install VS Code (Ubuntu/Debian via Microsoft repo)
sudo apt install code

# Install VSCodium
sudo apt install codium

# Install Neovim
sudo apt install neovim
```

### 5.2 Integrated Development Environments (IDEs)

| IDE | Language Focus | Notes |
|-----|---------------|-------|
| **JetBrains IntelliJ IDEA** | Java/Kotlin | Industry standard for JVM |
| **PyCharm** | Python | Best Python IDE |
| **CLion** | C/C++ | JetBrains C/C++ IDE |
| **Eclipse** | Java, C++, others | Open source, plugin-based |
| **NetBeans** | Java, PHP | Apache project |
| **Android Studio** | Android | Google's official Android IDE |
| **Arduino IDE** | Arduino/C++ | Embedded development |

### 5.3 Version Control

```bash
# Git essentials
git init                          # Initialize repo
git clone <url>                   # Clone remote repo
git status                        # Working tree status
git add .                         # Stage all changes
git commit -m "message"           # Commit staged changes
git push origin main              # Push to remote
git pull                          # Fetch + merge remote changes
git log --oneline --graph         # Visual commit history

# GUI Git clients
gitg                              # GNOME Git viewer
gitk                              # Tcl/Tk Git visualizer
git-cola                          # Cross-platform GUI
```

### 5.4 Terminals and Shell Tools

| Tool | Purpose |
|------|---------|
| **tmux** | Terminal multiplexer — split panes, sessions |
| **screen** | Older terminal multiplexer |
| **zsh + Oh My Zsh** | Enhanced shell with themes and plugins |
| **fish** | User-friendly shell, autosuggestions |
| **fzf** | Fuzzy file/history finder |
| **ripgrep (rg)** | Faster grep alternative |
| **bat** | `cat` with syntax highlighting |
| **eza / exa** | Modern `ls` replacement |
| **htop / btop** | Interactive process viewer |

```bash
# Install tmux
sudo apt install tmux

# Install zsh + Oh My Zsh
sudo apt install zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# Install bat
sudo apt install bat
batcat file.txt      # Ubuntu (binary named batcat)

# Install ripgrep
sudo apt install ripgrep
rg "search term" /path/
```

---

## 6. Communication

### 6.1 Instant Messaging and Chat

| Application | Protocol | Notes |
|-------------|----------|-------|
| **Signal** | Signal | Encrypted, cross-platform |
| **Telegram** | Telegram | Large groups, bots, media |
| **Discord** | Proprietary | Gaming/community, voice/video |
| **Element (Matrix)** | Matrix | Open, federated, end-to-end encrypted |
| **Slack** | Proprietary | Workplace messaging |
| **Fractal** | Matrix | GNOME-native Matrix client |
| **Pidgin** | Multi-protocol | XMPP, IRC, legacy protocols |
| **HexChat** | IRC | Full-featured IRC client |

```bash
# Install Telegram
sudo apt install telegram-desktop

# Install Signal (Ubuntu)
wget -O- https://updates.signal.org/desktop/apt/keys.asc \
  | gpg --dearmor | sudo tee /usr/share/keyrings/signal-desktop-keyring.gpg > /dev/null
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/signal-desktop-keyring.gpg] \
  https://updates.signal.org/desktop/apt xenial main" \
  | sudo tee /etc/apt/sources.list.d/signal-xenial.list
sudo apt update && sudo apt install signal-desktop

# Install Element (Matrix client)
flatpak install flathub im.riot.Riot
```

### 6.2 Video Conferencing

| Application | Notes |
|-------------|-------|
| **Zoom** | Proprietary, widely used |
| **Microsoft Teams** | Proprietary, enterprise |
| **Jitsi Meet** | Open source, browser-based |
| **Google Meet** | Browser-based |
| **Webex** | Cisco, enterprise |

```bash
# Install Zoom (Ubuntu/Debian — download .deb from zoom.us)
sudo dpkg -i zoom_amd64.deb
sudo apt install -f    # Fix any dependency issues
```

### 6.3 RSS Readers

| Application | Notes |
|-------------|-------|
| **Newsflash** | GNOME-native, modern UI |
| **Liferea** | GTK, long-standing, reliable |
| **Akregator** | KDE-native, Kontact integration |
| **Thunderbird** | Built-in RSS reader |

---

## 7. System Utilities

### 7.1 File Managers

| File Manager | DE | Features |
|-------------|-----|---------|
| **Nautilus (Files)** | GNOME | Clean UI, extensions, search |
| **Dolphin** | KDE | Dual-pane, tabs, powerful |
| **Thunar** | XFCE | Fast, lightweight, custom actions |
| **Nemo** | Cinnamon | Nautilus fork with more features |
| **PCManFM** | LXDE/LXQt | Very lightweight |
| **Double Commander** | Any | Dual-pane, Total Commander-like |
| **Ranger** | TUI | Vim-keyed terminal file manager |
| **Midnight Commander (mc)** | TUI | Classic two-panel terminal manager |

```bash
# Install Ranger
sudo apt install ranger

# Install Midnight Commander
sudo apt install mc

# Launch MC
mc
```

### 7.2 Archive Managers

| Tool | Type | Notes |
|------|------|-------|
| **File Roller** | GUI | GNOME archive manager |
| **Ark** | GUI | KDE archive manager |
| **PeaZip** | GUI | Cross-platform, many formats |
| **tar** | CLI | Standard Linux archiver |
| **zip / unzip** | CLI | ZIP format |
| **7zip (7z)** | CLI/GUI | High compression, many formats |

```bash
# Create tar archive
tar -czvf archive.tar.gz /path/to/dir

# Extract tar archive
tar -xzvf archive.tar.gz

# Create zip
zip -r archive.zip /path/to/dir

# Extract zip
unzip archive.zip

# Use 7zip
7z a archive.7z /path/to/dir     # Create
7z x archive.7z                  # Extract
```

### 7.3 Disk and Storage Tools

| Tool | Type | Purpose |
|------|------|---------|
| **GParted** | GUI | Partition editor |
| **GNOME Disks** | GUI | Format, check, benchmark disks |
| **Baobab** | GUI | Disk usage analyzer (GNOME) |
| **KDiskFree** | GUI | Disk usage viewer (KDE) |
| **Filelight** | GUI | Sunburst disk usage map (KDE) |
| **ncdu** | TUI | Fast disk usage analyzer |
| **df / du** | CLI | Disk free / disk usage |

```bash
# Check disk usage by directory
ncdu /

# Disk space overview
df -h

# Directory sizes
du -sh /home/*
du -sh /* 2>/dev/null | sort -rh | head -10
```

### 7.4 System Monitoring

| Tool | Type | Purpose |
|------|------|---------|
| **GNOME System Monitor** | GUI | CPU, RAM, processes, network |
| **KSysGuard** | GUI | KDE system monitor |
| **Mission Center** | GUI | Modern GNOME task manager |
| **htop** | TUI | Interactive process viewer |
| **btop** | TUI | Modern resource monitor |
| **glances** | TUI/Web | Cross-platform system overview |
| **iotop** | TUI | Disk I/O per process |
| **nethogs** | TUI | Network usage per process |

```bash
# Install htop
sudo apt install htop

# Install btop
sudo apt install btop

# Install glances
sudo apt install glances
glances              # TUI mode
glances -w           # Web server mode (browser at :61208)

# Monitor disk I/O
sudo iotop

# Monitor network per process
sudo nethogs
```

---

## 8. Package Management (GUI)

### 8.1 Software Centers

| Application | Distro | Notes |
|-------------|--------|-------|
| **GNOME Software** | Ubuntu, Fedora | Flatpak + native packages |
| **KDE Discover** | Kubuntu, KDE Neon | Flatpak + native + Snap |
| **Ubuntu Software** | Ubuntu | Snap-focused variant of GNOME Software |
| **Synaptic** | Debian/Ubuntu | Full APT front-end, advanced |
| **Pamac** | Manjaro, Arch | AUR + official repos |

```bash
# Install Synaptic (powerful APT GUI)
sudo apt install synaptic

# Launch Synaptic
sudo synaptic
```

### 8.2 Flatpak and Snap

| Format | Manager | Repository | Notes |
|--------|---------|-----------|-------|
| **Flatpak** | `flatpak` | Flathub | Sandboxed, distro-agnostic |
| **Snap** | `snap` | Snap Store | Canonical-backed, auto-update |
| **AppImage** | chmod+run | Self-contained | Portable, no install needed |

```bash
# --- Flatpak ---
flatpak install flathub org.gimp.GIMP
flatpak run org.gimp.GIMP
flatpak update                        # Update all Flatpaks
flatpak list                          # List installed

# --- Snap ---
sudo snap install vlc
sudo snap list                        # List installed snaps
sudo snap refresh                     # Update all snaps

# --- AppImage ---
chmod +x application.AppImage
./application.AppImage
```

!!! tip
    **Flatpak** is generally preferred over Snap for its open governance model and Flathub's wide application selection. Add the Flathub remote with: `flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo`

---

## 📋 Quick Reference Cheat Sheet

```bash
# --- Web Browsers ---
sudo apt install firefox chromium-browser

# --- Office ---
sudo apt install libreoffice
flatpak install flathub org.onlyoffice.desktopeditors

# --- Media ---
sudo apt install vlc mpv audacity kdenlive

# --- Graphics ---
sudo apt install gimp inkscape krita darktable

# --- Screenshots ---
flameshot gui                  # Interactive screenshot
flameshot full -p ~/Pictures   # Full screenshot to Pictures

# --- Screen Recording ---
obs                            # OBS Studio

# --- Development ---
sudo apt install git neovim tmux ripgrep bat

# --- Communication ---
sudo apt install telegram-desktop
flatpak install flathub im.riot.Riot   # Element (Matrix)

# --- File Management ---
sudo apt install ranger mc ncdu

# --- Archives ---
tar -czvf out.tar.gz dir/      # Create tarball
tar -xzvf archive.tar.gz       # Extract tarball
7z a out.7z dir/               # Create 7zip archive

# --- System Monitoring ---
sudo apt install htop btop glances
glances                        # System overview TUI
sudo iotop                     # Disk I/O monitor

# --- Flatpak Setup ---
sudo apt install flatpak
flatpak remote-add --if-not-exists flathub \
  https://flathub.org/repo/flathub.flatpakrepo
flatpak install flathub <app-id>
flatpak update
```

| Application | Category | Install |
|-------------|----------|---------|
| `firefox` | Browser | `sudo apt install firefox` |
| `vlc` | Video player | `sudo apt install vlc` |
| `libreoffice` | Office suite | `sudo apt install libreoffice` |
| `gimp` | Image editor | `sudo apt install gimp` |
| `inkscape` | Vector graphics | `sudo apt install inkscape` |
| `kdenlive` | Video editor | `sudo apt install kdenlive` |
| `obs-studio` | Screen recording | `sudo apt install obs-studio` |
| `flameshot` | Screenshot | `sudo apt install flameshot` |
| `htop` | Process monitor | `sudo apt install htop` |
| `ranger` | Terminal file manager | `sudo apt install ranger` |