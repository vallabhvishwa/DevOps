# The Complete DevOps Engineer's Reference Guide
## Part 1: Linux Mastery

> **About This Guide:** This is a comprehensive, book-style reference designed to teach you everything you need to know without requiring external resources. Every concept is explained from first principles with complete depth.

---

# Chapter 1: Understanding Linux - The Foundation

## 1.1 What is Linux?

Linux is a free, open-source operating system kernel created by Linus Torvalds in 1991. When people say "Linux," they usually mean a complete operating system that includes the Linux kernel plus a collection of software tools, libraries, and utilities (often from the GNU project). This combination is more accurately called "GNU/Linux."

### The Kernel Explained

The **kernel** is the core component of any operating system. It's the software that sits between your hardware and your applications. Think of it as a translator and manager:

```
┌─────────────────────────────────────────────────────────────┐
│                     User Applications                        │
│              (browsers, editors, your apps)                  │
├─────────────────────────────────────────────────────────────┤
│                      System Libraries                        │
│                    (glibc, libssl, etc.)                    │
├─────────────────────────────────────────────────────────────┤
│                     System Calls Interface                   │
│           (the API between userspace and kernel)            │
├─────────────────────────────────────────────────────────────┤
│                        Linux Kernel                          │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │
│  │ Process  │ │  Memory  │ │   File   │ │ Network  │       │
│  │ Manager  │ │ Manager  │ │ Systems  │ │  Stack   │       │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘       │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐                    │
│  │  Device  │ │ Security │ │   IPC    │                    │
│  │ Drivers  │ │ (SELinux)│ │          │                    │
│  └──────────┘ └──────────┘ └──────────┘                    │
├─────────────────────────────────────────────────────────────┤
│                         Hardware                             │
│        (CPU, RAM, Disk, Network Cards, USB, etc.)           │
└─────────────────────────────────────────────────────────────┘
```

**What the kernel does:**

1. **Process Management**: Decides which programs run, when they run, and for how long. On a multi-core CPU, it distributes work across cores.

2. **Memory Management**: Allocates RAM to programs, ensures one program can't access another's memory (isolation), and handles virtual memory (using disk as extended RAM when needed).

3. **File System Management**: Provides a unified way to access files on different storage types (SSD, HDD, network storage, USB drives).

4. **Device Drivers**: Contains code to communicate with hardware devices. When you plug in a USB drive, the kernel's driver handles the communication.

5. **Network Stack**: Implements networking protocols (TCP/IP) so your applications can communicate over networks.

6. **Security**: Enforces permissions, user isolation, and security modules like SELinux or AppArmor.

### Linux Distributions

A Linux distribution (distro) packages the Linux kernel with additional software to create a complete operating system. Different distros are optimized for different use cases:

| Distribution | Base | Package Manager | Use Case |
|-------------|------|-----------------|----------|
| **Ubuntu** | Debian | apt/dpkg | General purpose, popular for servers and desktops |
| **Debian** | Independent | apt/dpkg | Stability-focused, very reliable |
| **RHEL** | Independent | yum/dnf/rpm | Enterprise, commercial support |
| **CentOS Stream** | RHEL | yum/dnf/rpm | Free RHEL-like for development |
| **Rocky Linux** | RHEL | yum/dnf/rpm | RHEL-compatible, community-driven |
| **AlmaLinux** | RHEL | yum/dnf/rpm | RHEL-compatible, community-driven |
| **Fedora** | Independent | dnf/rpm | Cutting-edge features, RHEL preview |
| **Alpine** | Independent | apk | Minimal, security-focused, container images |
| **Arch Linux** | Independent | pacman | Rolling release, latest software |
| **SUSE/openSUSE** | Independent | zypper/rpm | Enterprise, YaST configuration tool |

**For DevOps engineers**, you'll most commonly encounter:
- **Ubuntu** (especially Ubuntu Server LTS versions)
- **RHEL/Rocky/AlmaLinux** (enterprise environments)
- **Alpine** (container base images due to small size)
- **Debian** (container base images, stable servers)

---

## 1.2 The Linux File System Hierarchy

Every file and directory in Linux starts from a single root directory, denoted by `/`. This is fundamentally different from Windows, where each drive has its own root (C:\, D:\, etc.). In Linux, even other drives are mounted as directories under `/`.

### Complete Directory Structure Explained

