# Linux Crash Course - Networking

## Table of Contents
- [Network Configuration Basics](#network-configuration-basics)
- [IP Command (iproute2)](#ip-command-iproute2)
- [Network Connectivity Testing](#network-connectivity-testing)
- [DNS and Name Resolution](#dns-and-name-resolution)
- [Network Services and Ports](#network-services-and-ports)
- [Firewall Management (firewalld)](#firewall-management-firewalld)
- [Remote Access (SSH)](#remote-access-ssh)
- [File Transfer](#file-transfer)
- [Network Troubleshooting](#network-troubleshooting)
- [NetworkManager (nmcli)](#networkmanager-nmcli)

---

## Network Configuration Basics

### Network Configuration Files (RHEL)
```bash
# NetworkManager is the default in RHEL 7+
# Legacy network scripts deprecated in RHEL 8+

# NetworkManager connection files
/etc/NetworkManager/system-connections/    # Connection profiles

# Hostname configuration
/etc/hostname                              # System hostname
/etc/hosts                                 # Local host resolution

# DNS configuration
/etc/resolv.conf                           # DNS servers (managed by NM)

# Network interface naming
# enp0s3 = en(ethernet) p0(PCI bus 0) s3(slot 3)
# ens192 = en(ethernet) s192(hotplug slot)
# eth0   = Legacy naming (if biosdevname=0 net.ifnames=0)
```

### View Current Configuration
```bash
# IP addresses
ip addr
ip a                                       # Short form

# Routing table
ip route
ip r

# DNS servers
cat /etc/resolv.conf

# Hostname
hostname
hostnamectl

# All network info
nmcli
```

---

## IP Command (iproute2)

The `ip` command replaces legacy tools (`ifconfig`, `route`, `arp`, `netstat`).

### ip addr - Manage IP Addresses
```bash
# Show all interfaces
ip addr
ip addr show
ip a

# Show specific interface
ip addr show dev enp0s3

# Show only IPv4
ip -4 addr

# Show only IPv6
ip -6 addr

# Add IP address (temporary)
sudo ip addr add 192.168.1.100/24 dev enp0s3

# Add with label
sudo ip addr add 192.168.1.101/24 dev enp0s3 label enp0s3:1

# Remove IP address
sudo ip addr del 192.168.1.100/24 dev enp0s3

# Flush all addresses
sudo ip addr flush dev enp0s3
```

### ip link - Manage Network Interfaces
```bash
# Show all interfaces
ip link
ip link show
ip l

# Show specific interface
ip link show dev enp0s3

# Bring interface up
sudo ip link set enp0s3 up

# Bring interface down
sudo ip link set enp0s3 down

# Set MTU
sudo ip link set enp0s3 mtu 9000

# Change MAC address
sudo ip link set enp0s3 down
sudo ip link set enp0s3 address 00:11:22:33:44:55
sudo ip link set enp0s3 up

# Show statistics
ip -s link
ip -s link show dev enp0s3
```

### ip route - Manage Routing
```bash
# Show routing table
ip route
ip route show
ip r

# Show route to destination
ip route get 8.8.8.8

# Add static route
sudo ip route add 10.0.0.0/8 via 192.168.1.1
sudo ip route add 10.0.0.0/8 via 192.168.1.1 dev enp0s3

# Add default gateway
sudo ip route add default via 192.168.1.1

# Delete route
sudo ip route del 10.0.0.0/8

# Delete default route
sudo ip route del default

# Replace route
sudo ip route replace 10.0.0.0/8 via 192.168.1.254
```

### ip neighbor - ARP Table
```bash
# Show ARP table (replaces arp -a)
ip neighbor
ip neigh
ip n

# Add static ARP entry
sudo ip neigh add 192.168.1.100 lladdr 00:11:22:33:44:55 dev enp0s3

# Delete entry
sudo ip neigh del 192.168.1.100 dev enp0s3

# Flush ARP cache
sudo ip neigh flush all
```

---

## Network Connectivity Testing

### ping - Test Connectivity
```bash
# Basic ping
ping hostname
ping 192.168.1.1
ping google.com

# Specific count
ping -c 4 google.com                      # 4 packets

# Interval
ping -i 0.5 google.com                    # Every 0.5 seconds

# Timeout
ping -W 2 google.com                      # 2 second timeout

# Packet size
ping -s 1000 google.com                   # 1000 byte packets

# Flood ping (root only)
sudo ping -f google.com

# IPv6
ping6 ::1
ping -6 google.com

# Don't resolve hostnames
ping -n 192.168.1.1
```

### traceroute / tracepath - Trace Route
```bash
# Trace route to destination
traceroute google.com
traceroute -n google.com                  # No DNS resolution

# Using ICMP (like Windows)
sudo traceroute -I google.com

# Using TCP
sudo traceroute -T -p 80 google.com

# tracepath (no root required)
tracepath google.com
tracepath -n google.com

# mtr - Combined ping + traceroute
mtr google.com
mtr -n google.com                         # No DNS
mtr -r -c 10 google.com                   # Report mode, 10 packets
```

### curl - Transfer Data
```bash
# Basic GET request
curl http://example.com
curl https://api.example.com/data

# Save to file
curl -o filename.html http://example.com
curl -O http://example.com/file.zip       # Keep original name

# Follow redirects
curl -L http://example.com

# Show headers
curl -I http://example.com                # Headers only
curl -i http://example.com                # Headers + body

# POST request
curl -X POST http://api.example.com/data
curl -X POST -d "param1=value1&param2=value2" http://api.example.com
curl -X POST -H "Content-Type: application/json" -d '{"key":"value"}' http://api.example.com

# Authentication
curl -u username:password http://example.com
curl -H "Authorization: Bearer TOKEN" http://api.example.com

# Verbose
curl -v http://example.com

# Silent mode
curl -s http://example.com

# Timeout
curl --connect-timeout 5 -m 10 http://example.com

# Download with progress
curl -# -O http://example.com/largefile.zip

# Test port connectivity
curl -v telnet://192.168.1.1:22
```

### wget - Download Files
```bash
# Download file
wget http://example.com/file.zip

# Save with different name
wget -O newname.zip http://example.com/file.zip

# Download in background
wget -b http://example.com/largefile.zip

# Resume download
wget -c http://example.com/largefile.zip

# Recursive download
wget -r http://example.com/
wget -r -l 2 http://example.com/          # Depth level 2

# Mirror website
wget -m http://example.com/

# Limit speed
wget --limit-rate=1m http://example.com/file.zip

# Download multiple files
wget -i urls.txt                          # URLs from file

# Quiet mode
wget -q http://example.com/file.zip
```

---

## DNS and Name Resolution

### Name Resolution Order
```bash
# /etc/nsswitch.conf defines order
cat /etc/nsswitch.conf | grep hosts
# hosts: files dns myhostname
# 1. files = /etc/hosts
# 2. dns = /etc/resolv.conf
# 3. myhostname = local hostname
```

### /etc/hosts - Local Resolution
```bash
# Static host entries
cat /etc/hosts

# Format:
# IP_address hostname [aliases...]
127.0.0.1   localhost localhost.localdomain
192.168.1.10 webserver web
192.168.1.20 database db mysql
```

### /etc/resolv.conf - DNS Servers
```bash
cat /etc/resolv.conf

# Format:
nameserver 8.8.8.8
nameserver 8.8.4.4
search example.com
domain example.com

# Note: Managed by NetworkManager
# Edit via nmcli, not directly
```

### DNS Lookup Tools

#### host
```bash
host google.com
host -t MX google.com                     # Mail servers
host -t NS google.com                     # Name servers
host -t A google.com                      # A records
host 8.8.8.8                              # Reverse lookup
```

#### dig
```bash
# Basic lookup
dig google.com

# Short answer
dig +short google.com

# Specific record type
dig google.com A
dig google.com MX
dig google.com NS
dig google.com TXT
dig google.com ANY

# Reverse lookup
dig -x 8.8.8.8

# Use specific DNS server
dig @8.8.8.8 google.com

# Trace DNS resolution
dig +trace google.com

# No comments (cleaner output)
dig +noall +answer google.com
```

#### nslookup
```bash
# Interactive mode
nslookup
> google.com
> set type=MX
> google.com
> exit

# Non-interactive
nslookup google.com
nslookup google.com 8.8.8.8              # Use specific DNS
nslookup -type=MX google.com
```

### getent - Query Name Services
```bash
# Lookup host (respects nsswitch.conf)
getent hosts google.com
getent hosts localhost

# All hosts
getent hosts
```

---

## Network Services and Ports

### ss - Socket Statistics (replaces netstat)
```bash
# All sockets
ss

# Listening sockets
ss -l

# TCP sockets
ss -t

# UDP sockets
ss -u

# Listening TCP
ss -lt

# With process info
ss -ltp
ss -ltpn                                  # Numeric ports

# All connections (listening + established)
ss -a
ss -atn                                   # All TCP, numeric

# Filter by state
ss state established
ss state listening
ss state time-wait

# Filter by port
ss -lt sport = :22                        # Source port 22
ss -lt dport = :80                        # Dest port 80
ss -tn dport = :443

# Filter by address
ss -tn dst 192.168.1.1
ss -tn src 192.168.1.100

# Show timers
ss -to

# Summary statistics
ss -s
```

### netstat (Legacy, but still available)
```bash
# Listening ports with programs
netstat -tlnp

# All connections
netstat -an

# Routing table
netstat -rn

# Interface statistics
netstat -i

# Breakdown:
# -t = TCP
# -u = UDP
# -l = Listening
# -n = Numeric (don't resolve)
# -p = Show process
# -a = All
# -r = Routing
```

### lsof - List Open Files (including network)
```bash
# Network connections
lsof -i

# Specific port
lsof -i :22
lsof -i :80

# TCP connections
lsof -i TCP

# Specific host
lsof -i @192.168.1.1

# By process
lsof -i -P -n | grep sshd

# Who's using a port
sudo lsof -i :8080
```

### Common Ports
| Port | Service |
|------|---------|
| 20/21 | FTP |
| 22 | SSH |
| 23 | Telnet |
| 25 | SMTP |
| 53 | DNS |
| 80 | HTTP |
| 110 | POP3 |
| 143 | IMAP |
| 443 | HTTPS |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 6379 | Redis |
| 8080 | HTTP Alt |

---

## Firewall Management (firewalld)

### Understanding firewalld
**firewalld** is the default firewall in RHEL 7+. It uses **zones** to define trust levels for network connections.

### Default Zones
| Zone | Description |
|------|-------------|
| drop | Drop all incoming, no reply |
| block | Reject incoming with message |
| public | Default, untrusted networks |
| external | External networks with NAT |
| dmz | DMZ with limited access |
| work | Work networks, somewhat trusted |
| home | Home networks, trusted |
| internal | Internal networks, trusted |
| trusted | All traffic allowed |

### firewall-cmd Basics
```bash
# Check status
sudo firewall-cmd --state
sudo systemctl status firewalld

# Start/stop firewall
sudo systemctl start firewalld
sudo systemctl stop firewalld
sudo systemctl enable firewalld

# Get default zone
sudo firewall-cmd --get-default-zone

# Get active zones
sudo firewall-cmd --get-active-zones

# List all zones
sudo firewall-cmd --get-zones

# Get zone info
sudo firewall-cmd --zone=public --list-all
sudo firewall-cmd --list-all                    # Default zone
sudo firewall-cmd --list-all-zones              # All zones
```

### Managing Services
```bash
# List available services
sudo firewall-cmd --get-services

# List enabled services
sudo firewall-cmd --list-services

# Add service (runtime)
sudo firewall-cmd --add-service=http

# Add service (permanent)
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload                      # Apply permanent

# Add service permanently and immediately
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --add-service=http

# Remove service
sudo firewall-cmd --remove-service=http
sudo firewall-cmd --permanent --remove-service=http

# Multiple services
sudo firewall-cmd --permanent --add-service={http,https,ssh}

# Check if service allowed
sudo firewall-cmd --query-service=http
```

### Managing Ports
```bash
# Add port
sudo firewall-cmd --add-port=8080/tcp
sudo firewall-cmd --permanent --add-port=8080/tcp

# Add port range
sudo firewall-cmd --permanent --add-port=5000-5100/tcp

# Remove port
sudo firewall-cmd --remove-port=8080/tcp
sudo firewall-cmd --permanent --remove-port=8080/tcp

# List ports
sudo firewall-cmd --list-ports

# Check port
sudo firewall-cmd --query-port=8080/tcp
```

### Managing Rich Rules
```bash
# Rich rules provide more control

# Allow specific source IP
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.100" accept'

# Allow port from specific IP
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" port port="3306" protocol="tcp" accept'

# Reject specific source
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.0.0.5" reject'

# Rate limiting
sudo firewall-cmd --permanent --add-rich-rule='rule service name="ssh" accept limit value="10/m"'

# Log and accept
sudo firewall-cmd --permanent --add-rich-rule='rule service name="ssh" log prefix="SSH " level="info" accept'

# List rich rules
sudo firewall-cmd --list-rich-rules

# Remove rich rule
sudo firewall-cmd --permanent --remove-rich-rule='rule family="ipv4" source address="192.168.1.100" accept'
```

### Zone Management
```bash
# Change default zone
sudo firewall-cmd --set-default-zone=home

# Assign interface to zone
sudo firewall-cmd --zone=trusted --change-interface=enp0s8
sudo firewall-cmd --permanent --zone=trusted --change-interface=enp0s8

# Get zone for interface
sudo firewall-cmd --get-zone-of-interface=enp0s3

# Add source to zone
sudo firewall-cmd --permanent --zone=trusted --add-source=192.168.1.0/24
```

### Port Forwarding
```bash
# Forward port 80 to 8080 locally
sudo firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toport=8080

# Forward to another host
sudo firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toport=80:toaddr=192.168.1.100

# Enable masquerading (required for forwarding to other hosts)
sudo firewall-cmd --permanent --add-masquerade

# Check
sudo firewall-cmd --query-masquerade
sudo firewall-cmd --list-forward-ports
```

### Reload and Save
```bash
# Reload firewall (apply permanent rules)
sudo firewall-cmd --reload

# Runtime to permanent
sudo firewall-cmd --runtime-to-permanent

# Check for differences
sudo firewall-cmd --list-all
sudo firewall-cmd --permanent --list-all
```

---

## Remote Access (SSH)

### SSH Client
```bash
# Basic connection
ssh user@hostname
ssh user@192.168.1.100

# Specific port
ssh -p 2222 user@hostname

# Verbose (debugging)
ssh -v user@hostname
ssh -vvv user@hostname                    # Very verbose

# Execute command
ssh user@hostname 'ls -la'
ssh user@hostname 'df -h; free -h'

# Forward X11 (GUI apps)
ssh -X user@hostname
ssh -Y user@hostname                      # Trusted forwarding

# Force password auth
ssh -o PreferredAuthentications=password user@hostname

# Disable host key checking (not recommended)
ssh -o StrictHostKeyChecking=no user@hostname
```

### SSH Key Authentication
```bash
# Generate key pair
ssh-keygen
ssh-keygen -t rsa -b 4096
ssh-keygen -t ed25519                     # Recommended
ssh-keygen -t ed25519 -C "email@example.com"

# Key files
~/.ssh/id_rsa                             # Private key
~/.ssh/id_rsa.pub                         # Public key
~/.ssh/id_ed25519                         # ED25519 private
~/.ssh/id_ed25519.pub                     # ED25519 public

# Copy public key to server
ssh-copy-id user@hostname
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@hostname
ssh-copy-id -p 2222 user@hostname

# Manual copy
cat ~/.ssh/id_ed25519.pub | ssh user@hostname 'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys'

# Set permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_rsa.pub
chmod 600 ~/.ssh/authorized_keys
```

### SSH Config File
```bash
# Create/edit ~/.ssh/config
vim ~/.ssh/config
```

```
# Default for all hosts
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3

# Specific host
Host webserver
    HostName 192.168.1.100
    User admin
    Port 2222
    IdentityFile ~/.ssh/web_key

Host db
    HostName database.example.com
    User dbadmin
    
Host jump
    HostName jumphost.example.com
    User jumper

# Use jump host
Host internal
    HostName 10.0.0.50
    User admin
    ProxyJump jump
```

```bash
# Now use aliases
ssh webserver
ssh db
ssh internal
```

### SSH Tunneling / Port Forwarding
```bash
# Local port forwarding (access remote service locally)
# Access remote:3306 via localhost:3307
ssh -L 3307:localhost:3306 user@remotehost
ssh -L 3307:dbserver:3306 user@jumphost    # Through jump host

# Remote port forwarding (expose local service remotely)
# Make local:8080 available on remote:9090
ssh -R 9090:localhost:8080 user@remotehost

# Dynamic port forwarding (SOCKS proxy)
ssh -D 1080 user@remotehost
# Configure browser to use SOCKS5 proxy localhost:1080

# Background tunnel
ssh -fN -L 3307:localhost:3306 user@remotehost
```

### SSH Server Configuration
```bash
# Config file
sudo vim /etc/ssh/sshd_config

# Common settings:
Port 22                                   # Change default port
PermitRootLogin no                        # Disable root login
PasswordAuthentication no                 # Key-only auth
PubkeyAuthentication yes
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
AllowUsers user1 user2
AllowGroups sshusers

# Restart after changes
sudo systemctl restart sshd
```

---

## File Transfer

### scp - Secure Copy
```bash
# Copy to remote
scp file.txt user@host:/path/
scp file.txt user@host:~/
scp file.txt user@host:/home/user/newname.txt

# Copy from remote
scp user@host:/path/file.txt ./
scp user@host:~/file.txt ./local_file.txt

# Copy directory (recursive)
scp -r directory/ user@host:/path/

# Specific port
scp -P 2222 file.txt user@host:/path/

# Preserve permissions/times
scp -p file.txt user@host:/path/

# Compress during transfer
scp -C largefile.zip user@host:/path/

# Between two remote hosts
scp user1@host1:/file.txt user2@host2:/path/
```

### rsync - Efficient Sync
```bash
# Basic sync (local)
rsync -av source/ destination/

# Sync to remote
rsync -av /local/path/ user@host:/remote/path/

# Sync from remote
rsync -av user@host:/remote/path/ /local/path/

# Common options
rsync -avz source/ dest/                  # Compress
rsync -avzP source/ dest/                 # Progress + partial
rsync -avz --delete source/ dest/         # Mirror (delete extra)
rsync -avz --exclude='*.log' source/ dest/  # Exclude pattern
rsync -avz --exclude-from=exclude.txt src/ dest/
rsync -avzn source/ dest/                 # Dry run

# Over SSH with options
rsync -avz -e "ssh -p 2222" source/ user@host:/dest/

# Backup with timestamp
rsync -avz --backup --backup-dir=/backup/$(date +%Y%m%d) source/ dest/

# Options breakdown:
# -a = archive (preserve everything)
# -v = verbose
# -z = compress
# -P = progress + partial (resume)
# -n = dry run
# --delete = delete files not in source
```

### sftp - Secure FTP
```bash
# Connect
sftp user@host
sftp -P 2222 user@host

# Commands in sftp:
sftp> pwd                                 # Remote working directory
sftp> lpwd                                # Local working directory
sftp> ls                                  # List remote
sftp> lls                                 # List local
sftp> cd /path                            # Change remote dir
sftp> lcd /path                           # Change local dir
sftp> get file.txt                        # Download
sftp> get -r directory/                   # Download directory
sftp> put file.txt                        # Upload
sftp> put -r directory/                   # Upload directory
sftp> mkdir newdir                        # Create remote dir
sftp> rm file.txt                         # Delete remote file
sftp> exit                                # Quit
```

---

## Network Troubleshooting

### Troubleshooting Steps
```
1. Check physical connection / link status
   → ip link show

2. Check IP configuration
   → ip addr show
   → ip route show

3. Check local connectivity
   → ping localhost
   → ping gateway

4. Check DNS resolution
   → dig google.com
   → cat /etc/resolv.conf

5. Check remote connectivity
   → ping remote_host
   → traceroute remote_host

6. Check port connectivity
   → ss -lt
   → nc -zv host port

7. Check firewall
   → firewall-cmd --list-all
   → iptables -L -n

8. Check logs
   → journalctl -u NetworkManager
   → dmesg | grep -i eth
```

### netcat (nc) - Network Swiss Army Knife
```bash
# Test port connectivity
nc -zv hostname 22
nc -zv hostname 80-90                     # Port range

# Listen on port
nc -l 1234                                # Listen on 1234
nc -l -p 1234                             # Same

# Connect and chat
nc hostname 1234                          # Connect to listener

# Transfer file
# Receiver:
nc -l 1234 > received_file.txt
# Sender:
nc hostname 1234 < file.txt

# Port scanning (basic)
nc -zv hostname 20-25

# HTTP request
echo -e "GET / HTTP/1.1\r\nHost: example.com\r\n\r\n" | nc example.com 80
```

### tcpdump - Packet Capture
```bash
# Capture on interface
sudo tcpdump -i enp0s3

# Limit packets
sudo tcpdump -i enp0s3 -c 100

# Don't resolve names
sudo tcpdump -i enp0s3 -n

# Filter by host
sudo tcpdump -i enp0s3 host 192.168.1.100

# Filter by port
sudo tcpdump -i enp0s3 port 80
sudo tcpdump -i enp0s3 port 22 or port 443

# Filter by protocol
sudo tcpdump -i enp0s3 icmp
sudo tcpdump -i enp0s3 tcp
sudo tcpdump -i enp0s3 udp

# Save to file
sudo tcpdump -i enp0s3 -w capture.pcap

# Read from file
tcpdump -r capture.pcap

# Verbose
sudo tcpdump -i enp0s3 -v
sudo tcpdump -i enp0s3 -vvv

# Show packet contents
sudo tcpdump -i enp0s3 -X
sudo tcpdump -i enp0s3 -A                 # ASCII only
```

### nmap - Network Scanner (if installed)
```bash
# Install
sudo dnf install nmap

# Basic host scan
nmap hostname

# Scan network
nmap 192.168.1.0/24

# Scan specific ports
nmap -p 22,80,443 hostname
nmap -p 1-1024 hostname
nmap -p- hostname                         # All ports

# Service detection
nmap -sV hostname

# OS detection
sudo nmap -O hostname

# Stealth scan
sudo nmap -sS hostname

# Ping scan (discover hosts)
nmap -sn 192.168.1.0/24
```

---

## NetworkManager (nmcli)

### nmcli Basics
```bash
# Show all connections
nmcli connection show
nmcli con show
nmcli c s

# Show active connections
nmcli con show --active

# Show devices
nmcli device status
nmcli dev status
nmcli d s

# Show device details
nmcli dev show enp0s3

# General status
nmcli general status
nmcli general hostname
```

### Managing Connections
```bash
# Bring up connection
nmcli con up "Connection Name"
nmcli con up enp0s3

# Bring down connection
nmcli con down "Connection Name"

# Reload connections
nmcli con reload

# Delete connection
nmcli con delete "Connection Name"
```

### Create/Modify Connections
```bash
# Create DHCP connection
nmcli con add con-name "MyConnection" type ethernet ifname enp0s3

# Create static IP connection
nmcli con add con-name "Static" type ethernet ifname enp0s3 \
  ipv4.addresses 192.168.1.100/24 \
  ipv4.gateway 192.168.1.1 \
  ipv4.dns "8.8.8.8,8.8.4.4" \
  ipv4.method manual

# Modify existing connection
nmcli con mod "MyConnection" ipv4.addresses 192.168.1.100/24
nmcli con mod "MyConnection" ipv4.gateway 192.168.1.1
nmcli con mod "MyConnection" ipv4.dns "8.8.8.8"
nmcli con mod "MyConnection" ipv4.method manual
nmcli con mod "MyConnection" connection.autoconnect yes

# Apply changes
nmcli con up "MyConnection"

# Add secondary IP
nmcli con mod "MyConnection" +ipv4.addresses 192.168.1.101/24

# Add DNS server
nmcli con mod "MyConnection" +ipv4.dns 1.1.1.1
```

### Hostname Management
```bash
# View hostname
hostnamectl
hostname

# Set hostname
sudo hostnamectl set-hostname newhostname
sudo hostnamectl set-hostname server.example.com

# Set transient hostname (current session)
sudo hostname tempname
```

### nmtui - Text UI for NetworkManager
```bash
# Interactive TUI
nmtui

# Direct to specific function
nmtui edit
nmtui connect
nmtui hostname
```

---

## Quick Reference Card

### IP Commands
| Command | Description |
|---------|-------------|
| `ip addr` | Show IP addresses |
| `ip link` | Show interfaces |
| `ip route` | Show routing table |
| `ip neigh` | Show ARP table |
| `ip addr add IP dev IF` | Add IP |
| `ip link set IF up/down` | Enable/disable |

### Connectivity
| Command | Description |
|---------|-------------|
| `ping host` | Test connectivity |
| `traceroute host` | Trace path |
| `dig domain` | DNS lookup |
| `curl URL` | HTTP request |
| `wget URL` | Download file |

### Ports/Sockets
| Command | Description |
|---------|-------------|
| `ss -tlnp` | Listening TCP ports |
| `ss -ulnp` | Listening UDP ports |
| `ss -atn` | All TCP connections |
| `lsof -i :PORT` | Process using port |
| `nc -zv host port` | Test port |

### firewall-cmd
| Command | Description |
|---------|-------------|
| `--list-all` | Show current config |
| `--add-service=X` | Allow service |
| `--add-port=X/tcp` | Allow port |
| `--permanent` | Make persistent |
| `--reload` | Apply changes |

### SSH/Transfer
| Command | Description |
|---------|-------------|
| `ssh user@host` | Remote shell |
| `ssh-keygen` | Generate keys |
| `ssh-copy-id user@host` | Copy public key |
| `scp file user@host:path` | Copy to remote |
| `rsync -avz src dest` | Sync files |

### nmcli
| Command | Description |
|---------|-------------|
| `nmcli con show` | List connections |
| `nmcli con up NAME` | Activate connection |
| `nmcli dev status` | Device status |
| `nmcli con add` | Create connection |
| `nmcli con mod` | Modify connection |

---

**[← Back to Index](README.md)**  
**Previous: [Process Management](05_process_management.md)**  
**Next: [Package Management](07_package_management.md)**
