# 🌐 Network Operations

> **Network operations** are a core Linux skill — whether you're diagnosing connectivity issues, transferring files, scanning hosts, or securing remote sessions, Linux ships with a powerful suite of networking tools that give you full visibility and control over every layer of the stack.

---

## 1. Overview of Linux Networking

Linux provides both low-level and high-level tools to configure interfaces, troubleshoot connections, transfer data, and monitor traffic.

| Category | Key Tools |
|---|---|
| Interface Configuration | `ip`, `ifconfig`, `nmcli`, `nmtui` |
| Connectivity Testing | `ping`, `ping6`, `traceroute`, `mtr` |
| DNS Resolution | `dig`, `nslookup`, `host`, `resolvectl` |
| Port & Service Scanning | `netstat`, `ss`, `nmap` |
| File Transfer | `wget`, `curl`, `scp`, `rsync`, `sftp` |
| Remote Access | `ssh`, `sshd`, `ssh-keygen` |
| Packet Analysis | `tcpdump`, `wireshark` |
| Routing | `ip route`, `route`, `traceroute` |
| Firewall | `firewalld`, `ufw`, `iptables`, `nftables` |
| Network Info | `hostname`, `ip addr`, `ip link` |

---

## 2. Network Interface Management

### 2.1 `ip` — The Modern Tool (Replaces `ifconfig`)

```bash
ip addr show                        # show all interfaces and IPs
ip addr show eth0                   # show specific interface
ip link show                        # show link-layer status
ip link show up                     # show only active interfaces
ip -s link show eth0                # show statistics (packets, errors)
ip -4 addr show                     # IPv4 only
ip -6 addr show                     # IPv6 only
```

**Configuring Interfaces (requires root):**

```bash
ip addr add 192.168.1.100/24 dev eth0      # assign IP address
ip addr del 192.168.1.100/24 dev eth0      # remove IP address
ip link set eth0 up                         # bring interface up
ip link set eth0 down                       # bring interface down
ip link set eth0 mtu 9000                   # set MTU (jumbo frames)
ip link set eth0 promisc on                 # enable promiscuous mode
```

**Routing:**

```bash
ip route show                              # show routing table
ip route show default                      # show default gateway
ip route add default via 192.168.1.1       # set default gateway
ip route add 10.0.0.0/8 via 192.168.1.1   # add static route
ip route del 10.0.0.0/8                    # delete route
ip route get 8.8.8.8                       # which route would be used
```

**ARP / Neighbours:**

```bash
ip neigh show                              # show ARP table
ip neigh show dev eth0                     # ARP for specific interface
ip neigh flush dev eth0                    # flush ARP cache
```

### 2.2 `ifconfig` — Legacy Tool (Still Common)

```bash
ifconfig                            # show all active interfaces
ifconfig -a                         # show all interfaces (including down)
ifconfig eth0                       # show specific interface
ifconfig eth0 up                    # bring interface up
ifconfig eth0 down                  # bring interface down
ifconfig eth0 192.168.1.50 netmask 255.255.255.0   # assign static IP
ifconfig eth0 mtu 1500              # set MTU
```

!!! info "ip vs ifconfig"
    `ifconfig` is deprecated on most modern distros. Use `ip` instead. On minimal installs, `ifconfig` may not be present — install `net-tools` to get it back.

### 2.3 NetworkManager Tools

```bash
nmcli device status                 # list all devices
nmcli connection show               # list all connections
nmcli connection show "Wired"       # details of a specific connection
nmcli device connect eth0           # connect a device
nmcli device disconnect eth0        # disconnect a device
nmcli networking off                # disable all networking
nmcli networking on                 # enable all networking

# Create a static IP connection
nmcli connection add type ethernet con-name "static" ifname eth0 \
  ip4 192.168.1.50/24 gw4 192.168.1.1

# Modify DNS
nmcli connection modify "static" ipv4.dns "8.8.8.8 1.1.1.1"

# Apply changes
nmcli connection up "static"
```

```bash
nmtui                               # terminal UI for NetworkManager
```