```
/ (root)
│
├── /bin (Essential User Binaries)
│   │
│   │   Contains essential command-line programs needed for the system
│   │   to boot and run in single-user mode. These are available to all users.
│   │
│   │   Examples:
│   │   - ls      : List directory contents
│   │   - cp      : Copy files
│   │   - mv      : Move/rename files
│   │   - rm      : Remove files
│   │   - cat     : Concatenate and display files
│   │   - echo    : Display text
│   │   - bash    : The Bourne Again Shell
│   │   - chmod   : Change file permissions
│   │   - chown   : Change file ownership
│   │   - mkdir   : Make directories
│   │   - rmdir   : Remove empty directories
│   │   - ln      : Create links
│   │   - pwd     : Print working directory
│   │   - df      : Disk free space
│   │   - mount   : Mount file systems
│   │   - umount  : Unmount file systems
│   │   - ps      : Process status
│   │   - kill    : Send signals to processes
│   │   - grep    : Search text patterns
│   │   - sed     : Stream editor
│   │   - tar     : Archive utility
│   │   - gzip    : Compression utility
│   │
│   │   NOTE: On modern systems, /bin is often a symbolic link to /usr/bin
│   │
│
├── /sbin (System Binaries)
│   │
│   │   Contains essential system administration programs. These are
│   │   typically only used by the root user or system administrators.
│   │
│   │   Examples:
│   │   - init       : The first process (PID 1)
│   │   - shutdown   : Shut down the system
│   │   - reboot     : Reboot the system
│   │   - fsck       : File system check/repair
│   │   - mkfs       : Make file system
│   │   - fdisk      : Partition table manipulator
│   │   - iptables   : Firewall configuration
│   │   - ip         : Network configuration
│   │   - ifconfig   : Legacy network configuration
│   │   - route      : Routing table management
│   │   - mount      : Mount file systems (also in /bin)
│   │   - lvm        : Logical Volume Manager
│   │   - mdadm      : RAID management
│   │
│   │   NOTE: On modern systems, /sbin is often a symbolic link to /usr/sbin
│   │
│
├── /etc (Configuration Files)
│   │
│   │   The brain of your Linux system. Contains all system-wide
│   │   configuration files. Application and service configurations
│   │   are stored here.
│   │
│   │   CRITICAL FILES - You must know these:
│   │
│   ├── /etc/passwd
│   │   │   User account information database
│   │   │   Format: username:x:UID:GID:comment:home:shell
│   │   │   
│   │   │   Example entry:
│   │   │   john:x:1001:1001:John Doe:/home/john:/bin/bash
│   │   │   
│   │   │   Fields explained:
│   │   │   - john     : Username
│   │   │   - x        : Password placeholder (actual password in /etc/shadow)
│   │   │   - 1001     : User ID (UID) - unique numeric identifier
│   │   │   - 1001     : Primary Group ID (GID)
│   │   │   - John Doe : GECOS field (comment, usually full name)
│   │   │   - /home/john : User's home directory
│   │   │   - /bin/bash  : User's login shell
│   │   │
│   │   │   Special users:
│   │   │   - root (UID 0)     : Superuser, full system access
│   │   │   - nobody (UID 65534): Unprivileged user for services
│   │   │   - System users (UID 1-999): For services/daemons
│   │
│   ├── /etc/shadow
│   │   │   Secure password storage (only readable by root)
│   │   │   Format: username:password_hash:lastchange:min:max:warn:inactive:expire
│   │   │
│   │   │   Example:
│   │   │   john:$6$xyz...abc:19000:0:99999:7:::
│   │   │
│   │   │   Password hash prefixes:
│   │   │   - $1$ : MD5 (weak, don't use)
│   │   │   - $5$ : SHA-256
│   │   │   - $6$ : SHA-512 (recommended)
│   │   │   - $y$ : yescrypt (modern, strongest)
│   │   │   - !!  : Account locked, no password set
│   │   │   - *   : Account disabled
│   │
│   ├── /etc/group
│   │   │   Group definitions
│   │   │   Format: groupname:x:GID:member1,member2,member3
│   │   │
│   │   │   Example:
│   │   │   developers:x:1002:john,jane,bob
│   │
│   ├── /etc/hosts
│   │   │   Static hostname to IP address mapping
│   │   │   Checked BEFORE DNS queries (by default)
│   │   │
│   │   │   Example:
│   │   │   127.0.0.1       localhost
│   │   │   192.168.1.100   webserver.local webserver
│   │   │   10.0.0.50       database.internal db
│   │
│   ├── /etc/hostname
│   │   │   Contains the system's hostname (single line)
│   │   │   Example content: myserver
│   │
│   ├── /etc/resolv.conf
│   │   │   DNS resolver configuration
│   │   │
│   │   │   Example:
│   │   │   nameserver 8.8.8.8
│   │   │   nameserver 8.8.4.4
│   │   │   search example.com internal.example.com
│   │   │
│   │   │   'search' directive: domains to append for short hostnames
│   │   │   If you type 'ping server', it tries:
│   │   │   - server
│   │   │   - server.example.com
│   │   │   - server.internal.example.com
│   │
│   ├── /etc/nsswitch.conf
│   │   │   Name Service Switch - defines lookup order
│   │   │
│   │   │   Example:
│   │   │   passwd:     files systemd
│   │   │   group:      files systemd
│   │   │   hosts:      files dns myhostname
│   │   │   
│   │   │   'hosts: files dns' means:
│   │   │   1. Check /etc/hosts first
│   │   │   2. Then query DNS
│   │
│   ├── /etc/fstab
│   │   │   File System Table - defines what to mount at boot
│   │   │
│   │   │   Format: device  mountpoint  fstype  options  dump  pass
│   │   │
│   │   │   Example:
│   │   │   UUID=abc-123  /           ext4    defaults        0  1
│   │   │   UUID=def-456  /home       ext4    defaults        0  2
│   │   │   UUID=ghi-789  none        swap    sw              0  0
│   │   │   /dev/sdb1     /data       xfs     defaults,noatime 0  2
│   │   │
│   │   │   Options explained:
│   │   │   - defaults : rw,suid,dev,exec,auto,nouser,async
│   │   │   - noatime  : Don't update access time (performance)
│   │   │   - ro       : Read-only
│   │   │   - noexec   : Can't execute binaries (security)
│   │   │   - nosuid   : Ignore SUID bits (security)
│   │   │
│   │   │   Pass field (last column):
│   │   │   - 0 : Don't check with fsck
│   │   │   - 1 : Check first (root filesystem)
│   │   │   - 2 : Check after root
│   │
│   ├── /etc/ssh/
│   │   ├── sshd_config    : SSH server configuration
│   │   ├── ssh_config     : SSH client system-wide defaults
│   │   ├── ssh_host_*     : Server's host keys
│   │   │
│   │   │   Important sshd_config settings:
│   │   │   Port 22                      # SSH port
│   │   │   PermitRootLogin no           # Disable root SSH login
│   │   │   PasswordAuthentication no    # Require key-based auth
│   │   │   PubkeyAuthentication yes     # Allow public key auth
│   │   │   AllowUsers admin deploy      # Only these users can SSH
│   │   │   MaxAuthTries 3               # Max login attempts
│   │   │   ClientAliveInterval 300      # Keep-alive every 5 min
│   │   │   ClientAliveCountMax 2        # Disconnect after 2 missed
│   │
│   ├── /etc/sudoers (and /etc/sudoers.d/)
│   │   │   Sudo (superuser do) configuration
│   │   │   ALWAYS edit with 'visudo' command (validates syntax)
│   │   │
│   │   │   Format: who  where=(as_whom)  what
│   │   │
│   │   │   Examples:
│   │   │   root    ALL=(ALL:ALL) ALL
│   │   │   # root can run any command as anyone from anywhere
│   │   │
│   │   │   %admin  ALL=(ALL) ALL
│   │   │   # Members of 'admin' group can run any command
│   │   │
│   │   │   john    ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx
│   │   │   # john can restart nginx without password
│   │   │
│   │   │   deploy  ALL=(ALL) NOPASSWD: ALL
│   │   │   # deploy user can run anything without password
│   │   │   # (common for automation, but use carefully)
│   │
│   ├── /etc/systemd/
│   │   │   Systemd configuration directory
│   │   ├── system/           : System unit files
│   │   ├── user/             : User unit files
│   │   ├── journald.conf     : Journal (logging) config
│   │   └── logind.conf       : Login manager config
│   │
│   ├── /etc/security/limits.conf
│   │   │   Resource limits for users/groups
│   │   │
│   │   │   Format: domain  type  item  value
│   │   │
│   │   │   Examples:
│   │   │   *       soft    nofile    65535
│   │   │   *       hard    nofile    65535
│   │   │   # All users can open up to 65535 files
│   │   │
│   │   │   @docker soft    nproc     unlimited
│   │   │   # docker group has unlimited processes
│   │   │
│   │   │   Types:
│   │   │   - soft : Can be changed by user up to hard limit
│   │   │   - hard : Maximum limit, only root can increase
│   │   │
│   │   │   Common items:
│   │   │   - nofile : Maximum open file descriptors
│   │   │   - nproc  : Maximum number of processes
│   │   │   - memlock: Maximum locked memory
│   │   │   - as     : Maximum address space (memory)
│   │
│   ├── /etc/sysctl.conf (and /etc/sysctl.d/)
│   │   │   Kernel parameter configuration
│   │   │
│   │   │   Examples:
│   │   │   vm.swappiness = 10
│   │   │   # How aggressively to swap (0-100, lower = less swapping)
│   │   │
│   │   │   net.ipv4.ip_forward = 1
│   │   │   # Enable IP forwarding (routing between interfaces)
│   │   │
│   │   │   net.core.somaxconn = 65535
│   │   │   # Maximum socket connection backlog
│   │   │
│   │   │   fs.file-max = 2097152
│   │   │   # Maximum file handles system-wide
│   │
│   ├── /etc/crontab
│   │   │   System-wide cron job schedule
│   │
│   ├── /etc/environment
│   │   │   System-wide environment variables
│   │   │   Format: VARIABLE="value"
│   │
│   └── /etc/profile (and /etc/profile.d/)
│       │   System-wide shell profile (executed on login)
│
├── /home (User Home Directories)
│   │
│   │   Each regular user has a directory here: /home/username
│   │   Contains user's personal files, configurations, and data.
│   │
│   │   Common files in a user's home:
│   │   ~/.bashrc        : Bash configuration (runs on each shell)
│   │   ~/.bash_profile  : Login shell configuration
│   │   ~/.profile       : Login shell configuration (sh-compatible)
│   │   ~/.ssh/          : SSH keys and known hosts
│   │   ~/.config/       : Application configurations (XDG standard)
│   │   ~/.local/        : User-local installed software
│   │
│   │   SSH directory (~/.ssh/):
│   │   ~/.ssh/id_rsa           : Private key (RSA)
│   │   ~/.ssh/id_rsa.pub       : Public key (RSA)
│   │   ~/.ssh/id_ed25519       : Private key (Ed25519, recommended)
│   │   ~/.ssh/id_ed25519.pub   : Public key (Ed25519)
│   │   ~/.ssh/authorized_keys  : Public keys allowed to log in
│   │   ~/.ssh/known_hosts      : Fingerprints of known servers
│   │   ~/.ssh/config           : SSH client configuration
│
├── /root (Root User's Home)
│   │
│   │   Home directory for the root (superuser) account.
│   │   Located at /root, not /home/root.
│   │   Same structure as regular user homes.
│
├── /var (Variable Data)
│   │
│   │   Files that change frequently during system operation.
│   │   Logs, caches, spool files, runtime data.
│   │
│   ├── /var/log/
│   │   │   System and application log files
│   │   │
│   │   │   CRITICAL LOG FILES:
│   │   │
│   │   ├── syslog (or messages)
│   │   │   │   General system messages
│   │   │   │   Kernel messages, service status, errors
│   │   │   │   Ubuntu/Debian: /var/log/syslog
│   │   │   │   RHEL/CentOS: /var/log/messages
│   │   │
│   │   ├── auth.log (or secure)
│   │   │   │   Authentication logs
│   │   │   │   SSH logins, sudo usage, PAM events
│   │   │   │   Ubuntu/Debian: /var/log/auth.log
│   │   │   │   RHEL/CentOS: /var/log/secure
│   │   │
│   │   ├── kern.log
│   │   │   │   Kernel messages
│   │   │   │   Hardware issues, driver messages
│   │   │
│   │   ├── dmesg
│   │   │   │   Kernel ring buffer (boot messages)
│   │   │   │   Hardware detection, driver loading
│   │   │
│   │   ├── cron
│   │   │   │   Cron job execution logs
│   │   │
│   │   ├── apt/ or yum.log
│   │   │   │   Package manager logs
│   │   │
│   │   ├── nginx/, apache2/, httpd/
│   │   │   │   Web server logs
│   │   │   │   access.log : Request logs
│   │   │   │   error.log  : Error logs
│   │   │
│   │   └── journal/
│   │       │   Systemd journal (binary format)
│   │       │   Access with 'journalctl' command
│   │
│   ├── /var/lib/
│   │   │   Application state data
│   │   │   Databases, package manager state, etc.
│   │   │
│   │   ├── docker/    : Docker images, containers, volumes
│   │   ├── mysql/     : MySQL databases
│   │   ├── postgresql/: PostgreSQL databases
│   │   ├── dpkg/      : dpkg package database
│   │   └── rpm/       : RPM package database
│   │
│   ├── /var/run/ -> /run
│   │   │   Runtime data (PIDs, sockets)
│   │   │   Cleared on reboot
│   │   │   Now typically a symlink to /run
│   │
│   ├── /var/cache/
│   │   │   Application cache data
│   │   ├── apt/       : Downloaded .deb packages
│   │   └── yum/       : Downloaded .rpm packages
│   │
│   ├── /var/spool/
│   │   │   Spool directories for queued jobs
│   │   ├── cron/      : User crontabs
│   │   ├── mail/      : Mail queue
│   │   └── cups/      : Print jobs
│   │
│   └── /var/tmp/
│       │   Temporary files (preserved across reboots)
│       │   Unlike /tmp, not cleared on reboot
│
├── /tmp (Temporary Files)
│   │
│   │   Temporary files, CLEARED ON REBOOT.
│   │   World-writable with sticky bit (only owner can delete their files).
│   │   Permissions: 1777 (drwxrwxrwt)
│   │   
│   │   The sticky bit (t) means:
│   │   - Anyone can create files
│   │   - Only the file owner (or root) can delete them
│   │   - Prevents users from deleting each other's temp files
│
├── /usr (User Programs - Secondary Hierarchy)
│   │
│   │   Second major hierarchy containing user programs.
│   │   Shareable, read-only data.
│   │   
│   │   On modern systems, /bin, /sbin, /lib are symlinks to /usr/
│   │
│   ├── /usr/bin/
│   │   │   User binaries (non-essential)
│   │   │   Most command-line programs
│   │
│   ├── /usr/sbin/
│   │   │   System administration binaries (non-essential)
│   │
│   ├── /usr/lib/
│   │   │   Libraries for /usr/bin and /usr/sbin
│   │
│   ├── /usr/lib64/
│   │   │   64-bit libraries (on 64-bit systems)
│   │
│   ├── /usr/include/
│   │   │   C/C++ header files for compiling software
│   │
│   ├── /usr/share/
│   │   │   Architecture-independent data
│   │   ├── man/       : Manual pages
│   │   ├── doc/       : Documentation
│   │   ├── zoneinfo/  : Timezone data
│   │   └── applications/ : Desktop entries
│   │
│   ├── /usr/local/
│   │   │   Locally installed software (not from package manager)
│   │   │   Has its own bin/, sbin/, lib/, share/ hierarchy
│   │   │   
│   │   │   When you compile from source:
│   │   │   ./configure --prefix=/usr/local
│   │   │   make && make install
│   │   │   
│   │   │   Binaries go to /usr/local/bin, libs to /usr/local/lib, etc.
│   │
│   └── /usr/src/
│       │   Source code (kernel headers, etc.)
│
├── /opt (Optional Software)
│   │
│   │   Third-party, self-contained applications.
│   │   Each application in its own subdirectory.
│   │
│   │   Examples:
│   │   /opt/google/chrome/
│   │   /opt/microsoft/azurecli/
│   │   /opt/containerd/
│   │
│   │   Convention: /opt/<vendor>/<application>
│
├── /proc (Process Information - Virtual Filesystem)
│   │
│   │   VIRTUAL filesystem - doesn't exist on disk.
│   │   Provides interface to kernel data structures.
│   │   Created dynamically by the kernel.
│   │
│   │   /proc/<PID>/       : Directory for each running process
│   │   ├── cmdline        : Command line that started the process
│   │   ├── environ        : Environment variables
│   │   ├── fd/            : Open file descriptors
│   │   ├── maps           : Memory mappings
│   │   ├── status         : Process status (memory, state, etc.)
│   │   ├── stat           : Process statistics (raw format)
│   │   ├── limits         : Resource limits
│   │   ├── cwd -> /path   : Symlink to current working directory
│   │   └── exe -> /path   : Symlink to executable
│   │
│   │   System information:
│   │   /proc/cpuinfo      : CPU information
│   │   /proc/meminfo      : Memory information
│   │   /proc/version      : Kernel version
│   │   /proc/uptime       : System uptime
│   │   /proc/loadavg      : Load averages
│   │   /proc/filesystems  : Supported filesystems
│   │   /proc/mounts       : Currently mounted filesystems
│   │   /proc/partitions   : Partition information
│   │   /proc/net/         : Network statistics
│   │   /proc/sys/         : Kernel parameters (sysctl)
│   │
│   │   Examples:
│   │   cat /proc/cpuinfo        # Show CPU details
│   │   cat /proc/meminfo        # Show memory details
│   │   cat /proc/1/cmdline      # Command for PID 1 (init/systemd)
│   │   cat /proc/self/status    # Status of current process
│
├── /sys (System Information - Virtual Filesystem)
│   │
│   │   VIRTUAL filesystem for kernel objects.
│   │   Provides interface to devices, drivers, and kernel features.
│   │   More structured than /proc.
│   │
│   │   /sys/class/        : Device classes
│   │   ├── net/           : Network interfaces
│   │   ├── block/         : Block devices
│   │   ├── tty/           : Terminal devices
│   │   └── power_supply/  : Battery information
│   │
│   │   /sys/devices/      : All devices in hierarchy
│   │   /sys/block/        : Block devices
│   │   /sys/fs/           : Filesystem information
│   │   ├── cgroup/        : Control groups
│   │   └── selinux/       : SELinux info
│   │
│   │   Examples:
│   │   cat /sys/class/net/eth0/address    # MAC address
│   │   cat /sys/class/net/eth0/speed      # Link speed
│   │   cat /sys/block/sda/size            # Disk size (sectors)
│
├── /dev (Device Files)
│   │
│   │   Device files - special files representing hardware.
│   │   Created by udev dynamically.
│   │
│   │   Types of device files:
│   │   - Block devices (b): Random access, fixed size blocks (disks)
│   │   - Character devices (c): Sequential access, byte streams
│   │
│   │   Common devices:
│   │   /dev/null          : Black hole - discards all input
│   │   /dev/zero          : Produces infinite null bytes
│   │   /dev/random        : Random number generator (blocking)
│   │   /dev/urandom       : Random number generator (non-blocking)
│   │   /dev/tty           : Current terminal
│   │   /dev/console       : System console
│   │   /dev/stdin         : Standard input (fd 0)
│   │   /dev/stdout        : Standard output (fd 1)
│   │   /dev/stderr        : Standard error (fd 2)
│   │
│   │   Block devices (storage):
│   │   /dev/sda           : First SCSI/SATA disk
│   │   /dev/sda1          : First partition on sda
│   │   /dev/sda2          : Second partition on sda
│   │   /dev/sdb           : Second disk
│   │   /dev/nvme0n1       : First NVMe disk
│   │   /dev/nvme0n1p1     : First partition on NVMe disk
│   │   /dev/vda           : First VirtIO disk (virtual machines)
│   │   /dev/xvda          : First Xen virtual disk (AWS)
│   │
│   │   Device mapper (LVM, RAID, encryption):
│   │   /dev/mapper/       : Device mapper devices
│   │   /dev/dm-0          : First device mapper device
│   │
│   │   Pseudo-terminals:
│   │   /dev/pts/          : Pseudo-terminal slaves (SSH sessions)
│
├── /lib (Essential Libraries) -> /usr/lib
│   │
│   │   Essential shared libraries for /bin and /sbin.
│   │   On modern systems, symlink to /usr/lib.
│   │
│   │   /lib/modules/      : Kernel modules
│   │   /lib/firmware/     : Device firmware
│
├── /lib64 (64-bit Libraries) -> /usr/lib64
│   │
│   │   64-bit libraries. Symlink to /usr/lib64 on modern systems.
│
├── /mnt (Temporary Mount Point)
│   │
│   │   Traditional location for temporarily mounting filesystems.
│   │   Used for manual mounts by administrators.
│   │
│   │   Example:
│   │   mount /dev/sdb1 /mnt
│   │   mount -t nfs server:/share /mnt
│
├── /media (Removable Media)
│   │
│   │   Mount point for removable media (USB, CD, etc.)
│   │   Usually auto-mounted by desktop environments.
│   │   
│   │   /media/<username>/USB_DRIVE
│   │   /media/<username>/CDROM
│
├── /srv (Service Data)
│   │
│   │   Data for services provided by the system.
│   │   Not commonly used; convention varies.
│   │   
│   │   Examples:
│   │   /srv/www/          : Web server files
│   │   /srv/ftp/          : FTP server files
│
├── /boot (Boot Files)
│   │
│   │   Files needed to boot the system.
│   │   Often a separate partition (especially with UEFI).
│   │
│   │   Contents:
│   │   vmlinuz-*          : Compressed Linux kernel
│   │   initrd.img-* or initramfs-* : Initial RAM disk
│   │   grub/              : GRUB bootloader files
│   │   efi/               : EFI boot files (if UEFI system)
│   │   config-*           : Kernel configuration
│   │   System.map-*       : Kernel symbol table
│
└── /run (Runtime Data)
    │
    │   Temporary runtime data since boot.
    │   CLEARED ON REBOOT.
    │   Stored in memory (tmpfs).
    │
    │   /run/lock/         : Lock files
    │   /run/user/<UID>/   : User runtime directory
    │   /run/systemd/      : Systemd runtime data
    │   /run/docker/       : Docker runtime data
    │   /run/containerd/   : Containerd runtime data
    │
    │   Example files:
    │   /run/sshd.pid      : SSH daemon PID
    │   /run/nginx.pid     : Nginx PID
```

