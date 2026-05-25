# 🖨️ Printing

> **Printing in Linux** is managed through the **CUPS** (Common Unix Printing System) framework — a modular, network-capable print system that handles everything from local USB printers to enterprise print servers, supporting hundreds of printer models through PPD driver files and a clean web-based management interface.

---

## 1. Overview of Linux Printing Architecture

```
Application (LibreOffice, Firefox, etc.)
        │
        ▼
  Print Dialog / API
        │
        ▼
    CUPS Scheduler  ──────────────────────────────────────
        │                                                 │
        ▼                                                 ▼
   Print Queue                                    /etc/cups/
  (spooler)                                  (config files)
        │
        ▼
  Filter Chain (Ghostscript, Foomatic, CUPS filters)
        │
        ▼
  Backend (usb://, socket://, lpd://, ipp://, smb://)
        │
        ▼
    Physical Printer
```

| Component | Role |
|---|---|
| **CUPS** | Core print system scheduler and spooler |
| **PPD files** | PostScript Printer Description — driver/capability info |
| **Ghostscript** | PostScript/PDF interpreter and rasteriser |
| **Foomatic** | Database + filter glue for non-PostScript printers |
| **CUPS-filters** | Format conversion filters (PDF → raster, etc.) |
| **Avahi/mDNS** | Printer auto-discovery on local network |
| **IPP** | Internet Printing Protocol — modern standard |

---

## 2. CUPS — Common Unix Printing System

### 2.1 CUPS Service Management

```bash
# systemd-based systems
systemctl status cups                   # check if running
systemctl start cups                    # start CUPS
systemctl stop cups                     # stop CUPS
systemctl restart cups                  # restart (apply config changes)
systemctl enable cups                   # start on boot
systemctl disable cups                  # don't start on boot

# SysV-style (older systems)
service cups status
service cups start
service cups restart
```

### 2.2 CUPS Web Interface

CUPS includes a built-in web admin interface accessible at:

```
http://localhost:631
```

| URL | Purpose |
|---|---|
| `http://localhost:631` | CUPS home page |
| `http://localhost:631/admin` | Administration — add/remove printers |
| `http://localhost:631/printers` | List all printers |
| `http://localhost:631/jobs` | View all print jobs |
| `http://localhost:631/help` | CUPS documentation |

!!! tip "Accessing the Web Interface"
    You may need to add your user to the `lpadmin` group to manage printers via the web UI:
    ```bash
    sudo usermod -aG lpadmin $USER
    ```
    Then log out and back in, or run `newgrp lpadmin`.

### 2.3 CUPS Configuration Files

| File / Directory | Purpose |
|---|---|
| `/etc/cups/cupsd.conf` | Main CUPS daemon configuration |
| `/etc/cups/printers.conf` | Installed printer definitions |
| `/etc/cups/classes.conf` | Printer class (group) definitions |
| `/etc/cups/ppd/` | PPD files for installed printers |
| `/etc/cups/cups-files.conf` | File/path settings (CUPS 1.6+) |
| `/var/spool/cups/` | Print job spool directory |
| `/var/log/cups/access_log` | CUPS access log |
| `/var/log/cups/error_log` | CUPS error log |
| `/var/log/cups/page_log` | Page accounting log |

```bash
# View main config
cat /etc/cups/cupsd.conf

# View installed printers
cat /etc/cups/printers.conf

# Tail the error log (debug printing issues)
tail -f /var/log/cups/error_log

# Check page log (who printed what)
cat /var/log/cups/page_log
```

### 2.4 Key `cupsd.conf` Directives

```apacheconf
# Listen on localhost only (default)
Listen localhost:631

# Allow all network interfaces
Listen 0.0.0.0:631

# Log levels: debug2, debug, info, warn, error, none
LogLevel warn
PageLogFormat

# Default authentication
DefaultAuthType Basic

# Allow browsing (network printer sharing)
Browsing On
BrowseLocalProtocols dnssd

# Access control example
<Location /admin>
  AuthType Default
  Require user @SYSTEM
  Order allow,deny
</Location>

# Allow access from local network
<Location />
  Order allow,deny
  Allow localhost
  Allow 192.168.1.0/24
</Location>
```