### 2.4 Wireless Interfaces

```bash
iwconfig                            # show wireless interface info (legacy)
iwlist wlan0 scan                   # scan for WiFi networks
iw dev wlan0 scan                   # modern wireless scan
iw dev wlan0 link                   # show current link info
nmcli device wifi list              # list WiFi networks (NetworkManager)
nmcli device wifi connect "SSID" password "pass"   # connect to WiFi
```

---

## 3. Hostname and DNS Configuration

### 3.1 Hostname

```bash
hostname                            # display current hostname
hostname -f                         # fully qualified domain name (FQDN)
hostname -i                         # IP address of hostname
hostname -I                         # all IPs of all interfaces
hostnamectl                         # detailed hostname info (systemd)
hostnamectl set-hostname newname    # permanently set hostname
```

### 3.2 `/etc/hosts` — Static Name Resolution

```bash
cat /etc/hosts
# Format: IP  hostname  alias
127.0.0.1   localhost
127.0.1.1   mymachine.local mymachine
192.168.1.10  server1.local server1
```

### 3.3 `/etc/resolv.conf` — DNS Configuration

```bash
cat /etc/resolv.conf
# nameserver  8.8.8.8
# nameserver  1.1.1.1
# search      local.domain
# domain      local.domain
```

!!! warning "resolv.conf on Systemd Systems"
    On systemd-resolved systems, `/etc/resolv.conf` is a symlink managed automatically. Edit DNS via `nmcli` or `/etc/systemd/resolved.conf` instead.

### 3.4 `/etc/nsswitch.conf` — Name Resolution Order

```bash
grep hosts /etc/nsswitch.conf
# hosts: files dns myhostname
# Resolution order: /etc/hosts first, then DNS, then hostname
```

---

## 4. DNS Lookup Tools

### 4.1 `dig` — DNS Information Groper

```bash
dig google.com                      # A record (IPv4)
dig google.com AAAA                 # IPv6 record
dig google.com MX                   # mail exchanger records
dig google.com NS                   # name server records
dig google.com TXT                  # TXT records (SPF, DKIM, etc.)
dig google.com ANY                  # all record types
dig @8.8.8.8 google.com             # query specific DNS server
dig +short google.com               # just the IP answer
dig +noall +answer google.com       # clean answer section only
dig -x 8.8.8.8                      # reverse DNS lookup (PTR)
dig +trace google.com               # trace full DNS resolution path
dig google.com +dnssec              # check DNSSEC
```

### 4.2 `nslookup` — Interactive DNS Lookup

```bash
nslookup google.com                 # basic lookup
nslookup google.com 8.8.8.8        # using specific nameserver
nslookup -type=MX google.com        # MX records
nslookup -type=NS google.com        # NS records

# Interactive mode
nslookup
> server 1.1.1.1
> set type=MX
> google.com
> exit
```

### 4.3 `host` — Simple DNS Lookup

```bash
host google.com                     # A record
host -t MX google.com               # MX record
host -t NS google.com               # NS record
host 8.8.8.8                        # reverse lookup
host -a google.com                  # all records
```

### 4.4 `resolvectl` — systemd-resolved Status

```bash
resolvectl status                   # DNS config per interface
resolvectl query google.com         # resolve a hostname
resolvectl statistics               # cache stats
resolvectl flush-caches             # flush DNS cache
resolvectl dns eth0 8.8.8.8         # set DNS for interface
```

---

## 5. Connectivity Testing

### 5.1 `ping` — Test Reachability

```bash
ping google.com                     # continuous ping (Ctrl+C to stop)
ping -c 4 google.com                # send exactly 4 packets
ping -i 0.5 google.com              # interval 0.5 seconds
ping -s 1400 google.com             # custom packet size (bytes)
ping -t 64 192.168.1.1             # set TTL
ping -W 2 google.com                # timeout 2 seconds per reply
ping -q -c 10 google.com            # quiet: summary only
ping6 ::1                           # ping IPv6 loopback
ping 192.168.1.1 -I eth0            # use specific interface
```

