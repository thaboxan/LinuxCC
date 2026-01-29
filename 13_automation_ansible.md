# Linux Crash Course - Automation with Ansible

## Table of Contents
- [Ansible Overview](#ansible-overview)
- [Installation and Setup](#installation-and-setup)
- [Inventory Management](#inventory-management)
- [Ad-hoc Commands](#ad-hoc-commands)
- [Playbooks](#playbooks)
- [Variables and Facts](#variables-and-facts)
- [Conditionals and Loops](#conditionals-and-loops)
- [Handlers](#handlers)
- [Templates (Jinja2)](#templates-jinja2)
- [Roles](#roles)
- [Ansible Vault](#ansible-vault)
- [Common Modules](#common-modules)
- [Best Practices](#best-practices)

---

## Ansible Overview

### What is Ansible?
```
┌─────────────────────────────────────────────────────────┐
│                   Ansible Architecture                   │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │              Control Node                        │    │
│  │  • Ansible installed here                       │    │
│  │  • Runs playbooks                               │    │
│  │  • No agents needed on targets                  │    │
│  └───────────────────────┬─────────────────────────┘    │
│                          │ SSH                           │
│          ┌───────────────┼───────────────┐              │
│          │               │               │              │
│          ▼               ▼               ▼              │
│  ┌───────────┐   ┌───────────┐   ┌───────────┐         │
│  │  Managed  │   │  Managed  │   │  Managed  │         │
│  │   Node 1  │   │   Node 2  │   │   Node 3  │         │
│  │  (Target) │   │  (Target) │   │  (Target) │         │
│  └───────────┘   └───────────┘   └───────────┘         │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### Key Concepts
| Concept | Description |
|---------|-------------|
| **Control Node** | Machine where Ansible is installed and runs |
| **Managed Node** | Target machines managed by Ansible |
| **Inventory** | List of managed nodes |
| **Playbook** | YAML file with automation tasks |
| **Task** | Single action to perform |
| **Module** | Unit of code Ansible executes |
| **Role** | Reusable collection of tasks, vars, templates |
| **Handler** | Task triggered by notification |
| **Facts** | System information gathered from nodes |

### Why Ansible?
- **Agentless** - Uses SSH, no software to install on targets
- **Simple** - YAML syntax, easy to read and write
- **Idempotent** - Safe to run multiple times
- **Powerful** - Thousands of modules available
- **Extensible** - Custom modules and plugins

---

## Installation and Setup

### Installing Ansible
```bash
# RHEL 9
sudo dnf install ansible-core

# With additional collections
sudo dnf install ansible

# Verify installation
ansible --version

# Check configuration
ansible-config dump
```

### Configuration File
```bash
# Priority order:
# 1. ANSIBLE_CONFIG environment variable
# 2. ./ansible.cfg (current directory)
# 3. ~/.ansible.cfg (home directory)
# 4. /etc/ansible/ansible.cfg (system-wide)

# Create project config
cat > ansible.cfg << 'EOF'
[defaults]
inventory = ./inventory
remote_user = ansible
host_key_checking = False
retry_files_enabled = False

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False
EOF
```

### SSH Key Setup
```bash
# Generate SSH key (if not exists)
ssh-keygen -t ed25519 -C "ansible"

# Copy key to managed nodes
ssh-copy-id user@managed-node

# Or use ansible to distribute keys
ansible all -m authorized_key -a "user=ansible key='{{ lookup('file', '~/.ssh/id_ed25519.pub') }}'" -k
```

---

## Inventory Management

### Static Inventory (INI Format)
```ini
# inventory or /etc/ansible/hosts

# Ungrouped hosts
server1.example.com
192.168.1.100

# Groups
[webservers]
web1.example.com
web2.example.com ansible_host=192.168.1.10

[dbservers]
db1.example.com
db2.example.com

# Group variables
[webservers:vars]
http_port=80
ansible_user=webadmin

# Nested groups
[production:children]
webservers
dbservers

# Host variables
[dbservers]
db1.example.com ansible_port=2222 db_name=production
```

### YAML Inventory
```yaml
# inventory.yml
all:
  hosts:
    server1.example.com:
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:
          ansible_host: 192.168.1.10
      vars:
        http_port: 80
    dbservers:
      hosts:
        db1.example.com:
          db_name: production
        db2.example.com:
    production:
      children:
        webservers:
        dbservers:
```

### Dynamic Inventory
```bash
# Use scripts or plugins
ansible-inventory -i dynamic_inventory.py --list

# AWS EC2 example (ansible.cfg)
[inventory]
enable_plugins = aws_ec2

# AWS EC2 inventory file (aws_ec2.yml)
plugin: aws_ec2
regions:
  - us-east-1
filters:
  tag:Environment: production
```

### Inventory Commands
```bash
# List all hosts
ansible-inventory --list
ansible-inventory --graph

# Test connectivity
ansible all -m ping
ansible webservers -m ping

# List hosts in group
ansible webservers --list-hosts
```

---

## Ad-hoc Commands

### Syntax
```bash
ansible <pattern> -m <module> -a "<arguments>"
```

### Common Ad-hoc Commands
```bash
# Ping all hosts
ansible all -m ping

# Run command
ansible all -m command -a "uptime"
ansible all -a "uptime"                    # command is default module

# Shell command (supports pipes, redirects)
ansible all -m shell -a "cat /etc/passwd | grep root"

# Copy file
ansible webservers -m copy -a "src=/tmp/file dest=/tmp/file mode=0644"

# Install package
ansible webservers -m dnf -a "name=httpd state=present" --become

# Start service
ansible webservers -m service -a "name=httpd state=started enabled=yes" --become

# Create user
ansible all -m user -a "name=john state=present" --become

# Gather facts
ansible server1 -m setup
ansible server1 -m setup -a "filter=ansible_distribution*"

# File operations
ansible all -m file -a "path=/tmp/test state=touch"
ansible all -m file -a "path=/tmp/testdir state=directory mode=0755"

# Fetch file from remote
ansible all -m fetch -a "src=/etc/hostname dest=/tmp/hostnames flat=yes"
```

### Patterns
```bash
# All hosts
ansible all -m ping

# Specific group
ansible webservers -m ping

# Multiple groups
ansible 'webservers:dbservers' -m ping

# Intersection (AND)
ansible 'webservers:&production' -m ping

# Exclusion
ansible 'all:!dbservers' -m ping

# Regex
ansible '~web[0-9]+' -m ping

# Specific host
ansible server1.example.com -m ping
```

---

## Playbooks

### Playbook Structure
```yaml
---
# playbook.yml
- name: Configure web servers           # Play name
  hosts: webservers                      # Target hosts
  become: yes                            # Run as root
  vars:                                  # Variables
    http_port: 80

  tasks:                                 # List of tasks
    - name: Install Apache               # Task name
      dnf:                               # Module
        name: httpd                      # Module arguments
        state: present

    - name: Start Apache
      service:
        name: httpd
        state: started
        enabled: yes

- name: Configure database servers       # Second play
  hosts: dbservers
  become: yes
  tasks:
    - name: Install MariaDB
      dnf:
        name: mariadb-server
        state: present
```

### Running Playbooks
```bash
# Run playbook
ansible-playbook playbook.yml

# Dry run (check mode)
ansible-playbook playbook.yml --check

# Show differences
ansible-playbook playbook.yml --check --diff

# Limit to specific hosts
ansible-playbook playbook.yml --limit web1

# Start at specific task
ansible-playbook playbook.yml --start-at-task "Start Apache"

# List tasks
ansible-playbook playbook.yml --list-tasks

# List hosts
ansible-playbook playbook.yml --list-hosts

# Step through tasks
ansible-playbook playbook.yml --step

# Verbose output
ansible-playbook playbook.yml -v      # -vv, -vvv, -vvvv for more
```

### Complete Playbook Example
```yaml
---
- name: Deploy web application
  hosts: webservers
  become: yes
  vars:
    app_name: mywebapp
    http_port: 80
    doc_root: /var/www/html

  tasks:
    - name: Install required packages
      dnf:
        name:
          - httpd
          - php
          - php-mysqlnd
        state: present

    - name: Create document root
      file:
        path: "{{ doc_root }}"
        state: directory
        owner: apache
        group: apache
        mode: '0755'

    - name: Deploy application files
      copy:
        src: files/index.php
        dest: "{{ doc_root }}/index.php"
        owner: apache
        group: apache
        mode: '0644'
      notify: Restart Apache

    - name: Configure firewall
      firewalld:
        service: http
        permanent: yes
        state: enabled
        immediate: yes

    - name: Ensure Apache is running
      service:
        name: httpd
        state: started
        enabled: yes

  handlers:
    - name: Restart Apache
      service:
        name: httpd
        state: restarted
```

---

## Variables and Facts

### Defining Variables
```yaml
# In playbook
- hosts: webservers
  vars:
    http_port: 80
    max_clients: 200

# From file
- hosts: webservers
  vars_files:
    - vars/common.yml
    - vars/webservers.yml

# Prompt user
- hosts: webservers
  vars_prompt:
    - name: username
      prompt: "Enter username"
      private: no
```

### Variable Files
```yaml
# vars/common.yml
---
ntp_server: time.example.com
dns_servers:
  - 8.8.8.8
  - 8.8.4.4

packages:
  - vim
  - wget
  - curl
```

### Variable Precedence (lowest to highest)
1. Role defaults
2. Inventory group_vars
3. Inventory host_vars
4. Playbook group_vars
5. Playbook host_vars
6. Host facts
7. Play vars
8. Play vars_files
9. Task vars
10. Block vars
11. Role vars
12. Include vars
13. Set_facts
14. Extra vars (-e)

### Registered Variables
```yaml
- name: Run command and register output
  command: hostname
  register: hostname_result

- name: Display hostname
  debug:
    msg: "Hostname is {{ hostname_result.stdout }}"
```

### Facts
```yaml
# Access facts
- name: Display distribution
  debug:
    msg: "OS is {{ ansible_distribution }} {{ ansible_distribution_version }}"

# Common facts
ansible_hostname            # Short hostname
ansible_fqdn                # Full hostname
ansible_distribution        # CentOS, RedHat, Ubuntu
ansible_distribution_version
ansible_os_family           # RedHat, Debian
ansible_kernel              # Kernel version
ansible_memtotal_mb         # Total memory
ansible_processor_vcpus     # CPU count
ansible_default_ipv4.address
ansible_all_ipv4_addresses

# Disable fact gathering (faster)
- hosts: all
  gather_facts: no
```

### Custom Facts
```bash
# Create on managed node: /etc/ansible/facts.d/custom.fact
[application]
name=myapp
version=1.0

# Access in playbook
{{ ansible_local.custom.application.name }}
```

---

## Conditionals and Loops

### When Conditionals
```yaml
- name: Install on RHEL
  dnf:
    name: httpd
    state: present
  when: ansible_distribution == "RedHat"

- name: Install on Debian
  apt:
    name: apache2
    state: present
  when: ansible_distribution == "Debian"

# Multiple conditions
- name: Install on RHEL 9
  dnf:
    name: httpd
  when:
    - ansible_distribution == "RedHat"
    - ansible_distribution_major_version == "9"

# OR condition
- name: Install on RHEL or CentOS
  dnf:
    name: httpd
  when: ansible_distribution == "RedHat" or ansible_distribution == "CentOS"

# Registered variable
- name: Check if file exists
  stat:
    path: /etc/myapp.conf
  register: config_file

- name: Create config if missing
  copy:
    src: myapp.conf
    dest: /etc/myapp.conf
  when: not config_file.stat.exists
```

### Loops
```yaml
# Simple loop
- name: Install packages
  dnf:
    name: "{{ item }}"
    state: present
  loop:
    - httpd
    - php
    - mariadb

# Better: pass list to module
- name: Install packages
  dnf:
    name:
      - httpd
      - php
      - mariadb
    state: present

# Loop with dict
- name: Create users
  user:
    name: "{{ item.name }}"
    groups: "{{ item.groups }}"
  loop:
    - { name: 'john', groups: 'wheel' }
    - { name: 'jane', groups: 'developers' }

# Loop with index
- name: Create files
  copy:
    content: "File {{ ansible_loop.index }}"
    dest: "/tmp/file{{ ansible_loop.index }}.txt"
  loop:
    - one
    - two
    - three
  loop_control:
    extended: yes

# Loop from variable
- name: Create users from list
  user:
    name: "{{ item }}"
  loop: "{{ users }}"
  vars:
    users:
      - alice
      - bob
      - charlie
```

### Blocks
```yaml
- name: Handle web server tasks
  block:
    - name: Install Apache
      dnf:
        name: httpd
        state: present

    - name: Start Apache
      service:
        name: httpd
        state: started

  rescue:
    - name: Handle failure
      debug:
        msg: "Web server setup failed!"

  always:
    - name: Always run this
      debug:
        msg: "This runs regardless of success/failure"

  when: ansible_distribution == "RedHat"
  become: yes
```

---

## Handlers

### Basic Handlers
```yaml
- hosts: webservers
  become: yes
  tasks:
    - name: Update Apache config
      copy:
        src: httpd.conf
        dest: /etc/httpd/conf/httpd.conf
      notify: Restart Apache

    - name: Update SSL config
      copy:
        src: ssl.conf
        dest: /etc/httpd/conf.d/ssl.conf
      notify:
        - Restart Apache
        - Reload Firewall

  handlers:
    - name: Restart Apache
      service:
        name: httpd
        state: restarted

    - name: Reload Firewall
      command: firewall-cmd --reload
```

### Handler Notes
- Handlers run once at end of play, even if notified multiple times
- Run in order defined, not order notified
- Use `--force-handlers` to run handlers even if play fails
- Use `meta: flush_handlers` to run handlers immediately

```yaml
- name: Flush handlers now
  meta: flush_handlers
```

---

## Templates (Jinja2)

### Template Basics
```jinja2
{# templates/httpd.conf.j2 #}
# Apache configuration for {{ ansible_hostname }}
# Generated by Ansible - DO NOT EDIT

ServerRoot "/etc/httpd"
Listen {{ http_port }}
ServerName {{ ansible_fqdn }}

DocumentRoot "{{ doc_root }}"

# Virtual hosts
{% for vhost in virtual_hosts %}
<VirtualHost *:{{ http_port }}>
    ServerName {{ vhost.name }}
    DocumentRoot {{ vhost.docroot }}
</VirtualHost>
{% endfor %}

# Security settings
{% if security_enabled %}
ServerTokens Prod
ServerSignature Off
{% endif %}
```

### Using Templates
```yaml
- name: Deploy Apache config
  template:
    src: httpd.conf.j2
    dest: /etc/httpd/conf/httpd.conf
    owner: root
    group: root
    mode: '0644'
    backup: yes
  notify: Restart Apache
```

### Jinja2 Features
```jinja2
{# Comments #}
{{ variable }}                              {# Variable #}
{{ variable | default('value') }}           {# Default value #}
{{ variable | upper }}                      {# Filter: uppercase #}
{{ variable | lower }}                      {# Filter: lowercase #}
{{ list | join(', ') }}                     {# Join list #}
{{ dict | to_nice_yaml }}                   {# Convert to YAML #}

{# Conditionals #}
{% if condition %}
content
{% elif other %}
other content
{% else %}
default content
{% endif %}

{# Loops #}
{% for item in list %}
{{ item }}
{% endfor %}

{% for key, value in dict.items() %}
{{ key }}: {{ value }}
{% endfor %}

{# Loop variables #}
{% for item in list %}
{{ loop.index }}        {# 1-indexed #}
{{ loop.index0 }}       {# 0-indexed #}
{{ loop.first }}        {# True if first #}
{{ loop.last }}         {# True if last #}
{% endfor %}
```

---

## Roles

### Role Structure
```
roles/
└── webserver/
    ├── tasks/
    │   └── main.yml           # Main tasks
    ├── handlers/
    │   └── main.yml           # Handlers
    ├── templates/
    │   └── httpd.conf.j2      # Templates
    ├── files/
    │   └── index.html         # Static files
    ├── vars/
    │   └── main.yml           # Role variables
    ├── defaults/
    │   └── main.yml           # Default variables (lowest priority)
    ├── meta/
    │   └── main.yml           # Role metadata, dependencies
    └── README.md              # Documentation
```

### Creating a Role
```bash
# Create role structure
ansible-galaxy init roles/webserver

# Or manually
mkdir -p roles/webserver/{tasks,handlers,templates,files,vars,defaults,meta}
```

### Role Files
```yaml
# roles/webserver/tasks/main.yml
---
- name: Install Apache
  dnf:
    name: httpd
    state: present

- name: Deploy config
  template:
    src: httpd.conf.j2
    dest: /etc/httpd/conf/httpd.conf
  notify: Restart Apache

- name: Start Apache
  service:
    name: httpd
    state: started
    enabled: yes
```

```yaml
# roles/webserver/handlers/main.yml
---
- name: Restart Apache
  service:
    name: httpd
    state: restarted
```

```yaml
# roles/webserver/defaults/main.yml
---
http_port: 80
doc_root: /var/www/html
```

```yaml
# roles/webserver/meta/main.yml
---
dependencies:
  - role: common
  - role: firewall
    vars:
      firewall_services:
        - http
        - https
```

### Using Roles
```yaml
# Playbook using roles
---
- hosts: webservers
  become: yes
  roles:
    - common
    - webserver
    - { role: database, when: "inventory_hostname in groups['dbservers']" }
    - role: security
      vars:
        security_level: high
```

### Ansible Galaxy
```bash
# Install role from Galaxy
ansible-galaxy install geerlingguy.apache

# Install from requirements file
ansible-galaxy install -r requirements.yml

# requirements.yml
---
roles:
  - name: geerlingguy.apache
  - name: geerlingguy.mysql
    version: 3.3.0
  - src: https://github.com/user/repo.git
    name: custom_role

collections:
  - name: ansible.posix
  - name: community.general
```

---

## Ansible Vault

### Encrypting Content
```bash
# Create encrypted file
ansible-vault create secrets.yml

# Encrypt existing file
ansible-vault encrypt vars/secrets.yml

# View encrypted file
ansible-vault view secrets.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# Decrypt file
ansible-vault decrypt secrets.yml

# Change password
ansible-vault rekey secrets.yml
```

### Using Vault in Playbooks
```bash
# Run playbook with vault password
ansible-playbook site.yml --ask-vault-pass

# Use password file
ansible-playbook site.yml --vault-password-file ~/.vault_pass

# Multiple vaults
ansible-playbook site.yml --vault-id dev@~/.vault_dev --vault-id prod@~/.vault_prod
```

### Encrypting Variables
```yaml
# Encrypt single variable
ansible-vault encrypt_string 'secret_value' --name 'db_password'

# Output (paste in vars file):
db_password: !vault |
  $ANSIBLE_VAULT;1.1;AES256
  61626364656667...
```

### Vault Best Practices
```yaml
# Keep encrypted and unencrypted vars separate
group_vars/
├── webservers/
│   ├── vars.yml           # Regular variables
│   └── vault.yml          # Encrypted variables

# Reference vault variables in regular files
# vars.yml
db_password: "{{ vault_db_password }}"

# vault.yml (encrypted)
vault_db_password: supersecret
```

---

## Common Modules

### Package Management
```yaml
# DNF/YUM
- dnf:
    name: httpd
    state: present    # present, absent, latest

# APT
- apt:
    name: apache2
    state: present
    update_cache: yes
```

### Files and Directories
```yaml
# Copy
- copy:
    src: file.txt
    dest: /tmp/file.txt
    owner: root
    mode: '0644'

# Template
- template:
    src: config.j2
    dest: /etc/app.conf

# File operations
- file:
    path: /tmp/dir
    state: directory    # directory, file, link, absent, touch
    mode: '0755'

# Lineinfile
- lineinfile:
    path: /etc/hosts
    line: "192.168.1.1 server1"
    state: present

# Blockinfile
- blockinfile:
    path: /etc/hosts
    block: |
      192.168.1.1 server1
      192.168.1.2 server2
```

### Services
```yaml
- service:
    name: httpd
    state: started      # started, stopped, restarted, reloaded
    enabled: yes

- systemd:
    name: httpd
    state: started
    daemon_reload: yes
```

### Users and Groups
```yaml
- user:
    name: john
    groups: wheel
    shell: /bin/bash
    state: present

- group:
    name: developers
    state: present
```

### Commands
```yaml
# command (no shell features)
- command: /usr/bin/uptime

# shell (pipes, redirects work)
- shell: cat /etc/passwd | grep root

# script
- script: scripts/setup.sh

# raw (no Python needed)
- raw: yum install -y python3
```

### Firewall
```yaml
- firewalld:
    service: http
    permanent: yes
    state: enabled
    immediate: yes

- firewalld:
    port: 8080/tcp
    permanent: yes
    state: enabled
```

---

## Best Practices

### Project Structure
```
ansible-project/
├── ansible.cfg                 # Configuration
├── inventory/
│   ├── production/
│   │   ├── hosts               # Production inventory
│   │   ├── group_vars/
│   │   └── host_vars/
│   └── staging/
│       ├── hosts               # Staging inventory
│       ├── group_vars/
│       └── host_vars/
├── playbooks/
│   ├── site.yml                # Master playbook
│   ├── webservers.yml
│   └── dbservers.yml
├── roles/
│   ├── common/
│   ├── webserver/
│   └── database/
├── files/
├── templates/
└── requirements.yml            # Galaxy dependencies
```

### Coding Standards
```yaml
# Always name tasks
- name: Install Apache          # Good
  dnf:
    name: httpd

- dnf: name=httpd               # Bad (no name, old syntax)

# Use YAML syntax, not key=value
- name: Copy file
  copy:
    src: file.txt
    dest: /tmp/file.txt         # Good

- copy: src=file.txt dest=/tmp/  # Bad

# Use FQCN for modules
- name: Install package
  ansible.builtin.dnf:          # Fully Qualified Collection Name
    name: httpd

# Quote strings that might be interpreted wrong
- name: Set variable
  set_fact:
    version: "1.0"              # Quote to ensure string
    enabled: true               # Boolean, no quotes
```

### Tips
```
┌─────────────────────────────────────────────────────────┐
│              Ansible Best Practices                      │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  1. Use Version Control                                 │
│     • Keep playbooks in Git                             │
│     • Review changes before applying                    │
│                                                          │
│  2. Test Before Production                              │
│     • Use --check mode                                  │
│     • Test on staging first                             │
│     • Use molecule for role testing                     │
│                                                          │
│  3. Keep Playbooks Simple                               │
│     • One purpose per playbook                          │
│     • Use roles for reusability                         │
│     • Small, focused tasks                              │
│                                                          │
│  4. Use Variables Properly                              │
│     • group_vars for group settings                     │
│     • host_vars for host-specific                       │
│     • Vault for secrets                                 │
│                                                          │
│  5. Document Everything                                 │
│     • README for each role                              │
│     • Comments in complex playbooks                     │
│     • Describe variable purposes                        │
│                                                          │
│  6. Idempotency                                         │
│     • Playbooks should be safe to run multiple times   │
│     • Avoid command/shell when modules exist           │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### Quick Reference

| Task | Command |
|------|---------|
| Run playbook | `ansible-playbook site.yml` |
| Dry run | `ansible-playbook site.yml --check` |
| Limit hosts | `ansible-playbook site.yml --limit web1` |
| Verbose | `ansible-playbook site.yml -vvv` |
| List tasks | `ansible-playbook site.yml --list-tasks` |
| Ad-hoc ping | `ansible all -m ping` |
| Ad-hoc command | `ansible all -a "uptime"` |
| Vault create | `ansible-vault create secrets.yml` |
| Vault edit | `ansible-vault edit secrets.yml` |
| Install role | `ansible-galaxy install role_name` |
| Create role | `ansible-galaxy init roles/myrole` |

---

**Previous: [Containers & Virtualization](12_containers_virtualization.md)**  
**Next: [Back to Index](README.md)**