---

## 1.3 Understanding Shells and the Command Line

### What is a Shell?

A shell is a command-line interpreter that provides a user interface to the operating system. When you type commands, the shell interprets them and executes the corresponding programs.

```
┌─────────────────────────────────────────────────────────────┐
│                        You (User)                            │
│                           │                                  │
│                           ▼                                  │
│                    ┌─────────────┐                           │
│                    │   Terminal  │  (Terminal Emulator)      │
│                    │  Emulator   │  (xterm, gnome-terminal,  │
│                    └──────┬──────┘   Windows Terminal, etc.) │
│                           │                                  │
│                           ▼                                  │
│                    ┌─────────────┐                           │
│                    │    Shell    │  (bash, zsh, sh, fish)    │
│                    │             │                           │
│                    └──────┬──────┘                           │
│                           │                                  │
│                           ▼                                  │
│                    ┌─────────────┐                           │
│                    │   Kernel    │                           │
│                    │             │                           │
│                    └─────────────┘                           │
└─────────────────────────────────────────────────────────────┘
```

### Common Shells

| Shell | Description |
|-------|-------------|
| **bash** | Bourne Again Shell - Default on most Linux distros. Most widely used. |
| **sh** | Original Bourne Shell - POSIX compliant, minimal features. |
| **zsh** | Z Shell - Extended bash with powerful features. Default on macOS. |
| **fish** | Friendly Interactive Shell - User-friendly, auto-suggestions. |
| **dash** | Debian Almquist Shell - Fast, minimal, used for scripts. |
| **ksh** | Korn Shell - Used in some enterprise environments. |

