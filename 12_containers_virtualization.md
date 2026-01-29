# Linux Crash Course - Containers & Virtualization

## Table of Contents
- [Containerization Overview](#containerization-overview)
- [Podman (RHEL Default)](#podman-rhel-default)
- [Docker Basics](#docker-basics)
- [Container Images](#container-images)
- [Container Networking](#container-networking)
- [Container Storage](#container-storage)
- [Container Management](#container-management)
- [Podman Compose](#podman-compose)
- [systemd Integration](#systemd-integration)
- [Virtualization with KVM](#virtualization-with-kvm)
- [Virtual Machine Management](#virtual-machine-management)
- [Best Practices](#best-practices)

---

## Containerization Overview

### Containers vs Virtual Machines
```
┌─────────────────────────────────────────────────────────┐
│              Containers vs Virtual Machines              │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Virtual Machines                 Containers             │
│  ┌─────────────────────┐         ┌─────────────────────┐│
│  │   App A  │   App B  │         │   App A  │   App B  ││
│  ├──────────┴──────────┤         ├──────────┴──────────┤│
│  │   Guest OS │Guest OS│         │  Container Runtime  ││
│  ├─────────────────────┤         │     (Podman/Docker) ││
│  │      Hypervisor     │         ├─────────────────────┤│
│  ├─────────────────────┤         │      Host OS        ││
│  │      Host OS        │         ├─────────────────────┤│
│  ├─────────────────────┤         │      Hardware       ││
│  │      Hardware       │         └─────────────────────┘│
│  └─────────────────────┘                                │
│                                                          │
│  • Full OS per VM                 • Share host kernel   │
│  • Heavy (GB)                     • Lightweight (MB)    │
│  • Minutes to start               • Seconds to start   │
│  • Strong isolation               • Process isolation  │
└─────────────────────────────────────────────────────────┘
```

### Container Components
| Component | Description |
|-----------|-------------|
| **Image** | Read-only template with application and dependencies |
| **Container** | Running instance of an image |
| **Registry** | Storage for container images (Docker Hub, Quay.io) |
| **Runtime** | Software that runs containers (runc, crun) |
| **Engine** | Tool to manage containers (Podman, Docker) |

---

## Podman (RHEL Default)

### Why Podman?
- **Daemonless** - No background service required
- **Rootless** - Run containers as non-root user
- **Docker-compatible** - Same CLI commands
- **Systemd integration** - Generate unit files
- **Pod support** - Group containers like Kubernetes

### Installation
```bash
# RHEL 8/9
sudo dnf install podman

# Verify installation
podman --version
podman info
```

### Basic Commands
```bash
# Pull an image
podman pull registry.access.redhat.com/ubi9/ubi
podman pull docker.io/library/nginx

# List images
podman images
podman image ls

# Run a container
podman run -it ubi9/ubi /bin/bash                    # Interactive
podman run -d --name webserver -p 8080:80 nginx      # Detached with port
podman run -d --name myapp -v /data:/app/data nginx  # With volume

# List containers
podman ps                              # Running
podman ps -a                           # All (including stopped)

# Container operations
podman stop container_name
podman start container_name
podman restart container_name
podman rm container_name
podman rm -f container_name            # Force remove running

# Execute command in container
podman exec -it container_name /bin/bash
podman exec container_name cat /etc/os-release

# View logs
podman logs container_name
podman logs -f container_name          # Follow

# Inspect container
podman inspect container_name

# Container stats
podman stats
podman top container_name
```

### Rootless Containers
```bash
# Run as regular user (no sudo needed)
podman run -d --name myweb -p 8080:80 nginx

# User namespace mapping
podman unshare cat /proc/self/uid_map

# Rootless storage location
~/.local/share/containers/
```

---

## Docker Basics

### Installation (if needed)
```bash
# Add Docker repository
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# Install Docker
sudo dnf install docker-ce docker-ce-cli containerd.io

# Start and enable
sudo systemctl enable --now docker

# Add user to docker group (avoid sudo)
sudo usermod -aG docker $USER
newgrp docker
```

### Docker Commands
```bash
# Same as Podman - replace 'podman' with 'docker'
docker pull nginx
docker run -d --name web -p 80:80 nginx
docker ps
docker logs web
docker exec -it web /bin/bash
docker stop web
docker rm web
```

### Podman vs Docker
| Feature | Podman | Docker |
|---------|--------|--------|
| Daemon | No (daemonless) | Yes (dockerd) |
| Rootless | Native | Requires config |
| CLI | Same syntax | Same syntax |
| Compose | podman-compose | docker-compose |
| Pods | Yes (native) | No (Swarm/K8s) |
| RHEL Default | Yes | No |

---

## Container Images

### Working with Images
```bash
# Search for images
podman search nginx
podman search --filter=is-official nginx

# Pull specific tag
podman pull nginx:latest
podman pull nginx:1.24-alpine

# List images
podman images
podman image ls --format "{{.Repository}}:{{.Tag}}"

# Remove images
podman rmi nginx
podman rmi -f nginx                    # Force
podman image prune                     # Remove unused
podman image prune -a                  # Remove all unused

# Image details
podman inspect nginx
podman history nginx
```

### Building Images (Containerfile/Dockerfile)
```dockerfile
# Containerfile (or Dockerfile)
FROM registry.access.redhat.com/ubi9/ubi

# Set labels
LABEL maintainer="admin@example.com"
LABEL version="1.0"

# Install packages
RUN dnf install -y httpd && \
    dnf clean all

# Copy files
COPY index.html /var/www/html/

# Set working directory
WORKDIR /var/www/html

# Expose port
EXPOSE 80

# Run command
CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]
```

### Building and Tagging
```bash
# Build image
podman build -t myapp:1.0 .
podman build -t myapp:1.0 -f Containerfile .

# Tag image
podman tag myapp:1.0 myapp:latest
podman tag myapp:1.0 registry.example.com/myapp:1.0

# Push to registry
podman login registry.example.com
podman push registry.example.com/myapp:1.0

# Save/Load images
podman save -o myapp.tar myapp:1.0
podman load -i myapp.tar
```

### Multi-stage Builds
```dockerfile
# Build stage
FROM golang:1.20 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# Production stage
FROM registry.access.redhat.com/ubi9/ubi-minimal
COPY --from=builder /app/myapp /usr/local/bin/
CMD ["myapp"]
```

---

## Container Networking

### Network Modes
```bash
# List networks
podman network ls

# Create network
podman network create mynetwork
podman network create --subnet 10.88.0.0/16 mynetwork

# Run container on network
podman run -d --name web --network mynetwork nginx

# Connect container to network
podman network connect mynetwork container_name
podman network disconnect mynetwork container_name

# Inspect network
podman network inspect mynetwork

# Remove network
podman network rm mynetwork
podman network prune                   # Remove unused
```

### Port Mapping
```bash
# Map host port to container port
podman run -d -p 8080:80 nginx                      # Host:Container
podman run -d -p 127.0.0.1:8080:80 nginx            # Bind to localhost
podman run -d -p 8080-8090:80-90 nginx              # Port range
podman run -d -P nginx                              # Random host ports

# View port mappings
podman port container_name
```

### Container DNS
```bash
# Containers on same network can resolve by name
podman run -d --name db --network mynet mysql
podman run -d --name app --network mynet myapp
# 'app' container can connect to 'db' by name
```

---

## Container Storage

### Volume Types
```bash
# Named volumes (managed by Podman)
podman volume create mydata
podman run -v mydata:/app/data nginx

# Bind mounts (host directory)
podman run -v /host/path:/container/path nginx
podman run -v /host/path:/container/path:Z nginx    # SELinux label

# tmpfs (in memory)
podman run --tmpfs /tmp nginx
```

### Volume Management
```bash
# List volumes
podman volume ls

# Inspect volume
podman volume inspect mydata

# Remove volume
podman volume rm mydata
podman volume prune                    # Remove unused

# Backup volume
podman run --rm -v mydata:/data -v $(pwd):/backup alpine \
    tar cvf /backup/backup.tar /data
```

### SELinux and Volumes
```bash
# :z = shared label (multiple containers)
podman run -v /data:/app/data:z nginx

# :Z = private label (single container)
podman run -v /data:/app/data:Z nginx

# Read-only
podman run -v /data:/app/data:ro nginx
```

---

## Container Management

### Resource Limits
```bash
# Memory limits
podman run -m 512m nginx
podman run --memory 1g --memory-swap 2g nginx

# CPU limits
podman run --cpus 0.5 nginx                         # Half a CPU
podman run --cpus 2 nginx                           # Two CPUs
podman run --cpu-shares 512 nginx                   # Relative weight

# Combined
podman run -d --name app -m 1g --cpus 1 -p 8080:80 nginx
```

### Health Checks
```bash
# In Containerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
    CMD curl -f http://localhost/ || exit 1

# At runtime
podman run -d --health-cmd "curl -f http://localhost/" \
    --health-interval 30s nginx

# Check health status
podman inspect --format='{{.State.Health.Status}}' container_name
```

### Container Cleanup
```bash
# Remove stopped containers
podman container prune

# Remove all unused (containers, images, networks)
podman system prune

# Full cleanup
podman system prune -a --volumes

# View disk usage
podman system df
```

---

## Podman Compose

### Installation
```bash
# Install podman-compose
sudo dnf install podman-compose

# Or via pip
pip3 install podman-compose
```

### Compose File Example
```yaml
# docker-compose.yml (or podman-compose.yml)
version: '3'
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html:Z
    depends_on:
      - db
    networks:
      - webnet

  db:
    image: mariadb:10
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: myapp
    volumes:
      - dbdata:/var/lib/mysql:Z
    networks:
      - webnet

volumes:
  dbdata:

networks:
  webnet:
```

### Compose Commands
```bash
# Start services
podman-compose up -d

# Stop services
podman-compose down

# View logs
podman-compose logs
podman-compose logs -f web

# Scale service
podman-compose up -d --scale web=3

# Execute command
podman-compose exec web /bin/sh
```

---

## systemd Integration

### Generate systemd Unit File
```bash
# Create unit file from running container
podman generate systemd --name mycontainer > ~/.config/systemd/user/mycontainer.service

# With restart policy
podman generate systemd --name mycontainer --restart-policy=always

# Generate for new container
podman generate systemd --new --name mycontainer
```

### Example Unit File
```ini
# ~/.config/systemd/user/container-web.service
[Unit]
Description=Web container
After=network-online.target

[Service]
Restart=on-failure
ExecStartPre=-/usr/bin/podman rm -f web
ExecStart=/usr/bin/podman run --name web -p 8080:80 nginx
ExecStop=/usr/bin/podman stop web

[Install]
WantedBy=default.target
```

### Managing Container Services
```bash
# Rootless (user service)
systemctl --user daemon-reload
systemctl --user enable --now container-web.service
systemctl --user status container-web.service

# Enable lingering (survive logout)
loginctl enable-linger $USER

# Root service
sudo systemctl enable --now container-web.service
```

---

## Virtualization with KVM

### KVM Overview
```
┌─────────────────────────────────────────────────────────┐
│                    KVM Architecture                      │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │  Virtual Machine (Guest)                          │   │
│  │  ┌────────────────────────────────────────────┐  │   │
│  │  │  Applications                               │  │   │
│  │  ├────────────────────────────────────────────┤  │   │
│  │  │  Guest OS (Linux, Windows)                 │  │   │
│  │  ├────────────────────────────────────────────┤  │   │
│  │  │  Virtual Hardware (CPU, RAM, Disk, NIC)    │  │   │
│  │  └────────────────────────────────────────────┘  │   │
│  └──────────────────────────────────────────────────┘   │
│                          │                               │
│  ┌──────────────────────────────────────────────────┐   │
│  │  QEMU (Emulation) + libvirt (Management)         │   │
│  └──────────────────────────────────────────────────┘   │
│                          │                               │
│  ┌──────────────────────────────────────────────────┐   │
│  │  KVM (Kernel-based Virtual Machine)              │   │
│  └──────────────────────────────────────────────────┘   │
│                          │                               │
│  ┌──────────────────────────────────────────────────┐   │
│  │  Linux Kernel + Hardware (VT-x/AMD-V)            │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### Installation
```bash
# Check virtualization support
grep -E 'vmx|svm' /proc/cpuinfo

# Install KVM and tools
sudo dnf install @virtualization

# Or individually
sudo dnf install qemu-kvm libvirt virt-install virt-viewer

# Start libvirtd
sudo systemctl enable --now libvirtd

# Verify
virsh list --all
```

### Network Configuration
```bash
# List networks
virsh net-list --all

# Start default network
virsh net-start default
virsh net-autostart default

# Create bridge network
sudo nmcli con add type bridge con-name br0 ifname br0
sudo nmcli con add type ethernet slave-type bridge con-name br0-port1 \
    ifname enp0s3 master br0
```

---

## Virtual Machine Management

### Creating VMs
```bash
# Using virt-install
sudo virt-install \
    --name rhel9-vm \
    --ram 2048 \
    --vcpus 2 \
    --disk path=/var/lib/libvirt/images/rhel9.qcow2,size=20 \
    --os-variant rhel9.0 \
    --network network=default \
    --graphics vnc \
    --cdrom /path/to/rhel9.iso

# Minimal (text install)
sudo virt-install \
    --name test-vm \
    --ram 1024 \
    --vcpus 1 \
    --disk size=10 \
    --os-variant generic \
    --network network=default \
    --graphics none \
    --console pty,target_type=serial \
    --location /path/to/iso \
    --extra-args 'console=ttyS0,115200n8 serial'
```

### virsh Commands
```bash
# List VMs
virsh list --all

# Start/Stop
virsh start vm-name
virsh shutdown vm-name                 # Graceful
virsh destroy vm-name                  # Force stop
virsh reboot vm-name

# Console access
virsh console vm-name                  # Serial console
virt-viewer vm-name                    # GUI

# VM info
virsh dominfo vm-name
virsh domblklist vm-name               # Disk info
virsh domiflist vm-name                # Network info

# Modify VM
virsh setmem vm-name 4G --config       # Change memory
virsh setvcpus vm-name 4 --config      # Change CPUs

# Delete VM
virsh undefine vm-name
virsh undefine --remove-all-storage vm-name
```

### Snapshots
```bash
# Create snapshot
virsh snapshot-create-as vm-name snapshot1 "Before update"

# List snapshots
virsh snapshot-list vm-name

# Revert to snapshot
virsh snapshot-revert vm-name snapshot1

# Delete snapshot
virsh snapshot-delete vm-name snapshot1
```

### VM Templates and Cloning
```bash
# Clone VM
virt-clone --original vm-name --name new-vm --auto-clone

# Export VM
virsh dumpxml vm-name > vm-name.xml

# Import VM
virsh define vm-name.xml
```

---

## Best Practices

### Container Best Practices
```
┌─────────────────────────────────────────────────────────┐
│              Container Best Practices                    │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  1. Use Official/Trusted Base Images                    │
│     • registry.access.redhat.com/ubi9/ubi               │
│     • Verify image signatures                           │
│                                                          │
│  2. Keep Images Small                                   │
│     • Use minimal base images (ubi-minimal, alpine)     │
│     • Multi-stage builds                                │
│     • Clean up in same RUN layer                        │
│                                                          │
│  3. Don't Run as Root                                   │
│     • USER directive in Containerfile                   │
│     • Rootless containers with Podman                   │
│                                                          │
│  4. One Process per Container                           │
│     • Easier to scale and manage                        │
│     • Use compose for multi-container apps              │
│                                                          │
│  5. Use Health Checks                                   │
│     • Monitor container health                          │
│     • Enable automatic restarts                         │
│                                                          │
│  6. Manage Secrets Properly                             │
│     • Use secrets management                            │
│     • Don't embed in images                             │
│                                                          │
│  7. Tag Images Properly                                 │
│     • Avoid 'latest' in production                      │
│     • Use semantic versioning                           │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### Security Considerations
```bash
# Run read-only root filesystem
podman run --read-only nginx

# Drop capabilities
podman run --cap-drop ALL --cap-add NET_BIND_SERVICE nginx

# No new privileges
podman run --security-opt no-new-privileges nginx

# User namespace
podman run --userns keep-id nginx

# Scan images for vulnerabilities
podman image scan nginx
```

### Quick Reference

| Task | Command |
|------|---------|
| Pull image | `podman pull nginx` |
| Run container | `podman run -d -p 8080:80 nginx` |
| List containers | `podman ps -a` |
| View logs | `podman logs container` |
| Stop container | `podman stop container` |
| Remove container | `podman rm container` |
| Build image | `podman build -t myapp .` |
| List images | `podman images` |
| Remove image | `podman rmi image` |
| Create volume | `podman volume create data` |
| Cleanup | `podman system prune -a` |

---

**Previous: [Boot & Troubleshooting](11_boot_troubleshooting.md)**  
**Next: [Automation with Ansible](13_automation_ansible.md)**
