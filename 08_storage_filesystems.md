# Linux Crash Course - Storage & Filesystems

## Table of Contents
- [Storage Concepts](#storage-concepts)
- [Viewing Storage Devices](#viewing-storage-devices)
- [Partitioning (fdisk, gdisk, parted)](#partitioning-fdisk-gdisk-parted)
- [Filesystems](#filesystems)
- [Mounting Filesystems](#mounting-filesystems)
- [LVM - Logical Volume Manager](#lvm---logical-volume-manager)
- [Swap Space](#swap-space)
- [Disk Quotas](#disk-quotas)
- [RAID Overview](#raid-overview)
- [Storage Troubleshooting](#storage-troubleshooting)

---

## Storage Concepts

### Storage Hierarchy
```
┌─────────────────────────────────────────────────────────┐
│                    Storage Stack                         │
├─────────────────────────────────────────────────────────┤
│                                                          │
│   ┌──────────────────────────────────────────────────┐  │
│   │              Filesystem (ext4, xfs)               │  │
│   │         - Files, directories, permissions         │  │
│   └───────────────────────┬──────────────────────────┘  │
│                           │                              │
│   ┌───────────────────────▼──────────────────────────┐  │
│   │     Logical Volumes (LVM) - Optional Layer        │  │
│   │         - Flexible volume management              │  │
│   └───────────────────────┬──────────────────────────┘  │
│                           │                              │
│   ┌───────────────────────▼──────────────────────────┐  │
│   │              Partitions                           │  │
│   │         - Divide physical disk                    │  │
│   └───────────────────────┬──────────────────────────┘  │
│                           │                              │
│   ┌───────────────────────▼──────────────────────────┐  │
│   │          Physical Storage (HDD, SSD, NVMe)        │  │
│   └──────────────────────────────────────────────────┘  │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### Partition Tables
| Type | Max Size | Max Partitions | Notes |
|------|----------|----------------|-------|
| MBR | 2 TB | 4 primary (or 3 primary + 1 extended) | Legacy, BIOS |
| GPT | 9.4 ZB | 128 | Modern, UEFI |

### Device Naming
```bash
# SATA/SAS/Virtual disks
/dev/sda              # First disk
/dev/sdb              # Second disk
/dev/sda1             # First partition on sda
/dev/sda2             # Second partition on sda

# NVMe disks
/dev/nvme0n1          # First NVMe disk
/dev/nvme0n1p1        # First partition
/dev/nvme0n1p2        # Second partition

# Virtual (virtio) disks
/dev/vda              # First virtio disk
/dev/vda1             # First partition

# LVM
/dev/mapper/vg_name-lv_name
/dev/vg_name/lv_name  # Same as above
```

---

## Viewing Storage Devices

### lsblk - List Block Devices
```bash
# Basic listing
lsblk

# With filesystem info
lsblk -f

# With size in bytes
lsblk -b

# All info
lsblk -a

# Output example:
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda           8:0    0   50G  0 disk 
├─sda1        8:1    0    1G  0 part /boot
└─sda2        8:2    0   49G  0 part 
  ├─rhel-root 253:0  0   45G  0 lvm  /
  └─rhel-swap 253:1  0    4G  0 lvm  [SWAP]
sdb           8:16   0  100G  0 disk 
└─sdb1        8:17   0  100G  0 part /data
```

### fdisk -l - List Partitions
```bash
# List all disks and partitions
sudo fdisk -l

# Specific disk
sudo fdisk -l /dev/sda
```

### blkid - Block Device Attributes
```bash
# Show all block devices with UUID, TYPE, LABEL
sudo blkid

# Specific device
sudo blkid /dev/sda1

# Output format:
/dev/sda1: UUID="abc123" TYPE="xfs" PARTUUID="xyz789"
```

### df - Disk Free Space
```bash
# Show mounted filesystems
df

# Human readable
df -h

# Include filesystem type
df -hT

# Specific filesystem
df -h /home

# Show inodes
df -i

# Output example:
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        45G   15G   30G  34% /
/dev/sda1       1.0G  200M  800M  20% /boot
```

### du - Disk Usage
```bash
# Directory size
du -sh /var/log

# Subdirectory breakdown
du -h --max-depth=1 /var

# Sort by size
du -sh /var/* | sort -h

# Largest directories
du -h --max-depth=1 / 2>/dev/null | sort -hr | head -20
```

---

## Partitioning (fdisk, gdisk, parted)

### fdisk - MBR Partitioning
```bash
# Start fdisk
sudo fdisk /dev/sdb

# fdisk commands (interactive):
m     # Help menu
p     # Print partition table
n     # New partition
d     # Delete partition
t     # Change partition type
w     # Write changes and exit
q     # Quit without saving
l     # List known partition types
```

#### fdisk Example Session
```bash
sudo fdisk /dev/sdb

Command (m for help): n                    # New partition
Partition type: p                          # Primary
Partition number: 1                        # First partition
First sector: [Enter for default]
Last sector: +10G                          # 10GB partition

Command (m for help): n                    # Another partition
Partition type: p
Partition number: 2
First sector: [Enter]
Last sector: [Enter for remaining space]

Command (m for help): p                    # Print to verify
Command (m for help): w                    # Write and exit

# Inform kernel of changes
sudo partprobe /dev/sdb
```

### gdisk - GPT Partitioning
```bash
# Start gdisk
sudo gdisk /dev/sdb

# gdisk commands (similar to fdisk):
?     # Help
p     # Print
n     # New partition
d     # Delete partition
t     # Change type
w     # Write
q     # Quit
i     # Information about partition
```

### parted - Both MBR and GPT
```bash
# Interactive mode
sudo parted /dev/sdb

# Common commands:
(parted) print                             # Show partitions
(parted) mklabel gpt                       # Create GPT table
(parted) mklabel msdos                     # Create MBR table
(parted) mkpart primary xfs 0% 50%         # Create partition
(parted) mkpart primary ext4 50% 100%      # Another partition
(parted) rm 1                              # Remove partition 1
(parted) quit                              # Exit

# Non-interactive (scripting)
sudo parted -s /dev/sdb mklabel gpt
sudo parted -s /dev/sdb mkpart primary xfs 0% 100%
```

### Partition Types
```bash
# Common partition type codes (fdisk/gdisk):
83    # Linux filesystem (fdisk)
8300  # Linux filesystem (gdisk)
82    # Linux swap (fdisk)
8200  # Linux swap (gdisk)
8e    # Linux LVM (fdisk)
8e00  # Linux LVM (gdisk)
fd    # Linux RAID (fdisk)
fd00  # Linux RAID (gdisk)
```

---

## Filesystems

### Filesystem Types
| Filesystem | Description | Max Size | Max File | Notes |
|------------|-------------|----------|----------|-------|
| XFS | Default RHEL | 8 EiB | 8 EiB | Can grow, not shrink |
| ext4 | Traditional Linux | 1 EiB | 16 TiB | Can grow and shrink |
| Btrfs | Next-gen | 16 EiB | 16 EiB | Snapshots, compression |
| vfat | FAT32 | 2 TiB | 4 GB | USB drives, EFI |

### Creating Filesystems

#### mkfs - Make Filesystem
```bash
# XFS (RHEL default)
sudo mkfs.xfs /dev/sdb1
sudo mkfs -t xfs /dev/sdb1

# ext4
sudo mkfs.ext4 /dev/sdb1
sudo mkfs -t ext4 /dev/sdb1

# With label
sudo mkfs.xfs -L "MyData" /dev/sdb1
sudo mkfs.ext4 -L "MyData" /dev/sdb1

# Force (overwrite existing)
sudo mkfs.xfs -f /dev/sdb1

# With specific block size
sudo mkfs.ext4 -b 4096 /dev/sdb1
```

### Filesystem Maintenance

#### XFS Tools
```bash
# Filesystem info
xfs_info /dev/sdb1
xfs_info /mount/point

# Check/repair (unmounted)
sudo xfs_repair /dev/sdb1
sudo xfs_repair -n /dev/sdb1              # Dry run

# Defragment
sudo xfs_fsr /mount/point

# Grow filesystem
sudo xfs_growfs /mount/point

# Freeze/unfreeze (for snapshots)
sudo xfs_freeze -f /mount/point
sudo xfs_freeze -u /mount/point
```

#### ext4 Tools
```bash
# Check filesystem (unmounted)
sudo fsck.ext4 /dev/sdb1
sudo e2fsck -f /dev/sdb1                  # Force check
sudo e2fsck -p /dev/sdb1                  # Auto-repair

# Filesystem info
sudo tune2fs -l /dev/sdb1
sudo dumpe2fs /dev/sdb1

# Set label
sudo tune2fs -L "NewLabel" /dev/sdb1

# Resize (can grow or shrink)
# Shrink requires unmount and fsck first
sudo resize2fs /dev/sdb1 20G              # Resize to 20G
sudo resize2fs /dev/sdb1                  # Expand to partition size
```

---

## Mounting Filesystems

### mount - Mount Filesystems
```bash
# Basic mount
sudo mount /dev/sdb1 /mnt
sudo mount /dev/sdb1 /mnt/data

# With filesystem type
sudo mount -t xfs /dev/sdb1 /mnt

# With options
sudo mount -o ro /dev/sdb1 /mnt           # Read-only
sudo mount -o rw,noexec /dev/sdb1 /mnt    # Read-write, no execute

# Mount by UUID (preferred)
sudo mount UUID="abc123-def456" /mnt

# Mount by label
sudo mount LABEL="MyData" /mnt

# Show mounted filesystems
mount
mount | grep sdb
cat /proc/mounts
findmnt
findmnt -t xfs                            # Only XFS
```

### Common Mount Options
| Option | Description |
|--------|-------------|
| `ro` | Read-only |
| `rw` | Read-write (default) |
| `noexec` | Don't allow execution |
| `nosuid` | Ignore setuid/setgid bits |
| `nodev` | Don't interpret device files |
| `noatime` | Don't update access time |
| `nodiratime` | Don't update directory access time |
| `defaults` | rw, suid, dev, exec, auto, nouser, async |
| `auto` | Mount with `mount -a` |
| `noauto` | Don't mount with `mount -a` |
| `user` | Allow user to mount |
| `nofail` | Don't fail boot if device missing |

### umount - Unmount Filesystems
```bash
# Unmount by mount point
sudo umount /mnt
sudo umount /mnt/data

# Unmount by device
sudo umount /dev/sdb1

# Force unmount
sudo umount -f /mnt

# Lazy unmount (detach and clean up when not busy)
sudo umount -l /mnt

# Check what's using the mount
lsof /mnt
fuser -m /mnt
fuser -km /mnt                            # Kill processes using it
```

### /etc/fstab - Persistent Mounts
```bash
# View fstab
cat /etc/fstab

# Format:
# device                mount_point  fstype  options         dump  fsck
UUID=abc123-def456     /data        xfs     defaults        0     2
/dev/sdb1              /backup      ext4    defaults,noatime 0    2
LABEL=MyData           /mnt/data    xfs     defaults,nofail 0     0

# Columns:
# 1. Device (UUID preferred)
# 2. Mount point
# 3. Filesystem type
# 4. Mount options
# 5. Dump (backup) - usually 0
# 6. fsck order (0=skip, 1=root, 2=other)
```

```bash
# Test fstab entry (doesn't require reboot)
sudo mount -a

# Verify
findmnt --verify

# Mount single fstab entry
sudo mount /data
```

### systemd Mount Units
```bash
# Alternative to fstab
# Create /etc/systemd/system/data.mount
```

```ini
[Unit]
Description=Data Partition

[Mount]
What=/dev/sdb1
Where=/data
Type=xfs
Options=defaults

[Install]
WantedBy=multi-user.target
```

```bash
# Enable and start
sudo systemctl daemon-reload
sudo systemctl enable --now data.mount
```

---

## LVM - Logical Volume Manager

### LVM Concepts
```
┌─────────────────────────────────────────────────────────┐
│                    LVM Architecture                      │
├─────────────────────────────────────────────────────────┤
│                                                          │
│   ┌──────────────────────────────────────────────────┐  │
│   │  Logical Volumes (LV)          /dev/vg_name/lv   │  │
│   │  ┌────────┐ ┌────────┐ ┌────────┐               │  │
│   │  │  root  │ │  home  │ │  data  │               │  │
│   │  │  20GB  │ │  50GB  │ │  30GB  │               │  │
│   │  └────────┘ └────────┘ └────────┘               │  │
│   └───────────────────────┬──────────────────────────┘  │
│                           │                              │
│   ┌───────────────────────▼──────────────────────────┐  │
│   │  Volume Group (VG)              vg_data (100GB)  │  │
│   │  - Pool of storage from one or more PVs         │  │
│   └───────────────────────┬──────────────────────────┘  │
│                           │                              │
│   ┌───────────────────────▼──────────────────────────┐  │
│   │  Physical Volumes (PV)                           │  │
│   │  ┌─────────────┐  ┌─────────────┐               │  │
│   │  │  /dev/sdb   │  │  /dev/sdc   │               │  │
│   │  │   50GB      │  │   50GB      │               │  │
│   │  └─────────────┘  └─────────────┘               │  │
│   └──────────────────────────────────────────────────┘  │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### LVM Commands Overview
```bash
# Physical Volumes
pvcreate    # Create PV
pvdisplay   # Display PV info
pvs         # Summary of PVs
pvscan      # Scan for PVs
pvremove    # Remove PV

# Volume Groups
vgcreate    # Create VG
vgdisplay   # Display VG info
vgs         # Summary of VGs
vgextend    # Add PV to VG
vgreduce    # Remove PV from VG
vgremove    # Remove VG

# Logical Volumes
lvcreate    # Create LV
lvdisplay   # Display LV info
lvs         # Summary of LVs
lvextend    # Extend LV
lvreduce    # Shrink LV
lvremove    # Remove LV
lvresize    # Resize LV
```

### Creating LVM Step by Step

#### 1. Create Physical Volumes
```bash
# Create PVs from partitions or whole disks
sudo pvcreate /dev/sdb
sudo pvcreate /dev/sdc
sudo pvcreate /dev/sdb1 /dev/sdc1         # From partitions

# View PVs
sudo pvs
sudo pvdisplay
```

#### 2. Create Volume Group
```bash
# Create VG from PVs
sudo vgcreate vg_data /dev/sdb /dev/sdc

# With specific PE size (default 4MB)
sudo vgcreate -s 16M vg_data /dev/sdb /dev/sdc

# View VGs
sudo vgs
sudo vgdisplay
sudo vgdisplay vg_data
```

#### 3. Create Logical Volumes
```bash
# Create LV with specific size
sudo lvcreate -L 20G -n lv_home vg_data

# Create LV with percentage of VG
sudo lvcreate -l 50%VG -n lv_data vg_data

# Use all remaining space
sudo lvcreate -l 100%FREE -n lv_backup vg_data

# View LVs
sudo lvs
sudo lvdisplay
sudo lvdisplay /dev/vg_data/lv_home
```

#### 4. Create Filesystem and Mount
```bash
# Create filesystem
sudo mkfs.xfs /dev/vg_data/lv_home
sudo mkfs.ext4 /dev/vg_data/lv_data

# Mount
sudo mkdir /home
sudo mount /dev/vg_data/lv_home /home

# Add to fstab
echo '/dev/vg_data/lv_home /home xfs defaults 0 2' | sudo tee -a /etc/fstab
```

### Extending LVM

#### Extend Volume Group (Add Disk)
```bash
# Add new disk as PV
sudo pvcreate /dev/sdd

# Add PV to VG
sudo vgextend vg_data /dev/sdd

# Verify
sudo vgs
```

#### Extend Logical Volume
```bash
# Extend by specific size
sudo lvextend -L +10G /dev/vg_data/lv_home

# Extend to specific size
sudo lvextend -L 50G /dev/vg_data/lv_home

# Extend by percentage
sudo lvextend -l +50%FREE /dev/vg_data/lv_home

# Extend and resize filesystem in one command
sudo lvextend -L +10G -r /dev/vg_data/lv_home

# Or resize filesystem separately
# XFS
sudo xfs_growfs /home

# ext4
sudo resize2fs /dev/vg_data/lv_home
```

### Reducing LVM (Careful!)

#### Reduce Logical Volume (ext4 only)
```bash
# XFS cannot be shrunk!
# ext4 can be shrunk:

# Unmount
sudo umount /data

# Check filesystem
sudo e2fsck -f /dev/vg_data/lv_data

# Resize filesystem first
sudo resize2fs /dev/vg_data/lv_data 20G

# Reduce LV
sudo lvreduce -L 20G /dev/vg_data/lv_data

# Or in one step
sudo lvreduce -L 20G -r /dev/vg_data/lv_data

# Remount
sudo mount /dev/vg_data/lv_data /data
```

#### Reduce Volume Group
```bash
# Move data off PV first
sudo pvmove /dev/sdc

# Remove PV from VG
sudo vgreduce vg_data /dev/sdc

# Remove PV label
sudo pvremove /dev/sdc
```

### LVM Snapshots
```bash
# Create snapshot
sudo lvcreate -L 5G -s -n lv_home_snap /dev/vg_data/lv_home

# Mount snapshot
sudo mount -o ro /dev/vg_data/lv_home_snap /mnt/snap

# Remove snapshot
sudo lvremove /dev/vg_data/lv_home_snap

# Restore from snapshot (DANGER: replaces original!)
sudo umount /home
sudo lvconvert --merge /dev/vg_data/lv_home_snap
```

---

## Swap Space

### View Swap
```bash
# Show swap usage
swapon --show
free -h
cat /proc/swaps
```

### Create Swap Partition
```bash
# Create partition (type 82/8200 for swap)
sudo fdisk /dev/sdb
# Create partition, change type to 82

# Format as swap
sudo mkswap /dev/sdb2

# Enable
sudo swapon /dev/sdb2

# Add to fstab
echo '/dev/sdb2 none swap defaults 0 0' | sudo tee -a /etc/fstab
```

### Create Swap File
```bash
# Create swap file
sudo dd if=/dev/zero of=/swapfile bs=1M count=2048
# Or faster:
sudo fallocate -l 2G /swapfile

# Set permissions
sudo chmod 600 /swapfile

# Format as swap
sudo mkswap /swapfile

# Enable
sudo swapon /swapfile

# Add to fstab
echo '/swapfile none swap defaults 0 0' | sudo tee -a /etc/fstab

# Verify
swapon --show
```

### Swap Priority
```bash
# Higher priority = used first
sudo swapon -p 10 /dev/sdb2

# In fstab
/dev/sdb2 none swap defaults,pri=10 0 0
```

### Swappiness
```bash
# View current (0-100, lower = less swap usage)
cat /proc/sys/vm/swappiness

# Temporary change
sudo sysctl vm.swappiness=10

# Permanent change
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

---

## Disk Quotas

### Enable Quotas
```bash
# Edit /etc/fstab to add quota options
/dev/sdb1 /data xfs defaults,usrquota,grpquota 0 2

# Remount
sudo mount -o remount /data

# Or for ext4, create quota files
sudo quotacheck -cugm /data
sudo quotaon /data
```

### XFS Quotas
```bash
# Enable in mount options (usrquota, grpquota, prjquota)

# Set user quota (soft limit, hard limit)
sudo xfs_quota -x -c 'limit bsoft=5g bhard=6g user1' /data

# Set group quota
sudo xfs_quota -x -c 'limit bsoft=10g bhard=12g -g group1' /data

# Report
sudo xfs_quota -x -c 'report -h' /data
sudo xfs_quota -x -c 'report -u' /data    # User quotas
sudo xfs_quota -x -c 'report -g' /data    # Group quotas

# Interactive
sudo xfs_quota -x /data
```

### ext4 Quotas
```bash
# Initialize quota database
sudo quotacheck -cugm /data

# Turn on quotas
sudo quotaon /data

# Set quota for user
sudo edquota user1
# Or
sudo setquota -u user1 5000000 6000000 0 0 /data
# Args: soft-blocks hard-blocks soft-inodes hard-inodes

# Set quota for group
sudo edquota -g group1
sudo setquota -g group1 10000000 12000000 0 0 /data

# Report
sudo repquota /data
sudo repquota -a                          # All filesystems

# Check user quota
quota -u user1
quota -g group1
```

---

## RAID Overview

### RAID Levels
| Level | Min Disks | Description | Fault Tolerance |
|-------|-----------|-------------|-----------------|
| RAID 0 | 2 | Striping | None |
| RAID 1 | 2 | Mirroring | 1 disk |
| RAID 5 | 3 | Striping + Parity | 1 disk |
| RAID 6 | 4 | Striping + Double Parity | 2 disks |
| RAID 10 | 4 | Mirror + Stripe | 1 per mirror |

### mdadm - Software RAID
```bash
# Install
sudo dnf install mdadm

# Create RAID 1
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

# Create RAID 5
sudo mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd

# View status
cat /proc/mdstat
sudo mdadm --detail /dev/md0

# Save configuration
sudo mdadm --detail --scan >> /etc/mdadm.conf

# Add spare disk
sudo mdadm --add /dev/md0 /dev/sde

# Remove failed disk
sudo mdadm --remove /dev/md0 /dev/sdc
sudo mdadm --add /dev/md0 /dev/sdf        # Add replacement

# Stop array
sudo mdadm --stop /dev/md0

# Assemble existing array
sudo mdadm --assemble /dev/md0 /dev/sdb /dev/sdc
```

---

## Storage Troubleshooting

### Common Issues

#### Check Disk Health
```bash
# SMART data (install smartmontools)
sudo dnf install smartmontools

sudo smartctl -a /dev/sda
sudo smartctl -H /dev/sda                 # Health status
sudo smartctl -t short /dev/sda           # Run test
```

#### Filesystem Check
```bash
# Check and repair (unmounted!)
# XFS
sudo xfs_repair /dev/sdb1

# ext4
sudo fsck.ext4 -f /dev/sdb1

# Force check on next boot
sudo touch /forcefsck
```

#### Disk I/O Monitoring
```bash
# iostat
iostat -x 2

# iotop (install iotop)
sudo iotop

# dstat
dstat -d
```

#### Find Large Files
```bash
# Find files over 100MB
sudo find / -type f -size +100M 2>/dev/null

# Disk usage by directory
sudo du -h --max-depth=1 / 2>/dev/null | sort -hr | head -20
```

#### Check Mount Issues
```bash
# View kernel messages
dmesg | grep -i "error\|fail\|mount"

# Journal
journalctl -xe

# Verify fstab
findmnt --verify
```

---

## Quick Reference Card

### Storage Info
| Command | Description |
|---------|-------------|
| `lsblk` | List block devices |
| `lsblk -f` | With filesystem info |
| `blkid` | Show UUIDs and types |
| `df -h` | Disk space usage |
| `du -sh DIR` | Directory size |
| `fdisk -l` | List partitions |

### Partitioning
| Command | Description |
|---------|-------------|
| `fdisk /dev/sdX` | MBR partitioning |
| `gdisk /dev/sdX` | GPT partitioning |
| `parted /dev/sdX` | Both MBR/GPT |
| `partprobe` | Reload partition table |

### Filesystems
| Command | Description |
|---------|-------------|
| `mkfs.xfs /dev/sdX1` | Create XFS |
| `mkfs.ext4 /dev/sdX1` | Create ext4 |
| `xfs_repair /dev/sdX1` | Check XFS |
| `fsck.ext4 /dev/sdX1` | Check ext4 |
| `xfs_growfs /mount` | Grow XFS |
| `resize2fs /dev/sdX1` | Resize ext4 |

### Mounting
| Command | Description |
|---------|-------------|
| `mount /dev/sdX1 /mnt` | Mount |
| `umount /mnt` | Unmount |
| `mount -a` | Mount all in fstab |
| `findmnt` | Show mounts |

### LVM
| Command | Description |
|---------|-------------|
| `pvcreate /dev/sdX` | Create PV |
| `vgcreate VG PV` | Create VG |
| `lvcreate -L SIZE -n LV VG` | Create LV |
| `lvextend -L +SIZE -r LV` | Extend LV |
| `pvs` / `vgs` / `lvs` | List PV/VG/LV |

### Swap
| Command | Description |
|---------|-------------|
| `swapon --show` | Show swap |
| `mkswap /dev/sdX` | Create swap |
| `swapon /dev/sdX` | Enable swap |
| `swapoff /dev/sdX` | Disable swap |

---

**[← Back to Index](README.md)**  
**Previous: [Package Management](07_package_management.md)**  
**Next: [Shell Scripting](09_shell_scripting.md)**