---

## 3. Managing Printers from the Command Line

### 3.1 Listing Printers

```bash
lpstat -p                           # list all printers and status
lpstat -p -d                        # list printers + show default
lpstat -a                           # list printers accepting jobs
lpstat -v                           # list printers with device URIs
lpstat -s                           # summary: default + all printers
lpstat -t                           # complete status (everything)
lpstat -l -p printername            # detailed info on specific printer

# Using lpoptions
lpoptions -l                        # list options for default printer
lpoptions -p HP_LaserJet -l         # options for specific printer

# CUPS command
lpinfo -v                           # list available device backends/URIs
lpinfo -m                           # list available PPD drivers/models
```

### 3.2 Setting the Default Printer

```bash
lpoptions -d HP_LaserJet            # set default printer
lpadmin -d HP_LaserJet              # alternative (persistent)
echo "PRINTER=HP_LaserJet" >> ~/.bashrc   # environment variable
export PRINTER=HP_LaserJet              # set for current session
export LPDEST=HP_LaserJet               # alternative env variable
```

### 3.3 Adding a Printer with `lpadmin`

```bash
# Basic syntax
lpadmin -p printername -E -v device_uri -m ppd_file

# Add USB printer
lpadmin -p MyPrinter \
    -E \
    -v usb://HP/LaserJet%20P1005 \
    -m drv:///hpijs.drv/hp-laserjet_p1005.ppd

# Add network printer (IPP)
lpadmin -p NetworkPrinter \
    -E \
    -v ipp://192.168.1.100/ipp/print \
    -m everywhere                       # IPP Everywhere (driverless)

# Add network printer (LPD/LPR)
lpadmin -p OldPrinter \
    -E \
    -v lpd://192.168.1.50/lp \
    -m drv:///foomatic.drv/printer-model.ppd

# Add printer with description and location
lpadmin -p OfficePrinter \
    -E \
    -v socket://192.168.1.200:9100 \
    -m everywhere \
    -D "Office Printer 3rd Floor" \
    -L "Building A, Floor 3"
```

**Common device URI formats:**

| URI Format | Backend | Use Case |
|---|---|---|
| `usb://Make/Model` | USB | Direct USB connection |
| `ipp://host/ipp/print` | IPP | Modern network printer |
| `ipps://host/ipp/print` | IPPS | IPP over TLS |
| `socket://host:9100` | AppSocket/JetDirect | HP JetDirect |
| `lpd://host/queue` | LPD/LPR | Legacy network |
| `smb://user:pass@host/share` | Samba | Windows shared printer |
| `http://host:631/printers/name` | IPP via HTTP | CUPS shared |
| `dnssd://...` | Bonjour | Auto-discovered |

### 3.4 Modifying and Removing Printers

```bash
# Enable / Disable printer
cupsenable printername              # enable (accept and process jobs)
cupsdisable printername             # disable (stop processing jobs)
cupsdisable -r "Offline for maintenance" printername  # with reason

# Accept / Reject jobs
cupsaccept printername              # accept new jobs
cupsreject printername              # reject new jobs
cupsreject -r "Out of paper" printername

# Set printer options
lpadmin -p printername -o media=A4
lpadmin -p printername -o sides=two-sided-long-edge
lpadmin -p printername -o ColorModel=Gray

# Remove a printer
lpadmin -x printername

# Rename (delete and re-add)
lpadmin -x OldName
lpadmin -p NewName -E -v uri -m ppd
```

---

## 4. Printing Files — `lp` and `lpr`

### 4.1 `lp` — Standard Print Command

