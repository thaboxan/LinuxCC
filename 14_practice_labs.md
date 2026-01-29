# Linux Crash Course - Practice Labs

Hands-on exercises to reinforce your learning. Each lab includes objectives, tasks, and solutions.

## Table of Contents
- [Linux Crash Course - Practice Labs](#linux-crash-course---practice-labs)
  - [Table of Contents](#table-of-contents)
  - [Lab Environment Setup](#lab-environment-setup)
    - [Option 1: Virtual Machine](#option-1-virtual-machine)
    - [Option 2: Cloud Instance](#option-2-cloud-instance)
    - [Option 3: WSL2 (Windows)](#option-3-wsl2-windows)
  - [Lab 1: File System Navigation](#lab-1-file-system-navigation)
    - [Objectives](#objectives)
    - [Tasks](#tasks)
  - [Lab 2: File Manipulation](#lab-2-file-manipulation)
    - [Objectives](#objectives-1)
    - [Tasks](#tasks-1)
  - [Lab 3: User and Permission Management](#lab-3-user-and-permission-management)
    - [Objectives](#objectives-2)
    - [Tasks](#tasks-2)
  - [Lab 4: Text Processing Challenge](#lab-4-text-processing-challenge)
    - [Objectives](#objectives-3)
    - [Setup](#setup)
    - [Tasks](#tasks-3)
  - [Lab 5: Process Management](#lab-5-process-management)
    - [Objectives](#objectives-4)
    - [Tasks](#tasks-4)
  - [Lab 6: Network Configuration](#lab-6-network-configuration)
    - [Objectives](#objectives-5)
    - [Tasks](#tasks-5)
  - [Lab 7: Package Management](#lab-7-package-management)
    - [Objectives](#objectives-6)
    - [Tasks](#tasks-6)
  - [Lab 8: Storage and LVM](#lab-8-storage-and-lvm)
    - [Objectives](#objectives-7)
    - [Prerequisites](#prerequisites)
    - [Tasks](#tasks-7)
  - [Lab 9: Shell Scripting](#lab-9-shell-scripting)
    - [Objectives](#objectives-8)
    - [Tasks](#tasks-8)
  - [Lab 10: SELinux Troubleshooting](#lab-10-selinux-troubleshooting)
    - [Objectives](#objectives-9)
    - [Tasks](#tasks-9)
  - [Lab 11: Boot Recovery](#lab-11-boot-recovery)
    - [Objectives](#objectives-10)
    - [Tasks (Use VM for safety!)](#tasks-use-vm-for-safety)
  - [Lab 12: Container Deployment](#lab-12-container-deployment)
    - [Objectives](#objectives-11)
    - [Tasks](#tasks-10)
  - [Lab 13: Ansible Automation](#lab-13-ansible-automation)
    - [Objectives](#objectives-12)
    - [Setup](#setup-1)
    - [Tasks](#tasks-11)
  - [Capstone Project](#capstone-project)
    - [Scenario](#scenario)
    - [Requirements](#requirements)
    - [Deliverables](#deliverables)
  - [Lab Completion Checklist](#lab-completion-checklist)

---

## Lab Environment Setup

### Option 1: Virtual Machine
```bash
# Recommended: Use VirtualBox or KVM with a RHEL-based distro
# - Rocky Linux 9
# - AlmaLinux 9
# - CentOS Stream 9
# - Fedora (latest)
```

### Option 2: Cloud Instance
```bash
# AWS, Azure, GCP, or DigitalOcean
# Launch a RHEL/CentOS instance (free tier available)
```

### Option 3: WSL2 (Windows)
```bash
# Limited but works for most exercises
wsl --install -d Ubuntu
```

---

## Lab 1: File System Navigation

### Objectives
- Navigate the Linux file system
- Understand the directory structure
- Use absolute and relative paths

### Tasks

**Task 1.1: Explore the File System**
```
1. Display your current working directory
2. Navigate to the root directory
3. List all directories in root with details
4. Navigate to /var/log using an absolute path
5. Go back to your home directory using three different methods
```

**Task 1.2: Directory Investigation**
```
1. Find out what's in /etc/sysconfig
2. Count how many files are in /etc
3. Find the largest file in /var/log
4. Determine what type of file /etc/passwd is
5. Display the directory tree of /usr (2 levels deep)
```

**Task 1.3: Path Mastery**
```
Starting from /home/user:
1. Navigate to /var/log using only relative paths
2. Navigate to /etc from /var/log using relative paths
3. Create a path that goes up 3 directories and into tmp
```

<details>
<summary>View Hint</summary>

- **Task 1.1:** Use `pwd` for current directory. Remember `~`, bare `cd`, and `$HOME` all go home.
- **Task 1.2:** Try `ls -lS` to sort by size, `file` command reveals file types, and `tree -L` limits depth.
- **Task 1.3:** Use `..` to go up one directory level. Count how many levels you need to ascend.

</details>

<details>
<summary>View Solutions</summary>

**Task 1.1:**
```bash
# 1. Display current directory
pwd

# 2. Navigate to root
cd /

# 3. List directories in root
ls -la /

# 4. Navigate to /var/log
cd /var/log

# 5. Go home (three methods)
cd ~
cd
cd $HOME
```

**Task 1.2:**
```bash
# 1. Contents of /etc/sysconfig
ls -la /etc/sysconfig

# 2. Count files in /etc
ls -1 /etc | wc -l
# or for all files including hidden
find /etc -maxdepth 1 -type f | wc -l

# 3. Largest file in /var/log
ls -lS /var/log | head -5
# or
du -a /var/log | sort -rn | head -5

# 4. File type of /etc/passwd
file /etc/passwd

# 5. Directory tree
tree -L 2 /usr
# or if tree not installed
find /usr -maxdepth 2 -type d
```

**Task 1.3:**
```bash
# From /home/user to /var/log
cd ../../var/log

# From /var/log to /etc
cd ../../etc

# Up 3 and into tmp
cd ../../../tmp
```
</details>

---

## Lab 2: File Manipulation

### Objectives
- Create, copy, move, and delete files and directories
- Work with links
- Understand file compression

### Tasks

**Task 2.1: Create a Project Structure**
```
Create the following directory structure in your home:
~/project/
├── src/
│   ├── main/
│   └── test/
├── docs/
├── config/
└── logs/

Then:
1. Create empty files: README.md, LICENSE, .gitignore in ~/project
2. Create config.yaml in the config directory
3. Create sample.log in logs with today's date in the filename
```

**Task 2.2: File Operations**
```
1. Copy the entire project directory to ~/project-backup
2. Move README.md to docs/
3. Rename LICENSE to LICENSE.txt
4. Create a symbolic link called 'latest-log' pointing to your log file
5. Create a hard link to config.yaml called config-backup.yaml
6. Delete the src/test directory
```

**Task 2.3: Compression Challenge**
```
1. Create a tar archive of the project directory
2. Create a gzip compressed tar archive
3. Create a bzip2 compressed tar archive
4. Compare the sizes of all three
5. Extract the gzip archive to /tmp/extracted
6. List contents of an archive without extracting
```

**Task 2.4: Find and Organize**
```
In /etc:
1. Find all files ending with .conf
2. Find all files modified in the last 24 hours
3. Find all files larger than 100KB
4. Find all symbolic links
5. Copy all .conf files to ~/project/config/ (use find with exec)
```

<details>
<summary>View Hint</summary>

- **Task 2.1:** Use `mkdir -p` with brace expansion `{dir1,dir2}` to create multiple directories at once.
- **Task 2.2:** `ln -s` creates symbolic links, `ln` (no flag) creates hard links.
- **Task 2.3:** Remember tar flags: `c` (create), `x` (extract), `z` (gzip), `j` (bzip2), `v` (verbose), `f` (file).
- **Task 2.4:** `find` with `-exec` runs commands on results. Use `{}` as placeholder, end with `\;`

</details>

<details>
<summary>View Solutions</summary>

**Task 2.1:**
```bash
# Create directory structure
mkdir -p ~/project/{src/{main,test},docs,config,logs}

# Create files
touch ~/project/{README.md,LICENSE,.gitignore}
touch ~/project/config/config.yaml
touch ~/project/logs/sample-$(date +%Y%m%d).log
```

**Task 2.2:**
```bash
# 1. Copy project
cp -r ~/project ~/project-backup

# 2. Move README
mv ~/project/README.md ~/project/docs/

# 3. Rename LICENSE
mv ~/project/LICENSE ~/project/LICENSE.txt

# 4. Symbolic link
ln -s ~/project/logs/sample-*.log ~/project/latest-log

# 5. Hard link
ln ~/project/config/config.yaml ~/project/config/config-backup.yaml

# 6. Delete test directory
rm -r ~/project/src/test
```

**Task 2.3:**
```bash
# 1. Tar archive
tar -cvf project.tar ~/project

# 2. Gzip compressed
tar -czvf project.tar.gz ~/project

# 3. Bzip2 compressed
tar -cjvf project.tar.bz2 ~/project

# 4. Compare sizes
ls -lh project.tar*

# 5. Extract to /tmp
mkdir -p /tmp/extracted
tar -xzvf project.tar.gz -C /tmp/extracted

# 6. List contents
tar -tzvf project.tar.gz
```

**Task 2.4:**
```bash
# 1. Find .conf files
find /etc -name "*.conf" 2>/dev/null

# 2. Modified in last 24 hours
find /etc -mtime -1 2>/dev/null

# 3. Larger than 100KB
find /etc -size +100k 2>/dev/null

# 4. Symbolic links
find /etc -type l 2>/dev/null

# 5. Copy .conf files (need sudo for some)
find /etc -name "*.conf" -exec cp {} ~/project/config/ \; 2>/dev/null
```
</details>

---

## Lab 3: User and Permission Management

### Objectives
- Create and manage users and groups
- Set file permissions
- Configure sudo access

### Tasks

**Task 3.1: User Management**
```
1. Create a new user called 'developer' with home directory
2. Create a user 'contractor' with expiry date 30 days from now
3. Create a group called 'devteam'
4. Add 'developer' to the 'devteam' group as secondary group
5. Lock the 'contractor' account
6. Change the shell for 'developer' to /bin/bash
7. Set password aging: max 90 days, warn 7 days before expiry
```

**Task 3.2: Permission Challenge**
```
Create a shared project directory with these requirements:
1. Create /opt/shared-project
2. Owner: developer, Group: devteam
3. devteam members can read, write, and execute
4. Others can only read
5. New files should inherit the group ownership (SGID)
6. Only file owners can delete their files (sticky bit)
```

**Task 3.3: ACL Configuration**
```
1. Create a file ~/secret.txt
2. Set permissions so only owner can access it (600)
3. Use ACL to give 'contractor' read-only access
4. Use ACL to give 'devteam' group read-write access
5. Verify ACLs are set correctly
6. Remove the ACL for contractor
```

**Task 3.4: Sudo Configuration**
```
1. Allow 'developer' to run all commands as root
2. Allow 'devteam' group to restart httpd without password
3. Allow 'contractor' to run only 'cat' and 'less' commands
4. Create an alias for network commands and allow developer to use them
5. Log all sudo commands to /var/log/sudo.log
```

<details>
<summary>View Hint</summary>

- **Task 3.1:** Use `useradd -m` for home directory, `-e` for expiry date, `chage` for password aging.
- **Task 3.2:** SGID = `chmod g+s`, sticky bit = `chmod +t`. Numeric: SGID=2, sticky=1 (prepend to permissions).
- **Task 3.3:** `setfacl -m` modifies ACLs, `getfacl` displays them, `setfacl -x` removes specific entries.
- **Task 3.4:** Always use `visudo` to edit sudoers safely. Use `%groupname` for groups.

</details>

<details>
<summary>View Solutions</summary>

**Task 3.1:**
```bash
# 1. Create developer
sudo useradd -m developer

# 2. Create contractor with expiry
sudo useradd -m -e $(date -d "+30 days" +%Y-%m-%d) contractor

# 3. Create group
sudo groupadd devteam

# 4. Add to group
sudo usermod -aG devteam developer

# 5. Lock account
sudo passwd -l contractor
# or
sudo usermod -L contractor

# 6. Change shell
sudo usermod -s /bin/bash developer

# 7. Password aging
sudo chage -M 90 -W 7 developer
```

**Task 3.2:**
```bash
# 1. Create directory
sudo mkdir -p /opt/shared-project

# 2. Set owner and group
sudo chown developer:devteam /opt/shared-project

# 3 & 4. Set permissions (rwxrwxr--)
sudo chmod 774 /opt/shared-project

# 5. Set SGID
sudo chmod g+s /opt/shared-project

# 6. Set sticky bit
sudo chmod +t /opt/shared-project

# Combined: chmod 3774 /opt/shared-project
# Verify
ls -ld /opt/shared-project
# drwxrws--T
```

**Task 3.3:**
```bash
# 1. Create file
touch ~/secret.txt

# 2. Set permissions
chmod 600 ~/secret.txt

# 3. ACL for contractor
setfacl -m u:contractor:r ~/secret.txt

# 4. ACL for devteam
setfacl -m g:devteam:rw ~/secret.txt

# 5. Verify
getfacl ~/secret.txt

# 6. Remove ACL
setfacl -x u:contractor ~/secret.txt
```

**Task 3.4:**
```bash
# Edit sudoers safely
sudo visudo

# 1. Developer all access
developer ALL=(ALL) ALL

# 2. devteam restart httpd (add to sudoers)
%devteam ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart httpd

# 3. Contractor limited commands
contractor ALL=(ALL) /usr/bin/cat, /usr/bin/less

# 4. Command alias for network commands
Cmnd_Alias NETWORK = /usr/bin/ip, /usr/bin/ss, /usr/bin/ping
developer ALL=(ALL) NETWORK

# 5. Logging (add to sudoers)
Defaults logfile="/var/log/sudo.log"
```
</details>

---

## Lab 4: Text Processing Challenge

### Objectives
- Master grep, sed, and awk
- Process log files
- Build complex pipelines

### Setup
```bash
# Create sample log file for exercises
cat << 'EOF' > ~/access.log
192.168.1.100 - - [15/Jan/2024:10:23:45 +0000] "GET /index.html HTTP/1.1" 200 1234
192.168.1.101 - - [15/Jan/2024:10:23:46 +0000] "POST /login HTTP/1.1" 200 567
192.168.1.100 - - [15/Jan/2024:10:23:47 +0000] "GET /dashboard HTTP/1.1" 200 8901
10.0.0.50 - - [15/Jan/2024:10:23:48 +0000] "GET /admin HTTP/1.1" 403 123
192.168.1.102 - - [15/Jan/2024:10:23:49 +0000] "GET /api/users HTTP/1.1" 200 2345
192.168.1.100 - - [15/Jan/2024:10:23:50 +0000] "DELETE /api/users/5 HTTP/1.1" 401 89
10.0.0.50 - - [15/Jan/2024:10:23:51 +0000] "GET /admin HTTP/1.1" 403 123
192.168.1.101 - - [15/Jan/2024:10:23:52 +0000] "GET /profile HTTP/1.1" 200 3456
192.168.1.103 - - [15/Jan/2024:10:23:53 +0000] "GET /search?q=test HTTP/1.1" 200 789
10.0.0.50 - - [15/Jan/2024:10:23:54 +0000] "POST /admin/login HTTP/1.1" 401 45
EOF

# Create sample data file
cat << 'EOF' > ~/employees.csv
id,name,department,salary,start_date
1,John Smith,Engineering,75000,2020-03-15
2,Jane Doe,Marketing,65000,2019-07-22
3,Bob Johnson,Engineering,80000,2018-01-10
4,Alice Brown,HR,55000,2021-06-01
5,Charlie Wilson,Engineering,90000,2017-09-30
6,Diana Ross,Marketing,70000,2020-11-15
7,Edward Lee,Finance,72000,2019-04-20
8,Fiona Chen,Engineering,85000,2018-08-05
9,George Davis,HR,52000,2022-02-28
10,Helen White,Finance,78000,2016-12-01
EOF
```

### Tasks

**Task 4.1: grep Mastery**
```
Using access.log:
1. Find all requests from 192.168.1.100
2. Find all failed requests (4xx status codes)
3. Count how many GET vs POST requests
4. Find requests to /admin (case insensitive)
5. Show requests with 3 lines of context after each match
6. Find lines that do NOT contain "200"
```

**Task 4.2: sed Transformations**
```
1. Replace all IP addresses with [REDACTED] in access.log
2. Remove the timestamp portion from each line
3. Change HTTP/1.1 to HTTP/2.0
4. Delete all lines containing "403"
5. Add "REVIEWED: " prefix to each line
6. Extract just the URL path from each request
```

**Task 4.3: awk Analysis**
```
Using access.log:
1. Print only IP addresses and status codes
2. Calculate total bytes transferred (last column)
3. Count requests per IP address
4. Find the IP with most failed requests (4xx)
5. Calculate average response size for successful requests

Using employees.csv:
6. Print names and salaries only
7. Calculate average salary per department
8. Find highest paid employee in each department
9. List employees who started after 2020
10. Calculate total salary budget
```

**Task 4.4: Pipeline Challenge**
```
Combine tools to answer:
1. What are the top 3 most frequent IP addresses in access.log?
2. Create a summary report showing: total requests, success rate, unique IPs
3. Extract all unique URLs and sort them alphabetically
4. From employees.csv, find the Engineering department's average salary
5. Create a formatted table of department headcount and total salary
```

<details>
<summary>View Hint</summary>

- **Task 4.1:** `grep -v` inverts match, `-c` counts, `-i` is case-insensitive, `-E` enables extended regex.
- **Task 4.2:** In sed, `s/pattern/replacement/g` replaces globally. Use `-E` for extended regex with capture groups.
- **Task 4.3:** In awk, `$1` is first field, `NR` is line number. Use `END{}` block for final calculations.
- **Task 4.4:** Pipe commands together: `sort | uniq -c | sort -rn | head` for frequency analysis.

</details>

<details>
<summary>View Solutions</summary>

**Task 4.1:**
```bash
# 1. Requests from specific IP
grep "192.168.1.100" ~/access.log

# 2. Failed requests (4xx)
grep -E '" 4[0-9]{2} ' ~/access.log

# 3. Count GET vs POST
grep -c "GET" ~/access.log
grep -c "POST" ~/access.log

# 4. Case insensitive /admin
grep -i "/admin" ~/access.log

# 5. Context after match
grep -A 3 "403" ~/access.log

# 6. NOT containing 200
grep -v "200" ~/access.log
```

**Task 4.2:**
```bash
# 1. Redact IPs
sed -E 's/[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+/[REDACTED]/g' ~/access.log

# 2. Remove timestamp
sed 's/\[.*\] //' ~/access.log

# 3. Change HTTP version
sed 's/HTTP\/1.1/HTTP\/2.0/g' ~/access.log

# 4. Delete 403 lines
sed '/403/d' ~/access.log

# 5. Add prefix
sed 's/^/REVIEWED: /' ~/access.log

# 6. Extract URL path
sed -E 's/.*"(GET|POST|DELETE) ([^ ]+).*/\2/' ~/access.log
```

**Task 4.3:**
```bash
# 1. IP and status
awk '{print $1, $9}' ~/access.log

# 2. Total bytes
awk '{sum += $10} END {print "Total bytes:", sum}' ~/access.log

# 3. Requests per IP
awk '{count[$1]++} END {for (ip in count) print ip, count[ip]}' ~/access.log

# 4. IP with most failed requests
awk '$9 ~ /^4/ {count[$1]++} END {for (ip in count) print ip, count[ip]}' ~/access.log | sort -k2 -rn | head -1

# 5. Average response size for 200
awk '$9 == 200 {sum += $10; count++} END {print "Average:", sum/count}' ~/access.log

# 6. Names and salaries
awk -F',' 'NR>1 {print $2, $4}' ~/employees.csv

# 7. Average salary per department
awk -F',' 'NR>1 {sum[$3]+=$4; count[$3]++} END {for (d in sum) print d, sum[d]/count[d]}' ~/employees.csv

# 8. Highest paid per department
awk -F',' 'NR>1 {if ($4 > max[$3]) {max[$3]=$4; name[$3]=$2}} END {for (d in max) print d, name[d], max[d]}' ~/employees.csv

# 9. Started after 2020
awk -F',' 'NR>1 && $5 > "2020-01-01" {print $2, $5}' ~/employees.csv

# 10. Total salary
awk -F',' 'NR>1 {sum += $4} END {print "Total budget:", sum}' ~/employees.csv
```

**Task 4.4:**
```bash
# 1. Top 3 IPs
awk '{print $1}' ~/access.log | sort | uniq -c | sort -rn | head -3

# 2. Summary report
echo "=== Access Log Summary ==="
echo "Total requests: $(wc -l < ~/access.log)"
echo "Successful (2xx): $(grep -c '" 2[0-9][0-9] ' ~/access.log)"
echo "Failed (4xx): $(grep -c '" 4[0-9][0-9] ' ~/access.log)"
echo "Unique IPs: $(awk '{print $1}' ~/access.log | sort -u | wc -l)"

# 3. Unique URLs
sed -E 's/.*"(GET|POST|DELETE) ([^ ]+).*/\2/' ~/access.log | sort -u

# 4. Engineering average
awk -F',' '$3 == "Engineering" {sum += $4; count++} END {print "Avg:", sum/count}' ~/employees.csv

# 5. Department summary table
echo "Department | Count | Total Salary"
echo "-----------|-------|-------------"
awk -F',' 'NR>1 {sum[$3]+=$4; count[$3]++} END {for (d in sum) printf "%-11s| %5d | %12d\n", d, count[d], sum[d]}' ~/employees.csv
```
</details>

---

## Lab 5: Process Management

### Objectives
- Monitor and manage processes
- Work with systemd services
- Schedule tasks with cron

### Tasks

**Task 5.1: Process Investigation**
```
1. Display all processes with full details
2. Find the PID of the sshd service
3. Display a process tree
4. Find the top 5 memory-consuming processes
5. Find the top 5 CPU-consuming processes
6. List all processes owned by root
7. Find zombie processes (if any)
```

**Task 5.2: Process Control**
```
1. Start a background process that sleeps for 1000 seconds
2. List your background jobs
3. Bring the job to foreground, then suspend it
4. Resume it in the background
5. Send SIGTERM to the process
6. Start a process immune to hangups (nohup)
7. Start a process with lower priority (nice value 10)
```

**Task 5.3: systemd Services**
```
1. Check if httpd is installed, if not install it
2. Start and enable the httpd service
3. Check the status with detailed output
4. View the last 50 log entries for httpd
5. Create a custom service that runs a script
6. Reload systemd to recognize your new service
7. Stop and disable httpd
```

**Task 5.4: Cron Scheduling**
```
Create cron jobs to:
1. Run a backup script every day at 2:30 AM
2. Clean /tmp every Sunday at midnight
3. Check disk space every 15 minutes
4. Run a report on the 1st of every month at 6 AM
5. Log system uptime every hour (append to a file)
6. Create an at job to run once in 2 hours
```

<details>
<summary>View Hint</summary>

- **Task 5.1:** `ps aux` shows all processes. `--sort=-%mem` sorts by memory descending. `pgrep` finds PIDs.
- **Task 5.2:** `&` runs in background, `jobs` lists them, `fg/bg` switch foreground/background, `Ctrl+Z` suspends.
- **Task 5.3:** `systemctl start/stop/enable/disable` controls services. `journalctl -u` shows service logs.
- **Task 5.4:** Cron format: `min hour day month weekday command`. Use `*/15` for "every 15 minutes".

</details>

<details>
<summary>View Solutions</summary>

**Task 5.1:**
```bash
# 1. All processes
ps aux

# 2. Find sshd PID
pgrep sshd
# or
ps aux | grep sshd

# 3. Process tree
pstree -p

# 4. Top 5 memory
ps aux --sort=-%mem | head -6

# 5. Top 5 CPU
ps aux --sort=-%cpu | head -6

# 6. Root processes
ps -U root -u root

# 7. Zombies
ps aux | grep -w Z
```

**Task 5.2:**
```bash
# 1. Background sleep
sleep 1000 &

# 2. List jobs
jobs

# 3. Foreground then suspend
fg %1
# Then Ctrl+Z

# 4. Resume in background
bg %1

# 5. Send SIGTERM
kill %1
# or kill <PID>

# 6. Nohup
nohup sleep 1000 &

# 7. Nice
nice -n 10 sleep 1000 &
```

**Task 5.3:**
```bash
# 1. Check and install
rpm -q httpd || sudo dnf install -y httpd

# 2. Start and enable
sudo systemctl start httpd
sudo systemctl enable httpd

# 3. Status
sudo systemctl status httpd -l

# 4. Logs
sudo journalctl -u httpd -n 50

# 5. Custom service
sudo cat << 'EOF' > /etc/systemd/system/myscript.service
[Unit]
Description=My Custom Script
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/myscript.sh

[Install]
WantedBy=multi-user.target
EOF

# 6. Reload
sudo systemctl daemon-reload

# 7. Stop and disable
sudo systemctl stop httpd
sudo systemctl disable httpd
```

**Task 5.4:**
```bash
# Edit crontab
crontab -e

# 1. Daily backup at 2:30 AM
30 2 * * * /usr/local/bin/backup.sh

# 2. Clean tmp Sunday midnight
0 0 * * 0 rm -rf /tmp/*

# 3. Disk space every 15 min
*/15 * * * * df -h >> /var/log/disk-check.log

# 4. Monthly report
0 6 1 * * /usr/local/bin/monthly-report.sh

# 5. Hourly uptime
0 * * * * uptime >> /var/log/uptime.log

# 6. At job
at now + 2 hours
# Enter command, then Ctrl+D
```
</details>

---

## Lab 6: Network Configuration

### Objectives
- Configure network interfaces
- Troubleshoot connectivity
- Manage firewall rules

### Tasks

**Task 6.1: Network Information**
```
1. Display all IP addresses on the system
2. Show the routing table
3. Display DNS servers being used
4. Show all listening ports with process names
5. Display network interface statistics
6. Test connectivity to 8.8.8.8 with 5 packets
7. Trace the route to google.com
```

**Task 6.2: NetworkManager Configuration**
```
1. List all network connections
2. Show detailed info for your active connection
3. Create a new connection profile with static IP:
   - IP: 192.168.100.50/24
   - Gateway: 192.168.100.1
   - DNS: 8.8.8.8, 8.8.4.4
4. Add a secondary IP to an interface
5. Set the connection to auto-connect
6. Bring the connection down and back up
```

**Task 6.3: Firewall Configuration**
```
1. Check if firewalld is running
2. List all active zones and their rules
3. Allow HTTP and HTTPS traffic permanently
4. Open port 8080 for TCP
5. Allow traffic from 192.168.1.0/24 network
6. Create a new zone called 'webserver'
7. Add port 3306 to internal zone only
8. Block all traffic from a specific IP
9. Reload the firewall
10. List all rules in rich format
```

**Task 6.4: Troubleshooting Scenario**
```
Scenario: A web server is not accessible from the network.
Perform these diagnostic steps:

1. Verify the web service is running
2. Check if it's listening on the expected port
3. Test local connectivity
4. Check firewall rules
5. Verify network interface is up
6. Check routing
7. Test DNS resolution
8. Document your findings
```

<details>
<summary>View Hint</summary>

- **Task 6.1:** `ip a` shows addresses, `ip r` shows routes, `ss -tulpn` shows listening ports with process names.
- **Task 6.2:** Use `nmcli con` for connections, `nmcli con mod` to modify, `nmcli con up/down` to toggle.
- **Task 6.3:** `firewall-cmd --permanent` makes changes persist. `--add-service`, `--add-port` open traffic.
- **Task 6.4:** Systematic approach: check service → port → firewall → interface → routing → DNS.

</details>

<details>
<summary>View Solutions</summary>

**Task 6.1:**
```bash
# 1. IP addresses
ip addr
# or
ip a

# 2. Routing table
ip route
# or
ip r

# 3. DNS servers
cat /etc/resolv.conf

# 4. Listening ports
ss -tulnp
# or
netstat -tulnp

# 5. Interface stats
ip -s link

# 6. Ping test
ping -c 5 8.8.8.8

# 7. Traceroute
traceroute google.com
# or
tracepath google.com
```

**Task 6.2:**
```bash
# 1. List connections
nmcli con show

# 2. Connection details
nmcli con show "Connection Name"

# 3. Create static connection
nmcli con add type ethernet con-name static-lab ifname eth0 \
  ip4 192.168.100.50/24 gw4 192.168.100.1
nmcli con mod static-lab ipv4.dns "8.8.8.8 8.8.4.4"

# 4. Add secondary IP
nmcli con mod static-lab +ipv4.addresses 192.168.100.51/24

# 5. Auto-connect
nmcli con mod static-lab autoconnect yes

# 6. Down and up
nmcli con down static-lab
nmcli con up static-lab
```

**Task 6.3:**
```bash
# 1. Check firewalld
sudo systemctl status firewalld

# 2. List zones
sudo firewall-cmd --list-all-zones

# 3. Allow HTTP/HTTPS
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https

# 4. Open port 8080
sudo firewall-cmd --permanent --add-port=8080/tcp

# 5. Allow network
sudo firewall-cmd --permanent --add-source=192.168.1.0/24

# 6. New zone
sudo firewall-cmd --permanent --new-zone=webserver

# 7. Add to internal zone
sudo firewall-cmd --permanent --zone=internal --add-port=3306/tcp

# 8. Block IP
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.0.0.100" reject'

# 9. Reload
sudo firewall-cmd --reload

# 10. List rules
sudo firewall-cmd --list-all
```

**Task 6.4:**
```bash
# Troubleshooting steps
# 1. Check service
sudo systemctl status httpd

# 2. Check listening port
ss -tlnp | grep 80

# 3. Local test
curl localhost

# 4. Firewall
sudo firewall-cmd --list-all

# 5. Interface
ip link show

# 6. Routing
ip route

# 7. DNS
nslookup google.com

# 8. Document
echo "Findings documented in /tmp/network-troubleshoot.txt"
```
</details>

---

## Lab 7: Package Management

### Objectives
- Install, update, and remove packages
- Manage repositories
- Query package information

### Tasks

**Task 7.1: DNF Operations**
```
1. Update the package cache
2. Search for packages related to 'web server'
3. Get information about the nginx package
4. Install nginx
5. List all installed packages
6. Find which package provides /usr/bin/dig
7. List the dependencies of httpd
8. Remove nginx and its unused dependencies
9. Downgrade a package (if possible)
10. View transaction history
```

**Task 7.2: Repository Management**
```
1. List all enabled repositories
2. List all available repositories
3. Add the EPEL repository
4. Disable a repository temporarily for an install
5. Create a local repository from a directory of RPMs
6. Clean all cached data
7. Verify the integrity of installed packages
```

**Task 7.3: RPM Operations**
```
1. Download an RPM without installing
2. Query all files in an installed package
3. Query which package owns /etc/passwd
4. Verify an installed package
5. Extract files from an RPM without installing
6. List all config files for a package
7. Check the signature of an RPM
```

**Task 7.4: Module Streams (RHEL 8+)**
```
1. List all available module streams
2. Find available versions of nodejs
3. Enable the nodejs:18 stream
4. Install nodejs from the stream
5. Switch to a different stream
6. Reset a module stream
```

<details>
<summary>View Hint</summary>

- **Task 7.1:** `dnf search` finds packages, `dnf info` shows details, `dnf provides` finds which package owns a file.
- **Task 7.2:** `dnf repolist` shows enabled repos. EPEL adds extra packages for Enterprise Linux.
- **Task 7.3:** `rpm -q` queries packages, `-l` lists files, `-f` finds owner, `-V` verifies integrity.
- **Task 7.4:** `dnf module list` shows streams, `enable` activates, `reset` clears selection.

</details>

<details>
<summary>View Solutions</summary>

**Task 7.1:**
```bash
# 1. Update cache
sudo dnf makecache

# 2. Search
dnf search web server

# 3. Package info
dnf info nginx

# 4. Install
sudo dnf install -y nginx

# 5. List installed
dnf list installed

# 6. Find provider
dnf provides /usr/bin/dig

# 7. Dependencies
dnf repoquery --requires httpd

# 8. Remove with deps
sudo dnf remove nginx
sudo dnf autoremove

# 9. Downgrade
sudo dnf downgrade package-name

# 10. History
dnf history
```

**Task 7.2:**
```bash
# 1. Enabled repos
dnf repolist

# 2. All repos
dnf repolist all

# 3. Add EPEL
sudo dnf install epel-release

# 4. Disable temp
sudo dnf --disablerepo=epel install package

# 5. Local repo
sudo dnf install createrepo
createrepo /path/to/rpms/
# Create repo file in /etc/yum.repos.d/

# 6. Clean cache
sudo dnf clean all

# 7. Verify packages
rpm -Va
```

**Task 7.3:**
```bash
# 1. Download only
dnf download nginx

# 2. List files
rpm -ql httpd

# 3. File owner
rpm -qf /etc/passwd

# 4. Verify package
rpm -V httpd

# 5. Extract without install
rpm2cpio package.rpm | cpio -idmv

# 6. Config files
rpm -qc httpd

# 7. Check signature
rpm -K package.rpm
```

**Task 7.4:**
```bash
# 1. List modules
dnf module list

# 2. nodejs versions
dnf module list nodejs

# 3. Enable stream
sudo dnf module enable nodejs:18

# 4. Install
sudo dnf module install nodejs:18

# 5. Switch stream
sudo dnf module reset nodejs
sudo dnf module enable nodejs:20

# 6. Reset
sudo dnf module reset nodejs
```
</details>

---

## Lab 8: Storage and LVM

### Objectives
- Create partitions and filesystems
- Manage LVM volumes
- Configure persistent mounts

### Prerequisites
```bash
# For this lab, you need a spare disk or can use a loop device
# Create a 1GB loop device for practice:
sudo dd if=/dev/zero of=/tmp/disk.img bs=1M count=1024
sudo losetup /dev/loop0 /tmp/disk.img
```

### Tasks

**Task 8.1: Partition Management**
```
Using /dev/loop0 (or spare disk):
1. View current partition table
2. Create a new GPT partition table
3. Create three partitions:
   - 200MB for /boot
   - 300MB for swap
   - 500MB for data
4. Verify the partitions were created
5. Create filesystems:
   - ext4 on partition 1
   - swap on partition 2
   - xfs on partition 3
```

**Task 8.2: LVM Configuration**
```
1. Create a physical volume on a partition
2. Create a volume group called 'datavg'
3. Create logical volumes:
   - 'appslv' - 100MB
   - 'datalv' - 200MB
4. Create xfs filesystem on appslv
5. Create ext4 filesystem on datalv
6. Extend appslv by 50MB
7. Extend the filesystem to use new space
8. Display all LVM information
```

**Task 8.3: Mounting and fstab**
```
1. Create mount points /mnt/apps and /mnt/data
2. Mount the logical volumes
3. Verify the mounts
4. Add entries to /etc/fstab for persistent mounting
5. Test fstab entries without rebooting
6. Mount with specific options (noexec, nosuid)
7. Create a bind mount
```

**Task 8.4: Swap Management**
```
1. Display current swap usage
2. Create a swap file of 256MB
3. Set up the swap area
4. Enable the swap file
5. Add to fstab for persistence
6. Verify swap is active
7. Set swappiness to 10
```

<details>
<summary>View Hint</summary>

- **Task 8.1:** Use `gdisk` for GPT partitions. Partition type `8200` is Linux swap.
- **Task 8.2:** LVM flow: `pvcreate` → `vgcreate` → `lvcreate`. Extend with `lvextend`, then grow filesystem.
- **Task 8.3:** Use UUID or LV path in `/etc/fstab`. Test with `mount -a` before rebooting.
- **Task 8.4:** Create swap file with `dd`, format with `mkswap`, enable with `swapon`. Add to fstab for persistence.

</details>

<details>
<summary>View Solutions</summary>

**Task 8.1:**
```bash
# 1. View partitions
sudo fdisk -l /dev/loop0

# 2 & 3. Create partitions
sudo gdisk /dev/loop0
# n, 1, enter, +200M, enter
# n, 2, enter, +300M, 8200 (swap)
# n, 3, enter, enter, enter
# w

# 4. Verify
lsblk /dev/loop0

# 5. Create filesystems
sudo mkfs.ext4 /dev/loop0p1
sudo mkswap /dev/loop0p2
sudo mkfs.xfs /dev/loop0p3
```

**Task 8.2:**
```bash
# 1. Create PV
sudo pvcreate /dev/loop0p3

# 2. Create VG
sudo vgcreate datavg /dev/loop0p3

# 3. Create LVs
sudo lvcreate -L 100M -n appslv datavg
sudo lvcreate -L 200M -n datalv datavg

# 4 & 5. Create filesystems
sudo mkfs.xfs /dev/datavg/appslv
sudo mkfs.ext4 /dev/datavg/datalv

# 6. Extend LV
sudo lvextend -L +50M /dev/datavg/appslv

# 7. Extend filesystem
sudo xfs_growfs /dev/datavg/appslv

# 8. Display info
sudo pvs
sudo vgs
sudo lvs
sudo pvdisplay
sudo vgdisplay
sudo lvdisplay
```

**Task 8.3:**
```bash
# 1. Create mount points
sudo mkdir -p /mnt/{apps,data}

# 2. Mount
sudo mount /dev/datavg/appslv /mnt/apps
sudo mount /dev/datavg/datalv /mnt/data

# 3. Verify
df -h | grep mnt
mount | grep mnt

# 4. Add to fstab
echo "/dev/datavg/appslv /mnt/apps xfs defaults 0 0" | sudo tee -a /etc/fstab
echo "/dev/datavg/datalv /mnt/data ext4 defaults 0 0" | sudo tee -a /etc/fstab

# 5. Test fstab
sudo umount /mnt/apps /mnt/data
sudo mount -a

# 6. Mount with options
sudo mount -o noexec,nosuid /dev/datavg/appslv /mnt/apps

# 7. Bind mount
sudo mount --bind /mnt/apps /mnt/apps-mirror
```

**Task 8.4:**
```bash
# 1. Current swap
swapon --show
free -h

# 2. Create swap file
sudo dd if=/dev/zero of=/swapfile bs=1M count=256

# 3. Setup swap
sudo chmod 600 /swapfile
sudo mkswap /swapfile

# 4. Enable
sudo swapon /swapfile

# 5. Add to fstab
echo "/swapfile none swap sw 0 0" | sudo tee -a /etc/fstab

# 6. Verify
swapon --show

# 7. Set swappiness
sudo sysctl vm.swappiness=10
echo "vm.swappiness=10" | sudo tee -a /etc/sysctl.conf
```
</details>

---

## Lab 9: Shell Scripting

### Objectives
- Write practical shell scripts
- Handle arguments and errors
- Create automated solutions

### Tasks

**Task 9.1: Basic Script**
```
Create a script called 'sysinfo.sh' that:
1. Displays a header with the current date/time
2. Shows the hostname
3. Shows the OS version
4. Shows CPU information (model name)
5. Shows memory usage
6. Shows disk usage
7. Shows logged-in users
8. Formats output nicely with separators
```

**Task 9.2: User Input and Arguments**
```
Create a script called 'usermgmt.sh' that:
1. Accepts a command-line argument: create, delete, or info
2. Prompts for username
3. Performs the appropriate action:
   - create: creates user with home directory
   - delete: removes user and home directory
   - info: shows user information
4. Handles errors (user exists, doesn't exist, etc.)
5. Logs actions to /var/log/usermgmt.log
```

**Task 9.3: Log Analyzer**
```
Create a script called 'loganalyzer.sh' that:
1. Takes a log file as argument
2. Counts total lines
3. Finds the most common IP address
4. Counts error occurrences (4xx and 5xx)
5. Finds peak hour (most requests)
6. Outputs a summary report
7. Optionally exports to CSV with -c flag
```

**Task 9.4: Backup Script**
```
Create a script called 'backup.sh' that:
1. Takes source directory as argument
2. Creates timestamped backup in /backup
3. Compresses the backup with gzip
4. Keeps only last 7 backups (rotation)
5. Logs all actions with timestamps
6. Sends email on failure (simulate with echo)
7. Returns proper exit codes
```

**Task 9.5: Service Monitor**
```
Create a script called 'monitor.sh' that:
1. Monitors a list of services (httpd, sshd, firewalld)
2. Checks if each service is running
3. Attempts to restart stopped services
4. Logs all status changes
5. Runs in a loop with configurable interval
6. Handles graceful shutdown on SIGTERM
```

<details>
<summary>View Hint</summary>

- **Task 9.1:** Use command substitution `$(command)` to embed output. `free -h` shows human-readable memory.
- **Task 9.2:** Use `case` statements for menu options. `id username` checks if user exists (exit code 0 = exists).
- **Task 9.3:** `awk '{print $1}'` extracts first field. Pipe through `sort | uniq -c | sort -rn` for frequency.
- **Task 9.4:** Use `ls -1t` to list files by time, `tail -n +N` to skip first N-1 files for rotation.
- **Task 9.5:** Use `trap` to catch signals. `systemctl is-active --quiet` returns 0 if service is running.

</details>

<details>
<summary>View Solutions</summary>

**Task 9.1: sysinfo.sh**
```bash
#!/bin/bash

echo "========================================"
echo "       SYSTEM INFORMATION REPORT"
echo "========================================"
echo "Generated: $(date)"
echo "========================================"
echo ""
echo "HOSTNAME: $(hostname)"
echo ""
echo "--- OS Information ---"
cat /etc/os-release | grep -E "^(NAME|VERSION)="
echo ""
echo "--- CPU Information ---"
grep "model name" /proc/cpuinfo | head -1
echo "CPU Cores: $(nproc)"
echo ""
echo "--- Memory Usage ---"
free -h | grep -E "^(Mem|Swap)"
echo ""
echo "--- Disk Usage ---"
df -h | grep -E "^(/dev|Filesystem)"
echo ""
echo "--- Logged-in Users ---"
who
echo ""
echo "========================================"
echo "           END OF REPORT"
echo "========================================"
```

**Task 9.2: usermgmt.sh**
```bash
#!/bin/bash

LOG_FILE="/var/log/usermgmt.log"

log_action() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | sudo tee -a "$LOG_FILE"
}

if [[ $# -ne 1 ]]; then
    echo "Usage: $0 {create|delete|info}"
    exit 1
fi

read -p "Enter username: " USERNAME

case $1 in
    create)
        if id "$USERNAME" &>/dev/null; then
            echo "Error: User $USERNAME already exists"
            exit 1
        fi
        sudo useradd -m "$USERNAME"
        if [[ $? -eq 0 ]]; then
            echo "User $USERNAME created successfully"
            log_action "Created user: $USERNAME"
        else
            echo "Failed to create user"
            exit 1
        fi
        ;;
    delete)
        if ! id "$USERNAME" &>/dev/null; then
            echo "Error: User $USERNAME does not exist"
            exit 1
        fi
        read -p "Are you sure? (y/n): " CONFIRM
        if [[ $CONFIRM == "y" ]]; then
            sudo userdel -r "$USERNAME"
            echo "User $USERNAME deleted"
            log_action "Deleted user: $USERNAME"
        fi
        ;;
    info)
        if ! id "$USERNAME" &>/dev/null; then
            echo "Error: User $USERNAME does not exist"
            exit 1
        fi
        echo "--- User Information ---"
        id "$USERNAME"
        echo "Home: $(eval echo ~$USERNAME)"
        echo "Shell: $(grep "^$USERNAME:" /etc/passwd | cut -d: -f7)"
        echo "Groups: $(groups $USERNAME)"
        ;;
    *)
        echo "Invalid option. Use: create, delete, or info"
        exit 1
        ;;
esac
```

**Task 9.3: loganalyzer.sh**
```bash
#!/bin/bash

if [[ $# -lt 1 ]]; then
    echo "Usage: $0 <logfile> [-c]"
    exit 1
fi

LOG_FILE="$1"
CSV_OUTPUT=false

[[ "$2" == "-c" ]] && CSV_OUTPUT=true

if [[ ! -f "$LOG_FILE" ]]; then
    echo "Error: File not found"
    exit 1
fi

TOTAL_LINES=$(wc -l < "$LOG_FILE")
TOP_IP=$(awk '{print $1}' "$LOG_FILE" | sort | uniq -c | sort -rn | head -1)
ERRORS_4XX=$(grep -c '" 4[0-9][0-9] ' "$LOG_FILE")
ERRORS_5XX=$(grep -c '" 5[0-9][0-9] ' "$LOG_FILE")
PEAK_HOUR=$(awk '{print $4}' "$LOG_FILE" | cut -d: -f2 | sort | uniq -c | sort -rn | head -1)

echo "========================================"
echo "         LOG ANALYSIS REPORT"
echo "========================================"
echo "File: $LOG_FILE"
echo "Total Requests: $TOTAL_LINES"
echo "Top IP: $TOP_IP"
echo "4xx Errors: $ERRORS_4XX"
echo "5xx Errors: $ERRORS_5XX"
echo "Peak Hour: $PEAK_HOUR"
echo "========================================"

if $CSV_OUTPUT; then
    echo "metric,value" > report.csv
    echo "total_requests,$TOTAL_LINES" >> report.csv
    echo "errors_4xx,$ERRORS_4XX" >> report.csv
    echo "errors_5xx,$ERRORS_5XX" >> report.csv
    echo "CSV exported to report.csv"
fi
```

**Task 9.4: backup.sh**
```bash
#!/bin/bash

set -e

SOURCE_DIR="$1"
BACKUP_DIR="/backup"
LOG_FILE="/var/log/backup.log"
RETENTION=7

log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a "$LOG_FILE"
}

if [[ -z "$SOURCE_DIR" || ! -d "$SOURCE_DIR" ]]; then
    log "ERROR: Invalid source directory"
    echo "Would send failure email"
    exit 1
fi

TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_NAME="backup_${TIMESTAMP}.tar.gz"

mkdir -p "$BACKUP_DIR"

log "Starting backup of $SOURCE_DIR"

if tar -czvf "${BACKUP_DIR}/${BACKUP_NAME}" "$SOURCE_DIR" 2>/dev/null; then
    log "Backup created: $BACKUP_NAME"
else
    log "ERROR: Backup failed"
    echo "Would send failure email"
    exit 1
fi

# Rotation
BACKUP_COUNT=$(ls -1 "$BACKUP_DIR"/backup_*.tar.gz 2>/dev/null | wc -l)
if [[ $BACKUP_COUNT -gt $RETENTION ]]; then
    log "Rotating old backups (keeping last $RETENTION)"
    ls -1t "$BACKUP_DIR"/backup_*.tar.gz | tail -n +$((RETENTION + 1)) | xargs rm -f
fi

log "Backup completed successfully"
exit 0
```

**Task 9.5: monitor.sh**
```bash
#!/bin/bash

SERVICES=("httpd" "sshd" "firewalld")
INTERVAL=60
LOG_FILE="/var/log/monitor.log"

log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a "$LOG_FILE"
}

cleanup() {
    log "Monitor shutting down gracefully"
    exit 0
}

trap cleanup SIGTERM SIGINT

log "Service monitor started"

while true; do
    for SERVICE in "${SERVICES[@]}"; do
        if systemctl is-active --quiet "$SERVICE"; then
            : # Service running, do nothing
        else
            log "WARNING: $SERVICE is not running"
            log "Attempting to restart $SERVICE"
            if sudo systemctl start "$SERVICE" 2>/dev/null; then
                log "Successfully restarted $SERVICE"
            else
                log "ERROR: Failed to restart $SERVICE"
            fi
        fi
    done
    sleep "$INTERVAL"
done
```
</details>

---

## Lab 10: SELinux Troubleshooting

### Objectives
- Understand SELinux contexts
- Troubleshoot SELinux denials
- Configure SELinux policies

### Tasks

**Task 10.1: SELinux Basics**
```
1. Check the current SELinux mode
2. View the SELinux configuration file
3. Temporarily set SELinux to permissive
4. Set it back to enforcing
5. List all SELinux booleans related to httpd
6. View the SELinux context of /var/www/html
7. View process contexts for httpd
```

**Task 10.2: Context Management**
```
1. Create a directory /web/content
2. Create an index.html file in it
3. Try to serve it with Apache (will fail)
4. Check for SELinux denials
5. View the current context
6. Set the correct context (httpd_sys_content_t)
7. Verify the change and test Apache again
```

**Task 10.3: Boolean Management**
```
1. Try to have Apache connect to a database (simulated)
2. Find the relevant SELinux boolean
3. Enable the boolean permanently
4. Configure httpd to allow home directories
5. Allow httpd to send emails
6. List all currently enabled booleans
```

**Task 10.4: Troubleshooting Scenario**
```
Scenario: A custom web app on port 8888 won't start.

1. Start a simple web server on port 8888
2. Check for SELinux denials
3. Use audit2why to understand the denial
4. Add the port to the http_port_t type
5. Create a custom SELinux module if needed
6. Document the solution
```

<details>
<summary>View Hint</summary>

- **Task 10.1:** `getenforce` shows current mode, `setenforce 0/1` toggles temporarily. `getsebool -a` lists booleans.
- **Task 10.2:** `semanage fcontext` sets persistent contexts, `restorecon -Rv` applies them recursively.
- **Task 10.3:** Use `setsebool -P` for persistent boolean changes. Search booleans with `getsebool -a | grep keyword`.
- **Task 10.4:** `ausearch -m avc` finds denials, `audit2why` explains them, `semanage port -a` adds port labels.

</details>

<details>
<summary>View Solutions</summary>

**Task 10.1:**
```bash
# 1. Check mode
getenforce
sestatus

# 2. View config
cat /etc/selinux/config

# 3. Permissive
sudo setenforce 0

# 4. Enforcing
sudo setenforce 1

# 5. httpd booleans
getsebool -a | grep httpd

# 6. File context
ls -Z /var/www/html

# 7. Process context
ps auxZ | grep httpd
```

**Task 10.2:**
```bash
# 1 & 2. Create content
sudo mkdir -p /web/content
echo "<h1>Test</h1>" | sudo tee /web/content/index.html

# 3. Configure Apache (add to httpd.conf)
# DocumentRoot "/web/content"

# 4. Check denials
sudo ausearch -m avc -ts recent
sudo cat /var/log/audit/audit.log | grep denied

# 5. View context
ls -Z /web/content/

# 6. Set context
sudo semanage fcontext -a -t httpd_sys_content_t "/web/content(/.*)?"
sudo restorecon -Rv /web/content/

# 7. Verify
ls -Z /web/content/
sudo systemctl restart httpd
curl localhost
```

**Task 10.3:**
```bash
# 2. Find boolean
getsebool -a | grep -i httpd | grep -i db

# 3. Enable
sudo setsebool -P httpd_can_network_connect_db on

# 4. Home directories
sudo setsebool -P httpd_enable_homedirs on

# 5. Send emails
sudo setsebool -P httpd_can_sendmail on

# 6. List enabled
getsebool -a | grep -E "on$"
```

**Task 10.4:**
```bash
# 1. Start server (this might fail due to SELinux)
python3 -m http.server 8888

# 2. Check denials
sudo ausearch -m avc -ts recent | grep 8888

# 3. Analyze
sudo ausearch -m avc -ts recent | audit2why

# 4. Add port
sudo semanage port -a -t http_port_t -p tcp 8888

# Verify
sudo semanage port -l | grep http_port_t

# 5. If custom module needed
sudo ausearch -m avc -ts recent | audit2allow -M mywebapp
sudo semodule -i mywebapp.pp

# 6. Test
python3 -m http.server 8888 &
curl localhost:8888
```
</details>

---

## Lab 11: Boot Recovery

### Objectives
- Understand the boot process
- Recover from boot failures
- Reset root password

### Tasks (Use VM for safety!)

**Task 11.1: Boot Process Analysis**
```
1. View the GRUB configuration file
2. List available boot entries
3. View the kernel boot parameters
4. Check the default boot target
5. View boot messages with dmesg
6. Check boot time analysis with systemd-analyze
7. View the boot log with journalctl
```

**Task 11.2: Boot Target Management**
```
1. List all available targets
2. Check the current default target
3. Change default target to multi-user
4. Boot into single-user mode (rescue)
5. Boot into emergency mode
6. Restore graphical target
```

**Task 11.3: Root Password Reset**
```
Document the steps to reset a forgotten root password:
1. Interrupt the GRUB boot process
2. Edit the kernel parameters
3. Boot into single-user mode
4. Remount filesystem read-write
5. Change the password
6. Handle SELinux relabeling
7. Reboot
```

**Task 11.4: GRUB Recovery**
```
1. Create a backup of GRUB configuration
2. Modify a GRUB entry temporarily
3. Add custom kernel parameters
4. Reinstall GRUB (if needed)
5. Recover from a missing GRUB installation
```

<details>
<summary>View Hint</summary>

- **Task 11.1:** `/proc/cmdline` shows current boot parameters. `systemd-analyze blame` shows slow services.
- **Task 11.2:** `systemctl isolate target` switches targets live. `set-default` changes boot target.
- **Task 11.3:** Add `rd.break` to kernel line to break before root mount. Use `touch /.autorelabel` for SELinux.
- **Task 11.4:** `grubby` modifies boot entries. Backup configs before making changes. Reinstall with `grub2-install`.

</details>

<details>
<summary>View Solutions</summary>

**Task 11.1:**
```bash
# 1. GRUB config
cat /etc/default/grub
cat /boot/grub2/grub.cfg

# 2. Boot entries
sudo grubby --info=ALL

# 3. Kernel parameters
cat /proc/cmdline

# 4. Default target
systemctl get-default

# 5. Boot messages
dmesg | less
dmesg | grep -i error

# 6. Boot analysis
systemd-analyze
systemd-analyze blame
systemd-analyze critical-chain

# 7. Boot log
journalctl -b
journalctl -b -1  # Previous boot
```

**Task 11.2:**
```bash
# 1. List targets
systemctl list-units --type=target

# 2. Current default
systemctl get-default

# 3. Change to multi-user
sudo systemctl set-default multi-user.target

# 4. Rescue mode
sudo systemctl isolate rescue.target
# or add 'systemd.unit=rescue.target' to kernel params

# 5. Emergency mode
sudo systemctl isolate emergency.target

# 6. Restore graphical
sudo systemctl set-default graphical.target
```

**Task 11.3: Root Password Reset Steps**
```
1. Reboot and interrupt GRUB
   - Press 'e' at GRUB menu

2. Edit kernel line
   - Find line starting with 'linux'
   - Add: rd.break
   - Remove: rhgb quiet

3. Boot with Ctrl+X

4. Remount read-write
   mount -o remount,rw /sysroot

5. Chroot
   chroot /sysroot

6. Change password
   passwd root

7. SELinux fix
   touch /.autorelabel

8. Exit and reboot
   exit
   exit
   # System will relabel and reboot
```

**Task 11.4:**
```bash
# 1. Backup GRUB
sudo cp /etc/default/grub /etc/default/grub.backup
sudo cp -r /boot/grub2 /boot/grub2.backup

# 2. Temporary modification
# Press 'e' at boot menu, modify, Ctrl+X to boot

# 3. Add permanent kernel parameters
sudo grubby --update-kernel=ALL --args="audit=1"

# 4. Reinstall GRUB
# BIOS:
sudo grub2-install /dev/sda
# UEFI:
sudo grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg

# 5. Recovery from live media
# Boot from installation media
# Mount root filesystem
# Reinstall GRUB as above
```
</details>

---

## Lab 12: Container Deployment

### Objectives
- Run and manage containers with Podman
- Build custom images
- Deploy multi-container applications

### Tasks

**Task 12.1: Basic Container Operations**
```
1. Pull the nginx:alpine image
2. Run nginx container in background on port 8080
3. List running containers
4. View container logs
5. Execute a shell inside the container
6. Stop and remove the container
7. List all images and remove unused ones
```

**Task 12.2: Build Custom Image**
```
Create a custom image that:
1. Uses ubi9 as base
2. Installs httpd
3. Copies a custom index.html
4. Exposes port 80
5. Runs httpd in foreground
6. Build and tag as mywebapp:1.0
7. Run and test the container
```

**Task 12.3: Persistent Storage**
```
1. Create a volume called 'webdata'
2. Run a container with the volume mounted
3. Add files to the volume from inside container
4. Stop container and start a new one with same volume
5. Verify data persistence
6. Backup the volume to a tar file
7. Clean up containers but keep volume
```

**Task 12.4: Multi-Container Application**
```
Deploy a WordPress application:
1. Create a network for the application
2. Run MariaDB with environment variables
3. Run WordPress connected to MariaDB
4. Test the WordPress installation
5. Create a compose file for the deployment
6. Bring up/down with compose
```

<details>
<summary>View Hint</summary>

- **Task 12.1:** `podman run -d` runs detached, `-p` maps ports, `--name` assigns name. `exec -it` opens shell.
- **Task 12.2:** Containerfile: `FROM` sets base, `RUN` executes commands, `COPY` adds files, `CMD` sets default command.
- **Task 12.3:** `podman volume create` makes volumes. Mount with `-v volume:/path` in run command.
- **Task 12.4:** Create network with `podman network create`. Use `--network` and `-e` for environment variables.

</details>

<details>
<summary>View Solutions</summary>

**Task 12.1:**
```bash
# 1. Pull image
podman pull nginx:alpine

# 2. Run container
podman run -d --name web -p 8080:80 nginx:alpine

# 3. List containers
podman ps

# 4. View logs
podman logs web
podman logs -f web

# 5. Execute shell
podman exec -it web /bin/sh

# 6. Stop and remove
podman stop web
podman rm web

# 7. Clean images
podman images
podman image prune
```

**Task 12.2:**
```bash
# Create Containerfile
cat << 'EOF' > Containerfile
FROM registry.access.redhat.com/ubi9/ubi

RUN dnf install -y httpd && dnf clean all

COPY index.html /var/www/html/

EXPOSE 80

CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]
EOF

# Create index.html
echo "<h1>My Custom Web App</h1>" > index.html

# Build
podman build -t mywebapp:1.0 .

# Run and test
podman run -d --name myapp -p 8080:80 mywebapp:1.0
curl localhost:8080
```

**Task 12.3:**
```bash
# 1. Create volume
podman volume create webdata

# 2. Run with volume
podman run -d --name web -v webdata:/usr/share/nginx/html nginx

# 3. Add files
podman exec web sh -c 'echo "Hello" > /usr/share/nginx/html/test.txt'

# 4. New container, same volume
podman stop web && podman rm web
podman run -d --name web2 -v webdata:/usr/share/nginx/html nginx

# 5. Verify
podman exec web2 cat /usr/share/nginx/html/test.txt

# 6. Backup
podman run --rm -v webdata:/data -v $(pwd):/backup alpine tar cvf /backup/webdata.tar /data

# 7. Cleanup
podman stop web2 && podman rm web2
podman volume ls
```

**Task 12.4:**
```bash
# 1. Create network
podman network create wordpress-net

# 2. Run MariaDB
podman run -d --name db --network wordpress-net \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=wordpress \
  -e MYSQL_USER=wp \
  -e MYSQL_PASSWORD=wppass \
  mariadb:10

# 3. Run WordPress
podman run -d --name wp --network wordpress-net \
  -p 8080:80 \
  -e WORDPRESS_DB_HOST=db \
  -e WORDPRESS_DB_USER=wp \
  -e WORDPRESS_DB_PASSWORD=wppass \
  -e WORDPRESS_DB_NAME=wordpress \
  wordpress

# 4. Test
curl localhost:8080

# 5. Compose file
cat << 'EOF' > docker-compose.yml
version: '3'
services:
  db:
    image: mariadb:10
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wp
      MYSQL_PASSWORD: wppass
    networks:
      - wordpress-net

  wordpress:
    image: wordpress
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wp
      WORDPRESS_DB_PASSWORD: wppass
      WORDPRESS_DB_NAME: wordpress
    networks:
      - wordpress-net

networks:
  wordpress-net:
EOF

# 6. Up/Down
podman-compose up -d
podman-compose down
```
</details>

---

## Lab 13: Ansible Automation

### Objectives
- Write Ansible playbooks
- Manage inventory
- Create reusable roles

### Setup
```bash
# Install Ansible
sudo dnf install ansible-core

# Create project structure
mkdir -p ~/ansible-lab/{inventory,playbooks,roles}
cd ~/ansible-lab
```

### Tasks

**Task 13.1: Inventory and Ad-hoc Commands**
```
1. Create an inventory file with localhost
2. Add groups: webservers, dbservers
3. Test connectivity with ping module
4. Run a command on all hosts
5. Gather facts from localhost
6. Install a package using ad-hoc command
```

**Task 13.2: Basic Playbook**
```
Create a playbook that:
1. Installs httpd, php, mariadb-server
2. Starts and enables the services
3. Copies a custom index.php
4. Opens firewall ports
5. Runs on webservers group
6. Uses become for privilege escalation
```

**Task 13.3: Variables and Templates**
```
1. Create a variables file with:
   - http_port: 8080
   - admin_email: admin@example.com
2. Create a Jinja2 template for httpd.conf
3. Deploy the template with variables
4. Use facts to display system information
5. Use registered variables for conditional logic
```

**Task 13.4: Handlers and Roles**
```
1. Add handlers to restart services when config changes
2. Create a role called 'webserver'
3. Move tasks, handlers, templates to the role
4. Create a role called 'common' for base packages
5. Use roles in a main playbook
6. Add role dependencies
```

**Task 13.5: Complete Deployment**
```
Create a complete playbook that:
1. Sets up a LAMP stack
2. Deploys a sample PHP application
3. Configures the database
4. Sets up firewall rules
5. Implements proper error handling
6. Uses vault for sensitive data
7. Is idempotent (can run multiple times safely)
```

<details>
<summary>View Hint</summary>

- **Task 13.1:** Use `ansible all -m ping` for connectivity. Ad-hoc format: `ansible group -m module -a "args"`
- **Task 13.2:** Start with `hosts:`, `become: yes`, then `tasks:`. Use `dnf` module for packages, `service` for services.
- **Task 13.3:** Define variables in `vars:` section or separate file. Access with `{{ variable_name }}`. Use `template` module.
- **Task 13.4:** Handlers run only when notified and execute at end of play. Use `notify: handler_name` in tasks.
- **Task 13.5:** Use `ansible-vault create` for secrets. `block/rescue` provides error handling. Check mode tests idempotency.

</details>

<details>
<summary>View Solutions</summary>

**Task 13.1:**
```bash
# 1 & 2. Create inventory
cat << 'EOF' > inventory/hosts
[local]
localhost ansible_connection=local

[webservers]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11

[dbservers]
db1 ansible_host=192.168.1.20

[all:vars]
ansible_user=ansible
EOF

# 3. Test ping
ansible all -i inventory/hosts -m ping

# 4. Run command
ansible all -i inventory/hosts -a "uptime"

# 5. Gather facts
ansible localhost -i inventory/hosts -m setup

# 6. Install package
ansible localhost -i inventory/hosts -m dnf -a "name=vim state=present" --become
```

**Task 13.2:**
```yaml
# playbooks/webserver.yml
---
- name: Configure Web Server
  hosts: webservers
  become: yes

  tasks:
    - name: Install packages
      dnf:
        name:
          - httpd
          - php
          - mariadb-server
        state: present

    - name: Start and enable httpd
      service:
        name: httpd
        state: started
        enabled: yes

    - name: Start and enable mariadb
      service:
        name: mariadb
        state: started
        enabled: yes

    - name: Copy index.php
      copy:
        content: "<?php phpinfo(); ?>"
        dest: /var/www/html/index.php
        owner: apache
        group: apache
        mode: '0644'

    - name: Open firewall ports
      firewalld:
        service: http
        permanent: yes
        state: enabled
        immediate: yes
```

**Task 13.3:**
```yaml
# playbooks/vars/main.yml
---
http_port: 8080
admin_email: admin@example.com
server_name: "{{ ansible_fqdn }}"

# playbooks/templates/httpd.conf.j2
# Generated by Ansible
ServerRoot "/etc/httpd"
Listen {{ http_port }}
ServerAdmin {{ admin_email }}
ServerName {{ server_name }}

# playbooks/template-example.yml
---
- name: Deploy with templates
  hosts: webservers
  become: yes
  vars_files:
    - vars/main.yml

  tasks:
    - name: Deploy httpd configuration
      template:
        src: templates/httpd.conf.j2
        dest: /etc/httpd/conf/httpd.conf
        backup: yes
      notify: Restart httpd

    - name: Display system info
      debug:
        msg: "Running on {{ ansible_distribution }} {{ ansible_distribution_version }}"

    - name: Check httpd status
      command: systemctl is-active httpd
      register: httpd_status
      ignore_errors: yes

    - name: Start httpd if not running
      service:
        name: httpd
        state: started
      when: httpd_status.rc != 0

  handlers:
    - name: Restart httpd
      service:
        name: httpd
        state: restarted
```

**Task 13.4:**
```bash
# Create role structure
ansible-galaxy init roles/webserver
ansible-galaxy init roles/common

# roles/webserver/tasks/main.yml
---
- name: Install web packages
  dnf:
    name: "{{ web_packages }}"
    state: present

- name: Deploy configuration
  template:
    src: httpd.conf.j2
    dest: /etc/httpd/conf/httpd.conf
  notify: Restart httpd

- name: Start httpd
  service:
    name: httpd
    state: started
    enabled: yes

# roles/webserver/handlers/main.yml
---
- name: Restart httpd
  service:
    name: httpd
    state: restarted

# roles/webserver/defaults/main.yml
---
web_packages:
  - httpd
  - php

# Main playbook using roles
# playbooks/site.yml
---
- name: Configure all servers
  hosts: all
  become: yes
  roles:
    - common

- name: Configure web servers
  hosts: webservers
  become: yes
  roles:
    - webserver
```

**Task 13.5:**
```yaml
# playbooks/lamp-deploy.yml
---
- name: Deploy LAMP Application
  hosts: webservers
  become: yes
  vars_files:
    - vars/main.yml
    - vars/vault.yml  # Encrypted with ansible-vault

  tasks:
    - name: Install LAMP packages
      dnf:
        name:
          - httpd
          - php
          - php-mysqlnd
          - mariadb-server
          - python3-PyMySQL
        state: present

    - name: Start services
      service:
        name: "{{ item }}"
        state: started
        enabled: yes
      loop:
        - httpd
        - mariadb

    - name: Create database
      mysql_db:
        name: "{{ db_name }}"
        state: present
        login_unix_socket: /var/lib/mysql/mysql.sock

    - name: Create database user
      mysql_user:
        name: "{{ db_user }}"
        password: "{{ db_password }}"  # From vault
        priv: "{{ db_name }}.*:ALL"
        host: localhost
        state: present
        login_unix_socket: /var/lib/mysql/mysql.sock

    - name: Deploy application
      copy:
        src: files/app/
        dest: /var/www/html/
        owner: apache
        group: apache
      notify: Restart httpd

    - name: Configure firewall
      firewalld:
        service: "{{ item }}"
        permanent: yes
        state: enabled
        immediate: yes
      loop:
        - http
        - https

  handlers:
    - name: Restart httpd
      service:
        name: httpd
        state: restarted

# Create vault file
# ansible-vault create vars/vault.yml
# db_password: supersecretpassword

# Run with vault
# ansible-playbook playbooks/lamp-deploy.yml --ask-vault-pass
```
</details>

---

## Capstone Project

### Scenario
You are tasked with setting up a complete web application environment for a small company.

### Requirements

1. **Server Setup**
   - Create 2 users: webadmin (sudo access), developer (limited access)
   - Configure SSH key authentication
   - Set up proper permissions

2. **Web Server**
   - Install and configure Apache
   - Set up virtual hosts
   - Configure SSL (self-signed for testing)
   - Implement basic security

3. **Database**
   - Install MariaDB
   - Create database and user
   - Import sample data
   - Configure secure access

4. **Application**
   - Deploy a PHP application
   - Configure application settings
   - Set up log rotation

5. **Security**
   - Configure firewall rules
   - Set up SELinux properly
   - Implement fail2ban (optional)

6. **Automation**
   - Create backup script (cron)
   - Write monitoring script
   - Create Ansible playbook for deployment

7. **Documentation**
   - Document all steps
   - Create runbook for common tasks
   - Write troubleshooting guide

### Deliverables
- All configuration files
- Shell scripts
- Ansible playbooks
- Documentation

---

## Lab Completion Checklist

| Lab | Completed | Notes |
|-----|-----------|-------|
| Lab 1: File System Navigation | [ ] | |
| Lab 2: File Manipulation | [ ] | |
| Lab 3: User and Permissions | [ ] | |
| Lab 4: Text Processing | [ ] | |
| Lab 5: Process Management | [ ] | |
| Lab 6: Network Configuration | [ ] | |
| Lab 7: Package Management | [ ] | |
| Lab 8: Storage and LVM | [ ] | |
| Lab 9: Shell Scripting | [ ] | |
| Lab 10: SELinux | [ ] | |
| Lab 11: Boot Recovery | [ ] | |
| Lab 12: Containers | [ ] | |
| Lab 13: Ansible | [ ] | |
| Capstone Project | [ ] | |

---

**[Back to Index](README.md)**