**For DevOps**, focus on **bash** - it's the standard everywhere.

### Shell Configuration Files

When you log in or open a terminal, the shell reads configuration files in a specific order:

**For bash:**

```
Login Shell (ssh, console login):
1. /etc/profile           (system-wide)
2. /etc/profile.d/*.sh    (system-wide scripts)
3. ~/.bash_profile        (user, if exists)
   OR ~/.bash_login       (user, if .bash_profile doesn't exist)
   OR ~/.profile          (user, if neither above exist)
4. ~/.bashrc              (usually sourced by .bash_profile)

Non-Login Shell (new terminal window):
1. ~/.bashrc              (user)

Logout:
1. ~/.bash_logout         (user)
```

**Understanding the difference:**

- **Login shell**: Started when you log into the system (SSH, console, `su -`)
- **Non-login shell**: Started when you open a new terminal in a GUI or run `bash` command

**Typical ~/.bashrc structure:**

```bash
# ~/.bashrc

# If not running interactively, don't do anything
case $- in
    *i*) ;;
      *) return;;
esac

# History settings
HISTCONTROL=ignoreboth          # Ignore duplicates and commands starting with space
HISTSIZE=10000                  # Lines in memory
HISTFILESIZE=20000              # Lines in file
shopt -s histappend             # Append to history, don't overwrite

# Check window size after each command
shopt -s checkwinsize

# Make 'less' more friendly for non-text input files
[ -x /usr/bin/lesspipe ] && eval "$(SHELL=/bin/sh lesspipe)"

# Prompt configuration
PS1='\u@\h:\w\$ '               # user@host:directory$

# Color support
alias ls='ls --color=auto'
alias grep='grep --color=auto'

# Common aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
alias ..='cd ..'
alias ...='cd ../..'

# Custom PATH
export PATH="$HOME/bin:$HOME/.local/bin:$PATH"

# Custom environment variables
export EDITOR=vim
export VISUAL=vim

# Load additional configurations
if [ -d ~/.bashrc.d ]; then
    for file in ~/.bashrc.d/*.sh; do
        [ -r "$file" ] && source "$file"
    done
fi
```

### Shell Scripting Fundamentals

Every DevOps engineer must be proficient in shell scripting. Here's a comprehensive overview:

**Script Structure:**

```bash
#!/bin/bash
# ^^^ This is called a "shebang" - tells the system which interpreter to use

# Script description
# Author: Your Name
# Date: 2024-01-01
# Purpose: Describe what this script does

# Exit on error, undefined variables, pipe failures
set -euo pipefail

# Optional: Enable debugging
# set -x

# Variables
NAME="value"                    # No spaces around =
readonly CONSTANT="value"       # Cannot be changed
local LOCAL_VAR="value"         # Only in functions

# Using variables
echo "$NAME"                    # Use quotes to prevent word splitting
echo "${NAME}_suffix"           # Braces for clarity

# Command substitution
CURRENT_DATE=$(date +%Y-%m-%d)
FILE_COUNT=$(ls -1 | wc -l)

# Arithmetic
COUNT=$((1 + 2))
((COUNT++))
((COUNT += 5))

# Arrays
FRUITS=("apple" "banana" "cherry")
echo "${FRUITS[0]}"             # First element
echo "${FRUITS[@]}"             # All elements
echo "${#FRUITS[@]}"            # Array length

# Associative arrays (bash 4+)
declare -A PERSON
PERSON[name]="John"
PERSON[age]="30"
echo "${PERSON[name]}"
```

**Conditionals:**

```bash
# If statement
if [ condition ]; then
    # commands
elif [ condition ]; then
    # commands
else
    # commands
fi

# Test operators (inside [ ] or test command)

# String tests
[ -z "$VAR" ]       # True if VAR is empty
[ -n "$VAR" ]       # True if VAR is not empty
[ "$A" = "$B" ]     # String equality
[ "$A" != "$B" ]    # String inequality

# Numeric tests
[ "$A" -eq "$B" ]   # Equal
[ "$A" -ne "$B" ]   # Not equal
[ "$A" -lt "$B" ]   # Less than
[ "$A" -le "$B" ]   # Less than or equal
[ "$A" -gt "$B" ]   # Greater than
[ "$A" -ge "$B" ]   # Greater than or equal

# File tests
[ -e "$FILE" ]      # Exists
[ -f "$FILE" ]      # Is regular file
[ -d "$PATH" ]      # Is directory
[ -r "$FILE" ]      # Is readable
[ -w "$FILE" ]      # Is writable
[ -x "$FILE" ]      # Is executable
[ -s "$FILE" ]      # Is not empty (size > 0)
[ -L "$FILE" ]      # Is symbolic link

# Logical operators
[ condition1 ] && [ condition2 ]    # AND
[ condition1 ] || [ condition2 ]    # OR
[ ! condition ]                     # NOT

# Modern bash: [[ ]] - preferred over [ ]
# Supports pattern matching, no word splitting issues
[[ "$VAR" == pattern* ]]    # Pattern matching
[[ "$VAR" =~ regex ]]       # Regex matching
[[ -z "$VAR" && -n "$OTHER" ]]  # Multiple conditions

# Example
if [[ -f "/etc/passwd" ]]; then
    echo "Password file exists"
fi

if [[ "$STATUS" == "running" && "$HEALTH" == "healthy" ]]; then
    echo "Service is operational"
fi
```

**Loops:**

```bash
# For loop - iterate over list
for item in apple banana cherry; do
    echo "$item"
done

# For loop - iterate over array
FRUITS=("apple" "banana" "cherry")
for fruit in "${FRUITS[@]}"; do
    echo "$fruit"
done

# For loop - C-style
for ((i=0; i<10; i++)); do
    echo "$i"
done

# For loop - iterate over files
for file in /var/log/*.log; do
    echo "Processing: $file"
done

# For loop - iterate over command output
for user in $(cut -d: -f1 /etc/passwd); do
    echo "User: $user"
done

# While loop
COUNT=0
while [ $COUNT -lt 10 ]; do
    echo "$COUNT"
    ((COUNT++))
done

# While loop - read lines from file
while IFS= read -r line; do
    echo "Line: $line"
done < /etc/passwd

# While loop - infinite with break
while true; do
    if [ condition ]; then
        break
    fi
    sleep 1
done

# Until loop (opposite of while)
COUNT=0
until [ $COUNT -ge 10 ]; do
    echo "$COUNT"
    ((COUNT++))
done
```

**Functions:**

```bash
# Function definition (two equivalent syntaxes)
function my_function {
    # commands
}

my_function() {
    # commands
}

# Function with arguments
greet() {
    local name="$1"          # First argument
    local greeting="${2:-Hello}"  # Second argument with default
    echo "$greeting, $name!"
}

greet "John"           # Output: Hello, John!
greet "John" "Hi"      # Output: Hi, John!

# Function with return value
# (return is for exit status 0-255, use echo for data)
is_valid() {
    if [[ "$1" =~ ^[0-9]+$ ]]; then
        return 0   # Success (true)
    else
        return 1   # Failure (false)
    fi
}

if is_valid "123"; then
    echo "Valid number"
fi

# Function returning data
get_timestamp() {
    echo "$(date +%Y%m%d_%H%M%S)"
}

TIMESTAMP=$(get_timestamp)
echo "$TIMESTAMP"
```