```bash
lp file.pdf                         # print to default printer
lp -d HP_LaserJet file.pdf          # print to specific printer
lp -n 3 file.pdf                    # print 3 copies
lp -P 2,4,6-10 file.pdf             # print pages 2, 4, and 6–10
lp -q 10 file.pdf                   # set priority (1=low, 100=high, default=50)
lp -t "My Print Job" file.pdf       # set job title
lp -H hold file.pdf                 # hold job — don't print yet
lp -H immediate file.pdf            # print immediately
lp -H restart file.pdf              # restart a completed job

# Print multiple files
lp file1.pdf file2.pdf file3.txt

# Print from stdin
echo "Hello Printer" | lp
cat report.txt | lp -d HP_LaserJet

# Print with options
lp -o media=A4 file.pdf
lp -o sides=two-sided-long-edge file.pdf      # duplex (long edge)
lp -o sides=two-sided-short-edge file.pdf     # duplex (short edge)
lp -o sides=one-sided file.pdf                # single sided
lp -o landscape file.pdf                       # landscape orientation
lp -o portrait file.pdf                        # portrait orientation
lp -o fit-to-page file.pdf                     # scale to fit
lp -o number-up=2 file.pdf                     # 2 pages per sheet
lp -o number-up=4 file.pdf                     # 4 pages per sheet
lp -o outputorder=reverse file.pdf             # reverse page order
lp -o ColorModel=Gray file.pdf                 # grayscale
lp -o Resolution=600dpi file.pdf               # set resolution
```

### 4.2 `lpr` — BSD-Style Print Command

```bash
lpr file.pdf                        # print to default printer
lpr -P HP_LaserJet file.pdf         # specific printer
lpr -#3 file.pdf                    # 3 copies
lpr -o landscape file.pdf           # landscape
lpr -o media=A4 file.pdf            # paper size
lpr -o sides=two-sided-long-edge file.pdf  # duplex

# Print stdin
echo "test" | lpr
cat file.txt | lpr -P OfficePrinter
```

!!! info "lp vs lpr"
    Both commands print files. `lp` is the System V (CUPS native) command with more options. `lpr` is the BSD legacy command. On modern Linux with CUPS, both work and accept similar `-o` options. Prefer `lp` for scripts.

---

## 5. Managing Print Jobs

### 5.1 Viewing Jobs

```bash
lpq                                 # show jobs in default printer queue
lpq -P HP_LaserJet                  # queue for specific printer
lpq -a                              # all queues, all jobs
lpq -l                              # verbose job listing

lpstat -o                           # show all active jobs
lpstat -o HP_LaserJet               # jobs for specific printer
lpstat -u alice                     # jobs for user alice
lpstat -W completed                 # show completed jobs
lpstat -W not-completed             # show pending/active jobs
```

### 5.2 Cancelling Jobs

```bash
lprm                                # cancel current (most recent) job
lprm -P HP_LaserJet 42             # cancel job ID 42
lprm -P HP_LaserJet -              # cancel ALL jobs on printer

cancel 42                           # cancel job ID 42 (CUPS command)
cancel -a                           # cancel ALL jobs
cancel -a HP_LaserJet               # cancel all jobs on printer
cancel -u alice                     # cancel all jobs for user alice

# Get job IDs first
lpstat -o
```

### 5.3 Holding and Releasing Jobs

```bash
# Hold a job (prevent it from printing)
lp -i 42 -H hold                    # hold existing job 42
lp -H hold file.pdf                 # submit but hold immediately

# Release a held job
lp -i 42 -H resume                  # release job 42

# Using cancel and resubmit approach
lpstat -o                           # find job ID
lp -i JOB_ID -H release             # release
```

### 5.4 Moving Jobs Between Printers

```bash
lpmove 42 HP_LaserJet               # move job 42 to HP_LaserJet
lpmove HP_LaserJet-42 Canon_MG5700  # move using full job name
```

---

## 6. Print Filters and File Format Conversion

### 6.1 How CUPS Filters Work

CUPS converts documents through a chain of filters before sending to the printer:

```
PDF / PostScript / Text / Image
        │
        ▼
   CUPS Filter Chain
   (pstops, pdftops, rastertopdf,
    gstoraster, imagetoraster...)
        │
        ▼
   Printer-specific raster or
   PostScript / PCL / ESC/P
        │
        ▼
    Backend (USB / Network)
```

