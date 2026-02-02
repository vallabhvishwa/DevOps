# Comprehensive DevOps Learning Roadmap for AKS + Jenkins

> **Target Role:** Senior DevOps Engineer supporting AKS-based applications (Java/Node.js) with Jenkins CI/CD
> 
> **Goal:** Become an elite DevOps engineer who never gets stuck - mastering BAU processes, troubleshooting, and advanced operations

---

## Table of Contents

1. [Phase 1: Linux Mastery](#phase-1-linux-mastery)
2. [Phase 2: Networking Deep Dive](#phase-2-networking-deep-dive)
3. [Phase 3: Kubernetes Deep Fundamentals](#phase-3-kubernetes-deep-fundamentals)
4. [Phase 4: AKS (Azure Kubernetes Service) Mastery](#phase-4-aks-azure-kubernetes-service-mastery)
5. [Phase 5: Jenkins CI/CD for Kubernetes](#phase-5-jenkins-cicd-for-kubernetes)
6. [Phase 6: Application Runtime Knowledge](#phase-6-application-runtime-knowledge)
7. [Phase 7: Observability & Monitoring](#phase-7-observability--monitoring)
8. [Phase 8: Infrastructure as Code](#phase-8-infrastructure-as-code)
9. [Phase 9: Security Deep Dive](#phase-9-security-deep-dive)
10. [Phase 10: Troubleshooting Masterclass](#phase-10-troubleshooting-masterclass)
11. [Phase 11: GitOps & Advanced Deployment](#phase-11-gitops--advanced-deployment)
12. [Phase 12: Reliability Engineering](#phase-12-reliability-engineering)
13. [Phase 13: Essential Skills & Tools Reference](#phase-13-essential-skills--tools-reference)
14. [90-Day Intensive Plan](#90-day-intensive-plan)

---

## Phase 1: Linux Mastery

> Linux is the foundation of everything in DevOps. Master it deeply.

### 1.1 Linux Fundamentals

#### File System Hierarchy
```
/
├── /bin          → Essential user binaries (ls, cp, mv, cat)
├── /sbin         → System binaries (iptables, fdisk, init)
├── /etc          → Configuration files (CRITICAL - know this well)
│   ├── /etc/passwd, /etc/shadow, /etc/group
│   ├── /etc/hosts, /etc/resolv.conf, /etc/hostname
│   ├── /etc/fstab, /etc/mtab
│   ├── /etc/ssh/sshd_config
│   ├── /etc/systemd/
│   ├── /etc/sysctl.conf
│   └── /etc/security/limits.conf
├── /home         → User home directories
├── /root         → Root user home
├── /var          → Variable data
│   ├── /var/log  → System and application logs (CRITICAL)
│   ├── /var/lib  → State information
│   └── /var/run  → Runtime data (PIDs, sockets)
├── /tmp          → Temporary files (cleared on reboot)
├── /usr          → User programs and data
│   ├── /usr/bin  → User binaries
│   ├── /usr/lib  → Libraries
│   └── /usr/local → Locally installed software
├── /opt          → Optional/third-party software
├── /proc         → Virtual filesystem (process info, kernel params)
├── /sys          → Virtual filesystem (device/driver info)
├── /dev          → Device files
└── /mnt, /media  → Mount points
```

#### Essential Commands - Master These

```bash
# File Operations
ls -la                    # List with permissions
find / -name "*.log" -mtime -1   # Find files modified in last day
find / -type f -size +100M       # Find large files
locate <filename>         # Fast file search (uses db)
stat <file>              # Detailed file info

# Text Processing (CRITICAL for log analysis)
cat, head, tail, less, more
tail -f /var/log/syslog   # Follow log file
grep -r "pattern" /path   # Recursive search
grep -E "regex" file      # Extended regex
grep -v "exclude"         # Invert match
grep -A 3 -B 3 "pattern"  # Context lines
awk '{print $1, $3}'      # Column extraction
awk -F: '{print $1}' /etc/passwd
sed 's/old/new/g' file    # Stream editing
sed -i 's/old/new/g' file # In-place edit
cut -d: -f1 /etc/passwd   # Cut columns
sort, uniq, wc            # Sorting, unique, counting
tr 'a-z' 'A-Z'           # Translate characters
xargs                     # Build commands from input

# Compression
tar -czvf archive.tar.gz /path   # Create gzipped tarball
tar -xzvf archive.tar.gz         # Extract
gzip, gunzip, zcat
bzip2, bunzip2, bzcat
zip, unzip

# Disk & Storage
df -h                     # Disk usage (human readable)
du -sh /path              # Directory size
lsblk                     # Block devices
fdisk -l                  # Partition table
mount, umount
blkid                     # Block device IDs
iostat                    # I/O statistics

# Process Management
ps aux                    # All processes
ps -ef --forest          # Process tree
top, htop                # Interactive process viewer
pgrep, pkill             # Find/kill by name
kill -9 <pid>            # Force kill
killall <name>           # Kill by name
nice, renice             # Process priority
nohup command &          # Run immune to hangups
jobs, fg, bg             # Job control
lsof                     # List open files
lsof -i :8080           # What's using port 8080
fuser -v /path           # What's using a file

# Memory
free -h                  # Memory usage
vmstat                   # Virtual memory stats
cat /proc/meminfo        # Detailed memory info

# System Information
uname -a                 # Kernel info
cat /etc/os-release      # OS info
hostname                 # System hostname
uptime                   # System uptime
dmesg                    # Kernel ring buffer
journalctl               # Systemd journal
```

### 1.2 User & Permission Management

```bash
# User Management
useradd -m -s /bin/bash username
usermod -aG groupname username
userdel -r username
passwd username
id username
whoami, who, w

# Group Management
groupadd groupname
groupmod
groupdel

# File Permissions
chmod 755 file           # rwxr-xr-x
chmod u+x file           # Add execute for owner
chmod -R 644 /path       # Recursive
chown user:group file
chown -R user:group /path

# Special Permissions
chmod u+s file           # SUID (run as file owner)
chmod g+s directory      # SGID (inherit group)
chmod +t directory       # Sticky bit (only owner can delete)

# ACLs (Access Control Lists)
getfacl file
setfacl -m u:user:rwx file
setfacl -m g:group:rx file
setfacl -x u:user file   # Remove ACL

# Sudoers
visudo                   # Edit sudoers safely
# /etc/sudoers.d/        # Drop-in configs
```

#### Permission Reference
```
Permission  Numeric  Meaning
---------  -------  -------
rwx------    700    Owner full, others none
rwxr-xr-x    755    Owner full, others read+execute
rw-r--r--    644    Owner read+write, others read only
rw-rw-r--    664    Owner+group read+write, others read
rwx------    700    Scripts/executables (private)
```

### 1.3 Process & Service Management

#### Systemd (Modern Linux)
```bash
# Service Management
systemctl start|stop|restart|reload <service>
systemctl enable|disable <service>    # Auto-start on boot
systemctl status <service>
systemctl list-units --type=service
systemctl list-units --failed
systemctl daemon-reload               # After changing unit files

# Journalctl (Log viewing)
journalctl -u <service>               # Logs for service
journalctl -u <service> -f            # Follow logs
journalctl -u <service> --since "1 hour ago"
journalctl -u <service> -p err        # Only errors
journalctl -xe                        # Recent errors with context
journalctl --disk-usage               # Journal size
journalctl --vacuum-size=500M         # Clean up

# Unit Files
/etc/systemd/system/                  # Custom unit files
/lib/systemd/system/                  # Package-provided

# Example unit file
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/bin/start.sh
ExecStop=/opt/myapp/bin/stop.sh
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

#### Process Analysis
```bash
# CPU/Memory per process
ps aux --sort=-%mem | head -20       # Top memory consumers
ps aux --sort=-%cpu | head -20       # Top CPU consumers

# Process limits
cat /proc/<pid>/limits
ulimit -a                            # Current shell limits
cat /etc/security/limits.conf        # System limits

# Strace (System call tracing - CRITICAL for debugging)
strace -p <pid>                      # Attach to running process
strace -f command                    # Trace command + children
strace -e open,read,write command    # Filter syscalls
strace -c command                    # Summary/statistics

# Ltrace (Library call tracing)
ltrace command

# /proc filesystem
/proc/<pid>/cmdline                  # Command line
/proc/<pid>/environ                  # Environment variables
/proc/<pid>/fd/                      # Open file descriptors
/proc/<pid>/maps                     # Memory mappings
/proc/<pid>/status                   # Process status
```

### 1.4 Package Management

```bash
# Debian/Ubuntu (APT)
apt update                           # Update package lists
apt upgrade                          # Upgrade packages
apt install <package>
apt remove <package>
apt purge <package>                  # Remove with config
apt search <keyword>
apt show <package>
dpkg -l                              # List installed
dpkg -L <package>                    # Files in package
dpkg -S /path/to/file               # Which package owns file

# RHEL/CentOS (YUM/DNF)
yum update / dnf update
yum install <package>
yum remove <package>
yum search <keyword>
yum info <package>
yum list installed
rpm -qa                              # List all packages
rpm -ql <package>                    # Files in package
rpm -qf /path/to/file               # Which package owns file

# Package sources
/etc/apt/sources.list               # Debian/Ubuntu
/etc/apt/sources.list.d/            # Additional repos
/etc/yum.repos.d/                   # RHEL/CentOS repos
```

### 1.5 Storage & Filesystems

```bash
# Disk Partitioning
fdisk /dev/sda                       # MBR partitioning
gdisk /dev/sda                       # GPT partitioning
parted /dev/sda                      # Modern partitioning

# Filesystem Operations
mkfs.ext4 /dev/sda1
mkfs.xfs /dev/sda1
mount /dev/sda1 /mnt/data
umount /mnt/data

# /etc/fstab format
# <device>  <mount>  <type>  <options>  <dump>  <pass>
/dev/sda1   /data    ext4    defaults   0       2
UUID=xxx    /data    ext4    defaults   0       2

# LVM (Logical Volume Management)
# Physical Volumes
pvcreate /dev/sdb
pvdisplay
pvs

# Volume Groups
vgcreate vg_data /dev/sdb
vgdisplay
vgs
vgextend vg_data /dev/sdc

# Logical Volumes
lvcreate -L 10G -n lv_data vg_data
lvdisplay
lvs
lvextend -L +5G /dev/vg_data/lv_data
resize2fs /dev/vg_data/lv_data       # Extend ext4 filesystem
xfs_growfs /mnt/data                 # Extend XFS filesystem

# Disk I/O Analysis
iostat -x 1                          # Extended I/O stats
iotop                                # I/O by process
hdparm -Tt /dev/sda                  # Disk benchmark
```

### 1.6 System Performance & Tuning

```bash
# Overall System Performance
top / htop                           # Interactive monitoring
vmstat 1                             # Virtual memory stats
mpstat -P ALL 1                      # Per-CPU stats
pidstat 1                            # Per-process stats
sar                                  # System Activity Reporter

# Memory Analysis
free -h
cat /proc/meminfo
slabtop                              # Kernel slab cache
echo 3 > /proc/sys/vm/drop_caches   # Clear caches (careful!)

# CPU Analysis
cat /proc/cpuinfo
lscpu
mpstat -P ALL 1                      # Per-CPU utilization
perf top                             # CPU profiling
perf record -g command               # Profile command
perf report                          # View profile

# Kernel Parameters (sysctl)
sysctl -a                            # All parameters
sysctl vm.swappiness                 # Single parameter
sysctl -w vm.swappiness=10           # Set temporarily
# /etc/sysctl.conf                   # Persistent changes
sysctl -p                            # Reload

# Important sysctl settings
vm.swappiness = 10                   # Reduce swap usage
net.core.somaxconn = 65535          # Max socket connections
net.ipv4.tcp_max_syn_backlog = 65535
fs.file-max = 2097152               # Max open files system-wide
net.ipv4.ip_local_port_range = 1024 65535
```

### 1.7 Log Management

```bash
# Important Log Files
/var/log/syslog         # General system logs (Debian/Ubuntu)
/var/log/messages       # General system logs (RHEL/CentOS)
/var/log/auth.log       # Authentication logs
/var/log/secure         # Authentication logs (RHEL)
/var/log/kern.log       # Kernel logs
/var/log/dmesg          # Boot messages
/var/log/cron           # Cron job logs
/var/log/boot.log       # Boot process logs
/var/log/nginx/         # Nginx logs
/var/log/apache2/       # Apache logs

# Log Analysis Commands
tail -f /var/log/syslog              # Follow logs
grep -i error /var/log/syslog
zgrep -i error /var/log/syslog.*.gz  # Search compressed logs
less +F /var/log/syslog              # Follow mode in less
awk '/error/ {print $0}' /var/log/syslog

# Logrotate
/etc/logrotate.conf
/etc/logrotate.d/

# Example logrotate config
/var/log/myapp/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 0640 appuser appgroup
    postrotate
        systemctl reload myapp
    endscript
}
```

### 1.8 SSH & Remote Access

```bash
# SSH Client
ssh user@host
ssh -i ~/.ssh/key.pem user@host      # With key
ssh -p 2222 user@host                # Custom port
ssh -L 8080:localhost:80 user@host   # Local port forward
ssh -R 8080:localhost:80 user@host   # Remote port forward
ssh -D 1080 user@host                # SOCKS proxy
ssh -J jumphost user@target          # Jump host / bastion

# SSH Config (~/.ssh/config)
Host myserver
    HostName 192.168.1.100
    User admin
    Port 22
    IdentityFile ~/.ssh/mykey.pem
    ForwardAgent yes

Host bastion
    HostName bastion.example.com
    User admin

Host internal-*
    ProxyJump bastion
    User admin

# SSH Server (/etc/ssh/sshd_config)
Port 22
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AllowUsers admin deploy
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2

# SSH Key Management
ssh-keygen -t ed25519 -C "email@example.com"
ssh-keygen -t rsa -b 4096 -C "email@example.com"
ssh-copy-id user@host
ssh-add ~/.ssh/key                   # Add to agent
ssh-add -l                           # List keys in agent
```

### 1.9 Cron & Scheduled Tasks

```bash
# Crontab
crontab -e                           # Edit user crontab
crontab -l                           # List user crontab
crontab -r                           # Remove user crontab

# Cron Format
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of week (0 - 6) (Sunday=0)
# │ │ │ │ │
# * * * * * command

# Examples
0 * * * * /script.sh                 # Every hour
*/5 * * * * /script.sh               # Every 5 minutes
0 2 * * * /backup.sh                 # 2 AM daily
0 2 * * 0 /weekly.sh                 # 2 AM Sunday
0 0 1 * * /monthly.sh                # Midnight, 1st of month

# System cron directories
/etc/crontab                         # System crontab
/etc/cron.d/                         # Drop-in cron files
/etc/cron.daily/                     # Daily jobs
/etc/cron.weekly/                    # Weekly jobs
/etc/cron.monthly/                   # Monthly jobs

# Systemd Timers (Modern alternative)
systemctl list-timers
# Create .timer unit file alongside .service
```

---

## Phase 2: Networking Deep Dive

> Networking issues are the #1 cause of application problems. Master this.

### 2.1 Networking Fundamentals

#### OSI Model & TCP/IP
```
OSI Layer    TCP/IP Layer   Protocols                  Devices
─────────────────────────────────────────────────────────────────
7 Application  ┐
6 Presentation ├ Application   HTTP, HTTPS, DNS,         Proxy,
5 Session      ┘               SSH, FTP, SMTP            Load Balancer

4 Transport      Transport     TCP, UDP                  Firewall

3 Network        Internet      IP, ICMP, ARP            Router, L3 Switch

2 Data Link   ┐                
1 Physical    ┘  Network       Ethernet, WiFi           Switch, NIC
                 Access
```

#### IP Addressing & Subnetting
```
IPv4 Address Classes (Legacy):
Class A: 1.0.0.0   - 126.255.255.255  (/8)   16M hosts
Class B: 128.0.0.0 - 191.255.255.255  (/16)  65K hosts
Class C: 192.0.0.0 - 223.255.255.255  (/24)  254 hosts

Private IP Ranges (RFC 1918):
10.0.0.0/8        (10.0.0.0 - 10.255.255.255)
172.16.0.0/12     (172.16.0.0 - 172.31.255.255)
192.168.0.0/16    (192.168.0.0 - 192.168.255.255)

CIDR Notation:
/32 = 1 IP (host route)
/31 = 2 IPs (point-to-point)
/30 = 4 IPs (2 usable)
/29 = 8 IPs (6 usable)
/28 = 16 IPs (14 usable)
/27 = 32 IPs (30 usable)
/26 = 64 IPs (62 usable)
/25 = 128 IPs (126 usable)
/24 = 256 IPs (254 usable) - typical subnet
/16 = 65,536 IPs
/8  = 16,777,216 IPs

Subnet Calculation Example:
192.168.10.0/26
Network: 192.168.10.0
Broadcast: 192.168.10.63
Usable: 192.168.10.1 - 192.168.10.62
Subnet Mask: 255.255.255.192
```

### 2.2 Linux Network Configuration

```bash
# Modern Tools (iproute2)
ip addr show                         # Show IP addresses
ip addr add 192.168.1.10/24 dev eth0
ip addr del 192.168.1.10/24 dev eth0
ip link show                         # Show interfaces
ip link set eth0 up|down
ip route show                        # Show routing table
ip route add 10.0.0.0/8 via 192.168.1.1
ip route add default via 192.168.1.1
ip neigh show                        # ARP table

# Legacy Tools (still useful)
ifconfig
route -n
netstat -tuln                        # Listening ports
netstat -anp                         # All connections with PIDs
arp -a

# Modern alternatives
ss -tuln                             # Listening sockets (faster than netstat)
ss -anp                              # All sockets with processes
ss -s                                # Socket statistics

# Network Configuration Files
# Debian/Ubuntu (Netplan - modern)
/etc/netplan/*.yaml

# Example netplan config
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 192.168.1.10/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]

# Apply netplan
netplan apply

# RHEL/CentOS (NetworkManager)
nmcli device status
nmcli connection show
nmcli connection modify eth0 ipv4.addresses 192.168.1.10/24
nmcli connection modify eth0 ipv4.gateway 192.168.1.1
nmcli connection modify eth0 ipv4.dns "8.8.8.8 8.8.4.4"
nmcli connection modify eth0 ipv4.method manual
nmcli connection up eth0
```

### 2.3 DNS Deep Dive

```bash
# DNS Resolution
/etc/resolv.conf                     # DNS configuration
/etc/hosts                           # Local host resolution
/etc/nsswitch.conf                   # Name resolution order

# DNS Lookup Tools
nslookup example.com
dig example.com                      # Detailed DNS info
dig +short example.com               # Quick answer
dig @8.8.8.8 example.com            # Use specific DNS server
dig example.com ANY                  # All record types
dig example.com MX                   # Mail records
dig example.com TXT                  # TXT records
dig -x 8.8.8.8                      # Reverse lookup
dig +trace example.com              # Full resolution path
host example.com                     # Simple lookup

# DNS Record Types
A       → IPv4 address
AAAA    → IPv6 address
CNAME   → Canonical name (alias)
MX      → Mail exchange
TXT     → Text record (SPF, DKIM, etc.)
NS      → Name server
SOA     → Start of Authority
PTR     → Pointer (reverse DNS)
SRV     → Service record

# DNS Troubleshooting
# Check if DNS is working
dig google.com
# Check specific DNS server
dig @10.0.0.2 internal.domain.com
# Trace DNS resolution
dig +trace domain.com
# Check DNS response time
dig domain.com | grep "Query time"
```

### 2.4 TCP/UDP & Ports

```bash
# Common Ports to Know
Port    Service          Protocol
────────────────────────────────
20/21   FTP              TCP
22      SSH              TCP
23      Telnet           TCP
25      SMTP             TCP
53      DNS              TCP/UDP
67/68   DHCP             UDP
80      HTTP             TCP
110     POP3             TCP
123     NTP              UDP
143     IMAP             TCP
443     HTTPS            TCP
465     SMTPS            TCP
514     Syslog           UDP
587     SMTP (submission) TCP
993     IMAPS            TCP
995     POP3S            TCP
3306    MySQL            TCP
5432    PostgreSQL       TCP
6379    Redis            TCP
8080    HTTP Alt         TCP
8443    HTTPS Alt        TCP
27017   MongoDB          TCP

# Kubernetes Specific
2379    etcd client      TCP
2380    etcd peer        TCP
6443    Kubernetes API   TCP
10250   Kubelet API      TCP
10251   Kube-scheduler   TCP
10252   Kube-controller  TCP
30000-32767  NodePort    TCP

# Check port usage
ss -tuln | grep :80
lsof -i :80
fuser 80/tcp
netstat -tuln | grep :80

# Port scanning (from allowed networks only!)
nc -zv host 80                       # Check single port
nc -zv host 20-30                    # Port range
nmap -p 80 host                      # Nmap scan
nmap -sT host                        # TCP connect scan
nmap -sV host                        # Service detection
```

### 2.5 Network Troubleshooting Tools

```bash
# Connectivity Testing
ping host                            # ICMP echo
ping -c 4 host                       # 4 pings only
ping -i 0.2 host                     # 0.2s interval
ping6 host                           # IPv6

# Path Analysis
traceroute host                      # Trace path
traceroute -n host                   # No DNS resolution
traceroute -T host                   # TCP (if ICMP blocked)
mtr host                             # Continuous traceroute
mtr -r -c 10 host                    # Report mode

# HTTP/HTTPS Testing
curl -v https://example.com          # Verbose output
curl -I https://example.com          # Headers only
curl -o /dev/null -s -w "%{http_code}\n" https://example.com
curl -k https://example.com          # Ignore SSL errors
curl --connect-timeout 5 host        # Connection timeout
curl -x proxy:port url               # Via proxy
curl -H "Header: value" url          # Custom header
curl -X POST -d "data" url           # POST request
wget -q -O- https://example.com      # Output to stdout
wget --spider url                    # Just check if exists

# TCP Connection Testing
telnet host port                     # Basic TCP test
nc -v host port                      # Netcat verbose
nc -z host port                      # Zero I/O mode (just check)
nc -zv host 20-30                    # Port range scan
nc -l -p 8080                        # Listen on port

# SSL/TLS Testing
openssl s_client -connect host:443
openssl s_client -connect host:443 -servername hostname  # SNI
openssl s_client -connect host:443 </dev/null | openssl x509 -text

# Packet Capture (tcpdump) - CRITICAL SKILL
tcpdump -i eth0                      # Capture on interface
tcpdump -i any                       # All interfaces
tcpdump -i eth0 port 80              # Filter by port
tcpdump -i eth0 host 192.168.1.100   # Filter by host
tcpdump -i eth0 src 192.168.1.100    # Filter by source
tcpdump -i eth0 dst 192.168.1.100    # Filter by destination
tcpdump -i eth0 tcp                  # TCP only
tcpdump -i eth0 udp                  # UDP only
tcpdump -i eth0 icmp                 # ICMP only
tcpdump -i eth0 'port 80 and host 192.168.1.100'
tcpdump -i eth0 -w capture.pcap      # Write to file
tcpdump -r capture.pcap              # Read from file
tcpdump -i eth0 -A                   # ASCII output
tcpdump -i eth0 -X                   # Hex + ASCII
tcpdump -i eth0 -n                   # Don't resolve hostnames
tcpdump -i eth0 -c 100               # Capture 100 packets only

# Common tcpdump filters
'tcp port 80'                        # HTTP traffic
'tcp port 443'                       # HTTPS traffic
'tcp port 53 or udp port 53'        # DNS traffic
'host 10.0.0.1 and port 22'         # SSH to specific host
'tcp[tcpflags] & (tcp-syn) != 0'    # TCP SYN packets
'tcp[tcpflags] & (tcp-rst) != 0'    # TCP RST packets
```

### 2.6 Firewall & iptables

```bash
# iptables Chains
INPUT   → Incoming traffic to local system
OUTPUT  → Outgoing traffic from local system
FORWARD → Traffic passing through (routing)

# iptables Basics
iptables -L -n -v                    # List all rules
iptables -L INPUT -n --line-numbers  # With line numbers
iptables -S                          # Print rules as commands

# Add rules
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
iptables -A INPUT -s 192.168.1.0/24 -j ACCEPT
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -j DROP            # Default deny

# Insert at position
iptables -I INPUT 1 -p tcp --dport 22 -j ACCEPT

# Delete rules
iptables -D INPUT -p tcp --dport 80 -j ACCEPT
iptables -D INPUT 3                  # By line number

# NAT (Network Address Translation)
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080

# Save/Restore rules
iptables-save > /etc/iptables.rules
iptables-restore < /etc/iptables.rules

# Modern: nftables (replacement for iptables)
nft list ruleset
nft add table inet filter
nft add chain inet filter input { type filter hook input priority 0 \; }
nft add rule inet filter input tcp dport 22 accept

# Firewalld (RHEL/CentOS)
firewall-cmd --state
firewall-cmd --list-all
firewall-cmd --add-port=80/tcp --permanent
firewall-cmd --add-service=http --permanent
firewall-cmd --reload

# UFW (Ubuntu)
ufw status
ufw allow 22/tcp
ufw allow from 192.168.1.0/24
ufw enable
```

### 2.7 Load Balancing Concepts

```
Load Balancing Algorithms:
├── Round Robin           → Equal distribution, simple
├── Weighted Round Robin  → Based on server capacity
├── Least Connections     → Send to server with fewest connections
├── IP Hash              → Same client IP → same server (sticky)
├── Weighted Response    → Based on response times
└── Random               → Random server selection

Layer 4 (Transport) vs Layer 7 (Application):
┌─────────────────┬─────────────────────────────────────────────────┐
│ Layer 4 (TCP)   │ - Fast, simple                                  │
│                 │ - Based on IP/port only                         │
│                 │ - Can't inspect content                         │
│                 │ - Azure Load Balancer, AWS NLB                  │
├─────────────────┼─────────────────────────────────────────────────┤
│ Layer 7 (HTTP)  │ - Content-aware routing                         │
│                 │ - Path-based routing                            │
│                 │ - Host-based routing                            │
│                 │ - SSL termination                               │
│                 │ - Azure App Gateway, AWS ALB, NGINX, HAProxy    │
└─────────────────┴─────────────────────────────────────────────────┘

Health Checks:
├── TCP connect      → Just check if port responds
├── HTTP GET         → Check for 200 OK
├── HTTP with path   → GET /health returns 200
└── Custom script    → Application-specific checks
```

### 2.8 Kubernetes Networking

```
Kubernetes Network Model:
├── Pod Networking
│   ├── Each pod gets unique IP
│   ├── Pods can communicate directly (no NAT)
│   ├── CNI plugins implement this (Calico, Cilium, Azure CNI)
│   └── Pod CIDR: e.g., 10.244.0.0/16
│
├── Service Networking
│   ├── ClusterIP  → Internal only (10.96.0.0/12 default)
│   ├── NodePort   → Expose on each node (30000-32767)
│   ├── LoadBalancer → External load balancer (cloud)
│   └── ExternalName → DNS CNAME to external service
│
├── Ingress
│   ├── HTTP/HTTPS routing
│   ├── Path-based routing
│   ├── Host-based routing
│   ├── TLS termination
│   └── Ingress controllers: NGINX, Traefik, AGIC
│
├── Network Policies
│   ├── Pod-to-pod firewall rules
│   ├── Ingress (incoming) rules
│   ├── Egress (outgoing) rules
│   └── Namespace isolation
│
└── DNS (CoreDNS)
    ├── <service>.<namespace>.svc.cluster.local
    ├── <pod-ip-dashed>.<namespace>.pod.cluster.local
    └── Headless services for pod discovery
```

#### Kubernetes Network Troubleshooting
```bash
# DNS troubleshooting
kubectl run test --rm -it --image=busybox -- nslookup kubernetes
kubectl run test --rm -it --image=busybox -- nslookup <service>.<namespace>
kubectl get pods -n kube-system -l k8s-app=kube-dns

# Check CoreDNS logs
kubectl logs -n kube-system -l k8s-app=kube-dns

# Network debugging pod
kubectl run netshoot --rm -it --image=nicolaka/netshoot -- bash
# Inside:
curl http://service:port
dig service.namespace.svc.cluster.local
tcpdump -i eth0

# Check service endpoints
kubectl get endpoints <service>
kubectl describe svc <service>

# Check network policies
kubectl get networkpolicies
kubectl describe networkpolicy <name>

# Pod-to-pod connectivity
kubectl exec -it pod1 -- ping <pod2-ip>
kubectl exec -it pod1 -- curl http://<pod2-ip>:port

# Service connectivity
kubectl exec -it pod1 -- curl http://<service>:<port>
kubectl exec -it pod1 -- curl http://<service>.<namespace>:<port>
```

### 2.9 Common Network Issues & Solutions

| Issue | Symptoms | Diagnosis | Solution |
|-------|----------|-----------|----------|
| DNS failure | Connection refused, name resolution fails | `dig`, `nslookup`, check `/etc/resolv.conf` | Check DNS server, CoreDNS pods, network policies |
| Port not listening | Connection refused | `ss -tuln`, `netstat -tuln` | Start service, check binding address |
| Firewall blocking | Connection timeout | `iptables -L`, `tcpdump` | Add firewall rule, check NSG/security groups |
| MTU issues | Large packets fail, SSH works but HTTP hangs | `ping -M do -s 1472 host` | Adjust MTU, enable PMTU discovery |
| NAT issues | Can reach internet but not internal | Check NAT rules, routing | Configure NAT, check routing tables |
| Routing issues | Packets not reaching destination | `ip route`, `traceroute` | Add routes, fix gateway configuration |
| SSL/TLS issues | SSL handshake failures | `openssl s_client`, check certs | Fix certificates, check expiry |
| TCP RST | Immediate connection reset | `tcpdump`, check app logs | App crash, port reuse, firewall |
| Connection timeout | No response | `tcpdump`, `ping`, `traceroute` | Network path issue, firewall, host down |

---

## Phase 3: Kubernetes Deep Fundamentals

### 3.1 Core Kubernetes Architecture

```
Kubernetes Control Plane Components:
├── kube-apiserver
│   ├── REST API for all cluster operations
│   ├── Authentication & Authorization
│   ├── Admission Controllers
│   └── Validation & Mutation
│
├── etcd
│   ├── Distributed key-value store
│   ├── All cluster state stored here
│   ├── Raft consensus algorithm
│   └── Backup/restore critical
│
├── kube-scheduler
│   ├── Assigns pods to nodes
│   ├── Considers: resources, affinity, taints
│   ├── Extensible with custom schedulers
│   └── Scheduling framework
│
├── kube-controller-manager
│   ├── Node Controller
│   ├── ReplicaSet Controller
│   ├── Deployment Controller
│   ├── Endpoint Controller
│   ├── Service Account Controller
│   └── Many more...
│
└── cloud-controller-manager (AKS)
    ├── Node Controller (cloud-specific)
    ├── Route Controller
    └── Service Controller (LoadBalancer)

Node Components:
├── kubelet
│   ├── Node agent, runs on each node
│   ├── Manages pod lifecycle
│   ├── Reports node status
│   └── Executes pod specs
│
├── kube-proxy
│   ├── Network proxy on each node
│   ├── Implements Service abstraction
│   ├── iptables or IPVS mode
│   └── Load balances to pod endpoints
│
└── Container Runtime
    ├── containerd (default)
    ├── CRI-O
    └── Implements CRI (Container Runtime Interface)
```

### 3.2 Kubernetes Objects - Master Every Single One

#### Workload Resources
```yaml
# Pod - Basic unit of deployment
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  containers:
  - name: app
    image: myapp:1.0
    ports:
    - containerPort: 8080
    resources:
      requests:
        memory: "256Mi"
        cpu: "250m"
      limits:
        memory: "512Mi"
        cpu: "500m"
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
    env:
    - name: DB_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_host
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secrets
          key: database_password
    volumeMounts:
    - name: config-volume
      mountPath: /config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
---
# Deployment - Declarative updates for pods
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: myapp:1.0
---
# StatefulSet - For stateful applications
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
---
# DaemonSet - One pod per node
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      tolerations:
      - operator: Exists
      containers:
      - name: node-exporter
        image: prom/node-exporter
---
# Job - Run to completion
apiVersion: batch/v1
kind: Job
metadata:
  name: backup
spec:
  backoffLimit: 4
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: backup
        image: backup-tool
        command: ["./backup.sh"]
---
# CronJob - Scheduled jobs
apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-backup
spec:
  schedule: "0 2 * * *"
  concurrencyPolicy: Forbid
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: backup
            image: backup-tool
```

#### Service & Networking
```yaml
# ClusterIP Service (internal)
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
---
# LoadBalancer Service (external)
apiVersion: v1
kind: Service
metadata:
  name: myapp-external
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"  # Internal LB
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
---
# Ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - myapp.example.com
    secretName: tls-secret
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
---
# NetworkPolicy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-network-policy
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    - namespaceSelector:
        matchLabels:
          name: monitoring
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
```

#### Configuration & Storage
```yaml
# ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_host: "mysql.default.svc.cluster.local"
  log_level: "info"
  config.yaml: |
    server:
      port: 8080
    database:
      pool_size: 10
---
# Secret
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  database_password: cGFzc3dvcmQxMjM=  # base64 encoded
  api_key: c2VjcmV0a2V5  # base64 encoded
---
# PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: managed-premium
  resources:
    requests:
      storage: 100Gi
---
# StorageClass (Azure Disk)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-premium
provisioner: disk.csi.azure.com
parameters:
  skuName: Premium_LRS
  cachingmode: ReadOnly
reclaimPolicy: Retain
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

#### Security Resources
```yaml
# ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-service-account
  annotations:
    azure.workload.identity/client-id: "<managed-identity-client-id>"
---
# Role (namespace-scoped)
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]
---
# RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
subjects:
- kind: ServiceAccount
  name: app-service-account
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
---
# ClusterRole (cluster-scoped)
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]
---
# ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: node-reader-binding
subjects:
- kind: ServiceAccount
  name: app-service-account
  namespace: default
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

### 3.3 kubectl Mastery - Be Lightning Fast

```bash
# Resource Operations
kubectl get pods -o wide                    # Extra info (IP, node)
kubectl get pods -o yaml                    # Full YAML output
kubectl get pods -o json                    # JSON output
kubectl get pods -o custom-columns='NAME:.metadata.name,STATUS:.status.phase'
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
kubectl get pods --sort-by=.metadata.creationTimestamp
kubectl get pods --field-selector=status.phase=Running
kubectl get pods -l app=myapp               # By label
kubectl get pods -l 'app in (myapp,yourapp)'
kubectl get all                             # All resources
kubectl get all -A                          # All namespaces

# Debugging
kubectl describe pod <pod>                  # Detailed info
kubectl logs <pod>                          # Container logs
kubectl logs <pod> -c <container>           # Specific container
kubectl logs <pod> --previous               # Previous container
kubectl logs <pod> -f                       # Follow logs
kubectl logs -l app=myapp                   # By label
kubectl logs --since=1h <pod>               # Last hour
kubectl logs --tail=100 <pod>               # Last 100 lines

# Exec into pods
kubectl exec -it <pod> -- /bin/bash
kubectl exec -it <pod> -c <container> -- /bin/bash
kubectl exec <pod> -- cat /etc/config

# Port forwarding
kubectl port-forward <pod> 8080:80
kubectl port-forward svc/<service> 8080:80
kubectl port-forward deploy/<deployment> 8080:80

# Copy files
kubectl cp <pod>:/path/to/file ./local-file
kubectl cp ./local-file <pod>:/path/to/file

# Resource usage
kubectl top pods
kubectl top pods --containers
kubectl top nodes

# Deployment operations
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>
kubectl rollout undo deployment/<name> --to-revision=2
kubectl rollout restart deployment/<name>

# Scaling
kubectl scale deployment/<name> --replicas=5
kubectl autoscale deployment/<name> --min=3 --max=10 --cpu-percent=80

# Node operations
kubectl cordon <node>                       # Mark unschedulable
kubectl uncordon <node>                     # Mark schedulable
kubectl drain <node> --ignore-daemonsets    # Drain node
kubectl taint nodes <node> key=value:NoSchedule
kubectl label nodes <node> type=gpu

# Quick debugging
kubectl run debug --rm -it --image=busybox -- sh
kubectl run debug --rm -it --image=nicolaka/netshoot -- bash
kubectl debug <pod> -it --image=busybox

# Events (critical for troubleshooting)
kubectl get events --sort-by='.lastTimestamp'
kubectl get events -A --sort-by='.lastTimestamp'
kubectl get events --field-selector type=Warning

# Context & configuration
kubectl config get-contexts
kubectl config use-context <context>
kubectl config set-context --current --namespace=<ns>
kubectx                                     # Switch context (plugin)
kubens                                      # Switch namespace (plugin)

# Explain resources
kubectl explain pod
kubectl explain pod.spec.containers
kubectl explain pod --recursive

# Dry run & diff
kubectl apply -f file.yaml --dry-run=client
kubectl apply -f file.yaml --dry-run=server
kubectl diff -f file.yaml

# API resources
kubectl api-resources
kubectl api-versions
```

### 3.4 Container Runtime Deep Dive

```bash
# containerd (via crictl)
crictl ps                               # List containers
crictl pods                             # List pods
crictl images                           # List images
crictl logs <container-id>              # Container logs
crictl exec -it <container-id> sh       # Exec into container
crictl stats                            # Container stats
crictl inspect <container-id>           # Inspect container

# Container debugging
crictl inspect <container-id> | jq '.status.state'
crictl inspect <container-id> | jq '.info.runtimeSpec'

# Image operations
crictl pull <image>
crictl rmi <image>

# containerd configuration
/etc/containerd/config.toml

# cgroups (resource limits)
cat /sys/fs/cgroup/memory/memory.limit_in_bytes
cat /sys/fs/cgroup/cpu/cpu.cfs_quota_us
cat /sys/fs/cgroup/cpu/cpu.cfs_period_us

# Check container namespaces
lsns                                    # List namespaces
nsenter --target <pid> --net ip addr    # Enter network namespace
nsenter --target <pid> --pid --mount top # Enter pid/mount namespace
```

---

## Phase 4: AKS (Azure Kubernetes Service) Mastery

### 4.1 AKS Architecture & Components

```
AKS-Specific Components:
├── Managed Control Plane
│   ├── Azure manages: API server, etcd, scheduler, controller-manager
│   ├── SLA tiers (Free, Standard, Premium)
│   └── Control plane logs (kube-apiserver, kube-controller-manager, etc.)
│
├── Node Pools
│   ├── System vs User node pools
│   ├── VM Scale Sets (VMSS) backend
│   ├── Spot instances
│   ├── Node pool modes (System, User)
│   └── Node images (Ubuntu, Windows, Azure Linux, custom)
│
├── Networking Models
│   ├── kubenet (basic, NAT-based)
│   ├── Azure CNI (VNet-integrated pods)
│   ├── Azure CNI Overlay
│   ├── Azure CNI with Dynamic IP Allocation
│   └── Bring Your Own CNI (Cilium, Calico)
│
├── Identity & Access
│   ├── Managed Identity (System/User-assigned)
│   ├── Azure AD integration (AKS-managed vs BYO)
│   ├── Azure RBAC for Kubernetes
│   ├── Workload Identity (pod-level Azure access)
│   └── Kubelet identity
│
└── Add-ons & Extensions
    ├── Azure Policy for AKS
    ├── Azure Monitor Container Insights
    ├── Azure Key Vault Secrets Provider
    ├── Application Gateway Ingress Controller (AGIC)
    ├── Open Service Mesh
    ├── Dapr
    ├── KEDA
    └── GitOps (Flux v2)
```

### 4.2 AKS Networking - Critical Knowledge

| Topic | What to Master |
|-------|----------------|
| VNet Integration | Subnet sizing, IP planning, CNI modes |
| Load Balancers | Standard LB, Internal LB, annotations |
| Ingress Options | NGINX, AGIC, Traefik, Contour |
| Egress | Outbound types (loadBalancer, userDefinedRouting, NAT Gateway) |
| Private Clusters | Private endpoint, DNS configuration, bastion access |
| DNS | CoreDNS customization, Azure Private DNS zones |
| Network Policies | Azure NPM vs Calico vs Cilium |
| Service Mesh | Istio, Linkerd, Open Service Mesh |

#### AKS Networking CIDR Planning
```
Example VNet: 10.0.0.0/8

Subnets:
├── AKS Nodes:      10.1.0.0/16    (65,534 IPs)
├── AKS Pods:       10.2.0.0/16    (65,534 IPs) - if using Azure CNI
├── AKS Services:   10.3.0.0/16    (65,534 IPs)
├── App Gateway:    10.4.0.0/24    (254 IPs)
├── Azure Firewall: 10.5.0.0/24    (254 IPs)
├── Bastion:        10.6.0.0/27    (30 IPs)
├── Private Endpoints: 10.7.0.0/24 (254 IPs)
└── Reserved:       10.8.0.0/13    (future growth)

Azure CNI IP Calculation:
Pods per node (default): 30
Nodes: 100
Total pod IPs needed: 30 × 100 = 3,000
Add 20% buffer: 3,600 IPs → /20 subnet minimum
```

### 4.3 AKS Storage

```
Storage Options:
├── Azure Disk
│   ├── Managed disks (Standard HDD, Standard SSD, Premium SSD, Ultra)
│   ├── Disk encryption (PMK, CMK, double encryption)
│   ├── CSI driver configuration
│   └── Snapshot and restore
│
├── Azure Files
│   ├── SMB and NFS protocols
│   ├── Premium vs Standard
│   ├── Private endpoints
│   └── Large file share support
│
├── Azure Blob (CSI)
│   ├── BlobFuse2
│   └── NFS 3.0 protocol
│
└── Best Practices
    ├── Access modes for different workloads
    ├── Reclaim policies
    ├── Volume expansion
    └── Backup strategies (Velero)
```

### 4.4 AKS Security

```
Security Areas:
├── Cluster Security
│   ├── API server authorized IP ranges
│   ├── Private clusters
│   ├── Azure AD authentication
│   ├── Kubernetes RBAC + Azure RBAC
│   ├── Pod Security Standards (restricted, baseline, privileged)
│   └── Defender for Containers
│
├── Workload Security
│   ├── Workload Identity (AAD Pod Identity successor)
│   ├── Secrets management (Key Vault CSI driver)
│   ├── Image scanning (Defender, Trivy)
│   ├── Admission controllers (Gatekeeper/OPA, Kyverno)
│   └── Runtime security (Falco, Sysdig)
│
├── Network Security
│   ├── Network Policies
│   ├── Azure Firewall integration
│   ├── Private Link
│   └── mTLS (service mesh)
│
└── Supply Chain Security
    ├── Image signing (Notation, Cosign)
    ├── SBOM generation
    ├── Vulnerability scanning
    └── Registry security (ACR)
```

### 4.5 AKS Operations & Maintenance

| Task | Details |
|------|---------|
| Upgrades | Control plane upgrades, node pool upgrades, node image upgrades, blue-green upgrades |
| Scaling | Cluster autoscaler, HPA, VPA, KEDA |
| Backup/DR | Velero, Azure Backup for AKS, cross-region DR |
| Maintenance Windows | Planned maintenance, node surge settings |
| Cost Management | Spot nodes, reserved instances, right-sizing, start/stop cluster |

#### AKS CLI Commands
```bash
# Cluster operations
az aks show -g <rg> -n <cluster>
az aks get-credentials -g <rg> -n <cluster>
az aks get-credentials -g <rg> -n <cluster> --admin  # Admin creds

# Upgrades
az aks get-upgrades -g <rg> -n <cluster>
az aks upgrade -g <rg> -n <cluster> --kubernetes-version <version>
az aks nodepool upgrade -g <rg> --cluster-name <cluster> -n <nodepool> --kubernetes-version <version>

# Node pools
az aks nodepool list -g <rg> --cluster-name <cluster>
az aks nodepool add -g <rg> --cluster-name <cluster> -n <nodepool> --node-count 3 --node-vm-size Standard_DS2_v2
az aks nodepool scale -g <rg> --cluster-name <cluster> -n <nodepool> --node-count 5
az aks nodepool delete -g <rg> --cluster-name <cluster> -n <nodepool>

# Diagnostics
az aks get-versions -l eastus
az aks check-acr -g <rg> -n <cluster> --acr <acrname>
az aks kollect -g <rg> -n <cluster> --storage-account <storage>  # Collect diagnostics

# Start/Stop (cost saving)
az aks stop -g <rg> -n <cluster>
az aks start -g <rg> -n <cluster>
```

---

## Phase 5: Jenkins CI/CD for Kubernetes

### 5.1 Jenkins Core Mastery

```
Jenkins Fundamentals:
├── Architecture
│   ├── Controller (master) vs Agents
│   ├── Distributed builds
│   ├── High availability patterns
│   └── Jenkins Configuration as Code (JCasC)
│
├── Pipeline Types
│   ├── Declarative Pipeline (preferred)
│   ├── Scripted Pipeline
│   └── Multibranch Pipeline
│
├── Jenkinsfile Deep Dive
│   ├── stages, steps, post actions
│   ├── parallel execution
│   ├── when conditions
│   ├── input steps
│   ├── environment variables
│   ├── credentials binding
│   ├── shared libraries
│   └── matrix builds
│
└── Plugin Ecosystem
    ├── Kubernetes plugin (dynamic agents)
    ├── Docker Pipeline
    ├── Azure Credentials
    ├── Pipeline Utility Steps
    ├── Blue Ocean
    ├── Warnings Next Generation
    └── SonarQube Scanner
```

### 5.2 Jenkins + Kubernetes Integration

```groovy
// Pod template for dynamic agents
pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.8-openjdk-17
    command: ['sleep', 'infinity']
    resources:
      requests:
        memory: "1Gi"
        cpu: "500m"
  - name: docker
    image: docker:dind
    securityContext:
      privileged: true
  - name: kubectl
    image: bitnami/kubectl
    command: ['sleep', 'infinity']
  - name: az-cli
    image: mcr.microsoft.com/azure-cli
    command: ['sleep', 'infinity']
'''
        }
    }
    stages {
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn clean package'
                }
            }
        }
        stage('Docker Build') {
            steps {
                container('docker') {
                    sh 'docker build -t myapp:${BUILD_NUMBER} .'
                }
            }
        }
        stage('Push to ACR') {
            steps {
                container('az-cli') {
                    sh '''
                    az acr login --name myacr
                    docker push myacr.azurecr.io/myapp:${BUILD_NUMBER}
                    '''
                }
            }
        }
        stage('Deploy to AKS') {
            steps {
                container('kubectl') {
                    sh '''
                    kubectl set image deployment/myapp myapp=myacr.azurecr.io/myapp:${BUILD_NUMBER}
                    kubectl rollout status deployment/myapp
                    '''
                }
            }
        }
    }
}
```

### 5.3 Complete CI/CD Pipeline Patterns for AKS

```
Pipeline Stages (Master These):
├── Source
│   ├── Git checkout
│   ├── Branch strategies (GitFlow, trunk-based)
│   └── Commit validation
│
├── Build
│   ├── Dependency caching
│   ├── Parallel builds
│   ├── Build artifacts
│   └── Build metadata
│
├── Test
│   ├── Unit tests
│   ├── Integration tests
│   ├── Code coverage
│   ├── Static analysis (SonarQube)
│   └── Security scanning (SAST)
│
├── Package
│   ├── Docker multi-stage builds
│   ├── Image tagging strategies
│   ├── Push to ACR
│   └── Image signing
│
├── Security Scan
│   ├── Container scanning (Trivy, Aqua)
│   ├── Dependency scanning
│   ├── License compliance
│   └── SBOM generation
│
├── Deploy
│   ├── Helm chart packaging
│   ├── Kustomize overlays
│   ├── Blue-green deployments
│   ├── Canary deployments
│   └── Progressive rollouts
│
└── Verify
    ├── Smoke tests
    ├── E2E tests
    ├── Performance tests
    └── Rollback triggers
```

### 5.4 Jenkins Pipeline Best Practices

```groovy
// Production-grade Jenkinsfile structure
@Library('shared-libs') _

pipeline {
    agent none  // Dynamic per stage
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 1, unit: 'HOURS')
        disableConcurrentBuilds()
        timestamps()
        ansiColor('xterm')
    }
    
    environment {
        REGISTRY = 'myacr.azurecr.io'
        IMAGE_NAME = 'myapp'
        KUBECONFIG = credentials('aks-kubeconfig')
    }
    
    stages {
        stage('Checkout') {
            agent { kubernetes { inheritFrom 'default' } }
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT_SHORT = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    env.IMAGE_TAG = "${BUILD_NUMBER}-${GIT_COMMIT_SHORT}"
                }
            }
        }
        
        stage('Build & Test') {
            parallel {
                stage('Java Build') {
                    agent { kubernetes { inheritFrom 'maven' } }
                    steps {
                        container('maven') {
                            sh 'mvn clean verify'
                        }
                    }
                    post {
                        always {
                            junit '**/target/surefire-reports/*.xml'
                            jacoco execPattern: '**/target/jacoco.exec'
                        }
                    }
                }
                stage('Security Scan') {
                    agent { kubernetes { inheritFrom 'security' } }
                    steps {
                        container('trivy') {
                            sh 'trivy fs --security-checks vuln,config .'
                        }
                    }
                }
            }
        }
        
        stage('Build & Push Image') {
            agent { kubernetes { inheritFrom 'docker' } }
            steps {
                container('docker') {
                    withCredentials([usernamePassword(credentialsId: 'acr-creds', usernameVariable: 'ACR_USER', passwordVariable: 'ACR_PASS')]) {
                        sh '''
                        docker build -t ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} .
                        docker login ${REGISTRY} -u ${ACR_USER} -p ${ACR_PASS}
                        docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                        '''
                    }
                }
            }
        }
        
        stage('Deploy to Dev') {
            when { branch 'develop' }
            agent { kubernetes { inheritFrom 'kubectl' } }
            steps {
                container('kubectl') {
                    sh '''
                    helm upgrade --install myapp ./charts/myapp \
                        --namespace dev \
                        --set image.tag=${IMAGE_TAG} \
                        --wait --timeout 5m
                    '''
                }
            }
        }
        
        stage('Deploy to Staging') {
            when { branch 'main' }
            agent { kubernetes { inheritFrom 'kubectl' } }
            steps {
                container('kubectl') {
                    sh '''
                    helm upgrade --install myapp ./charts/myapp \
                        --namespace staging \
                        --set image.tag=${IMAGE_TAG} \
                        --wait --timeout 5m
                    '''
                }
            }
        }
        
        stage('Deploy to Production') {
            when { branch 'main' }
            input {
                message "Deploy to production?"
                ok "Deploy"
                submitter "admin,devops-team"
            }
            agent { kubernetes { inheritFrom 'kubectl' } }
            steps {
                container('kubectl') {
                    sh '''
                    helm upgrade --install myapp ./charts/myapp \
                        --namespace production \
                        --set image.tag=${IMAGE_TAG} \
                        --wait --timeout 10m
                    '''
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        failure {
            slackSend(channel: '#devops-alerts', color: 'danger', 
                      message: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
        }
        success {
            slackSend(channel: '#devops-notifications', color: 'good',
                      message: "Build Succeeded: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
        }
    }
}
```

---

## Phase 6: Application Runtime Knowledge

### 6.1 Java Applications

```
Java DevOps Essentials:
├── Build Tools
│   ├── Maven (pom.xml, lifecycle, plugins, profiles)
│   ├── Gradle (build.gradle, tasks, dependencies)
│   └── Dependency management (BOM, version catalogs)
│
├── JVM Tuning for Containers
│   ├── Container-aware JVM flags
│   │   ├── -XX:+UseContainerSupport
│   │   ├── -XX:MaxRAMPercentage=75.0
│   │   ├── -XX:InitialRAMPercentage=50.0
│   │   └── -XX:+UseG1GC or -XX:+UseZGC
│   ├── Memory settings (Heap vs Native memory)
│   ├── CPU limits impact on JVM
│   └── GC tuning for containers
│
├── Common Java Frameworks
│   ├── Spring Boot
│   │   ├── Actuator endpoints (/health, /metrics, /info)
│   │   ├── Externalized configuration
│   │   ├── Profile-based configuration
│   │   └── Graceful shutdown
│   ├── Quarkus (native builds)
│   └── Micronaut
│
├── Observability
│   ├── Micrometer metrics
│   ├── Distributed tracing (OpenTelemetry)
│   ├── Structured logging (logback, log4j2)
│   └── JMX metrics
│
└── Troubleshooting
    ├── Thread dumps (jstack, jcmd)
    ├── Heap dumps (jmap, jcmd)
    ├── Flight Recorder (JFR)
    ├── Memory leak detection
    ├── CPU profiling
    └── Remote debugging
```

#### Java Troubleshooting Commands
```bash
# JVM information
jcmd <pid> VM.version
jcmd <pid> VM.flags
jcmd <pid> VM.system_properties
jcmd <pid> VM.command_line

# Thread dump (for deadlock/performance issues)
jcmd <pid> Thread.print
jstack <pid>
kill -3 <pid>  # Sends SIGQUIT, dumps to stdout

# Heap dump (for memory issues)
jcmd <pid> GC.heap_dump /tmp/heapdump.hprof
jmap -dump:format=b,file=/tmp/heapdump.hprof <pid>

# GC information
jcmd <pid> GC.heap_info
jcmd <pid> GC.class_histogram
jstat -gc <pid> 1000  # GC stats every 1s

# Flight Recorder
jcmd <pid> JFR.start duration=60s filename=/tmp/recording.jfr
jcmd <pid> JFR.dump filename=/tmp/recording.jfr
jcmd <pid> JFR.stop

# In Kubernetes
kubectl exec -it <pod> -- jcmd 1 Thread.print
kubectl exec -it <pod> -- jcmd 1 GC.heap_info
kubectl exec -it <pod> -- jcmd 1 GC.heap_dump /tmp/heap.hprof
kubectl cp <pod>:/tmp/heap.hprof ./heap.hprof
```

### 6.2 Node.js/JavaScript Applications

```
Node.js DevOps Essentials:
├── Package Management
│   ├── npm (package.json, package-lock.json)
│   ├── yarn (yarn.lock)
│   ├── pnpm (pnpm-lock.yaml)
│   └── Dependency security (npm audit)
│
├── Runtime Configuration
│   ├── Node.js memory limits
│   │   └── --max-old-space-size=<MB>
│   ├── Cluster mode (PM2, native cluster)
│   ├── Environment variables
│   └── .env handling (dotenv)
│
├── Common Frameworks
│   ├── Express.js
│   ├── NestJS
│   ├── Fastify
│   └── Next.js (SSR/SSG)
│
├── Build Process
│   ├── TypeScript compilation
│   ├── Bundlers (webpack, esbuild, vite)
│   ├── Tree shaking
│   └── Source maps
│
├── Health & Metrics
│   ├── Health check endpoints
│   ├── Prometheus metrics (prom-client)
│   ├── OpenTelemetry SDK
│   └── Structured logging (pino, winston)
│
└── Troubleshooting
    ├── Node.js debugging (--inspect)
    ├── Memory profiling
    ├── Event loop monitoring
    ├── Heap snapshots
    └── CPU profiling
```

#### Node.js Troubleshooting
```bash
# Debug mode
node --inspect app.js                # Debug on 9229
node --inspect=0.0.0.0:9229 app.js   # Remote debug

# Memory profiling
node --expose-gc --inspect app.js
# In Chrome DevTools: chrome://inspect

# Generate heap snapshot
node --heapsnapshot-signal=SIGUSR2 app.js
kill -USR2 <pid>  # Triggers heap snapshot

# CPU profiling
node --prof app.js
node --prof-process isolate-*.log > profile.txt

# In Kubernetes
kubectl port-forward <pod> 9229:9229
# Then connect Chrome DevTools to localhost:9229

# Common issues
# EVENT LOOP BLOCKED:
# - Check for sync operations
# - Use clinic.js for diagnosis
npm install -g clinic
clinic doctor -- node app.js

# MEMORY LEAK:
# - Use --max-old-space-size
# - Check for growing arrays/objects
# - Use heapdump module
```

### 6.3 Containerization Best Practices

```dockerfile
# Java multi-stage Dockerfile
FROM maven:3.9-eclipse-temurin-21 AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests

FROM eclipse-temurin:21-jre-alpine
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
USER appuser
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget -q --spider http://localhost:8080/actuator/health || exit 1
ENTRYPOINT ["java", "-XX:MaxRAMPercentage=75.0", "-jar", "app.jar"]
```

```dockerfile
# Node.js multi-stage Dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM node:20-alpine
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
USER appuser
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget -q --spider http://localhost:3000/health || exit 1
CMD ["node", "dist/main.js"]
```

---

## Phase 7: Observability & Monitoring

### 7.1 Metrics Stack

```
Metrics Architecture:
├── Collection
│   ├── Prometheus (scraping, PromQL)
│   ├── Azure Monitor / Container Insights
│   ├── Datadog / New Relic / Dynatrace
│   └── OpenTelemetry Collector
│
├── Kubernetes Metrics
│   ├── kube-state-metrics (object states)
│   ├── metrics-server (resource metrics)
│   ├── node-exporter (node-level)
│   └── cAdvisor (container-level)
│
├── Application Metrics
│   ├── RED method (Rate, Errors, Duration)
│   ├── USE method (Utilization, Saturation, Errors)
│   ├── Golden signals
│   └── SLIs/SLOs/SLAs
│
└── Visualization
    ├── Grafana dashboards
    ├── Azure Workbooks
    └── Alerting rules
```

#### Essential PromQL Queries
```promql
# Container CPU usage
sum(rate(container_cpu_usage_seconds_total{container!=""}[5m])) by (pod)

# Container memory usage
sum(container_memory_working_set_bytes{container!=""}) by (pod)

# Request rate
sum(rate(http_requests_total[5m])) by (service)

# Error rate
sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))

# Request latency (p99)
histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))

# Pod restarts
sum(kube_pod_container_status_restarts_total) by (pod)

# Available replicas
kube_deployment_status_replicas_available

# Node CPU
100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

### 7.2 Logging Stack

```
Logging Architecture:
├── Collection
│   ├── Fluent Bit / Fluentd
│   ├── Azure Monitor agent
│   ├── Vector
│   └── Promtail (Loki)
│
├── Storage & Query
│   ├── Azure Log Analytics
│   ├── Elasticsearch
│   ├── Loki
│   └── Splunk
│
├── Best Practices
│   ├── Structured logging (JSON)
│   ├── Correlation IDs
│   ├── Log levels
│   ├── Sensitive data masking
│   └── Log retention policies
│
└── Kubernetes Logs
    ├── Container stdout/stderr
    ├── Application logs
    ├── Audit logs
    └── Control plane logs
```

#### Azure Log Analytics (KQL) Queries
```kusto
// Container logs
ContainerLogV2
| where ContainerName == "myapp"
| where TimeGenerated > ago(1h)
| project TimeGenerated, LogEntry
| order by TimeGenerated desc

// Error logs
ContainerLogV2
| where LogEntry contains "error" or LogEntry contains "exception"
| summarize count() by bin(TimeGenerated, 5m)

// Pod restarts
KubePodInventory
| where TimeGenerated > ago(24h)
| where ClusterName == "mycluster"
| summarize RestartCount = sum(PodRestartCount) by Name, bin(TimeGenerated, 1h)
| where RestartCount > 0

// Resource usage
Perf
| where ObjectName == "K8SContainer"
| where CounterName == "cpuUsageNanoCores"
| summarize avg(CounterValue) by InstanceName, bin(TimeGenerated, 5m)
```

### 7.3 Distributed Tracing

```
Tracing Stack:
├── Standards
│   ├── OpenTelemetry (preferred)
│   ├── W3C Trace Context
│   └── Baggage propagation
│
├── Backends
│   ├── Jaeger
│   ├── Zipkin
│   ├── Azure Application Insights
│   └── Tempo (Grafana)
│
└── Implementation
    ├── Auto-instrumentation
    ├── Manual instrumentation
    ├── Sampling strategies
    └── Span attributes
```

### 7.4 Alerting & On-Call

| Topic | What to Master |
|-------|----------------|
| Alert Design | Actionable alerts, runbooks, severity levels |
| Tools | Azure Alerts, PagerDuty, OpsGenie, Prometheus Alertmanager |
| SLO-based Alerting | Error budgets, burn rate alerts |
| Incident Management | Incident response, post-mortems, blameless culture |

---

## Phase 8: Infrastructure as Code

### 8.1 Terraform for AKS

```hcl
# Example AKS Terraform configuration
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstate"
    container_name       = "tfstate"
    key                  = "aks.tfstate"
  }
}

resource "azurerm_kubernetes_cluster" "aks" {
  name                = var.cluster_name
  location            = var.location
  resource_group_name = var.resource_group_name
  dns_prefix          = var.cluster_name
  kubernetes_version  = var.kubernetes_version

  default_node_pool {
    name                = "system"
    node_count          = var.system_node_count
    vm_size             = var.system_node_size
    vnet_subnet_id      = var.subnet_id
    enable_auto_scaling = true
    min_count           = 2
    max_count           = 5
  }

  identity {
    type = "SystemAssigned"
  }

  network_profile {
    network_plugin    = "azure"
    network_policy    = "calico"
    load_balancer_sku = "standard"
  }

  oms_agent {
    log_analytics_workspace_id = var.log_analytics_workspace_id
  }

  azure_active_directory_role_based_access_control {
    managed                = true
    azure_rbac_enabled     = true
    admin_group_object_ids = var.admin_group_ids
  }

  tags = var.tags
}

resource "azurerm_kubernetes_cluster_node_pool" "user" {
  name                  = "user"
  kubernetes_cluster_id = azurerm_kubernetes_cluster.aks.id
  vm_size              = var.user_node_size
  node_count           = var.user_node_count
  vnet_subnet_id       = var.subnet_id
  enable_auto_scaling  = true
  min_count            = 1
  max_count            = 10

  node_labels = {
    "nodepool" = "user"
  }
}
```

### 8.2 Helm Charts

```
Helm Mastery:
├── Chart Structure
│   ├── Chart.yaml
│   ├── values.yaml
│   ├── templates/
│   ├── helpers (_helpers.tpl)
│   └── dependencies
│
├── Template Functions
│   ├── Built-in functions
│   ├── Sprig library
│   ├── Flow control
│   ├── Named templates
│   └── Template debugging
│
├── Advanced Patterns
│   ├── Library charts
│   ├── Umbrella charts
│   ├── Hooks (pre/post install, upgrade, delete)
│   └── Tests
│
└── Operations
    ├── helm install/upgrade/rollback
    ├── helm diff (plugin)
    ├── helm secrets
    └── Chart repositories
```

#### Helm Commands
```bash
# Basic operations
helm install <release> <chart>
helm upgrade <release> <chart>
helm rollback <release> <revision>
helm uninstall <release>

# With options
helm install myapp ./charts/myapp \
  --namespace production \
  --create-namespace \
  -f values-prod.yaml \
  --set image.tag=v1.0.0 \
  --wait --timeout 10m

# Debugging
helm template <release> <chart>        # Render templates locally
helm install --dry-run --debug        # Simulate install
helm get manifest <release>           # Get deployed manifest
helm get values <release>             # Get deployed values
helm history <release>                # Release history

# Repository management
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo nginx
```

### 8.3 Kustomize

```yaml
# kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- deployment.yaml
- service.yaml
- configmap.yaml

namespace: production

namePrefix: prod-

commonLabels:
  environment: production
  app: myapp

images:
- name: myapp
  newName: myacr.azurecr.io/myapp
  newTag: v1.0.0

configMapGenerator:
- name: app-config
  files:
  - config.yaml

secretGenerator:
- name: app-secrets
  literals:
  - API_KEY=secret123

patches:
- path: patches/replica-count.yaml
- path: patches/resources.yaml
```

---

## Phase 9: Security Deep Dive

### 9.1 Supply Chain Security

```
Security Pipeline:
├── Source Code
│   ├── Secret scanning (gitleaks, trufflehog)
│   ├── SAST (SonarQube, Semgrep)
│   └── Pre-commit hooks
│
├── Dependencies
│   ├── SCA (Dependabot, Snyk, OWASP DC)
│   ├── License compliance
│   └── SBOM (Syft, Trivy)
│
├── Build
│   ├── Reproducible builds
│   ├── Signed commits
│   └── Build provenance (SLSA)
│
├── Container Images
│   ├── Base image selection
│   ├── Vulnerability scanning
│   ├── Image signing (Cosign, Notation)
│   └── Distroless/minimal images
│
└── Runtime
    ├── Admission control
    ├── Runtime scanning
    └── Network policies
```

### 9.2 Kubernetes Security

```
K8s Security Layers:
├── Authentication
│   ├── Azure AD integration
│   ├── Service accounts
│   ├── OIDC providers
│   └── Certificate-based auth
│
├── Authorization
│   ├── RBAC (Role, ClusterRole)
│   ├── Azure RBAC for K8s
│   ├── Aggregated ClusterRoles
│   └── Audit logging
│
├── Admission Control
│   ├── Built-in controllers
│   ├── OPA/Gatekeeper
│   ├── Kyverno
│   └── Pod Security Admission
│
├── Pod Security
│   ├── Security contexts
│   ├── Read-only root filesystem
│   ├── Non-root containers
│   ├── Capability dropping
│   └── Seccomp profiles
│
└── Network Security
    ├── Network policies
    ├── Service mesh mTLS
    ├── Egress controls
    └── Private clusters
```

#### Secure Pod Example
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: myapp:latest
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
          - ALL
    resources:
      limits:
        memory: "512Mi"
        cpu: "500m"
      requests:
        memory: "256Mi"
        cpu: "250m"
    volumeMounts:
    - name: tmp
      mountPath: /tmp
  volumes:
  - name: tmp
    emptyDir: {}
  automountServiceAccountToken: false
```

---

## Phase 10: Troubleshooting Masterclass

### 10.1 Systematic Troubleshooting Framework

```
Troubleshooting Methodology:
├── 1. Identify Symptoms
│   ├── What is the user-visible impact?
│   ├── When did it start?
│   ├── What changed recently?
│   └── Scope: cluster-wide or specific workload?
│
├── 2. Gather Data
│   ├── kubectl describe/logs/events
│   ├── Metrics and dashboards
│   ├── Distributed traces
│   └── Node/pod status
│
├── 3. Form Hypothesis
│   ├── Based on symptoms and data
│   ├── Consider common failure modes
│   └── Check recent changes
│
├── 4. Test & Validate
│   ├── Reproduce if possible
│   ├── Isolate variables
│   └── Verify fixes
│
└── 5. Document & Prevent
    ├── Update runbooks
    ├── Add monitoring
    └── Post-mortem if severe
```

### 10.2 Common Issues & Solutions Matrix

| Category | Symptom | Investigation Commands | Likely Causes |
|----------|---------|----------------------|---------------|
| **Pod Scheduling** | Pod stuck in Pending | `kubectl describe pod`, `kubectl get events`, `kubectl describe nodes` | Resource constraints, node affinity, taints, PVC binding |
| **Pod Startup** | CrashLoopBackOff | `kubectl logs --previous`, `kubectl describe pod` | App crash, missing config, health check fail |
| **Pod Startup** | ImagePullBackOff | `kubectl describe pod`, check ACR | Image not found, auth failure, network |
| **Networking** | Service unreachable | `kubectl get endpoints`, `kubectl exec -- nslookup` | No ready endpoints, wrong selector, network policy |
| **Networking** | DNS resolution fails | `kubectl exec -- nslookup kubernetes`, check CoreDNS | CoreDNS issues, network policy |
| **Storage** | PVC stuck Pending | `kubectl describe pvc`, `kubectl get sc` | No matching PV, storage class issues |
| **Resources** | OOMKilled | `kubectl describe pod`, check limits | Memory limits too low, memory leak |
| **Resources** | CPU throttling | Check metrics, `kubectl top` | CPU limits too low, noisy neighbors |
| **Node** | Node NotReady | `kubectl describe node`, check kubelet logs | Disk pressure, memory pressure, network |
| **Ingress** | 502/503 errors | Check ingress controller logs, backend pods | Backend not ready, health checks |

### 10.3 Advanced Debugging Tools

```bash
# Debug containers
kubectl debug -it <pod> --image=nicolaka/netshoot --target=<container>

# Network debugging
kubectl run netshoot --rm -it --image=nicolaka/netshoot -- /bin/bash
# Inside: tcpdump, dig, curl, nmap, iperf, etc.

# Node debugging
kubectl debug node/<node-name> -it --image=ubuntu

# Java debugging
kubectl exec -it <pod> -- jcmd 1 VM.flags
kubectl exec -it <pod> -- jcmd 1 GC.heap_info
kubectl exec -it <pod> -- jcmd 1 Thread.print

# Node.js debugging
kubectl exec -it <pod> -- node --inspect=0.0.0.0:9229

# Resource debugging
kubectl top pods --containers
kubectl top nodes

# Events timeline
kubectl get events --sort-by='.lastTimestamp' -A
```

### 10.4 AKS-Specific Troubleshooting

```
AKS Issues:
├── Control Plane
│   ├── Check AKS health in Azure Portal
│   ├── Review diagnostic logs
│   ├── az aks show/get-credentials
│   └── Check API server availability
│
├── Networking
│   ├── NSG rules
│   ├── Azure CNI IP exhaustion
│   ├── DNS resolution (Azure Private DNS)
│   ├── Load balancer health probes
│   └── UDR configurations
│
├── Identity
│   ├── Managed identity permissions
│   ├── Workload identity federation
│   ├── AAD authentication issues
│   └── ACR authentication
│
└── Node Issues
    ├── VMSS instances
    ├── Node image updates
    ├── Node pool scaling
    └── Extension issues
```

---

## Phase 11: GitOps & Advanced Deployment

### 11.1 GitOps with Flux v2

```
Flux Components:
├── Source Controller (Git, Helm, OCI)
├── Kustomize Controller
├── Helm Controller
├── Notification Controller
└── Image Automation Controller

Implementation:
├── Repository structure
├── Kustomization manifests
├── HelmRelease CRDs
├── Multi-cluster patterns
└── Secret management (SOPS, Sealed Secrets)
```

### 11.2 Progressive Delivery

```
Strategies:
├── Blue-Green Deployments
│   ├── Full environment swap
│   └── Instant rollback capability
│
├── Canary Deployments
│   ├── Traffic splitting
│   ├── Metrics-based promotion
│   └── Flagger automation
│
├── A/B Testing
│   ├── Feature flags
│   └── Header-based routing
│
└── Tools
    ├── Argo Rollouts
    ├── Flagger
    └── Istio traffic management
```

---

## Phase 12: Reliability Engineering

### 12.1 SRE Practices

```
Reliability Concepts:
├── SLIs, SLOs, SLAs
│   ├── Defining meaningful indicators
│   ├── Setting realistic objectives
│   └── Error budget policies
│
├── Capacity Planning
│   ├── Load testing (k6, Locust, JMeter)
│   ├── Right-sizing resources
│   ├── HPA tuning
│   └── Cluster autoscaler optimization
│
├── Chaos Engineering
│   ├── Chaos Mesh
│   ├── Litmus
│   ├── Azure Chaos Studio
│   └── Failure injection patterns
│
└── Disaster Recovery
    ├── Backup strategies (Velero)
    ├── Multi-region deployments
    ├── RTO/RPO planning
    └── Runbook automation
```

---

## Phase 13: Essential Skills & Tools Reference

### 13.1 Must-Know CLI Tools

| Tool | Purpose |
|------|---------|
| `kubectl` | Kubernetes operations |
| `helm` | Package management |
| `kustomize` | Configuration management |
| `az` | Azure CLI |
| `terraform` | Infrastructure as Code |
| `docker/podman` | Container operations |
| `crictl` | Container runtime debugging |
| `k9s` | Terminal UI for K8s |
| `stern` | Multi-pod log tailing |
| `kubectx/kubens` | Context/namespace switching |
| `jq/yq` | JSON/YAML processing |
| `curl/httpie` | API testing |

### 13.2 Knowledge Certifications Path

```
Recommended Certifications:
├── Kubernetes
│   ├── CKA (Certified Kubernetes Administrator)
│   ├── CKAD (Certified Kubernetes Application Developer)
│   └── CKS (Certified Kubernetes Security Specialist)
│
├── Azure
│   ├── AZ-104 (Azure Administrator)
│   ├── AZ-400 (Azure DevOps Engineer Expert)
│   └── AZ-305 (Azure Solutions Architect)
│
└── Other
    ├── HashiCorp Terraform Associate
    ├── LPIC-1 / RHCSA (Linux)
    └── AWS Solutions Architect (for multi-cloud)
```

### 13.3 Daily BAU Checklist

```
Morning Checks:
□ Cluster health (nodes, control plane)
□ Critical workload status
□ Recent deployments
□ Alert dashboard
□ Resource utilization trends
□ Certificate expiration
□ Backup job status

Weekly Reviews:
□ Security scan results
□ Cost analysis
□ Capacity planning
□ Incident review
□ Changelog review
□ Dependency updates
```

---

## 90-Day Intensive Plan

| Week | Focus Area | Deliverable |
|------|-----------|-------------|
| 1-2 | Linux fundamentals, CLI mastery | Complete all Linux sections, set up lab environment |
| 3-4 | Networking deep dive | Build network troubleshooting toolkit, practice tcpdump/wireshark |
| 5-6 | Kubernetes fundamentals | Deploy complex multi-tier app, practice all kubectl commands |
| 7-8 | AKS specifics | Production-grade AKS cluster with Terraform |
| 9-10 | Jenkins pipelines | Full CI/CD for Java + Node.js apps |
| 11-12 | Observability | Complete monitoring stack (Prometheus, Grafana, Loki) |
| 13-14 | Security | Hardened cluster with policy enforcement |
| 15-16 | Troubleshooting | Document 30+ troubleshooting scenarios with solutions |
| 17-18 | DR/Reliability | Backup, restore, chaos testing exercises |
| 19-20 | Advanced topics | GitOps, service mesh, progressive delivery |
| 21-22 | Certifications prep | CKA exam preparation |
| 23-24 | Integration | End-to-end project combining all skills |

---

## Learning Resources

### Official Documentation (Read Cover to Cover)
- Kubernetes.io - Concepts, Tasks, Tutorials
- Microsoft Learn - AKS Learning Paths
- Jenkins.io - Pipeline Documentation
- Linux man pages - Commands reference

### Hands-On Practice
1. **Set up a personal AKS cluster** - Practice everything
2. **Break things intentionally** - Learn failure modes
3. **Build full CI/CD pipelines** - End to end
4. **Contribute to open source** - Real-world experience

### Books
- "The Linux Command Line" by William Shotts
- "Kubernetes Up & Running" by Brendan Burns
- "Production Kubernetes" by Josh Rosso
- "Site Reliability Engineering" by Google
- "The Phoenix Project" for DevOps culture
- "TCP/IP Illustrated" by W. Richard Stevens

### Practice Platforms
- Katacoda / Killercoda - Interactive scenarios
- Linux Academy / A Cloud Guru
- CKA/CKAD exam simulators
- Azure hands-on labs

---

> **Remember:** This roadmap is comprehensive but not exhaustive - the field evolves constantly. The key is to build deep fundamentals while staying current with new developments. Focus on hands-on practice over passive learning. Break things, fix them, and document everything.

**Version:** 1.0  
**Last Updated:** February 2026
