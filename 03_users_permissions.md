# Linux Crash Course - Users & Permissions

## Table of Contents
- [Understanding Users and Groups](#understanding-users-and-groups)
- [User Management Commands](#user-management-commands)
- [Group Management Commands](#group-management-commands)
- [Password Management](#password-management)
- [Understanding Permissions](#understanding-permissions)
- [Changing Permissions (chmod)](#changing-permissions-chmod)
- [Changing Ownership (chown, chgrp)](#changing-ownership-chown-chgrp)
- [Special Permissions](#special-permissions)
- [Default Permissions (umask)](#default-permissions-umask)
- [Access Control Lists (ACLs)](#access-control-lists-acls)
- [Sudo and Root Access](#sudo-and-root-access)

---

## Understanding Users and Groups

### User Types in Linux
```
┌─────────────────────────────────────────────────────────┐
│                    Linux Users                           │
├─────────────────────────────────────────────────────────┤
│  Root User (UID 0)                                       │
│  • Superuser with unlimited privileges                   │
│  • Can access all files and run all commands            │
│  • Home directory: /root                                 │
├─────────────────────────────────────────────────────────┤
│  System Users (UID 1-999 in RHEL)                       │
│  • Service accounts for daemons/services                │
│  • Usually cannot login interactively                   │
│  • Examples: apache, nginx, mysql, nobody               │
├─────────────────────────────────────────────────────────┤
│  Regular Users (UID 1000+)                              │
│  • Normal user accounts                                  │
│  • Limited privileges                                    │
│  • Home directory: /home/username                       │
└─────────────────────────────────────────────────────────┘
```

### Important User Files
| File | Description |
|------|-------------|
| `/etc/passwd` | User account information |
| `/etc/shadow` | Encrypted passwords and aging info |
| `/etc/group` | Group definitions |
| `/etc/gshadow` | Group passwords (rarely used) |
| `/etc/login.defs` | Default settings for user creation |
| `/etc/skel/` | Skeleton directory for new users |

### Understanding /etc/passwd
```bash
cat /etc/passwd

# Format:
# username:x:UID:GID:comment:home_directory:shell

# Example:
john:x:1001:1001:John Doe:/home/john:/bin/bash
│    │ │    │    │         │          │
│    │ │    │    │         │          └── Login shell
│    │ │    │    │         └── Home directory
│    │ │    │    └── Comment/Full name (GECOS)
│    │ │    └── Primary GID
│    │ └── UID
│    └── Password placeholder (actual in /etc/shadow)
└── Username
```

### Understanding /etc/shadow
```bash
sudo cat /etc/shadow

# Format:
# username:password:lastchange:min:max:warn:inactive:expire:reserved

# Example:
john:$6$xyz...abc:19384:0:99999:7:::
│    │            │     │ │     │
│    │            │     │ │     └── Days before warning
│    │            │     │ └── Maximum days between changes
│    │            │     └── Minimum days between changes
│    │            └── Days since Jan 1, 1970 of last change
│    └── Encrypted password ($6$ = SHA-512)
└── Username

# Password field special values:
# !! or !  = Account locked/no password set
# *        = Account disabled
# Empty    = No password required (dangerous!)
```

---

## User Management Commands

### useradd - Create New User
```bash
# Basic user creation
sudo useradd username

# Common options
sudo useradd -m username              # Create home directory
sudo useradd -m -s /bin/bash username # Specify shell
sudo useradd -u 1500 username         # Specify UID
sudo useradd -g groupname username    # Specify primary group
sudo useradd -G group1,group2 user    # Add to supplementary groups
sudo useradd -c "Full Name" username  # Add comment
sudo useradd -d /custom/home username # Custom home directory
sudo useradd -e 2026-12-31 username   # Account expiration date
sudo useradd -r serviceuser           # Create system user

# Full example
sudo useradd -m -s /bin/bash -c "John Doe" -G wheel,developers john

# View defaults
useradd -D
```

### usermod - Modify Existing User
```bash
# Change username
sudo usermod -l newname oldname

# Change home directory
sudo usermod -d /new/home -m username  # -m moves files

# Change shell
sudo usermod -s /bin/zsh username

# Change primary group
sudo usermod -g newgroup username

# Add to supplementary groups (APPEND)
sudo usermod -aG wheel username        # Add to wheel (sudo access)
sudo usermod -aG docker,libvirt user   # Add to multiple groups

# Replace supplementary groups (CAUTION!)
sudo usermod -G group1,group2 username # Replaces all groups!

# Lock account
sudo usermod -L username

# Unlock account
sudo usermod -U username

# Set expiration date
sudo usermod -e 2026-12-31 username

# Remove expiration
sudo usermod -e "" username

# Change UID
sudo usermod -u 2000 username
```

### userdel - Delete User
```bash
# Delete user (keeps home directory)
sudo userdel username

# Delete user AND home directory
sudo userdel -r username

# Force delete (even if logged in)
sudo userdel -f username

# Best practice: check for processes first
ps -u username
```

### id - Display User Information
```bash
# Current user info
id
# Output: uid=1000(john) gid=1000(john) groups=1000(john),10(wheel)

# Specific user
id username

# UID only
id -u username

# Primary GID only
id -g username

# All group names
id -Gn username

# All group IDs
id -G username
```

### whoami / who / w
```bash
# Current username
whoami
# Output: john

# Who is logged in
who
# Output: john  pts/0  2026-01-28 10:30 (192.168.1.100)

# Detailed login info
w
# Output shows: user, tty, from, login time, idle, what command

# Last logins
last
last username
last -n 10              # Last 10 entries

# Failed login attempts
sudo lastb
```

---

## Group Management Commands

### groupadd - Create Group
```bash
# Create group
sudo groupadd groupname

# Specify GID
sudo groupadd -g 2000 groupname

# Create system group
sudo groupadd -r systemgroup
```

### groupmod - Modify Group
```bash
# Rename group
sudo groupmod -n newname oldname

# Change GID
sudo groupmod -g 3000 groupname
```

### groupdel - Delete Group
```bash
# Delete group
sudo groupdel groupname

# Note: Cannot delete a group that is any user's primary group
```

### gpasswd - Group Administration
```bash
# Add user to group
sudo gpasswd -a username groupname

# Remove user from group
sudo gpasswd -d username groupname

# Set group administrators
sudo gpasswd -A admin1,admin2 groupname

# Set group members (replaces all)
sudo gpasswd -M user1,user2,user3 groupname
```

### groups - Show Group Membership
```bash
# Current user's groups
groups
# Output: john wheel developers

# Specific user's groups
groups username
```

### newgrp - Change Primary Group Temporarily
```bash
# Switch primary group for current session
newgrp developers

# Useful when creating files that need specific group ownership
```

---

## Password Management

### passwd - Change Password
```bash
# Change own password
passwd

# Change another user's password (root only)
sudo passwd username

# Lock account
sudo passwd -l username

# Unlock account
sudo passwd -u username

# Delete password (make empty - DANGEROUS!)
sudo passwd -d username

# Force password change on next login
sudo passwd -e username

# Set password status
passwd -S username
# Output: john PS 2026-01-28 0 99999 7 -1 (Password set, SHA512)
```

### chage - Password Aging
```bash
# View password aging info
sudo chage -l username

# Set password expiration (days since password change)
sudo chage -M 90 username            # Max 90 days

# Set minimum days between changes
sudo chage -m 7 username             # Min 7 days

# Set warning days before expiration
sudo chage -W 14 username            # Warn 14 days before

# Set account expiration date
sudo chage -E 2026-12-31 username

# Force password change on next login
sudo chage -d 0 username

# Set inactive days after password expires
sudo chage -I 30 username            # Lock after 30 days inactive

# Interactive mode
sudo chage username
```

---

## Understanding Permissions

### Permission Structure
```
-rwxr-xr-- 1 owner group size date filename
│└┬┘└┬┘└┬┘
│ │  │  └── Others (world) permissions
│ │  └── Group permissions
│ └── Owner (user) permissions
└── File type

Permission characters:
r = read    (4)
w = write   (2)
x = execute (1)
- = no permission (0)
```

### Permission Meanings
| Permission | On Files | On Directories |
|------------|----------|----------------|
| `r` (read) | View file contents | List directory contents (`ls`) |
| `w` (write) | Modify file contents | Create/delete files in directory |
| `x` (execute) | Run as program | Enter directory (`cd`) |

### Numeric (Octal) Permissions
```
r = 4
w = 2
x = 1

Calculate by adding:
rwx = 4+2+1 = 7
rw- = 4+2+0 = 6
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4
--- = 0+0+0 = 0

Common permissions:
777 = rwxrwxrwx (full access - DANGEROUS!)
755 = rwxr-xr-x (owner full, others read/execute)
700 = rwx------ (owner only)
644 = rw-r--r-- (owner read/write, others read)
600 = rw------- (owner read/write only)
```

---

## Changing Permissions (chmod)

### Symbolic Mode
```bash
# Syntax: chmod [who][operator][permission] file

# Who:
# u = user (owner)
# g = group
# o = others
# a = all (ugo)

# Operators:
# + = add permission
# - = remove permission
# = = set exact permission

# Examples
chmod u+x script.sh           # Add execute for owner
chmod g+w file.txt            # Add write for group
chmod o-r file.txt            # Remove read from others
chmod a+r file.txt            # Add read for all
chmod ug+rw file.txt          # Add read/write for user and group
chmod u=rwx,g=rx,o=r file.txt # Set exact permissions

# Recursive (directories)
chmod -R u+w directory/

# Common examples
chmod u+x script.sh           # Make script executable
chmod go-rwx private.txt      # Remove all for group/others
chmod a=r public.txt          # Read-only for everyone
```

### Numeric (Octal) Mode
```bash
# Set exact permissions using numbers
chmod 755 script.sh           # rwxr-xr-x
chmod 644 file.txt            # rw-r--r--
chmod 600 secret.txt          # rw-------
chmod 700 private_dir/        # rwx------
chmod 777 public/             # rwxrwxrwx (avoid!)

# Recursive
chmod -R 755 directory/

# Common permission sets
chmod 755 script.sh           # Executable scripts
chmod 644 document.txt        # Regular files
chmod 600 ~/.ssh/id_rsa       # SSH private key
chmod 700 ~/.ssh/             # SSH directory
chmod 640 /etc/passwd         # System files
```

### Reference Another File
```bash
# Copy permissions from another file
chmod --reference=source.txt target.txt
```

---

## Changing Ownership (chown, chgrp)

### chown - Change Owner and Group
```bash
# Change owner only
sudo chown newowner file.txt

# Change owner and group
sudo chown owner:group file.txt
sudo chown owner.group file.txt    # Alternative syntax

# Change group only (with colon)
sudo chown :newgroup file.txt

# Recursive
sudo chown -R owner:group directory/

# Don't follow symlinks
sudo chown -h owner:group symlink

# Only change if current owner matches
sudo chown --from=oldowner newowner file.txt

# Common examples
sudo chown root:root /etc/important.conf
sudo chown -R john:developers /var/www/project/
sudo chown apache:apache /var/www/html/
```

### chgrp - Change Group Only
```bash
# Change group
sudo chgrp groupname file.txt

# Recursive
sudo chgrp -R developers project/

# Note: chown :group achieves the same thing
```

---

## Special Permissions

### SUID (Set User ID)
```bash
# When set on executable: runs as file owner, not executor
# Numeric: 4000
# Symbol: s in user execute position

# View SUID files
ls -l /usr/bin/passwd
# -rwsr-xr-x 1 root root ... /usr/bin/passwd

# Set SUID
chmod u+s program
chmod 4755 program

# Remove SUID
chmod u-s program

# Find SUID files on system
sudo find / -perm -4000 -type f 2>/dev/null
```

### SGID (Set Group ID)
```bash
# On executable: runs as file group
# On directory: new files inherit directory's group
# Numeric: 2000
# Symbol: s in group execute position

# Set SGID on directory (common for shared folders)
chmod g+s shared_dir/
chmod 2775 shared_dir/

# New files in shared_dir will inherit the group

# Find SGID files
sudo find / -perm -2000 -type f 2>/dev/null
```

### Sticky Bit
```bash
# On directory: only file owner can delete their files
# Commonly used on /tmp
# Numeric: 1000
# Symbol: t in others execute position

ls -ld /tmp
# drwxrwxrwt 10 root root ... /tmp

# Set sticky bit
chmod +t directory/
chmod 1777 directory/

# Remove sticky bit
chmod -t directory/
```

### Special Permission Summary
```
┌─────────────┬────────┬─────────────────────────────────────┐
│ Permission  │ Octal  │ Effect                              │
├─────────────┼────────┼─────────────────────────────────────┤
│ SUID        │ 4000   │ Execute as file owner               │
│ SGID (file) │ 2000   │ Execute as file group               │
│ SGID (dir)  │ 2000   │ New files inherit directory group   │
│ Sticky      │ 1000   │ Only owner can delete files in dir  │
└─────────────┴────────┴─────────────────────────────────────┘

# Combined example
chmod 4755 file    # SUID + rwxr-xr-x
chmod 2775 dir     # SGID + rwxrwxr-x
chmod 1777 dir     # Sticky + rwxrwxrwx
chmod 6755 file    # SUID + SGID + rwxr-xr-x
```

---

## Default Permissions (umask)

### Understanding umask
```bash
# umask defines permissions REMOVED from new files/directories
# Default max permissions:
#   Files: 666 (rw-rw-rw-)
#   Directories: 777 (rwxrwxrwx)

# umask is subtracted:
# File permission = 666 - umask
# Directory permission = 777 - umask

# Common umask values:
# 022 → Files: 644, Dirs: 755 (default for most systems)
# 002 → Files: 664, Dirs: 775 (group write)
# 077 → Files: 600, Dirs: 700 (private)

# View current umask
umask
# Output: 0022

# View in symbolic form
umask -S
# Output: u=rwx,g=rx,o=rx
```

### Setting umask
```bash
# Set umask (current session)
umask 022
umask 077

# Permanent umask - add to:
# ~/.bashrc (user)
# /etc/profile or /etc/bashrc (system-wide)

# Example in ~/.bashrc
echo "umask 027" >> ~/.bashrc
```

### umask Calculation Table
| umask | File Result | Directory Result |
|-------|-------------|------------------|
| 000   | 666 (rw-rw-rw-) | 777 (rwxrwxrwx) |
| 022   | 644 (rw-r--r--) | 755 (rwxr-xr-x) |
| 027   | 640 (rw-r-----) | 750 (rwxr-x---) |
| 077   | 600 (rw-------) | 700 (rwx------) |
| 002   | 664 (rw-rw-r--) | 775 (rwxrwxr-x) |

---

## Access Control Lists (ACLs)

### Why ACLs?
Standard permissions only allow one user and one group. ACLs provide fine-grained access control for multiple users/groups.

### Check ACL Support
```bash
# Check if filesystem supports ACLs
mount | grep acl

# ACLs are usually enabled by default on ext4, xfs in RHEL
```

### getfacl - View ACLs
```bash
# View ACL
getfacl filename
getfacl directory/

# Output example:
# file: filename
# owner: john
# group: developers
# user::rw-
# user:jane:r--
# group::r--
# group:testers:rw-
# mask::rw-
# other::---
```

### setfacl - Set ACLs
```bash
# Add ACL for user
setfacl -m u:username:rwx file.txt

# Add ACL for group
setfacl -m g:groupname:rx file.txt

# Multiple entries
setfacl -m u:user1:rw,u:user2:r,g:group1:rw file.txt

# Remove specific ACL
setfacl -x u:username file.txt
setfacl -x g:groupname file.txt

# Remove all ACLs
setfacl -b file.txt

# Recursive
setfacl -R -m u:username:rx directory/

# Default ACL (for directories - inherited by new files)
setfacl -d -m u:username:rw directory/
setfacl -d -m g:groupname:rx directory/

# Copy ACL from one file to another
getfacl source.txt | setfacl --set-file=- target.txt
```

### ACL Mask
```bash
# The mask limits maximum permissions for named users/groups
# Effective permissions = ACL entry AND mask

# Set mask
setfacl -m m::rx file.txt

# Recalculate mask automatically
setfacl -m m:rwx file.txt
```

### ACL Examples
```bash
# Scenario: Shared project directory
# Owner: john, Group: developers
# Additional access: jane (read), testers group (read/write)

sudo mkdir /project
sudo chown john:developers /project
sudo chmod 770 /project

# Add ACLs
sudo setfacl -m u:jane:r-x /project
sudo setfacl -m g:testers:rwx /project

# Set defaults for new files
sudo setfacl -d -m u:jane:r-x /project
sudo setfacl -d -m g:testers:rwx /project

# Verify
getfacl /project
```

---

## Sudo and Root Access

### su - Switch User
```bash
# Switch to root (requires root password)
su
su -                    # Login shell (loads root's environment)

# Switch to another user
su username
su - username           # Login shell

# Run single command as another user
su -c "command" username
```

### sudo - Execute as Another User
```bash
# Run command as root
sudo command

# Run command as specific user
sudo -u username command

# Open root shell
sudo -i                 # Login shell
sudo -s                 # Non-login shell

# Edit file with sudo
sudo -e /etc/somefile   # Uses $EDITOR
sudoedit /etc/somefile  # Same as above

# List sudo privileges
sudo -l

# Validate/extend sudo timeout
sudo -v

# Invalidate sudo timestamp
sudo -k

# Run command in background
sudo -b command
```

### /etc/sudoers Configuration
```bash
# ALWAYS edit with visudo (validates syntax)
sudo visudo

# Basic syntax:
# user/group  host=(run_as_user:run_as_group)  commands

# Examples in sudoers:
root    ALL=(ALL)       ALL
# root can run any command as any user on any host

%wheel  ALL=(ALL)       ALL
# wheel group members can run anything with password

%wheel  ALL=(ALL)       NOPASSWD: ALL
# wheel group without password prompt

john    ALL=(ALL)       /usr/bin/systemctl, /usr/bin/dnf
# john can only run systemctl and dnf

jane    ALL=(ALL)       NOPASSWD: /usr/bin/systemctl restart httpd
# jane can restart httpd without password
```

### Sudoers Drop-in Directory
```bash
# Add custom rules in /etc/sudoers.d/ instead of editing main file
sudo visudo -f /etc/sudoers.d/developers

# Example content:
%developers ALL=(ALL) /usr/bin/docker, /usr/bin/podman
```

### Adding User to Sudo (wheel group in RHEL)
```bash
# Add user to wheel group
sudo usermod -aG wheel username

# Verify
groups username
sudo -l -U username
```

---

## Quick Reference Card

### User Commands
| Command | Description |
|---------|-------------|
| `useradd -m user` | Create user with home |
| `usermod -aG group user` | Add user to group |
| `userdel -r user` | Delete user and home |
| `passwd user` | Set/change password |
| `chage -l user` | View password aging |
| `id user` | Show UID/GID/groups |

### Group Commands
| Command | Description |
|---------|-------------|
| `groupadd group` | Create group |
| `groupmod -n new old` | Rename group |
| `groupdel group` | Delete group |
| `gpasswd -a user group` | Add to group |
| `gpasswd -d user group` | Remove from group |

### Permission Commands
| Command | Description |
|---------|-------------|
| `chmod 755 file` | Set permissions (octal) |
| `chmod u+x file` | Add execute for owner |
| `chmod -R g+w dir/` | Recursive group write |
| `chown user:group file` | Change owner/group |
| `chgrp group file` | Change group only |
| `umask 022` | Set default mask |

### ACL Commands
| Command | Description |
|---------|-------------|
| `getfacl file` | View ACLs |
| `setfacl -m u:user:rwx file` | Add user ACL |
| `setfacl -m g:group:rx file` | Add group ACL |
| `setfacl -x u:user file` | Remove user ACL |
| `setfacl -b file` | Remove all ACLs |
| `setfacl -d -m ...` | Set default ACL |

### Common Permissions
| Octal | Symbolic | Use Case |
|-------|----------|----------|
| 755 | rwxr-xr-x | Scripts, directories |
| 644 | rw-r--r-- | Regular files |
| 600 | rw------- | Private files |
| 700 | rwx------ | Private directories |
| 775 | rwxrwxr-x | Shared directories |

---

**[← Back to Index](README.md)**  
**Previous: [File Manipulation](02_file_manipulation.md)**  
**Next: [Text Processing](04_text_processing.md)**