### 6.2 Converting for Print with Command-Line Tools

```bash
# Text to PostScript
enscript -p output.ps file.txt          # text → PostScript
enscript -2r file.txt                   # 2-up landscape
enscript --font=Courier10 file.txt      # specify font

# PostScript to PDF
ps2pdf input.ps output.pdf
ps2pdf -dPDFSETTINGS=/printer input.ps output.pdf

# PDF to PostScript
pdf2ps input.pdf output.ps
pdftops input.pdf output.ps             # via poppler

# Image to PostScript
convert image.jpg -format ps image.ps   # ImageMagick
img2ps image.png                        # if available

# Print PostScript directly
lp -d HP_LaserJet output.ps

# Convert and print in one pipeline
enscript -p - file.txt | lp -d Printer
```

### 6.3 `a2ps` — Any to PostScript

```bash
a2ps file.txt                           # convert and print
a2ps -o output.ps file.txt              # convert to PS file only
a2ps -2 file.txt                        # 2 virtual pages per sheet
a2ps -r file.txt                        # landscape (rotated)
a2ps --font-size=10 file.txt            # font size
a2ps --header="My Header" file.txt      # custom header
a2ps --no-header file.txt               # suppress header
a2ps -P HP_LaserJet file.txt            # send to specific printer
a2ps *.py                               # multiple source files
```

---

## 7. Printer Classes

CUPS supports **printer classes** — logical groups of printers. Jobs sent to a class are forwarded to the first available member printer (load balancing / failover).

```bash
# Create a class
lpadmin -p Printer1 -c OfficeClass      # add Printer1 to OfficeClass
lpadmin -p Printer2 -c OfficeClass      # add Printer2 to OfficeClass

# List classes
lpstat -c                               # show all classes

# Print to a class
lp -d OfficeClass file.pdf             # CUPS picks an available printer

# Remove printer from class
lpadmin -p Printer1 -r OfficeClass

# View class config
cat /etc/cups/classes.conf
```

---

## 8. Network Printing

### 8.1 Sharing a Printer via CUPS

To share a local printer with other machines on the network:

```bash
# 1. Enable browsing and sharing in cupsd.conf
# /etc/cups/cupsd.conf:
# Listen 0.0.0.0:631
# Browsing On
# BrowseLocalProtocols dnssd

# 2. Mark printer as shared
lpadmin -p MyPrinter -o printer-is-shared=true

# Or via web interface: Administration → Modify Printer → Share This Printer

# 3. Allow access from network
# In cupsd.conf:
# <Location />
#   Order allow,deny
#   Allow localhost
#   Allow 192.168.1.0/24
# </Location>

systemctl restart cups
```

### 8.2 Connecting to a Shared CUPS Printer

```bash
# Add remote CUPS printer
lpadmin -p RemotePrinter \
    -E \
    -v ipp://192.168.1.10:631/printers/HP_LaserJet \
    -m everywhere

# Auto-discover via Avahi (if on same subnet)
avahi-browse -t -r _ipp._tcp        # discover IPP printers
avahi-browse -t -r _printer._tcp    # discover LPD printers

# Or simply add via CUPS web UI:
# http://localhost:631/admin → Add Printer → Internet Printing Protocol (IPP)
```

### 8.3 Printing to a Windows Shared Printer (Samba)

```bash
# Install Samba client
sudo apt install samba-client           # Debian/Ubuntu
sudo dnf install samba-client           # Fedora/RHEL

# List Windows shares
smbclient -L //192.168.1.5 -U username

# Add Windows shared printer
lpadmin -p WinPrinter \
    -E \
    -v smb://username:password@192.168.1.5/PrinterShare \
    -m drv:///hpijs.drv/hp-laserjet.ppd
```

---

## 9. Useful Printing Utilities

### 9.1 `lpinfo` — List Devices and Drivers

