# Linux Crash Course - Text Processing

## Table of Contents
- [grep - Pattern Searching](#grep---pattern-searching)
- [sed - Stream Editor](#sed---stream-editor)
- [awk - Pattern Scanning and Processing](#awk---pattern-scanning-and-processing)
- [cut - Extract Columns](#cut---extract-columns)
- [sort - Sort Lines](#sort---sort-lines)
- [uniq - Remove Duplicates](#uniq---remove-duplicates)
- [tr - Translate Characters](#tr---translate-characters)
- [diff and patch](#diff-and-patch)
- [Other Useful Tools](#other-useful-tools)
- [Combining Commands with Pipes](#combining-commands-with-pipes)
- [Wildcards (Glob Patterns)](#wildcards-glob-patterns)
- [find - Search for Files](#find---search-for-files)

---

## grep - Pattern Searching

**grep** = Global Regular Expression Print

### Basic Usage
```bash
# Search for pattern in file
grep "pattern" filename

# Search in multiple files
grep "pattern" file1.txt file2.txt
grep "pattern" *.txt

# Search recursively in directories
grep -r "pattern" /path/to/directory/
grep -R "pattern" .                    # Current directory
```

### Common Options
```bash
# Case insensitive
grep -i "pattern" file.txt

# Show line numbers
grep -n "pattern" file.txt

# Count matching lines
grep -c "pattern" file.txt

# Show only matching part (not whole line)
grep -o "pattern" file.txt

# Invert match (lines NOT matching)
grep -v "pattern" file.txt

# Whole word match
grep -w "word" file.txt

# Show lines before/after/around match
grep -B 3 "pattern" file.txt      # 3 lines Before
grep -A 3 "pattern" file.txt      # 3 lines After
grep -C 3 "pattern" file.txt      # 3 lines Context (before and after)

# List only filenames with matches
grep -l "pattern" *.txt

# List filenames WITHOUT matches
grep -L "pattern" *.txt

# Quiet mode (exit status only)
grep -q "pattern" file.txt && echo "Found"
```

### Regular Expressions
```bash
# Extended regex (-E or egrep)
grep -E "pattern1|pattern2" file.txt     # OR
grep -E "^start" file.txt                # Line starts with
grep -E "end$" file.txt                  # Line ends with
grep -E "^$" file.txt                    # Empty lines
grep -E "[0-9]+" file.txt                # One or more digits
grep -E "[a-zA-Z]+" file.txt             # One or more letters

# Basic regex patterns
.       # Any single character
*       # Zero or more of previous
^       # Start of line
$       # End of line
[]      # Character class
[^]     # Negated character class
\       # Escape special character

# Examples
grep "^#" config.txt                     # Lines starting with #
grep -v "^#" config.txt                  # Lines NOT starting with #
grep -v "^$" file.txt                    # Non-empty lines
grep "error\|warning" log.txt            # error OR warning
grep -E "192\.168\.[0-9]+\.[0-9]+" file  # IP addresses
```

### Practical Examples
```bash
# Search logs
grep -i "error" /var/log/messages
grep "Failed password" /var/log/secure

# Find processes
ps aux | grep httpd
ps aux | grep -v grep | grep httpd       # Exclude grep itself

# Search configuration
grep -r "Listen" /etc/httpd/

# Count occurrences
grep -c "GET" access.log
grep -o "GET\|POST" access.log | sort | uniq -c

# Find files containing text
grep -rl "TODO" ~/projects/
```

---

## sed - Stream Editor

**sed** = Stream EDitor - for parsing and transforming text

### Basic Syntax
```bash
sed 'command' filename
sed -e 'command1' -e 'command2' filename
sed -f script.sed filename
```

### Substitution (Most Common)
```bash
# Basic substitution (first occurrence per line)
sed 's/old/new/' file.txt

# Global substitution (all occurrences)
sed 's/old/new/g' file.txt

# Case insensitive
sed 's/old/new/gi' file.txt

# Substitute Nth occurrence
sed 's/old/new/2' file.txt            # 2nd occurrence
sed 's/old/new/3g' file.txt           # From 3rd occurrence onwards

# Different delimiters (useful for paths)
sed 's|/old/path|/new/path|g' file.txt
sed 's#old#new#g' file.txt

# In-place editing
sed -i 's/old/new/g' file.txt         # Modify file directly
sed -i.bak 's/old/new/g' file.txt     # Create backup first
```

### Line Selection
```bash
# Specific line
sed '5s/old/new/' file.txt            # Line 5 only
sed '5d' file.txt                     # Delete line 5

# Line range
sed '5,10s/old/new/g' file.txt        # Lines 5-10
sed '5,10d' file.txt                  # Delete lines 5-10

# From line to end
sed '5,$s/old/new/g' file.txt         # Line 5 to end

# Pattern match
sed '/pattern/s/old/new/g' file.txt   # Lines matching pattern
sed '/start/,/end/s/old/new/g' file   # Between patterns
```

### Common sed Commands
```bash
# d = delete
sed '/pattern/d' file.txt             # Delete matching lines
sed '/^#/d' file.txt                  # Delete comment lines
sed '/^$/d' file.txt                  # Delete empty lines

# p = print (use with -n)
sed -n '/pattern/p' file.txt          # Print only matching lines
sed -n '5,10p' file.txt               # Print lines 5-10
sed -n '1p' file.txt                  # Print first line

# a = append (after)
sed '/pattern/a New line' file.txt    # Add line after match

# i = insert (before)
sed '/pattern/i New line' file.txt    # Add line before match

# c = change (replace entire line)
sed '/pattern/c Replacement line' file.txt

# = = print line number
sed -n '/pattern/=' file.txt
```

### Practical Examples
```bash
# Remove blank lines and comments
sed '/^$/d; /^#/d' config.txt

# Replace multiple patterns
sed -e 's/foo/bar/g' -e 's/baz/qux/g' file.txt

# Add text at beginning/end of lines
sed 's/^/PREFIX: /' file.txt          # Add prefix
sed 's/$/ :SUFFIX/' file.txt          # Add suffix

# Extract between patterns
sed -n '/START/,/END/p' file.txt

# Replace in specific file types
find . -name "*.conf" -exec sed -i 's/old/new/g' {} \;

# Comment out lines
sed '/pattern/s/^/#/' file.txt

# Uncomment lines
sed 's/^#//' file.txt

# Remove leading/trailing whitespace
sed 's/^[ \t]*//' file.txt            # Leading
sed 's/[ \t]*$//' file.txt            # Trailing
sed 's/^[ \t]*//; s/[ \t]*$//' file   # Both

# Convert DOS to Unix line endings
sed -i 's/\r$//' file.txt
```

---

## awk - Pattern Scanning and Processing

**awk** = Pattern scanning and processing language (named after Aho, Weinberger, Kernighan)

### Basic Structure
```bash
awk 'pattern { action }' file
awk -F':' 'pattern { action }' file    # Set field separator
```

### Field Variables
```bash
# $0 = entire line
# $1, $2, etc. = fields (columns)
# NF = Number of Fields
# NR = Number of Record (line number)
# FS = Field Separator
# OFS = Output Field Separator

# Print first column
awk '{print $1}' file.txt

# Print multiple columns
awk '{print $1, $3}' file.txt

# Print last column
awk '{print $NF}' file.txt

# Print second to last
awk '{print $(NF-1)}' file.txt

# Print all except first column
awk '{$1=""; print $0}' file.txt

# Print with custom separator
awk -F':' '{print $1}' /etc/passwd
```

### Patterns and Conditions
```bash
# Pattern matching
awk '/pattern/' file.txt                      # Lines containing pattern
awk '/^root/' /etc/passwd                     # Lines starting with root
awk '!/pattern/' file.txt                     # Lines NOT matching

# Conditions
awk '$3 > 100' file.txt                       # Third field > 100
awk '$1 == "value"' file.txt                  # First field equals "value"
awk 'NR > 1' file.txt                         # Skip header (line > 1)
awk 'NF > 0' file.txt                         # Non-empty lines
awk 'NR >= 5 && NR <= 10' file.txt            # Lines 5-10

# Combine pattern and action
awk '/error/ {print $0}' log.txt
awk '$3 > 1000 {print $1, $3}' file.txt
```

### Built-in Functions
```bash
# String functions
awk '{print length($1)}' file.txt             # Length
awk '{print toupper($1)}' file.txt            # Uppercase
awk '{print tolower($1)}' file.txt            # Lowercase
awk '{print substr($1, 1, 3)}' file.txt       # Substring
awk '{gsub(/old/, "new"); print}' file.txt    # Replace

# Math functions
awk '{sum += $1} END {print sum}' file.txt    # Sum column
awk '{sum += $1} END {print sum/NR}' file     # Average
awk 'BEGIN {print sqrt(144)}'                 # Square root
awk 'BEGIN {print int(5.7)}'                  # Integer
```

### BEGIN and END Blocks
```bash
# BEGIN = before processing any lines
# END = after processing all lines

awk 'BEGIN {print "Header"} {print} END {print "Footer"}' file.txt

# Count lines
awk 'END {print NR}' file.txt

# Sum column with header
awk 'BEGIN {sum=0} {sum+=$1} END {print "Total:", sum}' file.txt

# Set output format
awk 'BEGIN {OFS=","} {print $1, $2, $3}' file.txt
```

### Practical Examples
```bash
# Parse /etc/passwd
awk -F':' '{print $1, $7}' /etc/passwd                    # User and shell
awk -F':' '$3 >= 1000 {print $1}' /etc/passwd             # Regular users
awk -F':' '$7 ~ /bash/ {print $1}' /etc/passwd            # Bash users

# Process logs
awk '{print $1}' access.log | sort | uniq -c | sort -rn   # Top IPs
awk '$9 >= 400' access.log                                # Error responses
awk '{sum+=$10} END {print sum}' access.log               # Total bytes

# Calculate
awk '{sum+=$1; count++} END {print sum/count}' numbers.txt  # Average
awk 'BEGIN {max=0} {if($1>max) max=$1} END {print max}' f   # Maximum

# Format output
awk '{printf "%-20s %10d\n", $1, $2}' file.txt            # Formatted

# Process CSV
awk -F',' '{print $1, $3}' data.csv

# Add line numbers
awk '{print NR": "$0}' file.txt

# Print specific columns with header
awk 'NR==1 || $3 > 100' file.txt                          # Header + matching
```

---

## cut - Extract Columns

### Basic Usage
```bash
# Cut by character position
cut -c1-10 file.txt                   # Characters 1-10
cut -c1,5,10 file.txt                 # Characters 1, 5, and 10
cut -c5- file.txt                     # From character 5 to end
cut -c-5 file.txt                     # First 5 characters

# Cut by delimiter and field
cut -d':' -f1 /etc/passwd             # First field (username)
cut -d':' -f1,7 /etc/passwd           # Fields 1 and 7
cut -d':' -f1-3 /etc/passwd           # Fields 1 through 3
cut -d',' -f2- file.csv               # Field 2 to end

# Default delimiter is TAB
cut -f1 file.txt                      # First tab-separated field
```

### Options
```bash
-d        # Delimiter
-f        # Fields
-c        # Characters
-b        # Bytes
--complement    # Everything EXCEPT specified
--output-delimiter=    # Change output delimiter
```

### Examples
```bash
# Extract usernames from passwd
cut -d':' -f1 /etc/passwd

# Get IP addresses from log
cut -d' ' -f1 access.log

# Extract columns from CSV and change delimiter
cut -d',' -f1,3 data.csv --output-delimiter=' '

# Remove first character from each line
cut -c2- file.txt

# Get everything except first field
cut -d':' -f2- /etc/passwd
```

---

## sort - Sort Lines

### Basic Usage
```bash
# Alphabetical sort
sort file.txt

# Reverse order
sort -r file.txt

# Numeric sort
sort -n numbers.txt

# Sort by specific column
sort -k2 file.txt                     # By 2nd field
sort -k2,2 file.txt                   # By 2nd field only
sort -k2n file.txt                    # 2nd field, numeric

# Sort with delimiter
sort -t':' -k3 -n /etc/passwd         # By UID
sort -t',' -k2 data.csv               # CSV by 2nd column

# Unique values only
sort -u file.txt

# Ignore case
sort -f file.txt

# Check if sorted
sort -c file.txt
```

### Options
```bash
-n        # Numeric sort
-r        # Reverse
-k        # Key (field to sort by)
-t        # Field delimiter
-u        # Unique (remove duplicates)
-f        # Ignore case
-h        # Human-readable numbers (1K, 2M)
-M        # Month sort (Jan, Feb, etc.)
-o        # Output file
```

### Examples
```bash
# Sort processes by memory
ps aux | sort -k4 -rn | head

# Sort by multiple columns
sort -k1,1 -k2,2n file.txt            # First alpha, then numeric

# Sort human-readable sizes
du -h | sort -h

# Sort by month
sort -M file.txt

# Sort and save
sort -o sorted.txt unsorted.txt

# Case-insensitive unique
sort -uf file.txt
```

---

## uniq - Remove Duplicates

### Basic Usage
```bash
# Remove adjacent duplicates (MUST BE SORTED FIRST!)
sort file.txt | uniq

# Count occurrences
sort file.txt | uniq -c

# Only show duplicates
sort file.txt | uniq -d

# Only show unique lines (no duplicates)
sort file.txt | uniq -u

# Ignore case
sort file.txt | uniq -i

# Ignore first N fields
sort file.txt | uniq -f 1

# Ignore first N characters
sort file.txt | uniq -s 5
```

### Examples
```bash
# Count unique values
cut -d':' -f7 /etc/passwd | sort | uniq -c

# Find duplicate lines
sort file.txt | uniq -d

# Top 10 most common items
sort access.log | uniq -c | sort -rn | head -10

# Count unique IPs
awk '{print $1}' access.log | sort | uniq -c | sort -rn
```

---

## tr - Translate Characters

### Basic Usage
```bash
# Replace characters
echo "hello" | tr 'a-z' 'A-Z'         # Lowercase to uppercase
echo "HELLO" | tr 'A-Z' 'a-z'         # Uppercase to lowercase

# Replace specific characters
echo "hello" | tr 'aeiou' '12345'     # Vowels to numbers

# Delete characters
echo "hello 123" | tr -d '0-9'        # Delete digits
echo "hello world" | tr -d ' '        # Delete spaces

# Squeeze repeated characters
echo "helloooo" | tr -s 'o'           # Reduce to single 'o'
tr -s ' ' < file.txt                  # Squeeze multiple spaces

# Complement (use characters NOT in set)
echo "hello123" | tr -cd '0-9'        # Keep only digits
echo "hello123" | tr -d '[:alpha:]'   # Delete all letters
```

### Character Classes
```bash
[:alpha:]     # Letters
[:digit:]     # Digits
[:alnum:]     # Alphanumeric
[:space:]     # Whitespace
[:lower:]     # Lowercase
[:upper:]     # Uppercase
[:punct:]     # Punctuation
[:print:]     # Printable characters
```

### Examples
```bash
# Convert case
tr '[:lower:]' '[:upper:]' < file.txt

# Remove non-printable characters
tr -cd '[:print:]' < file.txt

# Replace newlines with spaces
tr '\n' ' ' < file.txt

# Convert Windows line endings
tr -d '\r' < windows.txt > unix.txt

# Create simple cipher
echo "secret" | tr 'a-zA-Z' 'n-za-mN-ZA-M'  # ROT13
```

---

## diff and patch

### diff - Compare Files
```bash
# Basic diff
diff file1.txt file2.txt

# Unified format (most common)
diff -u file1.txt file2.txt

# Context format
diff -c file1.txt file2.txt

# Side by side
diff -y file1.txt file2.txt
diff -y --width=80 file1.txt file2.txt

# Ignore whitespace
diff -w file1.txt file2.txt          # Ignore all whitespace
diff -b file1.txt file2.txt          # Ignore whitespace changes

# Ignore case
diff -i file1.txt file2.txt

# Brief output (only if different)
diff -q file1.txt file2.txt

# Recursive (directories)
diff -r dir1/ dir2/

# Create patch file
diff -u old.txt new.txt > changes.patch
```

### patch - Apply Changes
```bash
# Apply patch
patch < changes.patch
patch file.txt < changes.patch

# Reverse patch
patch -R < changes.patch

# Dry run (test without applying)
patch --dry-run < changes.patch

# Backup original
patch -b < changes.patch

# Patch directory
patch -p1 < changes.patch
```

### diff Output Explained
```
# Normal diff:
2c2           # Line 2 changed
< old text    # Old content
---
> new text    # New content

3a4           # After line 3, add (line 4 in new)
> added line

5d4           # Delete line 5

# Unified diff:
--- old.txt
+++ new.txt
@@ -1,3 +1,3 @@
 unchanged
-removed
+added
 unchanged
```

---

## Other Useful Tools

### paste - Merge Files Side by Side
```bash
paste file1.txt file2.txt             # Tab separated
paste -d',' file1.txt file2.txt       # Comma separated
paste -s file.txt                     # Merge all lines into one
```

### join - Join Files on Common Field
```bash
# Files must be sorted on join field
join file1.txt file2.txt              # Join on first field
join -t':' -1 1 -2 3 f1 f2            # Custom field and delimiter
```

### split - Split File into Pieces
```bash
split -l 1000 file.txt                # 1000 lines each
split -b 10M file.txt                 # 10MB each
split -n 5 file.txt                   # Into 5 equal parts
split -l 1000 file.txt prefix_        # Custom prefix
```

### csplit - Split by Pattern
```bash
csplit file.txt '/pattern/' {*}       # Split at pattern
```

### column - Format into Columns
```bash
column -t file.txt                    # Auto-format table
column -t -s',' file.csv              # Format CSV
```

### expand/unexpand - Tab/Space Conversion
```bash
expand file.txt                       # Tabs to spaces
unexpand -a file.txt                  # Spaces to tabs
```

### nl - Number Lines
```bash
nl file.txt                           # Number non-empty lines
nl -ba file.txt                       # Number all lines
```

### rev - Reverse Line Characters
```bash
echo "hello" | rev                    # olleh
```

### tac - Reverse Line Order
```bash
tac file.txt                          # Print lines in reverse order
```

### fold - Wrap Lines
```bash
fold -w 80 file.txt                   # Wrap at 80 characters
```

---

## Combining Commands with Pipes

### Pipe Basics
```bash
command1 | command2 | command3

# Output of command1 becomes input of command2
```

### Practical Pipelines
```bash
# Find top 10 largest files
du -ah /var/log | sort -rh | head -10

# Count unique 404 errors by IP
grep " 404 " access.log | awk '{print $1}' | sort | uniq -c | sort -rn

# Find users with bash shell
grep bash /etc/passwd | cut -d':' -f1 | sort

# Extract and format data
cat data.txt | tr -s ' ' | cut -d' ' -f2,4 | sort -n | uniq

# Process log timestamps
awk '{print $4}' access.log | cut -d: -f2 | sort | uniq -c

# Find most common words
tr ' ' '\n' < file.txt | tr '[:upper:]' '[:lower:]' | sort | uniq -c | sort -rn | head

# Show disk usage by directory
du -h --max-depth=1 | sort -hr

# List installed packages (RHEL)
rpm -qa | sort | grep -i keyword

# Monitor real-time log for pattern
tail -f /var/log/messages | grep --line-buffered error

# Format process list
ps aux | awk '{printf "%-10s %5s %s\n", $1, $2, $11}'
```

### tee - Split Output
```bash
# Write to file AND display
command | tee output.txt

# Append instead of overwrite
command | tee -a output.txt

# Write to multiple files
command | tee file1.txt file2.txt

# Use with sudo
echo "text" | sudo tee /etc/somefile
```

### xargs - Build Commands from Input
```bash
# Basic usage
find . -name "*.txt" | xargs grep "pattern"

# Handle spaces in filenames
find . -name "*.txt" -print0 | xargs -0 grep "pattern"

# One argument at a time
cat list.txt | xargs -I {} mv {} {}.bak

# Parallel execution
cat files.txt | xargs -P 4 -I {} process {}

# Limit arguments per command
echo "1 2 3 4 5" | xargs -n 2 echo

# Prompt before execution
echo "file.txt" | xargs -p rm
```

---

## Wildcards (Glob Patterns)

### Basic Wildcards
```bash
*         # Match any characters (zero or more)
?         # Match exactly one character
[abc]     # Match any character in brackets
[a-z]     # Match any character in range
[!abc]    # Match any character NOT in brackets
[^abc]    # Same as [!abc]
```

### Wildcard Examples
```bash
# Match any files
ls *.txt                              # All .txt files
ls file*                              # Files starting with "file"
ls *log*                              # Files containing "log"

# Match single character
ls file?.txt                          # file1.txt, fileA.txt, etc.
ls ???.txt                            # Any 3-character name .txt

# Character classes
ls file[123].txt                      # file1.txt, file2.txt, file3.txt
ls file[a-z].txt                      # filea.txt through filez.txt
ls file[0-9][0-9].txt                 # file00.txt through file99.txt
ls [A-Z]*.txt                         # Files starting with uppercase
ls *[!0-9].txt                        # Files NOT ending with digit before .txt
```

### Extended Globs (bash)
```bash
# Enable extended globs
shopt -s extglob

# Extended patterns
?(pattern)        # Zero or one occurrence
*(pattern)        # Zero or more occurrences
+(pattern)        # One or more occurrences
@(pattern)        # Exactly one occurrence
!(pattern)        # Anything except pattern

# Examples
ls !(*.txt)                           # All files EXCEPT .txt
ls *.@(jpg|png|gif)                   # Images only
ls +([0-9]).txt                       # One or more digits .txt
```

---

## find - Search for Files

### Basic Syntax
```bash
find [path] [options] [expression]
```

### Search by Name
```bash
# Exact name
find /home -name "file.txt"

# Wildcard patterns (quote them!)
find . -name "*.txt"                  # All .txt files
find . -name "file*"                  # Files starting with "file"
find /var -name "*.log"               # All log files

# Case insensitive
find . -iname "*.TXT"                 # Matches .txt, .TXT, .Txt

# By path/directory pattern
find . -path "*config*"               # Path contains "config"
find . -path "*/src/*.java"           # Java files in src directories
```

### Search by Type
```bash
find . -type f                        # Files only
find . -type d                        # Directories only
find . -type l                        # Symbolic links
find . -type f -name "*.sh"           # Shell script files
```

### Search by Size
```bash
find . -size +100M                    # Larger than 100MB
find . -size -1k                      # Smaller than 1KB
find . -size 50M                      # Exactly 50MB
find /var -type f -size +10M          # Files over 10MB in /var

# Size suffixes: c (bytes), k (KB), M (MB), G (GB)
```

### Search by Time
```bash
# Modified time (days)
find . -mtime -7                      # Modified in last 7 days
find . -mtime +30                     # Modified more than 30 days ago
find . -mtime 0                       # Modified today

# Access time
find . -atime -7                      # Accessed in last 7 days

# Changed time (metadata)
find . -ctime -7                      # Changed in last 7 days

# Minutes instead of days
find . -mmin -60                      # Modified in last 60 minutes
find . -mmin +120                     # Modified more than 2 hours ago

# Newer than file
find . -newer reference.txt           # Newer than reference.txt
```

### Search by Permissions/Ownership
```bash
# By permission
find . -perm 644                      # Exactly 644
find . -perm -644                     # At least 644
find . -perm /u+x                     # User executable
find . -perm -u+x                     # User executable (all)

# By owner/group
find . -user john                     # Owned by john
find . -group developers              # Group is developers
find . -nouser                        # No valid owner
find . -nogroup                       # No valid group
```

### Combining Conditions
```bash
# AND (default)
find . -name "*.txt" -type f          # .txt AND file type

# OR
find . -name "*.txt" -o -name "*.log" # .txt OR .log
find . \( -name "*.jpg" -o -name "*.png" \) -type f

# NOT
find . ! -name "*.txt"                # NOT .txt
find . -not -user root                # NOT owned by root
```

### Actions
```bash
# Print (default)
find . -name "*.txt" -print

# Delete
find . -name "*.tmp" -delete
find /tmp -type f -mtime +7 -delete   # Delete old temp files

# Execute command
find . -name "*.sh" -exec chmod +x {} \;         # Make executable
find . -name "*.txt" -exec grep "pattern" {} \;  # Search in files
find . -name "*.log" -exec rm {} \;              # Remove files

# Execute with confirmation
find . -name "*.bak" -ok rm {} \;                # Prompt before delete

# Execute with multiple files (faster)
find . -name "*.txt" -exec grep "pattern" {} +

# Using xargs (handles large file lists)
find . -name "*.txt" | xargs grep "pattern"
find . -name "*.txt" -print0 | xargs -0 grep "pattern"  # Handle spaces
```

### Practical find Examples
```bash
# Find large files
find / -type f -size +100M 2>/dev/null

# Find recently modified configs
find /etc -name "*.conf" -mtime -1

# Find and remove old logs
find /var/log -name "*.log" -mtime +30 -delete

# Find empty files/directories
find . -type f -empty                 # Empty files
find . -type d -empty                 # Empty directories

# Find files with specific text
find . -name "*.py" -exec grep -l "import os" {} \;

# Find duplicate names
find . -name "*.txt" -printf "%f\n" | sort | uniq -d

# Find executables
find /usr/bin -type f -perm /u+x

# Find broken symlinks
find . -type l ! -exec test -e {} \; -print

# Find and change permissions
find . -type f -name "*.sh" -exec chmod 755 {} \;

# Find files modified between dates
find . -newermt "2024-01-01" ! -newermt "2024-02-01"
```

### find vs locate
```bash
# locate - fast but uses database
locate filename                       # Search filename database
sudo updatedb                         # Update database

# find - slower but real-time
find / -name filename                 # Search filesystem directly
```

---

## Quick Reference Card

### grep
| Command | Description |
|---------|-------------|
| `grep "pattern" file` | Search for pattern |
| `grep -i` | Case insensitive |
| `grep -r` | Recursive |
| `grep -v` | Invert match |
| `grep -n` | Show line numbers |
| `grep -c` | Count matches |
| `grep -E` | Extended regex |

### sed
| Command | Description |
|---------|-------------|
| `sed 's/old/new/' file` | Replace first |
| `sed 's/old/new/g' file` | Replace all |
| `sed -i` | In-place edit |
| `sed -n '/pattern/p'` | Print matching |
| `sed '/pattern/d'` | Delete matching |

### awk
| Command | Description |
|---------|-------------|
| `awk '{print $1}'` | Print first field |
| `awk -F':'` | Set delimiter |
| `awk '/pattern/'` | Match pattern |
| `awk '$3 > 10'` | Condition |
| `awk 'END {print NR}'` | Count lines |

### Other
| Command | Description |
|---------|-------------|
| `cut -d':' -f1` | Extract field |
| `sort -n` | Numeric sort |
| `sort -k2` | Sort by field 2 |
| `uniq -c` | Count occurrences |
| `tr 'a-z' 'A-Z'` | Translate chars |
| `diff -u f1 f2` | Compare files |

---

**[← Back to Index](README.md)**  
**Previous: [Users & Permissions](03_users_permissions.md)**  
**Next: [Process Management](05_process_management.md)**