**Reading ping output:**

```
64 bytes from 142.250.195.78: icmp_seq=1 ttl=116 time=12.4 ms
```

| Field | Meaning |
|---|---|
| `64 bytes` | Packet size received |
| `icmp_seq` | Sequence number |
| `ttl` | Time to live (hops remaining) |
| `time` | Round-trip time in ms |

### 5.2 `traceroute` / `tracepath` — Trace Network Path

```bash
traceroute google.com               # trace hops to destination
traceroute -n google.com            # numeric only (no DNS)
traceroute -I google.com            # use ICMP (like ping)
traceroute -T google.com            # use TCP (port 80, bypass firewalls)
traceroute -p 443 google.com        # use specific port
traceroute6 google.com              # IPv6 trace
tracepath google.com                # similar, no root needed
```

### 5.3 `mtr` — Continuous Traceroute (My Traceroute)

```bash
mtr google.com                      # interactive live traceroute
mtr -r -c 20 google.com             # report mode, 20 cycles
mtr -n google.com                   # no DNS resolution
mtr --tcp -P 443 google.com         # TCP mode on port 443
mtr --json google.com               # JSON output
```

!!! tip "mtr for Diagnosing Packet Loss"
    `mtr` shows per-hop packet loss and latency in real time, making it far more useful than traceroute alone for identifying where in the path a problem exists.

---

## 6. Port and Socket Monitoring

### 6.1 `ss` — Socket Statistics (Modern)

```bash
ss                                  # all sockets
ss -t                               # TCP sockets
ss -u                               # UDP sockets
ss -l                               # listening sockets
ss -lt                              # listening TCP
ss -lu                              # listening UDP
ss -p                               # show process name/PID
ss -n                               # numeric (no hostname/service lookup)
ss -tlnp                            # all listening TCP with PIDs (most useful)
ss -s                               # socket statistics summary
ss -a                               # all (listening + established)
ss dst 192.168.1.10                 # connections to specific host
ss sport = :22                      # connections on source port 22
ss state established                # only established connections
ss -tlnp | grep :80                 # who is listening on port 80
```

### 6.2 `netstat` — Legacy Socket Tool

```bash
netstat -tuln                       # listening TCP and UDP (numeric)
netstat -tulnp                      # add process info (root needed)
netstat -an                         # all connections, numeric
netstat -rn                         # routing table
netstat -i                          # interface statistics
netstat -s                          # per-protocol statistics
netstat -c                          # continuous refresh
```

!!! info "ss vs netstat"
    `netstat` is from the deprecated `net-tools` package. `ss` is faster, provides more detail, and is the modern replacement.

### 6.3 `nmap` — Network Scanner

```bash
nmap 192.168.1.1                    # scan top 1000 ports of a host
nmap 192.168.1.0/24                 # scan entire subnet
nmap -p 22,80,443 192.168.1.1      # scan specific ports
nmap -p 1-65535 192.168.1.1        # full port scan
nmap -sV 192.168.1.1               # detect service versions
nmap -O 192.168.1.1                # OS detection (root)
nmap -A 192.168.1.1                # aggressive: OS + version + scripts
nmap -sn 192.168.1.0/24            # ping sweep (no port scan)
nmap -Pn 192.168.1.1               # skip ping, scan anyway
nmap -sU 192.168.1.1               # UDP scan (root)
nmap -T4 192.168.1.1               # timing template (0-5, higher = faster)
nmap --open 192.168.1.0/24         # show only open ports
nmap -oN scan.txt 192.168.1.1      # save output to file (normal format)
nmap -oX scan.xml 192.168.1.1      # save as XML
```

!!! warning "Ethical Use of nmap"
    Only scan networks and hosts you own or have explicit permission to test. Unauthorized scanning may be illegal and violates network policies.

---

## 7. File Transfer Tools

### 7.1 `wget` — Non-Interactive Downloader