```bash
lpinfo -v                           # list all available device backends
lpinfo -m                           # list all PPD/driver files
lpinfo --include-schemes usb -v     # USB devices only
lpinfo --include-schemes socket -v  # network (JetDirect) devices
lpinfo -m | grep -i "hp laserjet"   # search for specific driver
lpinfo -m | grep -i canon           # search Canon drivers
```

### 9.2 `cupsctl` — CUPS Server Settings

```bash
cupsctl                             # show current server settings
cupsctl --debug-logging             # enable debug logging
cupsctl --no-debug-logging          # disable debug logging
cupsctl --remote-admin              # allow remote admin
cupsctl --no-remote-admin           # disable remote admin (more secure)
cupsctl --share-printers            # enable printer sharing
cupsctl --no-share-printers         # disable printer sharing
cupsctl --remote-any                # allow jobs from any host
cupsctl --no-remote-any             # jobs from localhost only
```

### 9.3 `cupstestppd` — Validate PPD Files

```bash
cupstestppd /etc/cups/ppd/MyPrinter.ppd    # validate PPD
cupstestppd -q /etc/cups/ppd/*.ppd         # validate all PPDs silently
```

### 9.4 `lpoptions` — Per-User Printer Options

```bash
lpoptions                           # show default printer + options
lpoptions -l                        # list all options with current values
lpoptions -p HP_LaserJet -l         # options for specific printer
lpoptions -d HP_LaserJet            # set as user default printer

# Set persistent user defaults
lpoptions -p HP_LaserJet -o sides=two-sided-long-edge
lpoptions -p HP_LaserJet -o media=A4
lpoptions -p HP_LaserJet -o ColorModel=Gray

# Remove a user default option
lpoptions -p HP_LaserJet -r sides

# User defaults are stored in
cat ~/.cups/lpoptions
```

---

## 10. PostScript and PDF Utilities

### 10.1 `ghostscript` (`gs`) — PostScript/PDF Engine

```bash
# Convert PDF to images
gs -dNOPAUSE -dBATCH -sDEVICE=jpeg -r150 \
   -sOutputFile=page_%03d.jpg input.pdf

# Compress a PDF (reduce file size)
gs -dNOPAUSE -dBATCH -sDEVICE=pdfwrite \
   -dCompatibilityLevel=1.4 \
   -dPDFSETTINGS=/screen \         # /screen, /ebook, /printer, /prepress
   -sOutputFile=compressed.pdf input.pdf

# Merge PDFs
gs -dNOPAUSE -dBATCH -sDEVICE=pdfwrite \
   -sOutputFile=merged.pdf \
   file1.pdf file2.pdf file3.pdf

# Extract page range from PDF
gs -dNOPAUSE -dBATCH -sDEVICE=pdfwrite \
   -dFirstPage=2 -dLastPage=5 \
   -sOutputFile=pages2-5.pdf input.pdf

# Get info about a PDF
gs -dNOPAUSE -dBATCH -sDEVICE=nullpage input.pdf
```

### 10.2 PDF Manipulation with `pdftk` (if installed)

```bash
pdftk input.pdf burst output page_%04d.pdf   # split into individual pages
pdftk file1.pdf file2.pdf cat output merged.pdf  # merge
pdftk input.pdf cat 1-5 output first5.pdf    # extract pages 1-5
pdftk input.pdf cat 1-endeast output rotated.pdf  # rotate
pdftk input.pdf dump_data                     # metadata
```

---

## 11. Troubleshooting Printing

### 11.1 Diagnosing Common Issues

```bash
# Is CUPS running?
systemctl status cups

# Can we reach the printer?
ping 192.168.1.100
nc -zv 192.168.1.100 9100           # test JetDirect port
nc -zv 192.168.1.100 631            # test IPP port

# Check queue status
lpstat -p -d
lpstat -t

# Check for held or stopped jobs
lpstat -o
lpq -a

# Check error log (most useful!)
tail -50 /var/log/cups/error_log
grep -i "error\|fail\|unable" /var/log/cups/error_log

# Increase log verbosity
cupsctl --debug-logging
tail -f /var/log/cups/error_log
# (then try printing)
cupsctl --no-debug-logging          # turn off after diagnosis
```