**Input/Output:**

```bash
# Reading input
read -p "Enter your name: " NAME
read -sp "Enter password: " PASSWORD  # Silent (no echo)
read -t 10 -p "Quick! " ANSWER        # 10 second timeout

# Output
echo "Hello World"              # With newline
echo -n "No newline"           # Without newline
echo -e "Tab:\there"           # Enable escape sequences
printf "Formatted: %s %d\n" "text" 42   # C-style printf

# Redirection
command > file          # Redirect stdout to file (overwrite)
command >> file         # Redirect stdout to file (append)
command 2> file         # Redirect stderr to file
command 2>&1            # Redirect stderr to stdout
command &> file         # Redirect both stdout and stderr (bash)
command > /dev/null     # Discard stdout
command 2>/dev/null     # Discard stderr
command &>/dev/null     # Discard all output

# Here documents (multi-line input)
cat << EOF
This is line 1
This is line 2
Variable: $NAME
EOF

# Here document without variable expansion
cat << 'EOF'
This $VAR will NOT be expanded
EOF

# Here strings
grep "pattern" <<< "$STRING"
```

**Error Handling:**

```bash
#!/bin/bash
set -euo pipefail

# set -e  : Exit on any error
# set -u  : Error on undefined variable
# set -o pipefail : Fail pipe if any command fails

# Trap for cleanup
cleanup() {
    echo "Cleaning up..."
    rm -f "$TEMP_FILE"
}
trap cleanup EXIT        # Run on exit
trap cleanup ERR         # Run on error
trap cleanup INT TERM    # Run on interrupt

# Custom error handling
die() {
    echo "ERROR: $1" >&2
    exit "${2:-1}"
}

# Usage
[[ -f "$CONFIG_FILE" ]] || die "Config file not found: $CONFIG_FILE" 2

# Try-catch pattern
if ! output=$(some_command 2>&1); then
    echo "Command failed: $output"
    exit 1
fi
```

**Best Practices:**

```bash
#!/bin/bash
#
# Script Name: deploy.sh
# Description: Deploy application to server
# Author: Your Name
# Version: 1.0.0
#

set -euo pipefail

# Constants
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_NAME="$(basename "$0")"

# Default values
ENVIRONMENT="${ENVIRONMENT:-development}"
VERBOSE="${VERBOSE:-false}"

# Logging functions
log_info() {
    echo "[INFO] $(date '+%Y-%m-%d %H:%M:%S') - $*"
}

log_error() {
    echo "[ERROR] $(date '+%Y-%m-%d %H:%M:%S') - $*" >&2
}

log_debug() {
    if [[ "$VERBOSE" == "true" ]]; then
        echo "[DEBUG] $(date '+%Y-%m-%d %H:%M:%S') - $*"
    fi
}

# Usage function
usage() {
    cat << EOF
Usage: $SCRIPT_NAME [OPTIONS]

Options:
    -e, --environment ENV   Environment (default: development)
    -v, --verbose           Enable verbose output
    -h, --help             Show this help message

Examples:
    $SCRIPT_NAME -e production
    $SCRIPT_NAME --environment staging --verbose
EOF
}

# Parse arguments
while [[ $# -gt 0 ]]; do
    case "$1" in
        -e|--environment)
            ENVIRONMENT="$2"
            shift 2
            ;;
        -v|--verbose)
            VERBOSE="true"
            shift
            ;;
        -h|--help)
            usage
            exit 0
            ;;
        *)
            log_error "Unknown option: $1"
            usage
            exit 1
            ;;
    esac
done

# Main function
main() {
    log_info "Starting deployment to $ENVIRONMENT"
    
    # Your deployment logic here
    
    log_info "Deployment complete"
}

# Run main function
main "$@"
```

---

# Chapter 2: Essential Linux Commands - Complete Reference

## 2.1 File and Directory Operations

### Navigating the Filesystem

```bash
# pwd - Print Working Directory
# Shows your current location in the filesystem
pwd
# Output: /home/john/projects

# cd - Change Directory
cd /path/to/directory      # Absolute path
cd relative/path           # Relative path
cd                         # Go to home directory
cd ~                       # Go to home directory
cd -                       # Go to previous directory
cd ..                      # Go up one level
cd ../..                   # Go up two levels
cd ~username               # Go to another user's home

# ls - List Directory Contents
ls                         # List current directory
ls /path                   # List specific directory
ls -l                      # Long format (permissions, size, date)
ls -la                     # Long format including hidden files
ls -lh                     # Human-readable sizes (KB, MB, GB)
ls -lt                     # Sort by modification time (newest first)
ls -ltr                    # Sort by time, reversed (oldest first)
ls -lS                     # Sort by size (largest first)
ls -R                      # Recursive (list subdirectories)
ls -d */                   # List only directories
ls -1                      # One file per line
ls --color=auto            # Colorized output

# Understanding ls -l output:
# drwxr-xr-x 2 john users 4096 Jan 15 10:30 documents
# │└┬┘└┬┘└┬┘ │  │     │     │        │        └─ Name
# │ │  │  │  │  │     │     │        └─ Modification time
# │ │  │  │  │  │     │     └─ Size in bytes
# │ │  │  │  │  │     └─ Group
# │ │  │  │  │  └─ Owner
# │ │  │  │  └─ Link count
# │ │  │  └─ Other permissions (r=read, w=write, x=execute)
# │ │  └─ Group permissions
# │ └─ Owner permissions
# └─ File type (d=directory, -=file, l=link, etc.)
```

### Creating and Removing Files/Directories

```bash
# mkdir - Make Directory
mkdir directory                    # Create single directory
mkdir -p path/to/nested/dir       # Create nested directories
mkdir -m 755 directory            # Create with specific permissions
mkdir dir1 dir2 dir3              # Create multiple directories

# rmdir - Remove Empty Directory
rmdir directory                   # Remove empty directory only

# rm - Remove Files and Directories
rm file                           # Remove file
rm -f file                        # Force remove (no confirmation)
rm -i file                        # Interactive (ask before each)
rm -r directory                   # Remove directory recursively
rm -rf directory                  # Force remove directory (DANGEROUS!)
rm -v file                        # Verbose (show what's being removed)

# CAUTION: rm -rf with wrong path can be devastating
# NEVER run: rm -rf /   or   rm -rf /*
# Always double-check your path before using rm -rf

# touch - Create Empty File or Update Timestamp
touch newfile                     # Create empty file
touch file1 file2 file3          # Create multiple files
touch -t 202401151030 file       # Set specific timestamp (YYYYMMDDhhmm)
touch -r reference target        # Copy timestamp from reference

# file - Determine File Type
file document.pdf                # Output: PDF document, version 1.4
file script.sh                   # Output: Bourne-Again shell script
file /bin/ls                     # Output: ELF 64-bit LSB executable
```

### Copying, Moving, and Linking

```bash
# cp - Copy Files and Directories
cp source destination            # Copy file
cp file1 file2 directory/       # Copy multiple files to directory
cp -r source_dir dest_dir       # Copy directory recursively
cp -a source dest               # Archive mode (preserves everything)
cp -p source dest               # Preserve permissions and timestamps
cp -i source dest               # Interactive (ask before overwrite)
cp -n source dest               # No clobber (don't overwrite)
cp -u source dest               # Update (only if source is newer)
cp -v source dest               # Verbose

# The -a flag is equivalent to -dR --preserve=all
# Preserves: mode, ownership, timestamps, links, xattr

# mv - Move or Rename Files
mv oldname newname              # Rename file
mv file directory/              # Move file to directory
mv file1 file2 directory/      # Move multiple files
mv -i source dest              # Interactive
mv -n source dest              # No clobber
mv -u source dest              # Update (only if source is newer)

# ln - Create Links
# Hard link: Another name for the same file (same inode)
ln file hardlink

# Symbolic (soft) link: Pointer to another file (different inode)
ln -s /path/to/target linkname
ln -s ../relative/path linkname

# Understanding links:
# Hard links:
#   - Same inode number as original
#   - Cannot span filesystems
#   - Cannot link to directories
#   - File deleted only when all hard links removed
#
# Symbolic links:
#   - Different inode, contains path to target
#   - Can span filesystems
#   - Can link to directories
#   - Becomes broken if target is deleted

# Check if file is a link
ls -l linkname                  # Shows: lrwxrwxrwx ... linkname -> target
readlink linkname              # Shows target path
readlink -f linkname           # Shows absolute target path
```

### Viewing File Contents