```bash
wget https://example.com/file.tar.gz          # download file
wget -O myfile.tar.gz https://example.com/f   # save with custom name
wget -P /tmp/ https://example.com/file        # download to directory
wget -c https://example.com/large.iso         # continue interrupted download
wget -q https://example.com/file              # quiet mode
wget --no-check-certificate https://...       # skip SSL verification
wget -r -np https://example.com/docs/         # recursive download
wget -m https://example.com/                  # mirror a website
wget --limit-rate=500k https://...            # limit download speed
wget -b https://example.com/large.iso        # background download
wget -i urls.txt                              # download list of URLs from file
wget --user=admin --password=pass https://... # HTTP auth
```

### 7.2 `curl` — Transfer Data with URLs

```bash
curl https://example.com                       # GET request, output to stdout
curl -o file.html https://example.com          # save to file
curl -O https://example.com/file.tar.gz        # save with original name
curl -L https://example.com                    # follow redirects
curl -I https://example.com                    # headers only (HEAD request)
curl -v https://example.com                    # verbose (show request/response)
curl -s https://example.com                    # silent (no progress)
curl -C - -O https://example.com/large.iso     # resume download
curl --limit-rate 500K https://...             # limit rate
curl -u user:pass https://example.com          # basic authentication
curl -k https://self-signed.example.com        # skip SSL verification
```

**HTTP Methods:**

```bash
curl -X POST -d "key=val" https://api.example.com/endpoint     # POST form data
curl -X POST -H "Content-Type: application/json" \
     -d '{"name":"Alice"}' https://api.example.com/users       # POST JSON
curl -X PUT -d @data.json https://api.example.com/item/1       # PUT from file
curl -X DELETE https://api.example.com/item/1                  # DELETE
curl -H "Authorization: Bearer TOKEN" https://api.example.com  # auth header
```

**Useful curl Flags:**

| Flag | Description |
|---|---|
| `-o FILE` | Save output to FILE |
| `-O` | Save with server filename |
| `-L` | Follow redirects |
| `-I` | Headers only |
| `-v` | Verbose output |
| `-s` | Silent (suppress progress) |
| `-k` | Insecure (skip SSL cert) |
| `-u user:pass` | Basic auth |
| `-H "Header: val"` | Add custom header |
| `-d "data"` | POST data |
| `-X METHOD` | HTTP method |
| `-b cookie.txt` | Send cookies |
| `-c cookie.txt` | Save cookies |
| `--proxy host:port` | Use proxy |

### 7.3 `scp` — Secure Copy (SSH-based)

```bash
scp file.txt user@host:/remote/path/          # local → remote
scp user@host:/remote/file.txt /local/path/   # remote → local
scp file.txt user@host:~                       # to home directory
scp -r dir/ user@host:/remote/                 # copy directory recursively
scp -P 2222 file.txt user@host:/path/          # non-standard SSH port
scp -i ~/.ssh/key.pem file.txt user@host:~     # use specific key
scp -C file.txt user@host:~                    # compress during transfer
scp user1@host1:/file user2@host2:/file        # remote to remote
```

### 7.4 `rsync` — Efficient Synchronisation

`rsync` transfers only changed blocks, making it ideal for backups and large syncs.

```bash
rsync -av src/ dest/                           # archive + verbose, local
rsync -av src/ user@host:/dest/                # push to remote
rsync -av user@host:/src/ dest/                # pull from remote
rsync -avz src/ user@host:/dest/               # compress during transfer
rsync -avP src/ user@host:/dest/               # progress bar + partial
rsync --delete src/ dest/                      # delete files not in src
rsync -n src/ dest/                            # dry run (simulate only)
rsync -e "ssh -p 2222" src/ user@host:/dest/   # custom SSH port
rsync --exclude="*.log" src/ dest/             # exclude pattern
rsync --exclude-from=excludes.txt src/ dest/   # exclude list from file
rsync --bwlimit=5000 src/ user@host:/dest/     # limit bandwidth (KB/s)
rsync --backup --backup-dir=/backup/ src/ dest/ # keep backup copies
```

**Common rsync flags:**

