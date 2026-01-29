# 🐧 Linux Crash Course

A comprehensive guide to Linux system administration, focused on RHEL-based distributions.

---

## 📚 Course Modules

| # | Module | Description |
|---|--------|-------------|
| 00 | [Quick Reference](00_quick_reference.md) | Command cheat sheet for quick lookup |
| 01 | [Introduction](01_intro.md) | History, kernel, distributions, open source |
| 02 | [File Manipulation](02_file_manipulation.md) | Navigation, files, directories, archives |
| 03 | [Users & Permissions](03_users_permissions.md) | Users, groups, chmod, chown, ACLs, sudo |
| 04 | [Text Processing](04_text_processing.md) | grep, sed, awk, cut, sort, find, wildcards |
| 05 | [Process Management](05_process_management.md) | ps, top, kill, systemd, cron, logging |
| 06 | [Networking](06_networking.md) | IP config, DNS, firewall, SSH, troubleshooting |
| 07 | [Package Management](07_package_management.md) | DNF, RPM, repositories, modules |
| 08 | [Storage & Filesystems](08_storage_filesystems.md) | Partitions, LVM, mounting, quotas, RAID |
| 09 | [Shell Scripting](09_shell_scripting.md) | Variables, loops, functions, automation |
| 10 | [SELinux & Security](10_selinux_security.md) | SELinux, hardening, auditing, best practices |
| 11 | [Boot & Troubleshooting](11_boot_troubleshooting.md) | Boot process, GRUB, recovery, diagnostics |
| 12 | [Containers & Virtualization](12_containers_virtualization.md) | Podman, Docker, VMs, systemd-nspawn |
| 13 | [Automation with Ansible](13_automation_ansible.md) | Playbooks, modules, roles, best practices |

---

## 🎯 Learning Path

### Beginner
1. Introduction → File Manipulation → Users & Permissions
2. Text Processing → Process Management

### Intermediate
3. Networking → Package Management → Storage
4. Shell Scripting → Containers

### Advanced
5. SELinux & Security → Boot & Troubleshooting
6. Automation with Ansible

---

## 🔧 Prerequisites

- Access to a Linux system (RHEL, CentOS, Rocky, AlmaLinux, or Fedora)
- Terminal/SSH access
- Basic familiarity with command line concepts

---

## 📖 How to Use This Guide

Each module contains:
- **Concept explanations** with diagrams
- **Command syntax** with options
- **Practical examples** you can run
- **Quick reference tables** for review
- **Navigation links** to related modules

### Tips
- Use `Ctrl+F` to search within a module
- Run commands in a test environment first
- Refer to man pages: `man <command>`
- Check the Quick Reference for common commands

---

## 🏷️ Certification Relevance

This material aligns with objectives for:
- **RHCSA** (Red Hat Certified System Administrator)
- **RHCE** (Red Hat Certified Engineer)
- **CompTIA Linux+**
- **LFCS** (Linux Foundation Certified System Administrator)

---

## 📝 Quick Command Reference

| Task | Command |
|------|---------|
| List files | `ls -la` |
| Change directory | `cd /path` |
| View file | `cat file` or `less file` |
| Edit file | `vim file` or `nano file` |
| Find files | `find / -name "*.txt"` |
| Search in files | `grep "pattern" file` |
| Check disk space | `df -h` |
| Check memory | `free -h` |
| View processes | `ps aux` or `top` |
| Check network | `ip a` or `ss -tuln` |
| Manage services | `systemctl status/start/stop service` |
| View logs | `journalctl -xe` |
| Package install | `dnf install package` |

---

## 🔗 Additional Resources

- [Red Hat Documentation](https://access.redhat.com/documentation/)
- [Linux man pages online](https://man7.org/linux/man-pages/)
- [RHEL System Administrator's Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9)
- [Explainshell](https://explainshell.com/) - Explain shell commands

---

## 📂 Repository Structure

```
LinuxCC/
├── README.md                      # This file - main index
├── 00_quick_reference.md          # Command cheat sheet
├── 01_intro.md                    # Linux introduction
├── 02_file_manipulation.md        # File system commands
├── 03_users_permissions.md        # User management
├── 04_text_processing.md          # Text tools (grep, sed, awk)
├── 05_process_management.md       # Process and service management
├── 06_networking.md               # Network configuration
├── 07_package_management.md       # Software management
├── 08_storage_filesystems.md      # Disk and storage
├── 09_shell_scripting.md          # Bash scripting
├── 10_selinux_security.md         # Security hardening
├── 11_boot_troubleshooting.md     # Boot process and recovery
├── 12_containers_virtualization.md # Containers and VMs
└── 13_automation_ansible.md       # Ansible basics
```

---

**Happy Learning! 🚀**