### 11.2 Common Problems and Fixes

| Problem | Likely Cause | Fix |
|---|---|---|
| Printer shows "stopped" | Error state or manual disable | `cupsenable printername` |
| Jobs stuck in queue | Backend error, printer offline | Check `error_log`, power cycle printer |
| "Rejecting jobs" | Queue set to reject | `cupsaccept printername` |
| No printers listed | CUPS not running | `systemctl start cups` |
| Access denied (web UI) | Not in `lpadmin` group | `usermod -aG lpadmin $USER` |
| USB printer not found | Module/driver issue | `lpinfo -v`, check `dmesg | grep usb` |
| Network printer unreachable | Firewall or IP issue | `ping`, `nc -zv host port` |
| Wrong paper size | Default not set | `lpadmin -p P -o media=A4` |
| Garbled output | Wrong driver/PPD | Reinstall printer with correct PPD |
| Very slow printing | Large job, filters | Check CUPS filters, use PDF |

### 11.3 Resetting a Stuck Printer

```bash
# Clear all jobs
cancel -a printername

# Disable then re-enable
cupsdisable printername
cupsenable printername

# Restart CUPS
systemctl restart cups

# Purge and re-add printer
lpadmin -x printername
# re-add with lpadmin -p ...

# Full reset of spool
systemctl stop cups
rm -f /var/spool/cups/[0-9]*        # remove job files
systemctl start cups
```

---

## 12. Environment Variables for Printing

| Variable | Purpose |
|---|---|
| `PRINTER` | Default printer name (overrides CUPS default) |
| `LPDEST` | Default printer (BSD legacy, alternative to PRINTER) |
| `CUPS_SERVER` | CUPS server hostname (default: localhost) |
| `IPP_PORT` | IPP port (default: 631) |

```bash
export PRINTER=HP_LaserJet          # set default printer for session
export CUPS_SERVER=printserver.local  # connect to remote CUPS server
lp file.pdf                         # uses HP_LaserJet on printserver.local
```

---

## 13. Quick Reference Cheat Sheet

### Printer Management

| Command | What it does |
|---|---|
| `lpstat -p -d` | List printers + show default |
| `lpstat -t` | Full CUPS status |
| `lpadmin -d printername` | Set default printer |
| `cupsenable printername` | Enable a printer |
| `cupsdisable printername` | Disable a printer |
| `cupsaccept printername` | Accept jobs |
| `cupsreject printername` | Reject jobs |
| `lpadmin -x printername` | Remove a printer |

### Printing Files

| Command | What it does |
|---|---|
| `lp file.pdf` | Print to default |
| `lp -d Printer file.pdf` | Print to specific printer |
| `lp -n 3 file.pdf` | 3 copies |
| `lp -P 2-5 file.pdf` | Pages 2–5 |
| `lp -o sides=two-sided-long-edge file.pdf` | Duplex |
| `lp -o media=A4 file.pdf` | A4 paper |
| `lp -o number-up=2 file.pdf` | 2-up |
| `echo "text" \| lp` | Print from stdin |

### Job Management

| Command | What it does |
|---|---|
| `lpq -a` | View all queues |
| `lpstat -o` | View active jobs |
| `cancel JOB_ID` | Cancel a job |
| `cancel -a` | Cancel all jobs |
| `lp -i JOB_ID -H hold` | Hold a job |
| `lp -i JOB_ID -H resume` | Release a held job |
| `lpmove JOB_ID Printer` | Move job to another printer |

### Diagnostics

| Command | What it does |
|---|---|
| `lpinfo -v` | List device URIs |
| `lpinfo -m` | List available drivers |
| `cupsctl --debug-logging` | Enable verbose logging |
| `tail -f /var/log/cups/error_log` | Watch error log live |
| `cupstestppd file.ppd` | Validate PPD file |