| Flag | Meaning |
|---|---|
| `-a` | Archive (recursive + preserves permissions, times, symlinks) |
| `-v` | Verbose |
| `-z` | Compress |
| `-P` | Progress + keep partial |
| `-n` | Dry run |
| `--delete` | Delete extraneous destination files |
| `-e ssh` | Use SSH as transport |
| `--exclude` | Exclude pattern |

!!! tip "Trailing Slash in rsync"
    `rsync -av src/ dest/` syncs the *contents* of src into dest. Without the trailing slash, `rsync -av src dest/` creates `dest/src/`. Always be intentional about trailing slashes.

### 7.5 `sftp` — Interactive Secure FTP

```bash
sftp user@host                      # open session
sftp -P 2222 user@host              # non-standard port
```

**Inside sftp session:**

```bash
ls                                  # list remote files
lls                                 # list local files
cd /remote/dir                      # change remote directory
lcd /local/dir                      # change local directory
get file.txt                        # download file
get -r directory/                   # download directory
put file.txt                        # upload file
put -r directory/                   # upload directory
pwd                                 # remote current directory
lpwd                                # local current directory
mkdir newdir                        # create remote directory
rm file.txt                         # delete remote file
bye                                 # close session
```

---

## 8. SSH — Secure Shell

### 8.1 Connecting

```bash
ssh user@hostname                   # basic connection
ssh user@192.168.1.10               # connect by IP
ssh -p 2222 user@host               # non-standard port
ssh -i ~/.ssh/id_rsa user@host      # specify identity file
ssh -v user@host                    # verbose (debug connection)
ssh -vvv user@host                  # maximum verbosity
ssh -X user@host                    # enable X11 forwarding (GUI apps)
ssh -A user@host                    # forward SSH agent
ssh -t user@jumphost ssh user@target  # SSH through a jump host
```

### 8.2 SSH Key Authentication

```bash
# Generate a key pair
ssh-keygen -t ed25519 -C "your@email.com"       # modern Ed25519 key
ssh-keygen -t rsa -b 4096 -C "your@email.com"   # RSA 4096-bit
ssh-keygen -t ed25519 -f ~/.ssh/mykey            # custom filename

# Copy public key to server
ssh-copy-id user@host                            # copies ~/.ssh/id_*.pub
ssh-copy-id -i ~/.ssh/mykey.pub user@host        # specific key
ssh-copy-id -p 2222 user@host                    # non-standard port

# Manually add key (if ssh-copy-id unavailable)
cat ~/.ssh/id_ed25519.pub | ssh user@host "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

### 8.3 SSH Config File (`~/.ssh/config`)

```bash
# ~/.ssh/config
Host myserver
    HostName 192.168.1.50
    User admin
    Port 2222
    IdentityFile ~/.ssh/mykey
    ForwardAgent yes

Host jumphost
    HostName jump.example.com
    User deploy
    Port 22

Host internal
    HostName 10.0.0.5
    User root
    ProxyJump jumphost
```

```bash
ssh myserver                        # uses config above
ssh internal                        # connects through jumphost automatically
```

### 8.4 Port Forwarding (SSH Tunnels)

```bash
# Local port forwarding (access remote service locally)
ssh -L 8080:localhost:80 user@host      # localhost:8080 → host's port 80
ssh -L 5432:db.internal:5432 user@jump # access DB through jump host

# Remote port forwarding (expose local service remotely)
ssh -R 8080:localhost:3000 user@host    # host's port 8080 → your local 3000

# Dynamic port forwarding (SOCKS proxy)
ssh -D 1080 user@host                   # SOCKS5 proxy on localhost:1080

