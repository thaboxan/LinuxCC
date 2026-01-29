# Linux Crash Course - Shell Scripting

## Table of Contents
- [Introduction to Shell Scripting](#introduction-to-shell-scripting)
- [Variables](#variables)
- [User Input](#user-input)
- [Operators and Arithmetic](#operators-and-arithmetic)
- [Conditional Statements](#conditional-statements)
- [Loops](#loops)
- [Functions](#functions)
- [Arrays](#arrays)
- [String Manipulation](#string-manipulation)
- [File Operations](#file-operations)
- [Command Line Arguments](#command-line-arguments)
- [Exit Codes and Error Handling](#exit-codes-and-error-handling)
- [Best Practices](#best-practices)
- [Practical Examples](#practical-examples)

---

## Introduction to Shell Scripting

### What is a Shell Script?
A shell script is a text file containing a series of commands that the shell executes sequentially. It automates repetitive tasks and combines multiple commands.

### Creating Your First Script
```bash
#!/bin/bash
# This is a comment
# Script: hello.sh

echo "Hello, World!"
echo "Today is $(date)"
echo "Current user: $USER"
```

### Making Script Executable
```bash
# Create the script
vim hello.sh

# Make it executable
chmod +x hello.sh

# Run the script
./hello.sh
# or
bash hello.sh
```

### Shebang Line
```bash
#!/bin/bash          # Use bash
#!/bin/sh            # Use sh (POSIX)
#!/usr/bin/env bash  # Find bash in PATH (portable)
#!/usr/bin/env python3  # For Python scripts
```

### Running Scripts
```bash
# With execute permission
./script.sh

# Without execute permission
bash script.sh
sh script.sh

# Source (run in current shell)
source script.sh
. script.sh
```

---

## Variables

### Variable Basics
```bash
# Assign variable (NO spaces around =)
name="John"
age=25
file_path="/var/log/messages"

# Use variable (with $)
echo $name
echo ${name}           # Preferred for clarity
echo "Hello, $name"
echo "Hello, ${name}!"

# Readonly variable
readonly PI=3.14159
PI=3.14               # Error: readonly variable

# Unset variable
unset name
```

### Variable Naming Rules
```bash
# Valid names
myVar="value"
my_var="value"
myVar2="value"
_myVar="value"

# Invalid names
2myVar="value"        # Cannot start with number
my-var="value"        # No hyphens
my var="value"        # No spaces
```

### Quoting
```bash
name="John Doe"

# Double quotes - variable expansion
echo "Hello, $name"   # Hello, John Doe

# Single quotes - literal string
echo 'Hello, $name'   # Hello, $name

# Backticks / $() - command substitution
echo "Date: $(date)"
echo "Date: `date`"   # Old style, avoid

# Escape character
echo "Price: \$100"   # Price: $100
echo "Tab:\tNewline:\n"
```

### Special Variables
```bash
$0          # Script name
$1 to $9    # Positional parameters
${10}       # 10th parameter and beyond
$#          # Number of arguments
$@          # All arguments (as separate words)
$*          # All arguments (as single word)
$$          # Current process ID
$?          # Exit status of last command
$!          # PID of last background process
$_          # Last argument of previous command
```

### Environment Variables
```bash
# Common environment variables
echo $HOME            # User home directory
echo $USER            # Current username
echo $PATH            # Executable search path
echo $PWD             # Current directory
echo $SHELL           # Current shell
echo $HOSTNAME        # System hostname
echo $RANDOM          # Random number (0-32767)
echo $LINENO          # Current line number

# Export variable (available to child processes)
export MY_VAR="value"

# List all environment variables
env
printenv
```

### Variable Substitution
```bash
name=""

# Default value (if unset or null)
echo ${name:-"default"}     # Use default, don't change name
echo ${name:="default"}     # Use default, set name to default
echo ${name:+"alternate"}   # Use alternate if name is set
echo ${name:?"error msg"}   # Display error if unset

# String length
str="Hello"
echo ${#str}                # 5

# Substring
str="Hello World"
echo ${str:0:5}             # Hello (start:length)
echo ${str:6}               # World (from position 6)
echo ${str: -5}             # World (last 5, note space)

# Pattern removal
file="/path/to/file.txt"
echo ${file#*/}             # path/to/file.txt (remove first match)
echo ${file##*/}            # file.txt (remove longest match)
echo ${file%/*}             # /path/to (remove from end, first match)
echo ${file%%/*}            # (empty - remove longest from end)

# Pattern replacement
str="hello world world"
echo ${str/world/there}     # hello there world (first)
echo ${str//world/there}    # hello there there (all)
echo ${str/#hello/hi}       # hi world world (beginning)
echo ${str/%world/earth}    # hello world earth (end)

# Case conversion (Bash 4+)
str="Hello World"
echo ${str,,}               # hello world (lowercase)
echo ${str^^}               # HELLO WORLD (uppercase)
echo ${str^}                # Hello World (first char upper)
```

---

## User Input

### read Command
```bash
# Basic input
echo "Enter your name:"
read name
echo "Hello, $name!"

# Prompt on same line
read -p "Enter your name: " name

# Silent input (passwords)
read -sp "Enter password: " password
echo

# Timeout
read -t 5 -p "Enter within 5 seconds: " answer

# Default value
read -p "Enter name [John]: " name
name=${name:-John}

# Read multiple values
read -p "Enter first and last name: " first last
echo "First: $first, Last: $last"

# Read into array
read -a colors -p "Enter colors: "
echo "First color: ${colors[0]}"

# Read specific number of characters
read -n 1 -p "Press any key..."
echo

# Read from file
while read line; do
    echo "$line"
done < file.txt
```

### select Menu
```bash
#!/bin/bash

PS3="Choose an option: "        # Custom prompt

select option in "Start" "Stop" "Restart" "Quit"; do
    case $option in
        Start)
            echo "Starting..."
            ;;
        Stop)
            echo "Stopping..."
            ;;
        Restart)
            echo "Restarting..."
            ;;
        Quit)
            echo "Goodbye!"
            break
            ;;
        *)
            echo "Invalid option"
            ;;
    esac
done
```

---

## Operators and Arithmetic

### Arithmetic Operations
```bash
# Using $(( ))
a=10
b=3

echo $((a + b))      # 13 (addition)
echo $((a - b))      # 7 (subtraction)
echo $((a * b))      # 30 (multiplication)
echo $((a / b))      # 3 (integer division)
echo $((a % b))      # 1 (modulus)
echo $((a ** b))     # 1000 (exponentiation)

# Assignment
((a++))              # Increment
((a--))              # Decrement
((a += 5))           # Add and assign
((a -= 5))           # Subtract and assign
((a *= 2))           # Multiply and assign
((a /= 2))           # Divide and assign

# Using let
let "result = a + b"
let a++

# Using expr (old style)
result=$(expr $a + $b)
result=$(expr $a \* $b)   # Must escape *

# Using bc for floating point
echo "scale=2; 10/3" | bc       # 3.33
result=$(echo "scale=2; 10/3" | bc)
```

### Comparison Operators

#### Integer Comparison
```bash
# Inside [[ ]] or [ ]
-eq     # Equal
-ne     # Not equal
-gt     # Greater than
-ge     # Greater than or equal
-lt     # Less than
-le     # Less than or equal

# Examples
if [[ $a -eq $b ]]; then
    echo "Equal"
fi

if [[ $a -gt 5 ]]; then
    echo "Greater than 5"
fi

# Inside (( )) - C-style
if (( a == b )); then echo "Equal"; fi
if (( a > 5 )); then echo "Greater"; fi
if (( a >= b )); then echo "Greater or equal"; fi
if (( a != b )); then echo "Not equal"; fi
```

#### String Comparison
```bash
=       # Equal (in [ ])
==      # Equal (in [[ ]], preferred)
!=      # Not equal
<       # Less than (alphabetically)
>       # Greater than
-z      # String is empty
-n      # String is not empty

# Examples
if [[ "$str1" == "$str2" ]]; then
    echo "Strings are equal"
fi

if [[ -z "$str" ]]; then
    echo "String is empty"
fi

if [[ -n "$str" ]]; then
    echo "String is not empty"
fi

# Pattern matching (in [[ ]])
if [[ "$name" == J* ]]; then
    echo "Starts with J"
fi

# Regex matching
if [[ "$email" =~ ^[a-z]+@[a-z]+\.[a-z]+$ ]]; then
    echo "Valid email format"
fi
```

#### File Test Operators
```bash
-e file     # File exists
-f file     # Is regular file
-d file     # Is directory
-L file     # Is symbolic link
-r file     # Is readable
-w file     # Is writable
-x file     # Is executable
-s file     # File size > 0
-O file     # Owned by current user
-G file     # Owned by current group
file1 -nt file2    # file1 newer than file2
file1 -ot file2    # file1 older than file2

# Examples
if [[ -e "$file" ]]; then
    echo "File exists"
fi

if [[ -d "$dir" ]]; then
    echo "Is a directory"
fi

if [[ -f "$file" && -r "$file" ]]; then
    echo "File exists and is readable"
fi
```

#### Logical Operators
```bash
# Inside [[ ]]
&&      # AND
||      # OR
!       # NOT

# Inside [ ]
-a      # AND
-o      # OR
!       # NOT

# Examples
if [[ $a -gt 5 && $a -lt 10 ]]; then
    echo "Between 5 and 10"
fi

if [[ $a -eq 1 || $a -eq 2 ]]; then
    echo "One or Two"
fi

if [[ ! -f "$file" ]]; then
    echo "File does not exist"
fi
```

---

## Conditional Statements

### if Statement
```bash
# Basic if
if [[ condition ]]; then
    commands
fi

# if-else
if [[ condition ]]; then
    commands
else
    other_commands
fi

# if-elif-else
if [[ condition1 ]]; then
    commands1
elif [[ condition2 ]]; then
    commands2
elif [[ condition3 ]]; then
    commands3
else
    default_commands
fi

# Practical example
#!/bin/bash
read -p "Enter a number: " num

if [[ $num -gt 0 ]]; then
    echo "Positive"
elif [[ $num -lt 0 ]]; then
    echo "Negative"
else
    echo "Zero"
fi
```

### case Statement
```bash
case $variable in
    pattern1)
        commands1
        ;;
    pattern2|pattern3)
        commands2
        ;;
    *)
        default_commands
        ;;
esac

# Practical example
#!/bin/bash
read -p "Enter a fruit: " fruit

case $fruit in
    apple|Apple)
        echo "It's an apple"
        ;;
    banana)
        echo "It's a banana"
        ;;
    [Oo]range)
        echo "It's an orange"
        ;;
    *)
        echo "Unknown fruit"
        ;;
esac
```

### Ternary-like Operations
```bash
# Using && and ||
[[ $a -gt $b ]] && echo "a is greater" || echo "b is greater or equal"

# Using arithmetic
max=$(( a > b ? a : b ))
```

---

## Loops

### for Loop
```bash
# Basic for loop
for i in 1 2 3 4 5; do
    echo "Number: $i"
done

# Range
for i in {1..10}; do
    echo "Number: $i"
done

# Range with step
for i in {0..100..10}; do
    echo "Number: $i"
done

# C-style for loop
for ((i=0; i<10; i++)); do
    echo "Number: $i"
done

# Loop through files
for file in *.txt; do
    echo "File: $file"
done

for file in /var/log/*; do
    echo "Processing: $file"
done

# Loop through command output
for user in $(cat /etc/passwd | cut -d: -f1); do
    echo "User: $user"
done

# Loop through array
colors=(red green blue)
for color in "${colors[@]}"; do
    echo "Color: $color"
done

# Loop with index
for i in "${!colors[@]}"; do
    echo "Index $i: ${colors[$i]}"
done
```

### while Loop
```bash
# Basic while
count=1
while [[ $count -le 5 ]]; do
    echo "Count: $count"
    ((count++))
done

# Infinite loop
while true; do
    echo "Press Ctrl+C to stop"
    sleep 1
done

# Read file line by line
while IFS= read -r line; do
    echo "$line"
done < file.txt

# Process with while
while read -r user shell; do
    echo "User: $user, Shell: $shell"
done < <(cat /etc/passwd | cut -d: -f1,7 | tr ':' ' ')

# While with command
while ping -c 1 google.com &>/dev/null; do
    echo "Network is up"
    sleep 5
done
echo "Network is down"
```

### until Loop
```bash
# Until (opposite of while)
count=1
until [[ $count -gt 5 ]]; do
    echo "Count: $count"
    ((count++))
done

# Wait for condition
until ping -c 1 server.example.com &>/dev/null; do
    echo "Waiting for server..."
    sleep 2
done
echo "Server is up!"
```

### Loop Control
```bash
# break - exit loop
for i in {1..10}; do
    if [[ $i -eq 5 ]]; then
        break
    fi
    echo $i
done

# continue - skip iteration
for i in {1..10}; do
    if [[ $i -eq 5 ]]; then
        continue
    fi
    echo $i
done

# break from nested loops
for i in {1..3}; do
    for j in {1..3}; do
        if [[ $j -eq 2 ]]; then
            break 2      # Break out of 2 levels
        fi
        echo "$i, $j"
    done
done
```

---

## Functions

### Defining Functions
```bash
# Style 1
function my_function {
    echo "Hello from function"
}

# Style 2 (POSIX compatible)
my_function() {
    echo "Hello from function"
}

# Calling function
my_function
```

### Function Arguments
```bash
greet() {
    echo "Hello, $1!"              # First argument
    echo "You are $2 years old"    # Second argument
    echo "All args: $@"
    echo "Number of args: $#"
}

greet "John" 25
```

### Return Values
```bash
# Return status (0-255)
is_even() {
    if (( $1 % 2 == 0 )); then
        return 0      # Success/true
    else
        return 1      # Failure/false
    fi
}

if is_even 4; then
    echo "4 is even"
fi

# Return string via echo
get_greeting() {
    echo "Hello, $1!"
}

message=$(get_greeting "World")
echo "$message"

# Return via global variable
calculate() {
    RESULT=$(( $1 + $2 ))
}

calculate 5 3
echo "Result: $RESULT"
```

### Local Variables
```bash
my_function() {
    local name="John"     # Local to function
    global_var="Global"   # Global
    echo "Inside: $name"
}

my_function
echo "Outside: $name"         # Empty
echo "Global: $global_var"    # Global
```

### Recursive Functions
```bash
# Factorial
factorial() {
    if [[ $1 -le 1 ]]; then
        echo 1
    else
        local prev=$(factorial $(( $1 - 1 )))
        echo $(( $1 * prev ))
    fi
}

echo "5! = $(factorial 5)"    # 120
```

---

## Arrays

### Indexed Arrays
```bash
# Declare array
declare -a my_array
my_array=(one two three)

# Alternative
colors[0]="red"
colors[1]="green"
colors[2]="blue"

# Access elements
echo ${colors[0]}             # First element
echo ${colors[-1]}            # Last element (Bash 4+)

# All elements
echo ${colors[@]}             # As separate words
echo ${colors[*]}             # As single word

# Array length
echo ${#colors[@]}

# Array indices
echo ${!colors[@]}

# Add element
colors+=(yellow)
colors+=("light blue")

# Remove element
unset colors[1]

# Slice
echo ${colors[@]:1:2}         # From index 1, 2 elements

# Loop through array
for color in "${colors[@]}"; do
    echo "$color"
done

# Loop with index
for i in "${!colors[@]}"; do
    echo "$i: ${colors[$i]}"
done
```

### Associative Arrays (Bash 4+)
```bash
# Declare
declare -A user

# Assign
user[name]="John"
user[age]=25
user[email]="john@example.com"

# Or inline
declare -A user=([name]="John" [age]=25 [email]="john@example.com")

# Access
echo ${user[name]}

# All keys
echo ${!user[@]}

# All values
echo ${user[@]}

# Check if key exists
if [[ -v user[name] ]]; then
    echo "Name exists"
fi

# Loop
for key in "${!user[@]}"; do
    echo "$key: ${user[$key]}"
done
```

---

## String Manipulation

### String Operations
```bash
str="Hello World"

# Length
echo ${#str}                  # 11

# Concatenation
str1="Hello"
str2="World"
str3="$str1 $str2"            # Hello World
str3="${str1}${str2}"         # HelloWorld

# Substring
echo ${str:0:5}               # Hello
echo ${str:6}                 # World
echo ${str: -5}               # World (note space)

# Find and replace
echo ${str/World/Universe}    # Hello Universe
echo ${str//o/0}              # Hell0 W0rld (all)

# Remove pattern
filename="document.txt"
echo ${filename%.txt}         # document
echo ${filename##*.}          # txt

# Case conversion (Bash 4+)
echo ${str,,}                 # hello world
echo ${str^^}                 # HELLO WORLD

# Check if contains substring
if [[ "$str" == *"World"* ]]; then
    echo "Contains World"
fi

# Split string
IFS=',' read -ra parts <<< "a,b,c,d"
for part in "${parts[@]}"; do
    echo "$part"
done
```

---

## File Operations

### Reading Files
```bash
# Read entire file
content=$(cat file.txt)
content=$(<file.txt)

# Read line by line
while IFS= read -r line; do
    echo "$line"
done < file.txt

# Read with line numbers
n=1
while IFS= read -r line; do
    echo "$n: $line"
    ((n++))
done < file.txt

# Read specific line
sed -n '5p' file.txt

# Read fields
while IFS=':' read -r user pass uid gid info home shell; do
    echo "User: $user, Shell: $shell"
done < /etc/passwd
```

### Writing Files
```bash
# Write (overwrite)
echo "content" > file.txt

# Append
echo "more content" >> file.txt

# Multiple lines
cat > file.txt << EOF
Line 1
Line 2
Line 3
EOF

# Without variable expansion
cat > file.txt << 'EOF'
$HOME stays as literal
EOF

# Write from variable
echo "$content" > file.txt
printf "%s\n" "$content" > file.txt
```

### File Checks
```bash
#!/bin/bash

file="$1"

if [[ ! -e "$file" ]]; then
    echo "File does not exist"
    exit 1
fi

if [[ -f "$file" ]]; then
    echo "Regular file"
elif [[ -d "$file" ]]; then
    echo "Directory"
elif [[ -L "$file" ]]; then
    echo "Symbolic link"
fi

if [[ -r "$file" ]]; then echo "Readable"; fi
if [[ -w "$file" ]]; then echo "Writable"; fi
if [[ -x "$file" ]]; then echo "Executable"; fi
if [[ -s "$file" ]]; then echo "Not empty"; fi
```

---

## Command Line Arguments

### Handling Arguments
```bash
#!/bin/bash

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "All arguments: $@"
echo "Number of arguments: $#"

# Shift arguments
while [[ $# -gt 0 ]]; do
    echo "Processing: $1"
    shift
done
```

### getopts - Parse Options
```bash
#!/bin/bash

usage() {
    echo "Usage: $0 [-v] [-f file] [-n number]"
    exit 1
}

verbose=false
file=""
number=0

while getopts "vf:n:h" opt; do
    case $opt in
        v)
            verbose=true
            ;;
        f)
            file="$OPTARG"
            ;;
        n)
            number="$OPTARG"
            ;;
        h)
            usage
            ;;
        \?)
            echo "Invalid option: -$OPTARG"
            usage
            ;;
        :)
            echo "Option -$OPTARG requires an argument"
            usage
            ;;
    esac
done

# Shift past processed options
shift $((OPTIND - 1))

# Remaining arguments
echo "Remaining args: $@"

echo "Verbose: $verbose"
echo "File: $file"
echo "Number: $number"
```

### Long Options with getopt
```bash
#!/bin/bash

# Using getopt (external command)
OPTS=$(getopt -o vf:n:h --long verbose,file:,number:,help -n "$0" -- "$@")
eval set -- "$OPTS"

verbose=false
file=""
number=0

while true; do
    case "$1" in
        -v|--verbose)
            verbose=true
            shift
            ;;
        -f|--file)
            file="$2"
            shift 2
            ;;
        -n|--number)
            number="$2"
            shift 2
            ;;
        -h|--help)
            echo "Usage: $0 [options]"
            exit 0
            ;;
        --)
            shift
            break
            ;;
        *)
            break
            ;;
    esac
done

echo "Verbose: $verbose"
echo "File: $file"
echo "Number: $number"
```

---

## Exit Codes and Error Handling

### Exit Codes
```bash
# Exit with code
exit 0          # Success
exit 1          # General error
exit 2          # Misuse of command

# Check exit code
command
if [[ $? -eq 0 ]]; then
    echo "Success"
else
    echo "Failed with code $?"
fi

# Common exit codes
# 0   - Success
# 1   - General error
# 2   - Misuse of command
# 126 - Permission denied
# 127 - Command not found
# 128+n - Killed by signal n
```

### Error Handling
```bash
#!/bin/bash

# Exit on error
set -e

# Exit on undefined variable
set -u

# Exit on pipe failure
set -o pipefail

# All together (recommended)
set -euo pipefail

# Trap errors
trap 'echo "Error on line $LINENO"; exit 1' ERR

# Cleanup on exit
cleanup() {
    rm -f "$temp_file"
    echo "Cleaned up"
}
trap cleanup EXIT

# Error handling function
die() {
    echo "ERROR: $1" >&2
    exit "${2:-1}"
}

# Usage
[[ -f "$file" ]] || die "File not found: $file" 2
```

### Debugging
```bash
# Run with debug
bash -x script.sh

# Enable debug in script
set -x          # Enable
set +x          # Disable

# Debug section
set -x
commands_to_debug
set +x

# Print commands before execution
#!/bin/bash -x

# Verbose mode
set -v          # Print input lines
set +v          # Disable
```

---

## Best Practices

### Script Template
```bash
#!/bin/bash
#
# Script: script_name.sh
# Description: What this script does
# Author: Your Name
# Date: 2026-01-28
# Version: 1.0
#

set -euo pipefail

# Constants
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_NAME="$(basename "$0")"

# Global variables
LOG_FILE="/var/log/${SCRIPT_NAME}.log"

# Functions
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

die() {
    log "ERROR: $1" >&2
    exit "${2:-1}"
}

usage() {
    cat << EOF
Usage: $SCRIPT_NAME [options]

Options:
    -h, --help      Show this help
    -v, --verbose   Enable verbose mode

EOF
    exit 0
}

cleanup() {
    # Cleanup code
    log "Cleanup complete"
}

# Main
main() {
    log "Starting $SCRIPT_NAME"
    
    # Main logic here
    
    log "Completed successfully"
}

# Traps
trap cleanup EXIT
trap 'die "Interrupted"' INT TERM

# Parse arguments
while [[ $# -gt 0 ]]; do
    case "$1" in
        -h|--help)
            usage
            ;;
        -v|--verbose)
            set -x
            shift
            ;;
        *)
            die "Unknown option: $1"
            ;;
    esac
done

# Run main
main "$@"
```

### Style Guidelines
```bash
# Use lowercase for variables
my_variable="value"

# Use UPPERCASE for constants/environment
readonly MAX_RETRIES=3
export PATH

# Use meaningful names
user_count=10           # Good
uc=10                   # Bad

# Quote variables
echo "$variable"        # Good
echo $variable          # Risky

# Use [[ ]] instead of [ ]
[[ -f "$file" ]]        # Preferred
[ -f "$file" ]          # POSIX, less features

# Use $(command) instead of backticks
result=$(ls -la)        # Preferred
result=`ls -la`         # Deprecated

# Check command existence
if command -v docker &>/dev/null; then
    echo "Docker is installed"
fi

# Use functions for repeated code
# Use local variables in functions
# Add comments for complex logic
# Handle errors appropriately
```

---

## Practical Examples

### Backup Script
```bash
#!/bin/bash
set -euo pipefail

SOURCE_DIR="${1:-/home}"
BACKUP_DIR="${2:-/backup}"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_${DATE}.tar.gz"

log() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*"; }

log "Starting backup of $SOURCE_DIR"

mkdir -p "$BACKUP_DIR"

tar -czf "${BACKUP_DIR}/${BACKUP_FILE}" "$SOURCE_DIR" 2>/dev/null

# Keep only last 7 backups
cd "$BACKUP_DIR"
ls -t backup_*.tar.gz | tail -n +8 | xargs -r rm -f

log "Backup complete: ${BACKUP_FILE}"
log "Size: $(du -h "${BACKUP_DIR}/${BACKUP_FILE}" | cut -f1)"
```

### Log Analyzer
```bash
#!/bin/bash

LOG_FILE="${1:-/var/log/messages}"

echo "=== Log Analysis for $LOG_FILE ==="
echo

echo "Error count by type:"
grep -i "error\|fail\|warning" "$LOG_FILE" 2>/dev/null | \
    awk '{print $5}' | sort | uniq -c | sort -rn | head -10

echo
echo "Top 10 IPs (if applicable):"
grep -oE '\b([0-9]{1,3}\.){3}[0-9]{1,3}\b' "$LOG_FILE" 2>/dev/null | \
    sort | uniq -c | sort -rn | head -10

echo
echo "Events by hour:"
awk '{print $3}' "$LOG_FILE" 2>/dev/null | cut -d: -f1 | \
    sort | uniq -c | sort -k2n
```

### System Health Check
```bash
#!/bin/bash

echo "=== System Health Check ==="
echo "Date: $(date)"
echo "Hostname: $(hostname)"
echo

echo "=== CPU Usage ==="
top -bn1 | head -5

echo
echo "=== Memory Usage ==="
free -h

echo
echo "=== Disk Usage ==="
df -h | grep -v tmpfs

echo
echo "=== Top Processes ==="
ps aux --sort=-%mem | head -6

echo
echo "=== Failed Services ==="
systemctl --failed 2>/dev/null || echo "N/A"

echo
echo "=== Recent Errors ==="
journalctl -p err --since "1 hour ago" 2>/dev/null | tail -10 || \
    tail -20 /var/log/messages | grep -i error
```

---

## Quick Reference Card

### Variable Operations
| Syntax | Description |
|--------|-------------|
| `${var:-default}` | Use default if unset |
| `${var:=default}` | Set default if unset |
| `${#var}` | String length |
| `${var:start:len}` | Substring |
| `${var/old/new}` | Replace first |
| `${var//old/new}` | Replace all |
| `${var#pattern}` | Remove prefix |
| `${var%pattern}` | Remove suffix |

### Test Operators
| Operator | Description |
|----------|-------------|
| `-eq`, `-ne` | Equal, not equal (int) |
| `-gt`, `-lt` | Greater, less than |
| `-ge`, `-le` | Greater/less or equal |
| `==`, `!=` | String comparison |
| `-z`, `-n` | Empty, not empty |
| `-e`, `-f`, `-d` | Exists, file, directory |
| `-r`, `-w`, `-x` | Readable, writable, executable |

### Control Flow
```bash
# If
if [[ cond ]]; then cmd; fi

# For
for i in list; do cmd; done
for ((i=0;i<10;i++)); do cmd; done

# While
while [[ cond ]]; do cmd; done

# Case
case $var in pat) cmd;; esac

# Function
func() { commands; }
```

---

**[← Back to Index](README.md)**  
**Previous: [Storage & Filesystems](08_storage_filesystems.md)**  
**Next: [SELinux & Security](10_selinux_security.md)**