```bash
# cat - Concatenate and Display
cat file                        # Display entire file
cat file1 file2                # Concatenate multiple files
cat -n file                    # Number all lines
cat -b file                    # Number non-empty lines
cat -s file                    # Squeeze multiple blank lines
cat -A file                    # Show all (tabs, line endings)

# less - Page Through File (Preferred for large files)
less file
# Navigation in less:
#   Space/f     : Forward one screen
#   b           : Back one screen
#   Enter/j     : Forward one line
#   k           : Back one line
#   g           : Go to beginning
#   G           : Go to end
#   /pattern    : Search forward
#   ?pattern    : Search backward
#   n           : Next search result
#   N           : Previous search result
#   q           : Quit
#   F           : Follow mode (like tail -f)

# more - Page Through File (Older, less features than 'less')
more file

# head - Display First Lines
head file                      # First 10 lines (default)
head -n 20 file               # First 20 lines
head -n -5 file               # All but last 5 lines
head -c 100 file              # First 100 bytes

# tail - Display Last Lines
tail file                      # Last 10 lines (default)
tail -n 20 file               # Last 20 lines
tail -n +5 file               # Starting from line 5
tail -f file                  # Follow (watch for new lines)
tail -F file                  # Follow with retry (handles rotation)
tail -f -n 100 file           # Follow, starting with last 100 lines

# Combining head and tail to get specific lines
head -n 20 file | tail -n 5   # Lines 16-20
sed -n '16,20p' file          # Lines 16-20 (better method)

# wc - Word Count
wc file                       # Lines, words, bytes
wc -l file                    # Lines only
wc -w file                    # Words only
wc -c file                    # Bytes only
wc -m file                    # Characters only

# od - Octal Dump (view binary files)
od -c file                    # Character representation
od -x file                    # Hexadecimal

# xxd - Hex dump
xxd file                      # Hex dump with ASCII
xxd -r hexdump original       # Reverse (hex to binary)

# strings - Extract printable strings from binary
strings /usr/bin/ls           # Show text in binary file
strings -n 10 binary          # Minimum 10 character strings
```

### Finding Files

```bash
# find - Search for files (recursive, powerful)

# Basic syntax: find [path] [conditions] [actions]

# Find by name
find /path -name "filename"           # Exact name
find /path -name "*.log"              # Pattern (case-sensitive)
find /path -iname "*.LOG"             # Pattern (case-insensitive)

# Find by type
find /path -type f                    # Regular files
find /path -type d                    # Directories
find /path -type l                    # Symbolic links
find /path -type b                    # Block devices
find /path -type c                    # Character devices

# Find by size
find /path -size +100M                # Larger than 100 MB
find /path -size -1k                  # Smaller than 1 KB
find /path -size 10M                  # Exactly 10 MB
# Size suffixes: c (bytes), k (KB), M (MB), G (GB)

# Find by time
find /path -mtime -7                  # Modified in last 7 days
find /path -mtime +30                 # Modified more than 30 days ago
find /path -mmin -60                  # Modified in last 60 minutes
find /path -atime -1                  # Accessed in last day
find /path -ctime -1                  # Changed (metadata) in last day
find /path -newer reference_file      # Newer than reference file

# Find by permissions
find /path -perm 644                  # Exactly 644
find /path -perm -644                 # At least 644 (has all these bits)
find /path -perm /644                 # Any of these bits
find /path -perm -u=x                 # Executable by owner

# Find by owner/group
find /path -user john                 # Owned by john
find /path -group developers          # Owned by group
find /path -uid 1000                  # By user ID
find /path -gid 1000                  # By group ID
find /path -nouser                    # No matching user
find /path -nogroup                   # No matching group

# Combining conditions
find /path -name "*.log" -mtime +30   # AND (default)
find /path -name "*.log" -o -name "*.txt"  # OR
find /path ! -name "*.log"            # NOT
find /path \( -name "*.log" -o -name "*.txt" \) -mtime -7  # Grouping

# Actions
find /path -name "*.log" -print       # Print path (default)
find /path -name "*.log" -delete      # Delete found files
find /path -name "*.sh" -exec chmod +x {} \;    # Execute command
find /path -name "*.log" -exec rm {} \;         # Remove each file
find /path -name "*.log" -exec rm {} +          # More efficient (batch)
find /path -name "*.txt" -exec grep "pattern" {} \;

# Limit depth
find /path -maxdepth 1 -name "*.log"  # Current directory only
find /path -mindepth 2 -name "*.log"  # Start from 2 levels deep

# locate - Fast file search (uses database)
locate filename                        # Search database
locate -i filename                     # Case-insensitive
updatedb                              # Update the database (as root)

# which - Find command location
which python                          # /usr/bin/python
which -a python                       # All matches in PATH

# whereis - Find binary, source, manual
whereis python                        # Binaries, source, man pages

# type - How command would be interpreted
type ls                               # ls is aliased to 'ls --color=auto'
type cd                               # cd is a shell builtin
type python                           # python is /usr/bin/python
```

## 2.2 Text Processing

Text processing is a critical skill for DevOps. You'll constantly work with logs, configuration files, and data streams.

### grep - Search Text Patterns

```bash
# Basic grep
grep "pattern" file                   # Search for pattern
grep "pattern" file1 file2            # Search multiple files
grep "pattern" *                      # Search all files in current dir
grep -r "pattern" /path               # Recursive search

# Common options
grep -i "pattern" file                # Case-insensitive
grep -v "pattern" file                # Invert match (lines NOT matching)
grep -n "pattern" file                # Show line numbers
grep -c "pattern" file                # Count matching lines
grep -l "pattern" *                   # List files with matches
grep -L "pattern" *                   # List files without matches
grep -w "word" file                   # Match whole words only
grep -x "exact line" file             # Match exact line

# Context
grep -A 3 "pattern" file              # 3 lines After match
grep -B 3 "pattern" file              # 3 lines Before match
grep -C 3 "pattern" file              # 3 lines Context (before and after)

# Regular expressions
grep "^start" file                    # Lines starting with "start"
grep "end$" file                      # Lines ending with "end"
grep "^$" file                        # Empty lines
grep "a.b" file                       # a, any char, b
grep "a.*b" file                      # a, any chars, b
grep "a\+b" file                      # a (one or more), b
grep "colou\?r" file                  # Optional u (color or colour)
grep "[0-9]" file                     # Any digit
grep "[a-zA-Z]" file                  # Any letter
grep "[^0-9]" file                    # Not a digit

# Extended regex (-E or egrep)
grep -E "pattern1|pattern2" file      # OR
grep -E "(group)+" file               # Grouping and quantifiers
grep -E "[0-9]{1,3}" file             # 1 to 3 digits
grep -E "\b\w+@\w+\.\w+\b" file       # Simple email pattern

# Perl regex (-P) - Most powerful
grep -P "\d{3}-\d{3}-\d{4}" file      # Phone number pattern
grep -P "(?<=@)\w+" file              # Lookbehind (domain from email)

# Practical examples
grep -r "ERROR" /var/log/             # Find errors in logs
grep -i "failed" /var/log/auth.log    # Find failed logins
grep -E "^[^#]" /etc/ssh/sshd_config  # Non-comment lines
grep -v "^#" file | grep -v "^$"      # No comments, no empty lines
ps aux | grep nginx                    # Find nginx processes
cat /etc/passwd | grep -E "^[^:]+:[^:]+:[0-9]{4,}"  # UID >= 1000
```

### sed - Stream Editor

sed is a powerful text transformation tool. It processes text line by line.

```bash
# Basic syntax: sed [options] 'command' file

# Substitution (most common use)
sed 's/old/new/' file                 # Replace first occurrence per line
sed 's/old/new/g' file                # Replace all occurrences (global)
sed 's/old/new/gi' file               # Global, case-insensitive
sed 's/old/new/2' file                # Replace only 2nd occurrence
sed 's/old/new/gp' file               # Print substituted lines
sed -n 's/old/new/gp' file            # Only print substituted lines

# Different delimiters (useful when pattern contains /)
sed 's|/path/old|/path/new|g' file
sed 's#old#new#g' file

# In-place editing
sed -i 's/old/new/g' file             # Edit file in place
sed -i.bak 's/old/new/g' file         # Create backup before editing

# Line selection
sed -n '5p' file                      # Print line 5
sed -n '5,10p' file                   # Print lines 5-10
sed -n '5,$p' file                    # Print from line 5 to end
sed -n '/pattern/p' file              # Print lines matching pattern
sed -n '/start/,/end/p' file          # Print from start to end pattern

# Deletion
sed '5d' file                         # Delete line 5
sed '5,10d' file                      # Delete lines 5-10
sed '/pattern/d' file                 # Delete matching lines
sed '/^$/d' file                      # Delete empty lines
sed '/^#/d' file                      # Delete comment lines

# Insertion and appending
sed '5i\New line before' file         # Insert before line 5
sed '5a\New line after' file          # Append after line 5
sed '/pattern/i\New line' file        # Insert before matching lines
sed '/pattern/a\New line' file        # Append after matching lines

# Multiple commands
sed -e 's/old/new/' -e 's/foo/bar/' file
sed 's/old/new/; s/foo/bar/' file

# Using capture groups
sed 's/\(.*\):\(.*\)/\2:\1/' file     # Swap around colon
sed -E 's/(.*):(.*)/\2:\1/' file      # Same with extended regex

# Practical examples
# Remove leading whitespace
sed 's/^[[:space:]]*//' file

# Remove trailing whitespace
sed 's/[[:space:]]*$//' file

# Remove both leading and trailing whitespace
sed 's/^[[:space:]]*//; s/[[:space:]]*$//' file

# Convert DOS line endings to Unix
sed 's/\r$//' file

# Add line numbers
sed = file | sed 'N;s/\n/\t/'

# Comment out lines containing pattern
sed '/pattern/s/^/#/' file

# Uncomment lines
sed 's/^#//' file

# Extract value from config file
sed -n 's/^key=\(.*\)/\1/p' config
```