# Background tunnel
ssh -fNL 8080:localhost:80 user@host    # -f background, -N no command
```

### 8.5 SSH Server Configuration (`/etc/ssh/sshd_config`)

```bash
Port 22                             # listening port
PermitRootLogin no                  # disable root login (recommended)
PasswordAuthentication no           # key-only login (recommended)
PubkeyAuthentication yes            # allow key auth
AuthorizedKeysFile .ssh/authorized_keys
MaxAuthTries 3                      # limit brute force
AllowUsers alice bob                # whitelist users
DenyUsers root                      # blacklist users
X11Forwarding no                    # disable GUI forwarding if unused
ClientAliveInterval 300             # keepalive every 5 minutes
```

```bash
systemctl restart sshd              # apply config changes
sshd -t                             # test config syntax before restart
```

### 8.6 Key Management

```bash
ssh-add ~/.ssh/id_ed25519           # add key to ssh-agent
ssh-add -l                          # list loaded keys
ssh-add -D                          # remove all keys from agent
eval $(ssh-agent)                   # start agent in current shell
ssh-keygen -lf ~/.ssh/id_ed25519.pub  # show key fingerprint
ssh-keygen -p -f ~/.ssh/id_rsa      # change key passphrase
ssh-keygen -R hostname              # remove host from known_hosts
```

---

## 9. Firewall Management

### 9.1 `ufw` — Uncomplicated Firewall (Debian/Ubuntu)

```bash
ufw status                          # show status and rules
ufw status verbose                  # detailed status
ufw status numbered                 # rules with numbers
ufw enable                          # enable firewall
ufw disable                         # disable firewall

# Allow / Deny by port
ufw allow 22                        # allow SSH
ufw allow 80/tcp                    # allow HTTP
ufw allow 443/tcp                   # allow HTTPS
ufw deny 23                         # deny Telnet
ufw allow 8000:9000/tcp             # allow port range

# Allow / Deny by service name
ufw allow ssh                       # allow SSH (by name)
ufw allow http                      # allow HTTP
ufw allow https                     # allow HTTPS

# Allow / Deny by IP
ufw allow from 192.168.1.0/24       # allow entire subnet
ufw allow from 10.0.0.5 to any port 22  # allow IP to port 22
ufw deny from 1.2.3.4               # block specific IP

# Delete rules
ufw delete allow 80/tcp
ufw delete 3                        # delete rule number 3
ufw reset                           # reset all rules
```

### 9.2 `firewalld` — Dynamic Firewall (RHEL/Fedora/CentOS)

```bash
firewall-cmd --state                # check if running
firewall-cmd --get-default-zone     # current default zone
firewall-cmd --get-active-zones     # active zones with interfaces
firewall-cmd --list-all             # list all rules in default zone

# Zones
firewall-cmd --get-zones                            # list zones
firewall-cmd --zone=public --list-all               # list rules in zone
firewall-cmd --set-default-zone=home                # change default zone
firewall-cmd --change-interface=eth0 --zone=trusted # assign interface to zone

# Allow / Deny Services
firewall-cmd --add-service=http --permanent          # allow HTTP permanently
firewall-cmd --add-service=https --permanent         # allow HTTPS
firewall-cmd --remove-service=telnet --permanent     # deny Telnet
firewall-cmd --list-services                         # list allowed services

# Allow / Deny Ports
firewall-cmd --add-port=8080/tcp --permanent         # allow port
firewall-cmd --remove-port=8080/tcp --permanent      # remove port

# Apply changes
firewall-cmd --reload                                # reload rules

# Rich rules (more specific)
firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.1.5" accept' --permanent
```

### 9.3 `iptables` — Low-Level Packet Filtering

```bash
iptables -L                         # list all rules
iptables -L -v -n                   # verbose, numeric
iptables -L INPUT -n                # INPUT chain only
iptables -F                         # flush all rules (careful!)
iptables -F INPUT                   # flush INPUT chain

# Allow / Deny
iptables -A INPUT -p tcp --dport 22 -j ACCEPT     # allow SSH
iptables -A INPUT -p tcp --dport 80 -j ACCEPT     # allow HTTP
iptables -A INPUT -s 192.168.1.0/24 -j ACCEPT     # allow subnet
iptables -A INPUT -s 1.2.3.4 -j DROP              # block IP

# Stateful rules
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -m state --state NEW -p tcp --dport 443 -j ACCEPT

