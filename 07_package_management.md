# Linux Crash Course - Package Management

## Table of Contents
- [Package Management Overview](#package-management-overview)
- [DNF Package Manager](#dnf-package-manager)
- [RPM Package Manager](#rpm-package-manager)
- [Repository Management](#repository-management)
- [Module Streams (RHEL 8+)](#module-streams-rhel-8)
- [Building from Source](#building-from-source)
- [Flatpak and Snap](#flatpak-and-snap)

---

## Package Management Overview

### Package Management in RHEL
```
┌─────────────────────────────────────────────────────────┐
│              RHEL Package Management                     │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │                    DNF                            │   │
│  │  High-level package manager                       │   │
│  │  Handles dependencies automatically               │   │
│  │  Works with repositories                          │   │
│  └───────────────────────┬──────────────────────────┘   │
│                          │                               │
│                          ▼                               │
│  ┌──────────────────────────────────────────────────┐   │
│  │                    RPM                            │   │
│  │  Low-level package manager                        │   │
│  │  Works with individual .rpm files                 │   │
│  │  No automatic dependency resolution               │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
└─────────────────────────────────────────────────────────┘

DNF = Dandified YUM (replaced yum in RHEL 8)
RPM = Red Hat Package Manager
```

### RPM Package Naming
```
httpd-2.4.53-7.el9.x86_64.rpm
│     │      │ │   │
│     │      │ │   └── Architecture (x86_64, noarch, i686)
│     │      │ └── Distribution (el9 = RHEL 9)
│     │      └── Release number
│     └── Version
└── Package name
```

---

## DNF Package Manager

### Basic DNF Commands

#### Searching Packages
```bash
# Search by name/description
dnf search httpd
dnf search "web server"

# Search in package name only
dnf search --name-only httpd

# Find which package provides a file
dnf provides /etc/httpd/conf/httpd.conf
dnf provides "*/httpd.conf"
dnf provides httpd

# List packages
dnf list                          # All available
dnf list installed                # Installed only
dnf list available                # Available (not installed)
dnf list updates                  # Available updates
dnf list httpd                    # Specific package
dnf list "http*"                  # Wildcards
```

#### Installing Packages
```bash
# Install package
sudo dnf install httpd
sudo dnf install httpd php mariadb       # Multiple

# Install from local RPM
sudo dnf install ./package.rpm

# Install from URL
sudo dnf install https://example.com/package.rpm

# Install specific version
sudo dnf install httpd-2.4.53

# Install without confirmation
sudo dnf install -y httpd

# Reinstall package
sudo dnf reinstall httpd

# Install only if not present
sudo dnf install --skip-installed httpd
```

#### Removing Packages
```bash
# Remove package
sudo dnf remove httpd

# Remove with dependencies (that become orphans)
sudo dnf remove --remove-unused httpd

# Remove multiple
sudo dnf remove httpd php mariadb

# Autoremove orphan dependencies
sudo dnf autoremove
```

#### Updating Packages
```bash
# Check for updates
dnf check-update

# Update specific package
sudo dnf update httpd

# Update all packages
sudo dnf update
sudo dnf upgrade                  # Same as update

# Update excluding packages
sudo dnf update --exclude=kernel*

# Security updates only
sudo dnf update --security

# Download without installing
sudo dnf download httpd
dnf download --resolve httpd      # With dependencies
```

#### Package Information
```bash
# Detailed package info
dnf info httpd

# Show dependencies
dnf deplist httpd
dnf repoquery --requires httpd
dnf repoquery --whatrequires httpd   # Reverse deps

# List files in package
dnf repoquery -l httpd
rpm -ql httpd                     # If installed

# Package changelog
dnf changelog httpd
```

### DNF Groups
```bash
# List available groups
dnf group list
dnf grouplist
dnf group list --hidden           # Include hidden

# Group info
dnf group info "Development Tools"

# Install group
sudo dnf group install "Development Tools"
sudo dnf groupinstall "Development Tools"
sudo dnf install @"Development Tools"      # Shorthand

# Remove group
sudo dnf group remove "Development Tools"
```

### DNF History
```bash
# View transaction history
dnf history
dnf history list
dnf history list 1-10

# Transaction details
dnf history info 15

# Undo transaction
sudo dnf history undo 15

# Redo transaction
sudo dnf history redo 15

# Rollback to transaction
sudo dnf history rollback 10
```

### DNF Cache
```bash
# Clean cache
sudo dnf clean all
sudo dnf clean packages           # Cached packages only
sudo dnf clean metadata           # Metadata only
sudo dnf clean dbcache            # DB cache only

# Rebuild cache
sudo dnf makecache
sudo dnf makecache --timer        # For timer use
```

### DNF Configuration
```bash
# Main config file
/etc/dnf/dnf.conf

# Common settings
cat /etc/dnf/dnf.conf
```

```ini
[main]
gpgcheck=1
installonly_limit=3
clean_requirements_on_remove=True
best=True
skip_if_unavailable=False
# Proxy settings
# proxy=http://proxy.example.com:8080
# proxy_username=user
# proxy_password=pass
```

---

## RPM Package Manager

### RPM Query Commands
```bash
# Query installed packages
rpm -qa                           # All installed
rpm -qa | grep httpd              # Filter
rpm -qa --last                    # By install date

# Query specific package
rpm -q httpd                      # Is it installed?
rpm -qi httpd                     # Information
rpm -ql httpd                     # List files
rpm -qc httpd                     # Config files only
rpm -qd httpd                     # Documentation only

# Query uninstalled package (.rpm file)
rpm -qip package.rpm              # Info
rpm -qlp package.rpm              # List files

# Find which package owns a file
rpm -qf /etc/httpd/conf/httpd.conf
rpm -qf $(which httpd)

# Verify package
rpm -V httpd                      # Check installed files
rpm -Va                           # Verify all packages

# Scripts in package
rpm -q --scripts httpd
```

### RPM Verification Output
```bash
rpm -V httpd

# Output format:
# SM5DLUGT c /path/to/file
# │││││││└─ Type: c=config, d=doc, g=ghost
# ││││││└── (reserved)
# │││││└─── Timestamp
# ││││└──── User ownership
# │││└───── Link path
# ││└────── Device
# │└─────── MD5 checksum
# └──────── Size
```

### RPM Install/Upgrade/Remove
```bash
# Install (rarely used directly - use dnf)
sudo rpm -ivh package.rpm         # Install verbose hash
sudo rpm -Uvh package.rpm         # Upgrade (or install if new)
sudo rpm -Fvh package.rpm         # Freshen (only if installed)

# Remove
sudo rpm -e httpd                 # Erase
sudo rpm -e --nodeps httpd        # Ignore dependencies

# Options:
# -i = install
# -U = upgrade
# -F = freshen
# -v = verbose
# -h = hash marks (progress)
# -e = erase
# --nodeps = no dependency check
# --force = force install
```

### RPM Signature Verification
```bash
# Import GPG key
sudo rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
sudo rpm --import https://www.redhat.com/security/team/key/

# Verify signature
rpm -K package.rpm
rpm --checksig package.rpm

# List imported keys
rpm -qa gpg-pubkey*
rpm -qi gpg-pubkey-xxx
```

### rpm2cpio - Extract RPM Contents
```bash
# Extract files from RPM without installing
rpm2cpio package.rpm | cpio -idmv

# List contents
rpm2cpio package.rpm | cpio -t

# Extract specific file
rpm2cpio package.rpm | cpio -ivd ./etc/httpd/conf/httpd.conf
```

---

## Repository Management

### Repository Configuration
```bash
# Repository config files
/etc/yum.repos.d/                 # Repository directory

# List enabled repos
dnf repolist
dnf repolist all                  # All (enabled + disabled)

# Repo info
dnf repoinfo
dnf repoinfo BaseOS
```

### Repository File Format
```bash
# Example: /etc/yum.repos.d/example.repo
cat /etc/yum.repos.d/redhat.repo
```

```ini
[rhel-9-for-x86_64-baseos-rpms]
name = Red Hat Enterprise Linux 9 for x86_64 - BaseOS (RPMs)
baseurl = https://cdn.redhat.com/content/dist/rhel9/$releasever/x86_64/baseos/os
enabled = 1
gpgcheck = 1
gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
sslverify = 1
sslcacert = /etc/rhsm/ca/redhat-uep.pem
sslclientkey = /etc/pki/entitlement/xxx-key.pem
sslclientcert = /etc/pki/entitlement/xxx.pem
```

### Managing Repositories
```bash
# Enable/disable repo
sudo dnf config-manager --enable repo-name
sudo dnf config-manager --disable repo-name

# Add repository
sudo dnf config-manager --add-repo https://example.com/repo.repo

# Use specific repo for command
sudo dnf install --repo=epel httpd

# Disable all repos, use specific one
sudo dnf install --disablerepo="*" --enablerepo=epel package
```

### EPEL Repository
```bash
# Extra Packages for Enterprise Linux
# Contains additional packages not in base RHEL

# Install EPEL (RHEL 9)
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm

# Or
sudo dnf install epel-release         # If available

# Verify
dnf repolist | grep epel
```

### Create Local Repository
```bash
# Install createrepo
sudo dnf install createrepo_c

# Create directory and add RPMs
mkdir -p /var/www/html/repo
cp *.rpm /var/www/html/repo/

# Create repository metadata
createrepo /var/www/html/repo/

# Update after adding packages
createrepo --update /var/www/html/repo/

# Create repo file on clients
cat > /etc/yum.repos.d/local.repo << EOF
[local-repo]
name=Local Repository
baseurl=http://server/repo/
enabled=1
gpgcheck=0
EOF
```

---

## Module Streams (RHEL 8+)

### Understanding Modules
Modules allow multiple versions of software to be available in the same repository.

```
Module = Collection of packages
Stream = Version of the module
Profile = Use case (client, server, default)
```

### Module Commands
```bash
# List modules
dnf module list
dnf module list php
dnf module list --installed
dnf module list --enabled

# Module info
dnf module info php
dnf module info php:8.1

# Enable stream
sudo dnf module enable php:8.1

# Install module (default profile)
sudo dnf module install php:8.1

# Install specific profile
sudo dnf module install php:8.1/devel

# Switch streams
sudo dnf module reset php
sudo dnf module enable php:8.2
sudo dnf module install php:8.2

# Disable module
sudo dnf module disable php

# Reset module (forget enabled stream)
sudo dnf module reset php
```

### Module Examples
```bash
# Install PHP 8.1
sudo dnf module enable php:8.1
sudo dnf module install php:8.1

# Install Node.js 18
sudo dnf module enable nodejs:18
sudo dnf module install nodejs:18

# Install PostgreSQL 15
sudo dnf module enable postgresql:15
sudo dnf module install postgresql:15/server
```

---

## Building from Source

### Development Tools
```bash
# Install development tools
sudo dnf group install "Development Tools"
sudo dnf install gcc gcc-c++ make

# Common build dependencies
sudo dnf install kernel-devel kernel-headers
sudo dnf install openssl-devel
sudo dnf install libcurl-devel
```

### Standard Build Process
```bash
# 1. Download source
wget https://example.com/software-1.0.tar.gz

# 2. Extract
tar -xzf software-1.0.tar.gz
cd software-1.0

# 3. Configure
./configure
./configure --prefix=/usr/local
./configure --help                # See options

# 4. Compile
make
make -j$(nproc)                   # Parallel build

# 5. Install
sudo make install

# 6. Optional cleanup
make clean
```

### Common Configure Options
```bash
--prefix=/usr/local               # Installation directory
--sysconfdir=/etc                 # Config files location
--enable-feature                  # Enable optional feature
--disable-feature                 # Disable feature
--with-library=/path              # Use specific library
--without-library                 # Build without library
```

### Using cmake
```bash
# Create build directory
mkdir build && cd build

# Configure
cmake ..
cmake -DCMAKE_INSTALL_PREFIX=/usr/local ..

# Build
make -j$(nproc)

# Install
sudo make install
```

---

## Flatpak and Snap

### Flatpak
```bash
# Install flatpak
sudo dnf install flatpak

# Add Flathub repository
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

# Search for apps
flatpak search firefox

# Install app
flatpak install flathub org.mozilla.firefox

# List installed
flatpak list

# Run app
flatpak run org.mozilla.firefox

# Update all
flatpak update

# Remove app
flatpak uninstall org.mozilla.firefox

# Remove unused
flatpak uninstall --unused
```

### Snap (if enabled)
```bash
# Install snapd
sudo dnf install snapd
sudo systemctl enable --now snapd.socket

# Create symlink
sudo ln -s /var/lib/snapd/snap /snap

# Search
snap find firefox

# Install
sudo snap install firefox

# List installed
snap list

# Update
sudo snap refresh

# Remove
sudo snap remove firefox

# Info
snap info firefox
```

---

## Quick Reference Card

### DNF Commands
| Command | Description |
|---------|-------------|
| `dnf search PKG` | Search packages |
| `dnf install PKG` | Install package |
| `dnf remove PKG` | Remove package |
| `dnf update` | Update all packages |
| `dnf update PKG` | Update specific package |
| `dnf info PKG` | Package information |
| `dnf list installed` | List installed |
| `dnf provides FILE` | Find package for file |
| `dnf history` | Transaction history |
| `dnf autoremove` | Remove orphans |

### DNF Groups
| Command | Description |
|---------|-------------|
| `dnf group list` | List groups |
| `dnf group install GROUP` | Install group |
| `dnf group remove GROUP` | Remove group |
| `dnf group info GROUP` | Group details |

### DNF Modules
| Command | Description |
|---------|-------------|
| `dnf module list` | List modules |
| `dnf module enable MOD:VER` | Enable stream |
| `dnf module install MOD` | Install module |
| `dnf module reset MOD` | Reset module |

### RPM Commands
| Command | Description |
|---------|-------------|
| `rpm -qa` | List all installed |
| `rpm -q PKG` | Query if installed |
| `rpm -qi PKG` | Package info |
| `rpm -ql PKG` | List files |
| `rpm -qf FILE` | Find owning package |
| `rpm -V PKG` | Verify package |
| `rpm -ivh FILE.rpm` | Install RPM |
| `rpm -e PKG` | Remove package |

### Repository Management
| Command | Description |
|---------|-------------|
| `dnf repolist` | List repos |
| `dnf config-manager --add-repo URL` | Add repo |
| `dnf config-manager --enable REPO` | Enable repo |
| `dnf config-manager --disable REPO` | Disable repo |
| `dnf clean all` | Clear cache |
| `dnf makecache` | Rebuild cache |

---

**Previous: [Networking](06_networking.md)**  
**Next: [Storage & Filesystems](08_storage_filesystems.md)**
