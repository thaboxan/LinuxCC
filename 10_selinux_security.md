# Linux Crash Course - SELinux & Security

## Table of Contents
- [Security Overview](#security-overview)
- [SELinux Fundamentals](#selinux-fundamentals)
- [SELinux Modes and Status](#selinux-modes-and-status)
- [SELinux Contexts](#selinux-contexts)
- [Managing SELinux Booleans](#managing-selinux-booleans)
- [SELinux Troubleshooting](#selinux-troubleshooting)
- [Firewall Security](#firewall-security)
- [User Security](#user-security)
- [SSH Security Hardening](#ssh-security-hardening)
- [File Security](#file-security)
- [Auditing and Logging](#auditing-and-logging)
- [Security Best Practices](#security-best-practices)

---

## Security Overview

### Linux Security Layers
```
┌─────────────────────────────────────────────────────────┐
│                 Linux Security Stack                     │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │  Application Security                            │    │
│  │  - Input validation, secure coding              │    │
│  └─────────────────────────────────────────────────┘    │
│                          │                               │
│  ┌─────────────────────────────────────────────────┐    │
│  │  SELinux / AppArmor (MAC)                       │    │
│  │  - Mandatory Access Control                      │    │
│  │  - Process confinement                          │    │
│  └─────────────────────────────────────────────────┘    │
│                          │                               │
│  ┌─────────────────────────────────────────────────┐    │
│  │  File Permissions (DAC)                          │    │
│  │  - Owner/Group/Other                             │    │
│  │  - rwx permissions                               │    │
│  └─────────────────────────────────────────────────┘    │
│                          │                               │
│  ┌─────────────────────────────────────────────────┐    │
│  │  Firewall (firewalld/iptables)                  │    │
│  │  - Network traffic filtering                     │    │
│  └─────────────────────────────────────────────────┘    │
│                          │                               │
│  ┌─────────────────────────────────────────────────┐    │
│  │  Kernel Security                                 │    │
│  │  - Kernel hardening, modules                    │    │
│  └─────────────────────────────────────────────────┘    │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### DAC vs MAC
| DAC (Discretionary) | MAC (Mandatory) |
|---------------------|-----------------|
| Owner controls access | System controls access |
| Traditional Unix permissions | SELinux policies |
| Users can change permissions | Policy-defined rules |
| chmod, chown | SELinux contexts |

---

## SELinux Fundamentals

### What is SELinux?
**Security-Enhanced Linux (SELinux)** is a Mandatory Access Control (MAC) system built into the Linux kernel. Developed by the NSA, it provides an additional layer of security beyond traditional file permissions.

### Key Concepts
```
┌─────────────────────────────────────────────────────────┐
│                 SELinux Concepts                         │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Subject (Process)                                       │
│  • Every process has an SELinux context                 │
│  • httpd_t, sshd_t, unconfined_t                       │
│                                                          │
│  Object (Resource)                                       │
│  • Files, ports, devices have contexts                  │
│  • httpd_sys_content_t, ssh_port_t                     │
│                                                          │
│  Policy                                                  │
│  • Rules defining what subjects can access              │
│  • Allow httpd_t to read httpd_sys_content_t           │
│                                                          │
│  Context = user:role:type:level                         │
│  • system_u:object_r:httpd_sys_content_t:s0            │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### SELinux Context Format
```
user:role:type:level

Example: system_u:object_r:httpd_sys_content_t:s0

user   = system_u (SELinux user)
role   = object_r (role for objects)
type   = httpd_sys_content_t (the most important part!)
level  = s0 (MLS/MCS level)

The TYPE is what SELinux primarily uses for access decisions.
```

---

## SELinux Modes and Status

### SELinux Modes
| Mode | Description |
|------|-------------|
| **Enforcing** | SELinux policy is enforced. Violations are blocked and logged. |
| **Permissive** | SELinux policy is not enforced but violations are logged. |
| **Disabled** | SELinux is completely disabled. |

### Check SELinux Status
```bash
# Current status
getenforce
# Output: Enforcing, Permissive, or Disabled

# Detailed status
sestatus
# Output:
# SELinux status:                 enabled
# SELinuxfs mount:                /sys/fs/selinux
# SELinux root directory:         /etc/selinux
# Loaded policy name:             targeted
# Current mode:                   enforcing
# Mode from config file:          enforcing
# Policy MLS status:              enabled
# Policy deny_unknown status:     allowed
# Memory protection checking:     actual (secure)
# Max kernel policy version:      33
```

### Change SELinux Mode

#### Temporary (until reboot)
```bash
# Set to permissive
sudo setenforce 0
sudo setenforce Permissive

# Set to enforcing
sudo setenforce 1
sudo setenforce Enforcing

# Note: Cannot switch to/from Disabled without reboot
```

#### Permanent
```bash
# Edit config file
sudo vim /etc/selinux/config

# Change SELINUX= line
SELINUX=enforcing      # Enable and enforce
SELINUX=permissive     # Enable but don't enforce
SELINUX=disabled       # Disable completely

# Reboot required for changes
sudo reboot

# IMPORTANT: If re-enabling after disabled,
# filesystem needs relabeling
sudo touch /.autorelabel
sudo reboot
```

### SELinux Policy Types
```bash
# View current policy
sestatus | grep "policy name"

# Common policies:
# targeted - Default, protects targeted processes
# minimum  - Subset of targeted
# mls      - Multi-Level Security (strict)

# Policy files location
ls /etc/selinux/
ls /etc/selinux/targeted/
```

---

## SELinux Contexts

### Viewing Contexts

#### File Contexts
```bash
# List files with context (-Z flag)
ls -Z /var/www/html/
# Output:
# system_u:object_r:httpd_sys_content_t:s0 index.html

# Detailed view
ls -laZ /var/www/html/
```

#### Process Contexts
```bash
# View process contexts
ps auxZ
ps -eZ

# Specific process
ps -Z -C httpd
# Output:
# system_u:system_r:httpd_t:s0 httpd
```

#### User Contexts
```bash
# Current user context
id -Z
# Output: unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

# SELinux user mappings
sudo semanage login -l
```

### Changing File Contexts

#### chcon - Temporary Context Change
```bash
# Change context (temporary - may be reset by restorecon)
sudo chcon -t httpd_sys_content_t /var/www/html/myfile.html

# Change recursively
sudo chcon -R -t httpd_sys_content_t /var/www/html/

# Copy context from another file
sudo chcon --reference=/var/www/html/index.html /var/www/html/newfile.html

# Change full context
sudo chcon -u system_u -r object_r -t httpd_sys_content_t file.html
```

#### restorecon - Restore Default Context
```bash
# Restore default context
sudo restorecon /var/www/html/myfile.html

# Recursive
sudo restorecon -R /var/www/html/

# Verbose
sudo restorecon -Rv /var/www/html/

# Show what would be changed (dry run)
sudo restorecon -Rvn /var/www/html/
```

#### semanage fcontext - Permanent Context Rules
```bash
# List file context rules
sudo semanage fcontext -l
sudo semanage fcontext -l | grep httpd

# Add new context rule (permanent)
sudo semanage fcontext -a -t httpd_sys_content_t "/web(/.*)?"

# Apply the rule
sudo restorecon -Rv /web/

# Modify existing rule
sudo semanage fcontext -m -t httpd_sys_rw_content_t "/web/uploads(/.*)?"

# Delete rule
sudo semanage fcontext -d "/web(/.*)?"

# Common types for web servers:
# httpd_sys_content_t     - Read-only web content
# httpd_sys_rw_content_t  - Read-write web content
# httpd_log_t             - Log files
# httpd_config_t          - Configuration files
```

### Port Contexts
```bash
# List port contexts
sudo semanage port -l
sudo semanage port -l | grep http

# Add port (allow httpd on port 8888)
sudo semanage port -a -t http_port_t -p tcp 8888

# Modify port type
sudo semanage port -m -t http_port_t -p tcp 8888

# Delete port rule
sudo semanage port -d -t http_port_t -p tcp 8888
```

---

## Managing SELinux Booleans

### What are Booleans?
Booleans are on/off switches that allow runtime modification of SELinux policy without changing policy files.

### Viewing Booleans
```bash
# List all booleans
getsebool -a

# Search for specific
getsebool -a | grep httpd
getsebool -a | grep ftp
getsebool -a | grep samba

# Check specific boolean
getsebool httpd_can_network_connect
# Output: httpd_can_network_connect --> off

# Detailed with description
sudo semanage boolean -l
sudo semanage boolean -l | grep httpd
```

### Setting Booleans

#### Temporary (until reboot)
```bash
# Enable boolean
sudo setsebool httpd_can_network_connect on

# Disable boolean
sudo setsebool httpd_can_network_connect off
```

#### Permanent
```bash
# Enable permanently (-P flag)
sudo setsebool -P httpd_can_network_connect on

# Disable permanently
sudo setsebool -P httpd_can_network_connect off
```

### Common Useful Booleans
```bash
# Web server (httpd)
httpd_can_network_connect        # Connect to network
httpd_can_network_connect_db     # Connect to database
httpd_enable_cgi                 # Execute CGI scripts
httpd_enable_homedirs            # Access user home dirs
httpd_read_user_content          # Read user content
httpd_use_nfs                    # Use NFS

# FTP (vsftpd)
ftpd_full_access                 # Full filesystem access
ftpd_anon_write                  # Anonymous write
ftpd_use_passive_mode            # Passive mode

# Samba
samba_enable_home_dirs           # Share home directories
samba_export_all_ro              # Export all read-only
samba_export_all_rw              # Export all read-write

# NFS
nfs_export_all_ro                # Export all read-only
nfs_export_all_rw                # Export all read-write

# General
allow_user_exec_content          # Execute in home
user_exec_content                # User execute scripts
```

---

## SELinux Troubleshooting

### Common SELinux Denials
```bash
# When SELinux blocks something:
# 1. Check audit log
# 2. Use audit2why
# 3. Apply appropriate fix
```

### Check Audit Logs
```bash
# View SELinux denials
sudo ausearch -m avc -ts recent
sudo ausearch -m avc -ts today

# Real-time monitoring
sudo tail -f /var/log/audit/audit.log | grep denied

# Using journalctl
sudo journalctl -t setroubleshoot
```

### audit2why - Understand Denials
```bash
# Analyze denial and suggest fix
sudo ausearch -m avc -ts recent | audit2why

# Common output types:
# - Missing boolean (setsebool)
# - Wrong file context (restorecon or semanage fcontext)
# - Need to create custom policy module
```

### audit2allow - Generate Policy
```bash
# Generate policy module from denials
sudo ausearch -m avc -ts recent | audit2allow -m mymodule > mymodule.te

# View what would be allowed
sudo ausearch -m avc -ts recent | audit2allow

# Create and install module
sudo ausearch -m avc -ts recent | audit2allow -M mymodule
sudo semodule -i mymodule.pp

# List installed modules
sudo semodule -l

# Remove module
sudo semodule -r mymodule
```

### setroubleshoot - GUI/CLI Helper
```bash
# Install
sudo dnf install setroubleshoot-server

# View alerts
sudo sealert -a /var/log/audit/audit.log

# Recent alerts
sudo sealert -l "*"

# Specific alert
sudo sealert -l <alert_id>
```

### Troubleshooting Workflow
```bash
# 1. Check if SELinux is causing the issue
sudo setenforce 0
# Test if problem goes away
sudo setenforce 1

# 2. Check the audit log
sudo ausearch -m avc -ts recent

# 3. Get suggestion
sudo ausearch -m avc -ts recent | audit2why

# 4. Apply fix (usually one of):
# a. Fix file context
sudo restorecon -Rv /path/to/files

# b. Set boolean
sudo setsebool -P <boolean> on

# c. Add port
sudo semanage port -a -t <type> -p tcp <port>

# d. Create custom policy (last resort)
sudo ausearch -m avc -ts recent | audit2allow -M myfix
sudo semodule -i myfix.pp
```

---

## Firewall Security

### firewalld Quick Security Commands
```bash
# Check status
sudo firewall-cmd --state
sudo firewall-cmd --list-all

# Lock down to minimum
sudo firewall-cmd --set-default-zone=drop

# Allow only specific services
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload

# Restrict SSH to specific IPs
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" service name="ssh" accept'
sudo firewall-cmd --permanent --remove-service=ssh
sudo firewall-cmd --reload

# Rate limiting
sudo firewall-cmd --permanent --add-rich-rule='rule service name="ssh" limit value="10/m" accept'

# Block specific IP
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.0.0.5" drop'

# Log dropped packets
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="0.0.0.0/0" log prefix="DROPPED: " level="info" drop'
```

### fail2ban - Intrusion Prevention
```bash
# Install
sudo dnf install epel-release
sudo dnf install fail2ban

# Enable and start
sudo systemctl enable --now fail2ban

# Configure
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo vim /etc/fail2ban/jail.local
```

```ini
# /etc/fail2ban/jail.local
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 5

[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/secure
maxretry = 3
bantime = 86400
```

```bash
# Restart
sudo systemctl restart fail2ban

# Check status
sudo fail2ban-client status
sudo fail2ban-client status sshd

# Unban IP
sudo fail2ban-client set sshd unbanip 192.168.1.100
```

---

## User Security

### Password Policies
```bash
# Password aging
sudo chage -M 90 username       # Max 90 days
sudo chage -m 7 username        # Min 7 days between changes
sudo chage -W 14 username       # Warn 14 days before expiry
sudo chage -E 2026-12-31 user   # Account expires

# View policy
sudo chage -l username

# Force password change
sudo chage -d 0 username

# Lock/unlock account
sudo usermod -L username        # Lock
sudo usermod -U username        # Unlock
sudo passwd -l username         # Lock
sudo passwd -u username         # Unlock
```

### Password Complexity (PAM)
```bash
# Edit PAM password requirements
sudo vim /etc/security/pwquality.conf
```

```ini
# /etc/security/pwquality.conf
minlen = 12
minclass = 3
maxrepeat = 2
maxclassrepeat = 4
dcredit = -1
ucredit = -1
lcredit = -1
ocredit = -1
difok = 3
```

### Account Lockout
```bash
# Edit PAM config
sudo vim /etc/pam.d/system-auth
sudo vim /etc/pam.d/password-auth

# Add after pam_env.so:
auth required pam_faillock.so preauth silent deny=5 unlock_time=900
auth required pam_faillock.so authfail deny=5 unlock_time=900

# Check failed attempts
sudo faillock --user username

# Reset lockout
sudo faillock --user username --reset
```

### Sudo Security
```bash
# Edit sudoers safely
sudo visudo

# Require password for sudo
Defaults timestamp_timeout=5    # 5 minute timeout
Defaults passwd_tries=3         # 3 attempts max

# Log all sudo commands
Defaults logfile="/var/log/sudo.log"

# Restrict sudo commands
username ALL=(ALL) /usr/bin/systemctl, /usr/bin/dnf

# No password for specific command
username ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart httpd
```

---

## SSH Security Hardening

### /etc/ssh/sshd_config Settings
```bash
sudo vim /etc/ssh/sshd_config
```

```bash
# Change default port
Port 2222

# Disable root login
PermitRootLogin no

# Use SSH key authentication only
PasswordAuthentication no
PubkeyAuthentication yes

# Disable empty passwords
PermitEmptyPasswords no

# Limit users/groups
AllowUsers admin deploy
AllowGroups sshusers

# Limit login attempts
MaxAuthTries 3

# Set idle timeout
ClientAliveInterval 300
ClientAliveCountMax 2

# Disable X11 forwarding if not needed
X11Forwarding no

# Use strong ciphers (RHEL 8+)
Ciphers aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr

# Use strong MACs
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com

# Use strong key exchange
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org

# Disable unused authentication methods
ChallengeResponseAuthentication no
KerberosAuthentication no
GSSAPIAuthentication no
```

```bash
# Validate configuration
sudo sshd -t

# Restart SSH
sudo systemctl restart sshd
```

### SSH Key Management
```bash
# Generate strong key
ssh-keygen -t ed25519 -C "user@host"
# Or RSA with 4096 bits
ssh-keygen -t rsa -b 4096 -C "user@host"

# Correct permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/config

# Restrict authorized_keys options
# In ~/.ssh/authorized_keys:
from="192.168.1.0/24",no-agent-forwarding,no-port-forwarding ssh-ed25519 AAAA...
```

---

## File Security

### Important Security Permissions
```bash
# Sensitive file permissions
chmod 600 /etc/shadow
chmod 600 ~/.ssh/id_*
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 644 /etc/passwd
chmod 644 /etc/group

# Find world-writable files
sudo find / -type f -perm -002 -ls 2>/dev/null

# Find world-writable directories
sudo find / -type d -perm -002 -ls 2>/dev/null

# Find SUID files
sudo find / -type f -perm -4000 -ls 2>/dev/null

# Find SGID files
sudo find / -type f -perm -2000 -ls 2>/dev/null

# Find files without owner
sudo find / -nouser -o -nogroup 2>/dev/null
```

### Immutable Files
```bash
# Make file immutable (even root can't modify)
sudo chattr +i /etc/passwd

# Remove immutable
sudo chattr -i /etc/passwd

# View attributes
lsattr /etc/passwd

# Append only
sudo chattr +a /var/log/secure
```

### File Integrity Monitoring (AIDE)
```bash
# Install AIDE
sudo dnf install aide

# Initialize database
sudo aide --init
sudo mv /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz

# Check for changes
sudo aide --check

# Update database after approved changes
sudo aide --update
sudo mv /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz
```

---

## Auditing and Logging

### auditd - System Auditing
```bash
# Status
sudo systemctl status auditd

# View audit rules
sudo auditctl -l

# Add file watch
sudo auditctl -w /etc/passwd -p wa -k passwd_changes
# -w = watch path
# -p = permissions (r=read, w=write, x=execute, a=attribute)
# -k = key for searching

# Add syscall audit
sudo auditctl -a always,exit -F arch=b64 -S execve -k commands

# Persistent rules (survives reboot)
sudo vim /etc/audit/rules.d/audit.rules
```

```bash
# /etc/audit/rules.d/audit.rules
-w /etc/passwd -p wa -k identity
-w /etc/group -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/sudoers -p wa -k sudo
-w /var/log/sudo.log -p wa -k sudo
-w /etc/ssh/sshd_config -p wa -k ssh
```

```bash
# Search audit logs
sudo ausearch -k passwd_changes
sudo ausearch -k passwd_changes -ts today
sudo ausearch -m USER_LOGIN -ts today

# Generate report
sudo aureport
sudo aureport -au              # Authentication
sudo aureport -l               # Logins
sudo aureport -f               # Files
sudo aureport -x               # Executables
```

### Important Log Files
```bash
/var/log/secure          # Authentication, sudo
/var/log/messages        # General system messages
/var/log/audit/audit.log # Audit daemon (SELinux, rules)
/var/log/boot.log        # Boot messages
/var/log/cron            # Cron jobs
/var/log/maillog         # Mail server

# View with journalctl
journalctl -u sshd           # SSH logs
journalctl -u httpd          # Apache logs
journalctl --since "1 hour ago"
journalctl -p err            # Errors only
journalctl -f                # Follow
```

---

## Security Best Practices

### System Hardening Checklist
```bash
# 1. Keep system updated
sudo dnf update -y

# 2. Enable SELinux
sudo setenforce 1
# Set SELINUX=enforcing in /etc/selinux/config

# 3. Configure firewall
sudo systemctl enable --now firewalld
sudo firewall-cmd --set-default-zone=drop
# Add only needed services

# 4. Disable unnecessary services
sudo systemctl disable --now telnet
sudo systemctl disable --now rsh
sudo systemctl disable --now rlogin

# 5. Secure SSH
# Edit /etc/ssh/sshd_config as shown above

# 6. Configure password policies
# Edit /etc/security/pwquality.conf

# 7. Enable auditing
sudo systemctl enable --now auditd

# 8. Regular security scans
sudo dnf install openscap-scanner scap-security-guide
sudo oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_standard \
    --results scan-results.xml \
    /usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml

# 9. Monitor logs
sudo journalctl -p err -b
sudo aureport --summary

# 10. Remove unnecessary packages
sudo dnf autoremove
```

### Quick Security Audit Commands
```bash
# Check for empty passwords
sudo awk -F: '($2 == "") {print $1}' /etc/shadow

# Check for UID 0 users (should only be root)
awk -F: '($3 == 0) {print $1}' /etc/passwd

# Find SUID/SGID files
sudo find / -type f \( -perm -4000 -o -perm -2000 \) -ls 2>/dev/null

# Check listening ports
sudo ss -tulpn

# Check failed logins
sudo lastb | head -20

# Check sudo usage
sudo grep sudo /var/log/secure | tail -20

# Check for rootkits (install rkhunter)
sudo dnf install rkhunter
sudo rkhunter --check
```

---

## Quick Reference Card

### SELinux Commands
| Command | Description |
|---------|-------------|
| `getenforce` | Show current mode |
| `setenforce 0/1` | Set permissive/enforcing |
| `sestatus` | Detailed status |
| `ls -Z` | Show file contexts |
| `ps -Z` | Show process contexts |
| `chcon -t TYPE FILE` | Change context (temp) |
| `restorecon -Rv PATH` | Restore default context |
| `semanage fcontext -a` | Add permanent context |
| `semanage port -a` | Add port context |
| `getsebool -a` | List booleans |
| `setsebool -P BOOL on` | Set boolean (permanent) |
| `ausearch -m avc` | Search denials |
| `audit2why` | Explain denial |
| `audit2allow` | Generate policy |

### File Contexts
| Type | Use |
|------|-----|
| `httpd_sys_content_t` | Web content (read) |
| `httpd_sys_rw_content_t` | Web content (read-write) |
| `ssh_home_t` | SSH keys |
| `user_home_t` | User home files |
| `var_log_t` | Log files |

### Security Commands
| Command | Description |
|---------|-------------|
| `passwd -l USER` | Lock account |
| `chage -l USER` | Show password aging |
| `faillock --user USER` | Show failed logins |
| `ausearch -k KEY` | Search audit logs |
| `aureport` | Audit report |
| `find / -perm -4000` | Find SUID files |
| `chattr +i FILE` | Make immutable |
| `lsattr FILE` | Show attributes |

---

**Previous: [Shell Scripting](09_shell_scripting.md)**  
**Next: [Boot Process & Troubleshooting](11_boot_troubleshooting.md)**