# Default policy
iptables -P INPUT DROP              # default deny all input
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Save / Restore
iptables-save > /etc/iptables/rules.v4
iptables-restore < /etc/iptables/rules.v4
```

---

## 10. Packet Analysis

### 10.1 `tcpdump` — Command-Line Packet Capture

```bash
tcpdump                                        # capture on default interface
tcpdump -i eth0                                # specific interface
tcpdump -i any                                 # all interfaces
tcpdump -n                                     # no DNS lookup
tcpdump -nn                                    # no DNS or port lookup
tcpdump -v                                     # verbose
tcpdump -vvv                                   # maximum verbosity
tcpdump -c 100                                 # capture only 100 packets
tcpdump -w capture.pcap                        # write to file
tcpdump -r capture.pcap                        # read from file
tcpdump -A                                     # print ASCII payload
tcpdump -X                                     # print hex and ASCII
```

**Filters:**

```bash
tcpdump host 192.168.1.1                       # traffic to/from host
tcpdump src 192.168.1.1                        # traffic FROM host
tcpdump dst 192.168.1.1                        # traffic TO host
tcpdump port 80                                # HTTP traffic
tcpdump port 443                               # HTTPS traffic
tcpdump not port 22                            # exclude SSH
tcpdump tcp                                    # TCP only
tcpdump udp                                    # UDP only
tcpdump icmp                                   # ICMP only
tcpdump -i eth0 'port 80 and host 192.168.1.5'  # combined filter
tcpdump -i eth0 'src 10.0.0.1 and dst port 22'  # source + dest port
tcpdump -i eth0 'not arp and not icmp'           # exclude noise
```

---

## 11. Network Performance and Diagnostics

### 11.1 `iperf3` — Bandwidth Testing

```bash
# On server
iperf3 -s                           # start server (port 5201)
iperf3 -s -p 9000                   # custom port

# On client
iperf3 -c 192.168.1.10             # test TCP bandwidth to server
iperf3 -c 192.168.1.10 -u          # UDP test
iperf3 -c 192.168.1.10 -t 30       # run for 30 seconds
iperf3 -c 192.168.1.10 -P 4        # 4 parallel streams
iperf3 -c 192.168.1.10 -R          # reverse (server → client)
iperf3 -c 192.168.1.10 -b 100M     # target 100 Mbps (UDP)
```

### 11.2 `netcat` (`nc`) — Swiss Army Knife

```bash
# Connectivity testing
nc -zv 192.168.1.1 22               # test if port 22 is open (-z = scan, -v = verbose)
nc -zv 192.168.1.1 20-25            # scan port range
nc -zu 192.168.1.1 53               # UDP port test

# Simple server / client
nc -l 1234                          # listen on port 1234
nc 192.168.1.1 1234                 # connect to listening server

# File transfer
nc -l 1234 > received.txt           # receive file on server
nc 192.168.1.1 1234 < file.txt      # send file from client

# Chat session
nc -l 1234                          # server listens
nc 192.168.1.1 1234                 # client connects (type to chat)

# Port scanning (basic)
nc -zv 192.168.1.1 1-1024 2>&1 | grep "succeeded"
```

### 11.3 Checking Network Interface Statistics

```bash
ip -s link show eth0                # packets, bytes, errors, drops
cat /proc/net/dev                   # raw interface statistics
ifstat                              # live interface bandwidth
sar -n DEV 1 5                      # network stats via sysstat (1s, 5 times)
nethogs                             # per-process bandwidth usage
nload eth0                          # visual bandwidth meter
```

---

## 12. Important Network Files and Directories

| File / Path | Purpose |
|---|---|
| `/etc/hosts` | Static hostname-to-IP mappings |
| `/etc/resolv.conf` | DNS nameserver configuration |
| `/etc/nsswitch.conf` | Name resolution order |
| `/etc/hostname` | System hostname |
| `/etc/network/interfaces` | Interface config (Debian/Ubuntu) |
| `/etc/sysconfig/network-scripts/` | Interface config (RHEL/CentOS) |
| `/etc/NetworkManager/` | NetworkManager config files |
| `/etc/ssh/sshd_config` | SSH server configuration |
| `/etc/ssh/ssh_config` | SSH client configuration |
| `/etc/services` | Port-to-service name mappings |
| `/etc/protocols` | Protocol numbers |
| `/proc/net/dev` | Real-time interface statistics |
| `/proc/net/tcp` | Active TCP connections |
| `/proc/net/route` | Kernel routing table |
| `~/.ssh/config` | User SSH client config |
| `~/.ssh/authorized_keys` | Accepted public keys for login |
| `~/.ssh/known_hosts` | Trusted remote host fingerprints |

---

## 13. Common Network Troubleshooting Workflow

### 13.1 Step-by-Step Diagnosis

```bash
# Step 1: Is the interface up?
ip link show
ip addr show