### awk - Pattern Scanning and Processing

awk is a programming language for text processing. It's excellent for columnar data.

```bash
# Basic syntax: awk 'pattern { action }' file

# Print columns (space/tab separated by default)
awk '{print $1}' file                 # First column
awk '{print $2}' file                 # Second column
awk '{print $NF}' file                # Last column
awk '{print $(NF-1)}' file            # Second to last column
awk '{print $1, $3}' file             # First and third columns
awk '{print $0}' file                 # Entire line

# Custom field separator
awk -F: '{print $1}' /etc/passwd      # Colon-separated
awk -F, '{print $2}' file.csv         # Comma-separated
awk -F'\t' '{print $1}' file          # Tab-separated

# Output field separator
awk -F: 'BEGIN{OFS=","} {print $1,$3,$4}' /etc/passwd

# Pattern matching
awk '/pattern/' file                  # Print matching lines
awk '!/pattern/' file                 # Print non-matching lines
awk '/pattern/ {print $1}' file       # Print $1 of matching lines
awk '$3 > 100' file                   # Third column > 100
awk '$1 == "value"' file              # First column equals "value"
awk 'NR > 1' file                     # Skip first line (header)
awk 'NR == 5' file                    # Only line 5
awk 'NR >= 5 && NR <= 10' file        # Lines 5-10

# Built-in variables
# NR  : Current line number (across all files)
# NF  : Number of fields in current line
# FS  : Field separator (input)
# OFS : Output field separator
# RS  : Record separator (input, default newline)
# ORS : Output record separator
# FILENAME : Current filename

# Arithmetic
awk '{sum += $1} END {print sum}' file            # Sum of column 1
awk '{sum += $1} END {print sum/NR}' file         # Average
awk 'BEGIN {max=0} $1>max {max=$1} END {print max}' file  # Maximum

# Formatting output
awk '{printf "%-20s %10d\n", $1, $2}' file        # Formatted output
# %s = string, %d = integer, %f = float
# - = left align, number = width

# Conditions
awk '{if ($1 > 100) print $0}' file
awk '{if ($1 > 100) print "big"; else print "small"}' file

# BEGIN and END blocks
awk 'BEGIN {print "Header"} {print} END {print "Footer"}' file
awk 'BEGIN {count=0} /pattern/ {count++} END {print count}' file

# Arrays
awk '{count[$1]++} END {for (i in count) print i, count[i]}' file

# Practical examples

# Sum disk usage by directory
du -s * | awk '{sum += $1} END {print sum/1024 " MB"}'

# Count lines per user in passwd
awk -F: '{shell[$NF]++} END {for (s in shell) print s, shell[s]}' /etc/passwd

# Extract specific field from JSON-like data
awk -F'"' '/"name"/ {print $4}' file

# Process Apache access log
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10

# Calculate percentage
awk '{print $1, $2, ($2/$1)*100 "%"}' file

# Convert CSV to TSV
awk -F, 'BEGIN{OFS="\t"} {$1=$1; print}' file.csv

# Join lines with comma
awk 'NR>1 {printf ","} {printf "%s", $0} END {print ""}' file
```

### Other Text Processing Tools

```bash
# sort - Sort lines
sort file                             # Alphabetical sort
sort -n file                          # Numeric sort
sort -r file                          # Reverse order
sort -k2 file                         # Sort by column 2
sort -k2n file                        # Sort by column 2, numerically
sort -k2,2 -k1,1 file                # Sort by col 2, then col 1
sort -t: -k3n /etc/passwd            # Sort passwd by UID
sort -u file                          # Sort and remove duplicates
sort -h file                          # Human numeric (1K, 2M, 3G)

# uniq - Report or filter repeated lines (requires sorted input)
uniq file                             # Remove adjacent duplicates
uniq -c file                          # Count occurrences
uniq -d file                          # Only show duplicates
uniq -u file                          # Only show unique lines

# Combined: find most common lines
sort file | uniq -c | sort -rn | head -10

# cut - Remove sections from lines
cut -d: -f1 /etc/passwd              # First field, colon-separated
cut -d: -f1,3 /etc/passwd            # Fields 1 and 3
cut -d: -f1-3 /etc/passwd            # Fields 1 through 3
cut -c1-10 file                      # Characters 1-10
cut -c5- file                        # From character 5 to end

# paste - Merge lines of files
paste file1 file2                     # Side by side, tab-separated
paste -d, file1 file2                # Comma-separated
paste -s file                         # All lines into one

# tr - Translate or delete characters
tr 'a-z' 'A-Z' < file                # Lowercase to uppercase
tr 'A-Z' 'a-z' < file                # Uppercase to lowercase
tr -d '\r' < file                    # Delete carriage returns
tr -s ' ' < file                     # Squeeze repeated spaces
tr ':' '\t' < file                   # Colons to tabs
echo "hello" | tr 'a-z' 'n-za-m'     # ROT13

# tee - Read from stdin, write to stdout and files
command | tee file                    # Write to file and stdout
command | tee -a file                 # Append to file
command | tee file1 file2            # Write to multiple files

# split - Split file into pieces
split -l 1000 file prefix_           # 1000 lines per file
split -b 10M file prefix_            # 10 MB per file

# csplit - Split file based on patterns
csplit file '/pattern/' '{*}'        # Split at pattern

# join - Join lines of two files on a common field
join file1 file2                      # Join on first field
join -1 2 -2 1 file1 file2           # Join on field 2 of file1, field 1 of file2

# column - Format into columns
column -t file                        # Table format
cat /etc/passwd | column -t -s:      # Using colon separator

# diff - Compare files
diff file1 file2                      # Show differences
diff -u file1 file2                   # Unified format (preferred)
diff -y file1 file2                   # Side by side
diff -r dir1 dir2                     # Compare directories

# comm - Compare sorted files
comm file1 file2                      # Three columns: only1, only2, both
comm -12 file1 file2                  # Only lines in both files

# expand/unexpand - Convert tabs/spaces
expand file                           # Tabs to spaces
unexpand file                         # Spaces to tabs
```

---

## 2.3 Process Management

Understanding processes is critical for system administration and troubleshooting.

### What is a Process?

A process is a running instance of a program. Every process has:

- **PID**: Process ID - unique identifier
- **PPID**: Parent Process ID - PID of the process that spawned it
- **UID/GID**: User/Group ID - who owns the process
- **State**: Running, Sleeping, Stopped, Zombie, etc.
- **Priority/Nice**: Scheduling priority
- **Memory**: Virtual memory usage
- **CPU time**: How much CPU time it has used

```
Process States:
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│    ┌──────────┐      fork()      ┌──────────┐              │
│    │  Parent  │ ────────────────▶│   New    │              │
│    │ Process  │                  │ Process  │              │
│    └──────────┘                  └────┬─────┘              │
│                                       │                     │
│                                       ▼                     │
│                              ┌──────────────┐               │
│                              │   Running    │◀──────┐       │
│                              │   (R)        │       │       │
│                              └──────┬───────┘       │       │
│                                     │               │       │
│              ┌──────────────────────┼───────────────┘       │
│              │                      │                       │
│              ▼                      ▼                       │
│     ┌──────────────┐       ┌──────────────┐                │
│     │   Sleeping   │       │   Stopped    │                │
│     │   (S or D)   │       │   (T)        │                │
│     └──────────────┘       └──────────────┘                │
│              │                      │                       │
│              │        exit()        │                       │
│              └──────────┬───────────┘                       │
│                         ▼                                   │
│                ┌──────────────┐                            │
│                │   Zombie     │ (Waiting for parent)       │
│                │   (Z)        │                            │
│                └──────┬───────┘                            │
│                       │ wait()                             │
│                       ▼                                    │
│                ┌──────────────┐                            │
│                │  Terminated  │                            │
│                └──────────────┘                            │
│                                                             │
└─────────────────────────────────────────────────────────────┘

Process States:
R - Running or runnable (on run queue)
S - Interruptible sleep (waiting for event)
D - Uninterruptible sleep (usually I/O)
T - Stopped (signal or debugger)
Z - Zombie (terminated but not reaped by parent)
```

### Viewing Processes

