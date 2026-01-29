# Linux Crash Course - Quick Reference

A condensed cheat sheet of essential Linux commands for quick lookup.

---

## Table of Contents
- [File System Navigation](#file-system-navigation)
- [File Operations](#file-operations)
- [Text Processing](#text-processing)
- [User & Permissions](#user--permissions)
- [Process Management](#process-management)
- [Networking](#networking)
- [Package Management](#package-management)
- [Storage & Disks](#storage--disks)
- [systemd Services](#systemd-services)
- [Logs & Troubleshooting](#logs--troubleshooting)
- [SELinux](#selinux)
- [Compression & Archives](#compression--archives)
- [SSH & Remote](#ssh--remote)
- [Useful Shortcuts](#useful-shortcuts)

---

## File System Navigation

```bash
pwd                     # Print working directory
cd /path                # Change to directory
cd ~                    # Go to home directory
cd -                    # Go to previous directory
cd ..                   # Go up one level

ls                      # List files
ls -la                  # List all with details
ls -lh                  # Human-readable sizes
ls -lt                  # Sort by time (newest first)
ls -lS                  # Sort by size (largest first)

tree                    # Directory tree structure
tree -L 2               # Limit to 2 levels
```

---

## File Operations

### Creating & Editing
```bash
touch file.txt          # Create empty file
mkdir dir               # Create directory
mkdir -p a/b/c          # Create nested directories

cat > file.txt          # Create and write (Ctrl+D to end)
vim file.txt            # Edit with vim
nano file.txt           # Edit with nano
```

### Viewing
```bash
cat file.txt            # Display entire file
less file.txt           # Page through file
head -n 20 file.txt     # First 20 lines
tail -n 20 file.txt     # Last 20 lines
tail -f file.txt        # Follow file (live)
```

### Copying, Moving, Deleting
```bash
cp file1 file2          # Copy file
cp -r dir1 dir2         # Copy directory recursively
mv file1 file2          # Move/rename
rm file                 # Delete file
rm -rf dir              # Delete directory (force, recursive)
```

### Finding Files
```bash
find /path -name "*.txt"            # Find by name
find /path -type f -size +100M      # Files over 100MB
find /path -mtime -7                # Modified in last 7 days
find /path -user john               # Owned by user
find /path -perm 755                # With specific permissions
find /path -name "*.log" -delete    # Find and delete

locate filename                     # Fast search (uses database)
updatedb                            # Update locate database
which command                       # Find command location
```

### Links
```bash
ln file hardlink        # Create hard link
ln -s file symlink      # Create symbolic link
readlink -f symlink     # Show link target
```

---

## Text Processing

### grep - Search Patterns
```bash
grep "pattern" file             # Search for pattern
grep -i "pattern" file          # Case insensitive
grep -r "pattern" /path         # Recursive search
grep -n "pattern" file          # Show line numbers
grep -v "pattern" file          # Invert match (NOT)
grep -c "pattern" file          # Count matches
grep -E "pat1|pat2" file        # Extended regex (OR)
grep -l "pattern" *.txt         # List matching files only
```

### sed - Stream Editor
```bash
sed 's/old/new/' file           # Replace first occurrence
sed 's/old/new/g' file          # Replace all occurrences
sed -i 's/old/new/g' file       # Edit in place
sed -n '5,10p' file             # Print lines 5-10
sed '/pattern/d' file           # Delete matching lines
sed '/^$/d' file                # Delete empty lines
```

### awk - Field Processing
```bash
awk '{print $1}' file           # Print first column
awk -F':' '{print $1}' file     # Custom delimiter
awk '/pattern/ {print}' file    # Print matching lines
awk '$3 > 100' file             # Condition on field
awk 'NR > 1' file               # Skip header line
awk '{sum+=$1} END {print sum}' # Sum column
```

### Other Text Tools
```bash
cut -d':' -f1 file              # Extract field 1
sort file                       # Sort lines
sort -n file                    # Numeric sort
sort -r file                    # Reverse sort
sort -u file                    # Unique sort
uniq                            # Remove adjacent duplicates
uniq -c                         # Count occurrences
wc -l file                      # Count lines
wc -w file                      # Count words
tr 'a-z' 'A-Z'                  # Translate to uppercase
diff file1 file2                # Compare files
```

---

## User & Permissions

### User Management
```bash
useradd username                # Create user
useradd -m -s /bin/bash user    # With home dir and shell
userdel -r username             # Delete user and home
usermod -aG group user          # Add user to group
passwd username                 # Set password
id username                     # Show user/group IDs
whoami                          # Current user
su - username                   # Switch user
sudo command                    # Run as root
```

### Group Management
```bash
groupadd groupname              # Create group
groupdel groupname              # Delete group
groups username                 # List user's groups
```

### Permissions
```bash
chmod 755 file                  # Set numeric permissions
chmod u+x file                  # Add execute for owner
chmod -R 644 dir                # Recursive
chown user:group file           # Change owner and group
chown -R user dir               # Recursive ownership

# Permission values: r=4, w=2, x=1
# 755 = rwxr-xr-x
# 644 = rw-r--r--
```

### Special Permissions
```bash
chmod u+s file                  # SUID (run as owner)
chmod g+s dir                   # SGID (inherit group)
chmod +t dir                    # Sticky bit
```

### ACLs
```bash
getfacl file                    # View ACLs
setfacl -m u:user:rwx file      # Set user ACL
setfacl -m g:group:rx file      # Set group ACL
setfacl -x u:user file          # Remove ACL
setfacl -b file                 # Remove all ACLs
```

---

## Process Management

### Viewing Processes
```bash
ps aux                          # All processes
ps -ef                          # Full format
ps aux | grep name              # Find process
pgrep -a processname            # Find by name
pstree                          # Process tree
top                             # Real-time monitor
htop                            # Better monitor (if installed)
```

### Managing Processes
```bash
kill PID                        # Terminate process
kill -9 PID                     # Force kill
killall processname             # Kill by name
pkill processname               # Kill by pattern

# Background/Foreground
command &                       # Run in background
jobs                            # List background jobs
fg %1                           # Bring job 1 to foreground
bg %1                           # Continue job in background
Ctrl+Z                          # Suspend current process
nohup command &                 # Run immune to hangup
```

### Priority
```bash
nice -n 10 command              # Start with lower priority
renice -n 5 -p PID              # Change priority
```

---

## Networking

### IP Configuration
```bash
ip a                            # Show IP addresses
ip r                            # Show routing table
ip link set eth0 up/down        # Enable/disable interface
nmcli con show                  # Show connections
nmcli con up "name"             # Activate connection
```

### Connectivity Testing
```bash
ping host                       # Test connectivity
ping -c 4 host                  # Send 4 packets
traceroute host                 # Trace route
mtr host                        # Continuous traceroute
```

### DNS
```bash
nslookup domain                 # DNS lookup
dig domain                      # Detailed DNS lookup
host domain                     # Simple lookup
cat /etc/resolv.conf            # View DNS servers
```

### Ports & Connections
```bash
ss -tuln                        # Listening ports (TCP/UDP)
ss -tunp                        # With process names
netstat -tuln                   # Legacy (if available)
lsof -i :80                     # What's using port 80
```

### Firewall (firewalld)
```bash
firewall-cmd --state                          # Check status
firewall-cmd --list-all                       # List rules
firewall-cmd --add-service=http --permanent   # Allow HTTP
firewall-cmd --add-port=8080/tcp --permanent  # Allow port
firewall-cmd --reload                         # Apply changes
```

---

## Package Management

### DNF (RHEL 8+)
```bash
dnf search package              # Search packages
dnf info package                # Package info
dnf install package             # Install
dnf remove package              # Remove
dnf update                      # Update all
dnf update package              # Update specific
dnf list installed              # List installed
dnf provides /path/file         # What provides file
dnf history                     # Transaction history
dnf clean all                   # Clear cache
```

### RPM
```bash
rpm -ivh package.rpm            # Install
rpm -Uvh package.rpm            # Upgrade
rpm -e package                  # Remove
rpm -qa                         # List all installed
rpm -qi package                 # Package info
rpm -ql package                 # List files in package
rpm -qf /path/file              # What package owns file
```

### Repositories
```bash
dnf repolist                    # List enabled repos
dnf repolist all                # List all repos
dnf config-manager --enable repo    # Enable repo
dnf config-manager --disable repo   # Disable repo
```

---

## Storage & Disks

### Disk Information
```bash
lsblk                           # List block devices
lsblk -f                        # With filesystem info
fdisk -l                        # List partitions
df -h                           # Disk space usage
du -sh dir                      # Directory size
du -h --max-depth=1             # Sizes of subdirectories
```

### Mounting
```bash
mount /dev/sdb1 /mnt            # Mount device
umount /mnt                     # Unmount
mount -a                        # Mount all in fstab
cat /etc/fstab                  # View persistent mounts
blkid                           # Show UUIDs
```

### Filesystem
```bash
mkfs.xfs /dev/sdb1              # Create XFS filesystem
mkfs.ext4 /dev/sdb1             # Create ext4 filesystem
xfs_repair /dev/sdb1            # Repair XFS
fsck /dev/sdb1                  # Check/repair filesystem
```

### LVM
```bash
pvs                             # List physical volumes
vgs                             # List volume groups
lvs                             # List logical volumes

pvcreate /dev/sdb               # Create PV
vgcreate vgname /dev/sdb        # Create VG
lvcreate -L 10G -n lvname vg    # Create LV
lvextend -L +5G /dev/vg/lv      # Extend LV
xfs_growfs /mount               # Grow XFS filesystem
resize2fs /dev/vg/lv            # Grow ext4 filesystem
```

---

## systemd Services

```bash
systemctl status service        # Check status
systemctl start service         # Start service
systemctl stop service          # Stop service
systemctl restart service       # Restart service
systemctl reload service        # Reload config
systemctl enable service        # Enable at boot
systemctl disable service       # Disable at boot
systemctl is-active service     # Check if running
systemctl is-enabled service    # Check if enabled
systemctl list-units --type=service     # List services
systemctl list-unit-files               # List all units
systemctl daemon-reload         # Reload systemd configs
```

---

## Logs & Troubleshooting

### journalctl (systemd logs)
```bash
journalctl                      # All logs
journalctl -b                   # Current boot
journalctl -b -1                # Previous boot
journalctl -u service           # Specific service
journalctl -f                   # Follow (live)
journalctl --since "1 hour ago" # Time-based
journalctl -p err               # Error priority only
journalctl -xe                  # Last entries + explanation
```

### Log Files
```bash
/var/log/messages               # General system logs
/var/log/secure                 # Authentication logs
/var/log/boot.log               # Boot messages
/var/log/cron                   # Cron job logs
/var/log/dnf.log                # Package manager logs
```

### System Info
```bash
uname -a                        # Kernel info
hostnamectl                     # Hostname and OS
cat /etc/os-release             # OS version
uptime                          # System uptime
free -h                         # Memory usage
lscpu                           # CPU info
dmidecode                       # Hardware info
```

---

## SELinux

```bash
getenforce                      # Current mode
setenforce 0                    # Set permissive (temp)
setenforce 1                    # Set enforcing (temp)

# Permanent: edit /etc/selinux/config

ls -Z file                      # View context
ps auxZ                         # Process contexts
chcon -t type file              # Change context (temp)
restorecon -Rv /path            # Restore default context
semanage fcontext -l            # List contexts
semanage fcontext -a -t type "/path(/.*)?"   # Add rule
setsebool -P boolean on         # Set boolean permanently
getsebool -a                    # List all booleans
ausearch -m avc -ts recent      # Recent denials
sealert -a /var/log/audit/audit.log         # Analyze denials
```

---

## Compression & Archives

### tar (Tape Archive)
```bash
tar -cvf archive.tar dir        # Create tar
tar -xvf archive.tar            # Extract tar
tar -tvf archive.tar            # List contents
tar -czvf archive.tar.gz dir    # Create gzip compressed
tar -xzvf archive.tar.gz        # Extract gzip
tar -cjvf archive.tar.bz2 dir   # Create bzip2 compressed
tar -xjvf archive.tar.bz2       # Extract bzip2
```

### Other Compression
```bash
gzip file                       # Compress (replaces file)
gunzip file.gz                  # Decompress
bzip2 file                      # Better compression
bunzip2 file.bz2                # Decompress bzip2
xz file                         # Best compression
unxz file.xz                    # Decompress xz
zip archive.zip files           # Create zip
unzip archive.zip               # Extract zip
```

---

## SSH & Remote

### SSH Connection
```bash
ssh user@host                   # Connect
ssh -p 2222 user@host           # Custom port
ssh -i key.pem user@host        # With key file
ssh-keygen                      # Generate key pair
ssh-copy-id user@host           # Copy public key
```

### File Transfer
```bash
scp file user@host:/path        # Copy to remote
scp user@host:/path/file .      # Copy from remote
scp -r dir user@host:/path      # Copy directory
rsync -avz src/ user@host:dst/  # Sync with compression
rsync -avz --delete src/ dst/   # Mirror (delete extra)
```

---

## Useful Shortcuts

### Keyboard Shortcuts
| Shortcut | Action |
|----------|--------|
| `Ctrl+C` | Cancel/kill current command |
| `Ctrl+Z` | Suspend current process |
| `Ctrl+D` | Logout / End input |
| `Ctrl+L` | Clear screen |
| `Ctrl+A` | Move to beginning of line |
| `Ctrl+E` | Move to end of line |
| `Ctrl+U` | Delete to beginning |
| `Ctrl+K` | Delete to end |
| `Ctrl+R` | Search command history |
| `Tab` | Auto-complete |
| `Tab Tab` | Show all completions |

### History
```bash
history                         # Show command history
!n                              # Run command #n
!!                              # Repeat last command
!string                         # Run last command starting with string
Ctrl+R                          # Search history
```

### Redirects & Pipes
```bash
command > file                  # Redirect stdout (overwrite)
command >> file                 # Redirect stdout (append)
command 2> file                 # Redirect stderr
command &> file                 # Redirect both
command1 | command2             # Pipe output
command | tee file              # Output + save to file
```

---

**[← Back to Index](README.md)**