# Step 2: Do we have an IP?
ip addr show eth0

# Step 3: Is the gateway reachable?
ip route show default
ping -c3 $(ip route | grep default | awk '{print $3}')

# Step 4: Is DNS working?
cat /etc/resolv.conf
dig google.com @8.8.8.8 +short

# Step 5: Can we reach the internet?
ping -c3 8.8.8.8          # test by IP (bypasses DNS)
ping -c3 google.com        # test DNS + connectivity

# Step 6: Is the target service running?
ss -tlnp | grep :80        # is HTTP server listening?
nc -zv 192.168.1.10 80     # can we reach it?

# Step 7: Is a firewall blocking?
iptables -L -n -v
ufw status verbose

# Step 8: What path does traffic take?
traceroute -n google.com
mtr -r google.com
```

### 13.2 Common Problems and Quick Fixes

| Problem | Likely Cause | Fix |
|---|---|---|
| No IP address | DHCP failure | `dhclient eth0` or `nmcli con up` |
| Can ping IP, not hostname | DNS misconfigured | Check `/etc/resolv.conf` |
| Can't reach gateway | Interface down | `ip link set eth0 up` |
| Service unreachable | Firewall blocking | Check `ufw`/`iptables` rules |
| SSH refused | sshd not running | `systemctl start sshd` |
| Slow connection | DNS timeout | Add faster nameserver |
| High packet loss | Network congestion / bad cable | Use `mtr` to identify hop |

---

## 14. Quick Reference Cheat Sheet

### Interface Management

| Command | What it does |
|---|---|
| `ip addr show` | Show all IPs |
| `ip link set eth0 up/down` | Enable/disable interface |
| `ip route show` | Show routing table |
| `ip route add default via GW` | Set default gateway |
| `ip neigh show` | Show ARP table |

### Connectivity

| Command | What it does |
|---|---|
| `ping -c4 host` | Test reachability |
| `traceroute -n host` | Trace path to host |
| `mtr -r host` | Report-mode traceroute |
| `dig host +short` | Quick DNS lookup |
| `nc -zv host port` | Test if port is open |

### Ports & Sockets

| Command | What it does |
|---|---|
| `ss -tlnp` | Listening TCP ports + PIDs |
| `ss -s` | Socket summary stats |
| `nmap -p- host` | Full port scan |
| `netstat -rn` | Routing table (legacy) |

### File Transfer

| Command | What it does |
|---|---|
| `wget -c URL` | Resume download |
| `curl -O URL` | Download with original name |
| `scp file user@host:~` | Secure copy to remote |
| `rsync -avP src/ dest/` | Sync with progress |
| `rsync --delete src/ dest/` | Mirror sync |

### SSH

| Command | What it does |
|---|---|
| `ssh -p PORT user@host` | Connect on custom port |
| `ssh-keygen -t ed25519` | Generate Ed25519 key |
| `ssh-copy-id user@host` | Install public key |
| `ssh -L 8080:localhost:80 host` | Local port forward |
| `ssh -D 1080 user@host` | SOCKS proxy |

### DNS

| Command | What it does |
|---|---|
| `dig domain` | Full DNS query |
| `dig domain +short` | IP only |
| `dig -x IP` | Reverse lookup |
| `dig @8.8.8.8 domain` | Query specific server |
| `resolvectl flush-caches` | Flush DNS cache |