```bash
# ps - Process Status

# Common usage patterns
ps                           # Processes in current terminal
ps aux                       # All processes, user-oriented format
ps -ef                       # All processes, full format
ps -ef --forest              # Process tree
ps -u username               # Processes by user
ps -p 1234                   # Specific PID
ps -C nginx                  # Processes by command name

# Understanding ps aux output:
# USER  PID %CPU %MEM   VSZ   RSS TTY STAT START   TIME COMMAND
# root    1  0.0  0.1 169292 13292 ?   Ss   Jan01   0:05 /sbin/init
#
# USER  : Process owner
# PID   : Process ID
# %CPU  : CPU usage percentage
# %MEM  : Memory usage percentage
# VSZ   : Virtual memory size (KB)
# RSS   : Resident Set Size - actual memory used (KB)
# TTY   : Terminal (? = no terminal)
# STAT  : Process state (see above)
# START : Start time
# TIME  : CPU time consumed
# COMMAND: Command line

# Custom output format
ps -eo pid,ppid,user,%cpu,%mem,stat,cmd --sort=-%mem | head -20
ps -eo pid,ppid,user,ni,pri,stat,cmd   # With nice and priority

# Filter high resource usage
ps aux --sort=-%cpu | head -10         # Top CPU consumers
ps aux --sort=-%mem | head -10         # Top memory consumers
ps aux --sort=-rss | head -10          # Top RSS (actual memory)

# top - Interactive process viewer
top
# Key commands in top:
#   q     : Quit
#   h     : Help
#   k     : Kill process (enter PID)
#   r     : Renice process (change priority)
#   u     : Filter by user
#   M     : Sort by memory
#   P     : Sort by CPU
#   1     : Toggle per-CPU stats
#   c     : Toggle full command line
#   V     : Forest view (tree)
#   d     : Change refresh interval

# Understanding top header:
# top - 14:30:00 up 5 days,  3:15,  2 users,  load average: 0.15, 0.10, 0.05
# Tasks: 250 total,   1 running, 249 sleeping,   0 stopped,   0 zombie
# %Cpu(s):  2.0 us,  1.0 sy,  0.0 ni, 97.0 id,  0.0 wa,  0.0 hi,  0.0 si
# MiB Mem :  16000.0 total,   8000.0 free,   4000.0 used,   4000.0 buff/cache
# MiB Swap:   2000.0 total,   2000.0 free,      0.0 used.  11000.0 avail Mem
#
# Load average: 1-min, 5-min, 15-min (how many processes want CPU)
#   - If load average > number of CPUs, system is overloaded
#
# CPU breakdown:
#   us (user)   : Time running user processes
#   sy (system) : Time running kernel processes
#   ni (nice)   : Time running niced user processes
#   id (idle)   : Time doing nothing
#   wa (iowait) : Time waiting for I/O
#   hi (hardware irq) : Time handling hardware interrupts
#   si (software irq) : Time handling software interrupts

# htop - Enhanced interactive viewer (install separately)
htop
# More user-friendly than top, with colors and mouse support

# pstree - Process tree
pstree                       # Show process tree
pstree -p                    # With PIDs
pstree -u                    # With usernames
pstree -a                    # With command arguments
pstree 1234                  # Tree starting from PID 1234
```

### Managing Processes

```bash
# Signals - ways to communicate with processes
# Common signals:
# SIGHUP  (1)  : Hangup - often used to reload config
# SIGINT  (2)  : Interrupt - Ctrl+C
# SIGQUIT (3)  : Quit - Ctrl+\ (produces core dump)
# SIGKILL (9)  : Kill - Cannot be caught or ignored (FORCE)
# SIGTERM (15) : Terminate - Polite request to stop (DEFAULT)
# SIGSTOP (19) : Stop - Cannot be caught (PAUSE)
# SIGCONT (18) : Continue - Resume stopped process
# SIGUSR1 (10) : User-defined signal 1
# SIGUSR2 (12) : User-defined signal 2

# kill - Send signal to process
kill 1234                    # Send SIGTERM (15) to PID 1234
kill -15 1234                # Same as above
kill -SIGTERM 1234           # Same as above
kill -9 1234                 # Force kill (SIGKILL)
kill -KILL 1234              # Same as above
kill -HUP 1234               # Reload configuration
kill -STOP 1234              # Pause process
kill -CONT 1234              # Resume process

# Always try SIGTERM first, only use SIGKILL as last resort
# SIGKILL doesn't give process chance to clean up

# killall - Kill processes by name
killall nginx                # Kill all nginx processes
killall -9 nginx             # Force kill all nginx
killall -u john              # Kill all processes by user

# pkill - Kill processes by pattern
pkill nginx                  # Kill processes matching "nginx"
pkill -9 -f "python script"  # Force kill matching full command line
pkill -u john                # Kill processes by user
pkill -P 1234                # Kill children of PID 1234

# pgrep - Find processes by pattern
pgrep nginx                  # PIDs of nginx processes
pgrep -l nginx               # PIDs and names
pgrep -a nginx               # PIDs and full command
pgrep -u john                # PIDs by user
pgrep -c nginx               # Count matching processes

# nice - Start process with priority
nice -n 10 command           # Start with lower priority (nice=10)
nice -n -10 command          # Start with higher priority (root only)
# Nice values: -20 (highest priority) to 19 (lowest priority)
# Default is 0

# renice - Change priority of running process
renice 10 -p 1234            # Set nice to 10 for PID 1234
renice -5 -p 1234            # Higher priority (root only)
renice 5 -u john             # Renice all of user's processes
renice 5 -g developers       # Renice all of group's processes

# nohup - Run immune to hangups
nohup command &              # Run in background, immune to SIGHUP
# Output goes to nohup.out by default
nohup command > output.log 2>&1 &

# disown - Remove job from shell's job table
command &                    # Start in background
disown                       # Disown last background job
disown %1                    # Disown job 1
disown -a                    # Disown all jobs

# Job control
command &                    # Run in background
jobs                         # List background jobs
fg                           # Bring last job to foreground
fg %1                        # Bring job 1 to foreground
bg                           # Resume stopped job in background
bg %1                        # Resume job 1 in background
Ctrl+Z                       # Stop (pause) current foreground job
Ctrl+C                       # Interrupt (kill) current foreground job
```

### Monitoring System Resources

```bash
# Memory
free                         # Memory usage
free -h                      # Human-readable
free -m                      # In megabytes
free -g                      # In gigabytes
free -s 1                    # Update every second

# Understanding free output:
#               total        used        free      shared  buff/cache   available
# Mem:          16000        4000        8000         100        4000       11000
# Swap:          2000           0        2000
#
# total     : Total physical memory
# used      : Used memory (includes buffers/cache)
# free      : Completely unused memory
# shared    : Memory used by tmpfs
# buff/cache: Memory used for buffers and cache (can be reclaimed)
# available : Memory available for new applications
#
# IMPORTANT: "available" is more useful than "free"
# Linux uses free RAM for buffers/cache, which can be freed if needed

# Detailed memory info
cat /proc/meminfo            # Very detailed memory info
vmstat 1                     # Virtual memory statistics every second

# vmstat columns:
# procs  : r (runnable), b (blocked)
# memory : swpd, free, buff, cache
# swap   : si (swap in), so (swap out)
# io     : bi (blocks in), bo (blocks out)
# system : in (interrupts), cs (context switches)
# cpu    : us, sy, id, wa, st

# CPU
lscpu                        # CPU information
cat /proc/cpuinfo            # Detailed CPU info
mpstat 1                     # CPU statistics every second
mpstat -P ALL 1              # Per-CPU statistics

# Uptime and load
uptime                       # Uptime and load averages
cat /proc/uptime             # Uptime in seconds
w                            # Who is logged in, load, activity

# I/O
iostat                       # I/O statistics
iostat -x 1                  # Extended stats every second
iotop                        # I/O usage by process (requires install)

# iostat columns:
# Device   : Device name
# tps      : Transfers per second
# kB_read/s: Kilobytes read per second
# kB_wrtn/s: Kilobytes written per second
# kB_read  : Total kilobytes read
# kB_wrtn  : Total kilobytes written
#
# Extended (-x) includes:
# r/s, w/s : Reads/writes per second
# rkB/s, wkB/s : KB read/written per second
# await    : Average time for requests (ms)
# %util    : Percentage of time device was busy

# All-in-one
sar                          # System Activity Reporter (if installed)
dstat                        # Versatile resource statistics (if installed)
glances                      # Complete overview (if installed)

# Process-specific resource usage
pidstat 1                    # Per-process CPU/memory every second
pidstat -d 1                 # Per-process I/O
pidstat -r 1                 # Per-process memory
```

---

This completes Part 1 of the comprehensive guide covering Linux fundamentals including the filesystem, shells, commands, and process management.

**Continue to Part 2 for:**
- User and Permission Management
- System Services (systemd)
- Package Management
- Storage and Filesystems
- Performance Tuning
- Troubleshooting
