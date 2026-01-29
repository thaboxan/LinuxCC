# Linux Crash Course - Process Management

## Table of Contents
- [Understanding Processes](#understanding-processes)
- [Viewing Processes (ps)](#viewing-processes-ps)
- [Real-time Monitoring (top, htop)](#real-time-monitoring-top-htop)
- [Managing Processes (kill, signals)](#managing-processes-kill-signals)
- [Background and Foreground Jobs](#background-and-foreground-jobs)
- [Process Priority (nice, renice)](#process-priority-nice-renice)
- [systemd and Services](#systemd-and-services)
- [Scheduling Tasks (cron, at)](#scheduling-tasks-cron-at)
- [System Logging](#system-logging)

---

## Understanding Processes

### What is a Process?
A **process** is an instance of a running program. Every command you run creates at least one process.

```
┌─────────────────────────────────────────────────────────┐
│                    Process Hierarchy                     │
├─────────────────────────────────────────────────────────┤
│  systemd (PID 1) ─── The first process, parent of all  │
│      │                                                   │
│      ├── sshd ─── SSH daemon                            │
│      │     └── sshd ─── SSH session                     │
│      │           └── bash ─── User shell                │
│      │                 └── vim ─── Editor               │
│      │                                                   │
│      ├── httpd ─── Web server                           │
│      │     ├── httpd ─── Worker process                 │
│      │     └── httpd ─── Worker process                 │
│      │                                                   │
│      └── crond ─── Cron scheduler                       │
└─────────────────────────────────────────────────────────┘
```

### Process States
| State | Symbol | Description |
|-------|--------|-------------|
| Running | R | Currently executing or ready to run |
| Sleeping | S | Waiting for an event (interruptible) |
| Disk Sleep | D | Waiting for I/O (uninterruptible) |
| Stopped | T | Stopped by signal or debugger |
| Zombie | Z | Completed but parent hasn't read exit status |

### Process Attributes
| Attribute | Description |
|-----------|-------------|
| PID | Process ID - unique identifier |
| PPID | Parent Process ID |
| UID | User ID running the process |
| GID | Group ID |
| TTY | Terminal associated with process |
| STAT | Process state |
| TIME | CPU time used |
| CMD | Command that started the process |

---

## Viewing Processes (ps)

### Basic ps Commands
```bash
# Current shell processes
ps

# All processes (BSD style)
ps aux

# All processes (Unix style)
ps -ef

# All processes with full format
ps -eF
```

### ps aux Explained
```bash
ps aux

# Output columns:
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1 171820 13456 ?        Ss   Jan01   0:05 /usr/lib/systemd/systemd
│            │   │    │     │     │   │         │      │      │   │
│            │   │    │     │     │   │         │      │      │   └── Command
│            │   │    │     │     │   │         │      │      └── CPU time
│            │   │    │     │     │   │         │      └── Start time
│            │   │    │     │     │   │         └── State
│            │   │    │     │     │   └── Terminal (? = no terminal)
│            │   │    │     │     └── Resident Set Size (KB in RAM)
│            │   │    │     └── Virtual memory (KB)
│            │   │    └── Memory usage %
│            │   └── CPU usage %
│            └── Process ID
└── User running process
```

### Common ps Options
```bash
# BSD style options (no dash)
ps aux                            # All processes, user-oriented
ps axjf                          # Process tree

# Unix style options (with dash)
ps -e                            # All processes
ps -f                            # Full format
ps -ef                           # All, full format
ps -eF                           # Extra full format
ps -ely                          # Long format with priority

# Specific user
ps -u username
ps aux | grep "^username"
ps -U username -u username u

# Specific process by PID
ps -p 1234
ps -p 1234,5678,9012

# Specific process by name
ps -C httpd
pgrep httpd

# Custom output format
ps -eo pid,ppid,user,%cpu,%mem,stat,cmd
ps -eo pid,ppid,user,nice,stat,cmd --sort=-%cpu

# Sort by CPU or memory
ps aux --sort=-%cpu | head       # Top CPU consumers
ps aux --sort=-%mem | head       # Top memory consumers

# Process tree
ps -ejH                          # Tree view
ps axjf                          # Forest view
pstree                           # Visual tree
pstree -p                        # With PIDs
pstree username                  # User's processes
```

### Finding Processes
```bash
# By name using grep
ps aux | grep httpd
ps aux | grep -v grep | grep httpd  # Exclude grep itself

# Using pgrep
pgrep httpd                      # Get PIDs
pgrep -l httpd                   # With process name
pgrep -u apache httpd            # By user
pgrep -a httpd                   # With full command
pgrep -c httpd                   # Count only

# Using pidof
pidof httpd                      # Get PIDs

# Process tree for specific process
pstree -p $(pgrep -o httpd)
```

---

## Real-time Monitoring (top, htop)

### top - Interactive Process Viewer
```bash
# Start top
top

# Top keyboard commands:
# q         → Quit
# h or ?    → Help
# Space     → Refresh now
# k         → Kill process (enter PID)
# r         → Renice process
# u         → Filter by user
# M         → Sort by memory
# P         → Sort by CPU (default)
# T         → Sort by time
# c         → Show full command line
# 1         → Show individual CPUs
# z         → Color mode
# W         → Save configuration
```

### top Header Explained
```
top - 10:30:45 up 15 days,  2:35,  3 users,  load average: 0.15, 0.20, 0.18
Tasks: 256 total,   1 running, 255 sleeping,   0 stopped,   0 zombie
%Cpu(s):  2.3 us,  1.2 sy,  0.0 ni, 96.2 id,  0.1 wa,  0.1 hi,  0.1 si,  0.0 st
MiB Mem :  16000.0 total,   8000.0 free,   5000.0 used,   3000.0 buff/cache
MiB Swap:   8000.0 total,   8000.0 free,      0.0 used.  10000.0 avail Mem

# Line 1: uptime, users, load average (1, 5, 15 min)
# Line 2: task counts by state
# Line 3: CPU breakdown
#   us = user space
#   sy = system/kernel
#   ni = nice (low priority)
#   id = idle
#   wa = I/O wait
#   hi = hardware interrupts
#   si = software interrupts
#   st = steal (virtualization)
# Lines 4-5: Memory and swap usage
```

### top Options
```bash
# Batch mode (for scripts)
top -b -n 1 > top_output.txt

# Specific user
top -u username

# Specific PIDs
top -p 1234,5678

# Update interval (seconds)
top -d 5

# Show threads
top -H

# Number of iterations
top -n 10                        # Run 10 times then exit
```

### htop - Enhanced top (if installed)
```bash
# Install htop (RHEL/CentOS)
sudo dnf install htop

# Run htop
htop

# htop advantages over top:
# - Better visual interface
# - Mouse support
# - Horizontal and vertical scrolling
# - Easy process killing (F9)
# - Easy search (F3)
# - Tree view (F5)
# - Sort columns (F6)
# - Filter (F4)
```

### Other Monitoring Tools
```bash
# vmstat - Virtual memory statistics
vmstat 2 5                       # Every 2 seconds, 5 times
# Output: procs, memory, swap, io, system, cpu

# iostat - I/O statistics
iostat 2 5
iostat -x                        # Extended stats

# mpstat - CPU statistics
mpstat                           # All CPUs average
mpstat -P ALL                    # Per CPU

# free - Memory usage
free -h                          # Human readable
free -s 2                        # Refresh every 2 seconds

# uptime - System load
uptime

# watch - Run command periodically
watch -n 2 'ps aux | head -10'   # Every 2 seconds
watch df -h                      # Watch disk space
```

---

## Managing Processes (kill, signals)

### Understanding Signals
Signals are software interrupts sent to processes.

| Signal | Number | Action | Description |
|--------|--------|--------|-------------|
| SIGHUP | 1 | Terminate | Hangup - often used to reload config |
| SIGINT | 2 | Terminate | Interrupt (Ctrl+C) |
| SIGQUIT | 3 | Core dump | Quit (Ctrl+\\) |
| SIGKILL | 9 | Terminate | Force kill (cannot be caught) |
| SIGTERM | 15 | Terminate | Graceful termination (default) |
| SIGSTOP | 19 | Stop | Pause process (cannot be caught) |
| SIGCONT | 18 | Continue | Resume stopped process |
| SIGUSR1 | 10 | User defined | Application specific |
| SIGUSR2 | 12 | User defined | Application specific |

### kill - Send Signals to Processes
```bash
# Default signal (SIGTERM - 15)
kill PID
kill 1234

# Specific signal by number
kill -9 1234                     # SIGKILL
kill -15 1234                    # SIGTERM

# Specific signal by name
kill -SIGKILL 1234
kill -KILL 1234
kill -HUP 1234                   # Reload configuration

# Kill multiple processes
kill 1234 5678 9012

# List all signals
kill -l
```

### killall - Kill by Name
```bash
# Kill all processes by name
killall httpd
killall -9 httpd                 # Force kill

# Interactive (confirm each)
killall -i httpd

# Kill by user
killall -u username

# Wait for processes to die
killall -w httpd

# Only show what would be killed
killall -s 0 httpd               # Dry run
```

### pkill - Kill by Pattern
```bash
# Kill by process name pattern
pkill httpd
pkill -9 httpd

# Kill by user
pkill -u username
pkill -u username httpd

# Kill by terminal
pkill -t pts/1

# Kill newest/oldest matching
pkill -n httpd                   # Newest
pkill -o httpd                   # Oldest

# Kill parent and children
pkill -P 1234                    # Children of PID 1234
```

### Practical Kill Examples
```bash
# Graceful shutdown
kill -15 $(pgrep nginx)

# Force kill unresponsive process
kill -9 $(pidof hung_process)

# Reload web server config
kill -HUP $(cat /var/run/nginx.pid)
systemctl reload nginx           # Preferred method

# Kill all user processes
pkill -9 -u troublemaker

# Kill processes using a file
fuser -k /var/log/myapp.log

# Kill processes on a port
fuser -k 8080/tcp
```

---

## Background and Foreground Jobs

### Running Processes in Background
```bash
# Start in background
command &
sleep 100 &

# Move running process to background
# 1. Press Ctrl+Z (stops the process)
# 2. Type 'bg' (continues in background)

# Start and immediately background
nohup command &                  # Survives logout
nohup command > output.log 2>&1 &
```

### Job Control Commands
```bash
# List background jobs
jobs
jobs -l                          # With PIDs

# Bring to foreground
fg                               # Last background job
fg %1                            # Job number 1
fg %+                            # Current job
fg %-                            # Previous job
fg %command                      # By command name

# Send to background
bg                               # Continue last stopped job
bg %1                            # Job number 1

# Stop/pause running process
Ctrl+Z                           # Sends SIGSTOP

# Terminate foreground process
Ctrl+C                           # Sends SIGINT
```

### nohup - Survive Logout
```bash
# Run command that survives logout
nohup command &

# With output redirection
nohup command > output.log 2>&1 &

# Check output
cat nohup.out                    # Default output file
```

### disown - Detach from Shell
```bash
# Start background job
command &

# Disown the job (survives shell exit)
disown %1
disown -a                        # All jobs
disown -h %1                     # Keep job, but don't SIGHUP on exit
```

### screen and tmux
```bash
# screen - Terminal multiplexer
screen                           # Start new session
screen -S mysession              # Named session
# Ctrl+A, D  = Detach
screen -ls                       # List sessions
screen -r mysession              # Reattach

# tmux - Better terminal multiplexer
tmux                             # Start new session
tmux new -s mysession            # Named session
# Ctrl+B, D  = Detach
tmux ls                          # List sessions
tmux attach -t mysession         # Reattach
```

---

## Process Priority (nice, renice)

### Understanding Priority
```
Priority Range: -20 (highest) to +19 (lowest)
Default: 0

Lower number = Higher priority
Higher number = Lower priority

Only root can set negative nice values
```

### nice - Start Process with Priority
```bash
# Start with nice value
nice command
nice -n 10 command               # Nice value 10 (lower priority)
nice -n -5 command               # Nice value -5 (higher priority, root only)

# Examples
nice -n 19 tar -czf backup.tar.gz /data    # Low priority backup
nice -n -10 critical_process               # High priority (root)

# Default nice value
nice                             # Shows 0
```

### renice - Change Running Process Priority
```bash
# Change by PID
renice -n 10 -p 1234
renice 10 1234                   # Same as above

# Change by user (all their processes)
renice -n 5 -u username
sudo renice -n -5 -u root        # Increase root process priority

# Change by process group
renice -n 10 -g 1234

# Examples
renice -n 19 -p $(pgrep rsync)   # Lower backup priority
sudo renice -n -10 -p $(pgrep mysql)  # Prioritize database
```

### Viewing Priority
```bash
# ps with nice value
ps -eo pid,nice,cmd
ps -eo pid,ni,pri,cmd            # ni=nice, pri=priority

# top shows NI column
top
```

---

## systemd and Services

### Understanding systemd
**systemd** is the system and service manager in RHEL 7+. It manages:
- System startup and services
- Logging (journald)
- Login sessions (logind)
- Network (networkd, optional)

### Unit Types
| Type | Extension | Description |
|------|-----------|-------------|
| Service | .service | Daemon or one-shot process |
| Socket | .socket | IPC or network socket |
| Target | .target | Group of units |
| Mount | .mount | Filesystem mount point |
| Timer | .timer | Scheduled activation |
| Device | .device | Hardware device |
| Path | .path | File/directory monitoring |

### systemctl - Service Management
```bash
# Start/Stop/Restart
sudo systemctl start httpd
sudo systemctl stop httpd
sudo systemctl restart httpd
sudo systemctl reload httpd      # Reload config without restart

# Enable/Disable (boot)
sudo systemctl enable httpd      # Start on boot
sudo systemctl disable httpd     # Don't start on boot
sudo systemctl enable --now httpd  # Enable and start

# Status
systemctl status httpd
systemctl is-active httpd
systemctl is-enabled httpd
systemctl is-failed httpd

# View all services
systemctl list-units --type=service
systemctl list-units --type=service --state=running
systemctl list-units --type=service --state=failed
systemctl list-unit-files --type=service

# Dependencies
systemctl list-dependencies httpd
systemctl list-dependencies --reverse httpd  # What depends on it

# Mask/Unmask (prevent starting)
sudo systemctl mask httpd        # Cannot be started
sudo systemctl unmask httpd
```

### Unit File Locations
```bash
/etc/systemd/system/             # Admin-created (highest priority)
/run/systemd/system/             # Runtime units
/usr/lib/systemd/system/         # Package-installed units

# View unit file
systemctl cat httpd

# Edit unit file (creates override)
sudo systemctl edit httpd        # Drop-in file
sudo systemctl edit --full httpd # Full copy to edit

# Reload after editing
sudo systemctl daemon-reload
```

### Creating a Service
```bash
# Create service file
sudo vim /etc/systemd/system/myapp.service
```

```ini
[Unit]
Description=My Custom Application
After=network.target
Wants=network-online.target

[Service]
Type=simple
User=myappuser
Group=myappgroup
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/bin/myapp --config /etc/myapp/config.yml
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
# Enable and start
sudo systemctl daemon-reload
sudo systemctl enable --now myapp
```

### Targets (Runlevels)
```bash
# Common targets
multi-user.target                # Text mode, networking (runlevel 3)
graphical.target                 # GUI mode (runlevel 5)
rescue.target                    # Single user (runlevel 1)
emergency.target                 # Minimal boot

# View default target
systemctl get-default

# Change default target
sudo systemctl set-default multi-user.target

# Change target now
sudo systemctl isolate multi-user.target

# Boot options (GRUB)
# Append to kernel line:
# systemd.unit=rescue.target
# systemd.unit=emergency.target
```

### journalctl - View Logs
```bash
# View all logs
journalctl

# Follow logs (like tail -f)
journalctl -f

# Specific unit
journalctl -u httpd
journalctl -u httpd -f

# Since boot
journalctl -b
journalctl -b -1                 # Previous boot

# By time
journalctl --since "2026-01-01"
journalctl --since "1 hour ago"
journalctl --since "2026-01-01" --until "2026-01-02"

# By priority
journalctl -p err                # Error and above
journalctl -p warning

# Kernel messages
journalctl -k
journalctl --dmesg

# Output formats
journalctl -o json
journalctl -o json-pretty
journalctl -o short-precise

# Disk usage
journalctl --disk-usage

# Clean old logs
sudo journalctl --vacuum-time=7d
sudo journalctl --vacuum-size=500M
```

---

## Scheduling Tasks (cron, at)

### cron - Recurring Tasks

#### Crontab Syntax
```
┌───────────── minute (0-59)
│ ┌───────────── hour (0-23)
│ │ ┌───────────── day of month (1-31)
│ │ │ ┌───────────── month (1-12)
│ │ │ │ ┌───────────── day of week (0-6, Sunday=0)
│ │ │ │ │
* * * * * command to execute

# Special characters:
*     Any value
,     List separator (1,3,5)
-     Range (1-5)
/     Step (*/5 = every 5)
```

#### Managing Crontab
```bash
# Edit user crontab
crontab -e

# List crontab
crontab -l

# Remove all cron jobs
crontab -r

# Edit another user's crontab (root)
sudo crontab -u username -e
```

#### Crontab Examples
```bash
# Every minute
* * * * * /path/to/script.sh

# Every 5 minutes
*/5 * * * * /path/to/script.sh

# Every hour at minute 30
30 * * * * /path/to/script.sh

# Daily at 2:30 AM
30 2 * * * /path/to/script.sh

# Weekly on Sunday at midnight
0 0 * * 0 /path/to/script.sh

# Monthly on 1st at 3 AM
0 3 1 * * /path/to/script.sh

# Monday to Friday at 9 AM
0 9 * * 1-5 /path/to/script.sh

# Every 15 minutes during business hours
*/15 9-17 * * 1-5 /path/to/script.sh

# With output handling
0 * * * * /path/to/script.sh >> /var/log/myjob.log 2>&1
0 * * * * /path/to/script.sh > /dev/null 2>&1  # No output
```

#### System Cron Directories
```bash
/etc/crontab              # System crontab
/etc/cron.d/              # Drop-in cron files
/etc/cron.hourly/         # Scripts run hourly
/etc/cron.daily/          # Scripts run daily
/etc/cron.weekly/         # Scripts run weekly
/etc/cron.monthly/        # Scripts run monthly

# View system crontab
cat /etc/crontab
ls /etc/cron.d/

# /etc/crontab format includes username:
# minute hour day month dow USER command
0 4 * * * root /usr/local/bin/backup.sh
```

### at - One-time Tasks
```bash
# Schedule a one-time job
at 2:30 PM
at> /path/to/script.sh
at> Ctrl+D

# Various time formats
at now + 5 minutes
at now + 1 hour
at now + 2 days
at 10:00 AM tomorrow
at 2:30 PM Dec 25
at midnight
at noon

# From file
at 2:30 PM -f /path/to/commands.txt

# List pending jobs
atq
at -l

# View job details
at -c job_number

# Remove job
atrm job_number
at -d job_number
```

### systemd Timers (Modern Alternative)
```bash
# Create timer file
sudo vim /etc/systemd/system/mybackup.timer
```

```ini
[Unit]
Description=Daily Backup Timer

[Timer]
OnCalendar=daily
OnCalendar=*-*-* 02:30:00
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
# Create corresponding service file
sudo vim /etc/systemd/system/mybackup.service
```

```ini
[Unit]
Description=Daily Backup

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
```

```bash
# Enable and start timer
sudo systemctl enable --now mybackup.timer

# List timers
systemctl list-timers
systemctl list-timers --all
```

---

## System Logging

### Log Locations
```bash
/var/log/                        # Main log directory
/var/log/messages                # General system log (RHEL)
/var/log/secure                  # Authentication log
/var/log/boot.log                # Boot messages
/var/log/cron                    # Cron job log
/var/log/maillog                 # Mail server log
/var/log/dmesg                   # Kernel ring buffer
/var/log/audit/audit.log         # SELinux audit log
/var/log/httpd/                  # Apache logs
/var/log/nginx/                  # Nginx logs
```

### Viewing Logs
```bash
# Traditional logs
tail -f /var/log/messages
tail -100 /var/log/secure
grep "error" /var/log/messages
less /var/log/messages

# journald (systemd)
journalctl
journalctl -u httpd -f
journalctl -p err
journalctl --since "1 hour ago"

# dmesg (kernel messages)
dmesg
dmesg -T                         # Human-readable timestamps
dmesg | tail -50
dmesg -w                         # Follow (like tail -f)
```

### logger - Write to Syslog
```bash
# Write message to syslog
logger "Test message"
logger -p auth.info "Auth message"
logger -t myapp "Application message"

# From script
logger -t backup "Backup completed successfully"
```

### rsyslog Configuration
```bash
# Main config
/etc/rsyslog.conf
/etc/rsyslog.d/*.conf

# Restart after changes
sudo systemctl restart rsyslog
```

---

## Quick Reference Card

### Process Viewing
| Command | Description |
|---------|-------------|
| `ps aux` | All processes (BSD) |
| `ps -ef` | All processes (Unix) |
| `ps -u user` | User's processes |
| `pgrep name` | Find PID by name |
| `top` | Interactive viewer |
| `htop` | Better top |

### Process Control
| Command | Description |
|---------|-------------|
| `kill PID` | Terminate (SIGTERM) |
| `kill -9 PID` | Force kill (SIGKILL) |
| `killall name` | Kill by name |
| `pkill pattern` | Kill by pattern |
| `Ctrl+C` | Interrupt (SIGINT) |
| `Ctrl+Z` | Suspend (SIGSTOP) |

### Job Control
| Command | Description |
|---------|-------------|
| `command &` | Run in background |
| `jobs` | List jobs |
| `fg` | Bring to foreground |
| `bg` | Continue in background |
| `nohup cmd &` | Survive logout |

### systemctl
| Command | Description |
|---------|-------------|
| `systemctl start svc` | Start service |
| `systemctl stop svc` | Stop service |
| `systemctl restart svc` | Restart service |
| `systemctl status svc` | View status |
| `systemctl enable svc` | Start on boot |
| `systemctl disable svc` | Don't start on boot |

### Cron Shortcuts
| Entry | Schedule |
|-------|----------|
| `@reboot` | At startup |
| `@hourly` | Every hour |
| `@daily` | Once a day |
| `@weekly` | Once a week |
| `@monthly` | Once a month |
| `@yearly` | Once a year |

---

**[← Back to Index](README.md)**  
**Previous: [Text Processing](04_text_processing.md)**  
**Next: [Networking](06_networking.md)**
