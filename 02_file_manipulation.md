# Linux Crash Course - File Manipulation Commands

## Table of Contents
- [Navigation Commands](#navigation-commands)
- [Listing Files and Directories](#listing-files-and-directories)
- [Creating Files and Directories](#creating-files-and-directories)
- [Copying, Moving, and Renaming](#copying-moving-and-renaming)
- [Deleting Files and Directories](#deleting-files-and-directories)
- [Viewing File Contents](#viewing-file-contents)
- [File Information and Types](#file-information-and-types)
- [Finding Files](#finding-files)
- [Links (Hard and Symbolic)](#links-hard-and-symbolic)
- [File Compression and Archives](#file-compression-and-archives)

---

## Navigation Commands

### pwd - Print Working Directory
Displays the current directory path.

```bash
pwd
# Output: /home/username
```

### cd - Change Directory
Navigate between directories.

```bash
# Basic navigation
cd /path/to/directory      # Go to absolute path
cd directory               # Go to relative path
cd ..                      # Go up one level (parent directory)
cd ../..                   # Go up two levels
cd ~                       # Go to home directory
cd                         # Go to home directory (shortcut)
cd -                       # Go to previous directory
cd /                       # Go to root directory

# Examples
cd /etc                    # Go to /etc
cd /var/log                # Go to /var/log
cd ~/Documents             # Go to Documents in home
cd ../../usr/bin           # Relative path navigation
```

### Directory Shortcuts
| Shortcut | Meaning |
|----------|---------|
| `.` | Current directory |
| `..` | Parent directory |
| `~` | Home directory |
| `-` | Previous directory |
| `/` | Root directory |

---

## Listing Files and Directories

### ls - List Directory Contents
The most commonly used command for viewing files and directories.

```bash
# Basic usage
ls                         # List current directory
ls /path/to/dir            # List specific directory
ls file1 file2             # List specific files

# Common options
ls -l                      # Long format (detailed)
ls -a                      # Show all files (including hidden)
ls -la                     # Long format + hidden files
ls -lh                     # Human-readable sizes
ls -lt                     # Sort by modification time
ls -ltr                    # Sort by time (reverse/oldest first)
ls -lS                     # Sort by size (largest first)
ls -R                      # Recursive (include subdirectories)
ls -d */                   # List only directories
ls -1                      # One file per line
ls -i                      # Show inode numbers
ls --color=auto            # Colorized output (default in RHEL)
```

### Understanding ls -l Output
```
-rw-r--r--. 1 user group 4096 Jan 28 10:30 filename.txt
│├──┼──┼──│ │  │    │     │        │          │
││  │  │  │ │  │    │     │        │          └── Filename
││  │  │  │ │  │    │     │        └── Modification date/time
││  │  │  │ │  │    │     └── File size (bytes)
││  │  │  │ │  │    └── Group owner
││  │  │  │ │  └── User owner
││  │  │  │ └── Number of hard links
││  │  │  └── SELinux context indicator (. or +)
││  │  └── Others permissions (r--)
││  └── Group permissions (r--)
│└── User/Owner permissions (rw-)
└── File type (- = file, d = directory, l = link)
```

### File Type Indicators
| Character | Type |
|-----------|------|
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |
| `c` | Character device |
| `b` | Block device |
| `s` | Socket |
| `p` | Named pipe (FIFO) |

---

## Creating Files and Directories

### touch - Create Empty Files
```bash
# Create a single file
touch filename.txt

# Create multiple files
touch file1.txt file2.txt file3.txt

# Create files with pattern
touch file{1..5}.txt       # Creates file1.txt through file5.txt
touch {a,b,c}.log          # Creates a.log, b.log, c.log

# Update timestamp of existing file
touch existingfile.txt     # Updates access/modification time
touch -a filename.txt      # Update only access time
touch -m filename.txt      # Update only modification time

# Create file with specific timestamp
touch -t 202601281200 file.txt   # YYYYMMDDhhmm
```

### mkdir - Make Directory
```bash
# Create a single directory
mkdir dirname

# Create multiple directories
mkdir dir1 dir2 dir3

# Create nested directories (parent directories)
mkdir -p parent/child/grandchild

# Create with specific permissions
mkdir -m 755 dirname
mkdir -m 700 private_dir

# Create multiple nested structures
mkdir -p project/{src,bin,lib,docs}
mkdir -p ~/projects/{web,api}/{src,test,docs}

# Verbose output
mkdir -v newdir
# Output: mkdir: created directory 'newdir'
```

### Creating Files with Content
```bash
# Using echo
echo "Hello World" > file.txt           # Create/overwrite
echo "New line" >> file.txt             # Append

# Using cat
cat > file.txt << EOF
Line 1
Line 2
Line 3
EOF

# Using printf
printf "Line 1\nLine 2\n" > file.txt

# Copy content from another file
cat source.txt > destination.txt
```

---

## Copying, Moving, and Renaming

### cp - Copy Files and Directories
```bash
# Copy a file
cp source.txt destination.txt

# Copy to a directory
cp file.txt /path/to/directory/
cp file.txt /path/to/directory/newname.txt

# Copy multiple files to a directory
cp file1.txt file2.txt /destination/

# Common options
cp -r source_dir/ dest_dir/     # Recursive (for directories)
cp -i file.txt dest/            # Interactive (prompt before overwrite)
cp -f file.txt dest/            # Force (no prompt)
cp -v file.txt dest/            # Verbose
cp -p file.txt dest/            # Preserve permissions/timestamps
cp -a source/ dest/             # Archive (preserve everything, recursive)
cp -u source.txt dest/          # Update (only if source is newer)
cp -n file.txt dest/            # No clobber (don't overwrite)

# Copy with backup
cp -b file.txt dest/            # Create backup of destination
cp --backup=numbered file.txt dest/  # Numbered backups

# Examples
cp -rv /etc/nginx/ ~/backup/    # Backup nginx config
cp -a /home/user/ /backup/user/ # Full backup with permissions
```

### mv - Move or Rename Files
```bash
# Rename a file
mv oldname.txt newname.txt

# Move file to directory
mv file.txt /path/to/directory/

# Move and rename
mv file.txt /path/to/directory/newname.txt

# Move multiple files
mv file1.txt file2.txt /destination/

# Common options
mv -i file.txt dest/            # Interactive (prompt before overwrite)
mv -f file.txt dest/            # Force
mv -v file.txt dest/            # Verbose
mv -n file.txt dest/            # No clobber
mv -u file.txt dest/            # Update only if source is newer

# Move directories
mv source_dir/ /new/location/

# Rename directory
mv olddir/ newdir/

# Examples
mv *.log /var/log/archive/      # Move all log files
mv ~/Downloads/*.pdf ~/Documents/PDFs/
```

---

## Deleting Files and Directories

### rm - Remove Files
```bash
# Remove a file
rm filename.txt

# Remove multiple files
rm file1.txt file2.txt file3.txt

# Common options
rm -i file.txt                  # Interactive (confirm each deletion)
rm -f file.txt                  # Force (no prompts, ignore nonexistent)
rm -v file.txt                  # Verbose

# Remove directories
rm -r directory/                # Recursive (required for directories)
rm -rf directory/               # Force recursive (DANGEROUS!)

# Remove empty directory
rm -d emptydir/

# Using wildcards
rm *.tmp                        # Remove all .tmp files
rm -f *.log                     # Force remove all log files

# Safe practices
rm -i *                         # Confirm each file
rm -I *.txt                     # Prompt once before removing many files
```

> ⚠️ **WARNING**: `rm -rf /` or `rm -rf *` can destroy your entire system. ALWAYS double-check your path!

### rmdir - Remove Empty Directories
```bash
# Remove empty directory
rmdir emptydir

# Remove nested empty directories
rmdir -p parent/child/grandchild    # Removes all if empty

# Verbose
rmdir -v emptydir

# Note: rmdir only works on EMPTY directories
# Use rm -r for directories with contents
```

### Safe Deletion with Trash
```bash
# RHEL doesn't have built-in trash from CLI
# Install trash-cli (if available) or use:
mkdir -p ~/.trash
alias trash='mv --target-directory=$HOME/.trash'

# Usage
trash file.txt                  # Moves to trash instead of deleting
```

---

## Viewing File Contents

### cat - Concatenate and Display
```bash
# Display file contents
cat file.txt

# Display multiple files
cat file1.txt file2.txt

# Display with line numbers
cat -n file.txt

# Display non-printing characters
cat -A file.txt                 # Show all (tabs, line endings)
cat -T file.txt                 # Show tabs as ^I
cat -E file.txt                 # Show line endings as $

# Concatenate files
cat file1.txt file2.txt > combined.txt

# Create file with cat
cat > newfile.txt
# Type content, press Ctrl+D to save
```

### less - Page Through Files
```bash
# View file with paging
less file.txt

# Navigation in less:
# Space/f     → Forward one page
# b           → Back one page
# Enter/j     → Forward one line
# k           → Back one line
# g           → Go to beginning
# G           → Go to end
# /pattern    → Search forward
# ?pattern    → Search backward
# n           → Next search result
# N           → Previous search result
# q           → Quit

# Options
less -N file.txt                # Show line numbers
less -S file.txt                # Don't wrap long lines
less +F file.txt                # Follow mode (like tail -f)
```

### more - Simple Pager
```bash
more file.txt                   # Basic paging
# Space to advance, q to quit
# less is generally preferred over more
```

### head - View Beginning of File
```bash
# Display first 10 lines (default)
head file.txt

# Display first N lines
head -n 20 file.txt
head -20 file.txt               # Same as above

# Display first N bytes
head -c 100 file.txt

# Multiple files
head file1.txt file2.txt
```

### tail - View End of File
```bash
# Display last 10 lines (default)
tail file.txt

# Display last N lines
tail -n 20 file.txt
tail -20 file.txt               # Same as above

# Display last N bytes
tail -c 100 file.txt

# Follow file (watch for changes) - VERY USEFUL!
tail -f /var/log/messages       # Follow log file
tail -F /var/log/messages       # Follow, retry if file rotates

# Follow multiple files
tail -f file1.log file2.log

# Start from line N
tail -n +5 file.txt             # Start from line 5 to end
```

### wc - Word Count
```bash
# Count lines, words, and bytes
wc file.txt
# Output: lines words bytes filename

# Specific counts
wc -l file.txt                  # Lines only
wc -w file.txt                  # Words only
wc -c file.txt                  # Bytes only
wc -m file.txt                  # Characters only

# Multiple files
wc -l *.txt                     # Line count for all txt files

# Common use with pipes
cat file.txt | wc -l            # Count lines
ls | wc -l                      # Count files in directory
```

---

## File Information and Types

### file - Determine File Type
```bash
# Determine file type
file filename.txt
# Output: filename.txt: ASCII text

file /bin/ls
# Output: /bin/ls: ELF 64-bit LSB executable...

file image.png
# Output: image.png: PNG image data, 800 x 600...

# Multiple files
file *

# Don't follow symlinks
file -h symlink
```

### stat - Detailed File Information
```bash
stat file.txt

# Output includes:
# - File name
# - Size
# - Blocks allocated
# - IO Block size
# - File type
# - Device
# - Inode number
# - Links count
# - Permissions (octal and symbolic)
# - Owner UID/GID
# - Access, Modify, Change times
# - SELinux context

# Specific format
stat -c %s file.txt             # Size only
stat -c %U file.txt             # Owner username
stat -c %a file.txt             # Permissions (octal)
stat -c %A file.txt             # Permissions (symbolic)
```

### du - Disk Usage
```bash
# Directory size
du directory/

# Human-readable
du -h directory/

# Summary only (total)
du -sh directory/

# All files and directories
du -ah directory/

# Max depth
du -h --max-depth=1 /home/

# Sort by size
du -sh * | sort -h

# Exclude patterns
du -sh --exclude="*.log" /var/
```

---

## Finding Files

### find - Search for Files
```bash
# Basic syntax
find [path] [options] [expression]

# Find by name
find /home -name "*.txt"        # Case-sensitive
find /home -iname "*.txt"       # Case-insensitive

# Find by type
find /var -type f               # Files only
find /var -type d               # Directories only
find /var -type l               # Symbolic links

# Find by size
find /home -size +100M          # Larger than 100MB
find /home -size -1k            # Smaller than 1KB
find /home -size 50M            # Exactly 50MB

# Find by time
find /var -mtime -7             # Modified in last 7 days
find /var -mtime +30            # Modified more than 30 days ago
find /var -mmin -60             # Modified in last 60 minutes
find /var -atime -1             # Accessed in last day

# Find by permissions
find /home -perm 755            # Exact permissions
find /home -perm -644           # At least these permissions
find /home -perm /u+x           # User has execute

# Find by owner
find /home -user username
find /var -group wheel

# Find empty files/directories
find /tmp -empty

# Execute command on results
find /tmp -name "*.tmp" -delete                    # Delete files
find /home -name "*.log" -exec rm {} \;            # Execute rm
find /home -name "*.sh" -exec chmod +x {} \;       # Make executable
find /var -name "*.conf" -exec grep -l "error" {} \;  # Search in files

# Combine conditions
find /home -name "*.txt" -size +1M                 # AND (default)
find /home -name "*.txt" -o -name "*.log"          # OR
find /home ! -name "*.txt"                         # NOT

# Limit depth
find /home -maxdepth 2 -name "*.txt"
find /home -mindepth 1 -maxdepth 3 -type d
```

### locate - Fast File Search
```bash
# Uses a database (faster than find)
locate filename

# Update database (requires root)
sudo updatedb

# Case-insensitive
locate -i filename

# Limit results
locate -n 10 filename

# Note: May not find recently created files until updatedb runs
```

### which / whereis / type
```bash
# Find executable location
which python
# Output: /usr/bin/python

# Find binary, source, and man page
whereis ls
# Output: ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz

# Show command type
type ls
# Output: ls is aliased to 'ls --color=auto'

type cd
# Output: cd is a shell builtin
```

---

## Links (Hard and Symbolic)

### Understanding Links
```
Hard Link:
┌─────────────┐     ┌─────────────┐
│  filename1  │────▶│    inode    │────▶ Data blocks
└─────────────┘     │   (12345)   │
                    └─────────────┘
┌─────────────┐           ▲
│  filename2  │───────────┘
└─────────────┘
(Both point to same inode/data)

Symbolic Link:
┌─────────────┐     ┌─────────────┐
│  symlink    │────▶│  /path/to/  │
└─────────────┘     │  original   │
                    └─────────────┘
(Points to the path, not the data)
```

### ln - Create Links
```bash
# Create hard link
ln original.txt hardlink.txt

# Create symbolic (soft) link
ln -s /path/to/original symlink

# Create symbolic link to directory
ln -s /var/log/ ~/logs

# Force overwrite existing link
ln -sf /new/target symlink

# Common options
ln -v source link               # Verbose
ln -i source link               # Interactive

# View link target
ls -l symlink
readlink symlink
readlink -f symlink             # Full canonical path
```

### Hard vs Symbolic Links
| Feature | Hard Link | Symbolic Link |
|---------|-----------|---------------|
| Cross filesystem | ❌ No | ✅ Yes |
| Link to directory | ❌ No (usually) | ✅ Yes |
| Original deleted | ✅ Data preserved | ❌ Broken link |
| Inode | Same as original | Different |
| Size | Same as original | Size of path string |

---

## File Compression and Archives

### tar - Archive Files
```bash
# Create archive
tar -cvf archive.tar files/     # Create verbose file
tar -cvf archive.tar file1 file2 dir/

# Extract archive
tar -xvf archive.tar            # Extract here
tar -xvf archive.tar -C /path/  # Extract to directory

# List contents
tar -tvf archive.tar

# Create compressed archives
tar -czvf archive.tar.gz files/ # gzip compression
tar -cjvf archive.tar.bz2 files/ # bzip2 compression
tar -cJvf archive.tar.xz files/  # xz compression

# Extract compressed archives
tar -xzvf archive.tar.gz
tar -xjvf archive.tar.bz2
tar -xJvf archive.tar.xz

# Options breakdown:
# c = create
# x = extract
# v = verbose
# f = file (archive name)
# z = gzip
# j = bzip2
# J = xz
# t = list
# C = change directory
```

### gzip / gunzip
```bash
# Compress file (replaces original)
gzip file.txt
# Creates file.txt.gz

# Decompress
gunzip file.txt.gz
# or
gzip -d file.txt.gz

# Keep original file
gzip -k file.txt

# Compress with level (1-9, 9=best)
gzip -9 file.txt

# View compressed file
zcat file.txt.gz
zless file.txt.gz
```

### bzip2 / bunzip2
```bash
# Compress (better ratio than gzip)
bzip2 file.txt
# Creates file.txt.bz2

# Decompress
bunzip2 file.txt.bz2
bzip2 -d file.txt.bz2

# Keep original
bzip2 -k file.txt

# View
bzcat file.txt.bz2
```

### xz / unxz
```bash
# Compress (best ratio, slower)
xz file.txt
# Creates file.txt.xz

# Decompress
unxz file.txt.xz
xz -d file.txt.xz

# Keep original
xz -k file.txt

# View
xzcat file.txt.xz
```

### zip / unzip
```bash
# Create zip archive
zip archive.zip file1 file2
zip -r archive.zip directory/   # Recursive for directories

# Extract
unzip archive.zip
unzip archive.zip -d /path/     # Extract to directory

# List contents
unzip -l archive.zip

# Options
zip -e archive.zip files/       # Encrypt with password
zip -u archive.zip newfile      # Update/add to existing zip
```

---

## Quick Reference Card

### Navigation
| Command | Description |
|---------|-------------|
| `pwd` | Print working directory |
| `cd dir` | Change to directory |
| `cd ..` | Go up one level |
| `cd ~` | Go to home |
| `cd -` | Go to previous directory |

### File Operations
| Command | Description |
|---------|-------------|
| `ls -la` | List all files (detailed) |
| `touch file` | Create empty file |
| `mkdir -p dir` | Create directory (with parents) |
| `cp -r src dest` | Copy (recursive) |
| `mv src dest` | Move/rename |
| `rm -rf dir` | Remove directory (force) |

### Viewing Files
| Command | Description |
|---------|-------------|
| `cat file` | Display file |
| `less file` | Page through file |
| `head -n 20 file` | First 20 lines |
| `tail -f file` | Follow file (live) |
| `wc -l file` | Count lines |

### Finding Files
| Command | Description |
|---------|-------------|
| `find /path -name "*.txt"` | Find by name |
| `find /path -type f -size +10M` | Find large files |
| `locate filename` | Fast search (database) |
| `which command` | Find command path |

### Compression
| Command | Description |
|---------|-------------|
| `tar -czvf a.tar.gz dir/` | Create tar.gz |
| `tar -xzvf a.tar.gz` | Extract tar.gz |
| `gzip file` | Compress with gzip |
| `unzip archive.zip` | Extract zip |

---

**Previous: [Introduction to Linux](01_intro.md)**  
**Next: [User and Permission Management](03_users_permissions.md)**
