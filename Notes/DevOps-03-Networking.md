# The Complete DevOps Engineer's Reference Guide
## Part 3: Networking Deep Dive

---

# Chapter 8: Networking Fundamentals

## 8.1 The OSI Model and TCP/IP

Understanding network layers is essential for troubleshooting. When something doesn't work, you need to know which layer is failing.

### The OSI Model (7 Layers)

```
OSI Model - The Theoretical Framework
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  Layer 7: Application                                               │
│  ├── What it does: Provides services directly to user applications  │
│  ├── Protocols: HTTP, HTTPS, FTP, SMTP, DNS, SSH, Telnet           │
│  ├── Data unit: Data                                                │
│  └── Example: Web browser requesting a webpage                      │
│                                                                     │
│  Layer 6: Presentation                                              │
│  ├── What it does: Data translation, encryption, compression        │
│  ├── Protocols: SSL/TLS, JPEG, GIF, ASCII, EBCDIC                  │
│  ├── Data unit: Data                                                │
│  └── Example: TLS encrypting data before transmission               │
│                                                                     │
│  Layer 5: Session                                                   │
│  ├── What it does: Establishes, manages, terminates connections     │
│  ├── Protocols: NetBIOS, RPC, SQL sessions                         │
│  ├── Data unit: Data                                                │
│  └── Example: Maintaining login session to a server                 │
│                                                                     │
│  Layer 4: Transport                                                 │
│  ├── What it does: End-to-end communication, reliability            │
│  ├── Protocols: TCP (reliable), UDP (fast)                         │
│  ├── Data unit: Segment (TCP) / Datagram (UDP)                     │
│  ├── Addressing: Port numbers (0-65535)                            │
│  └── Example: TCP ensuring all data arrives correctly               │
│                                                                     │
│  Layer 3: Network                                                   │
│  ├── What it does: Routing, logical addressing                      │
│  ├── Protocols: IP, ICMP, ARP, OSPF, BGP                           │
│  ├── Data unit: Packet                                              │
│  ├── Devices: Routers, Layer 3 switches                            │
│  ├── Addressing: IP addresses                                       │
│  └── Example: Router deciding which path to send packet             │
│                                                                     │
│  Layer 2: Data Link                                                 │
│  ├── What it does: Node-to-node data transfer, error detection      │
│  ├── Protocols: Ethernet, Wi-Fi (802.11), PPP                      │
│  ├── Data unit: Frame                                               │
│  ├── Devices: Switches, bridges                                     │
│  ├── Addressing: MAC addresses (48-bit, e.g., 00:1A:2B:3C:4D:5E)   │
│  └── Example: Switch forwarding frame based on MAC address          │
│                                                                     │
│  Layer 1: Physical                                                  │
│  ├── What it does: Physical transmission of bits                    │
│  ├── Protocols: Ethernet physical, USB, RS-232                     │
│  ├── Data unit: Bits                                                │
│  ├── Devices: Hubs, repeaters, cables, NICs                        │
│  └── Example: Electrical signals on copper cable                    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

Mnemonic to remember (top to bottom):
"All People Seem To Need Data Processing"
Application, Presentation, Session, Transport, Network, Data Link, Physical

Or bottom to top:
"Please Do Not Throw Sausage Pizza Away"
Physical, Data Link, Network, Transport, Session, Presentation, Application
```

### TCP/IP Model (4 Layers)

The TCP/IP model is the practical implementation used in real networks.

```
TCP/IP Model - The Practical Implementation
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  TCP/IP Layer      OSI Layers     Protocols                        │
│  ─────────────────────────────────────────────────────────────────  │
│                                                                     │
│  Application       7, 6, 5        HTTP, HTTPS, FTP, SSH, DNS,      │
│                                   SMTP, POP3, IMAP, SNMP, DHCP     │
│                                                                     │
│  Transport         4              TCP, UDP                          │
│                                                                     │
│  Internet          3              IP, ICMP, ARP, IGMP               │
│                                                                     │
│  Network Access    2, 1           Ethernet, Wi-Fi, PPP              │
│  (Link Layer)                                                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

Data Encapsulation Process:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  Application Layer:   [      DATA      ]                            │
│                              │                                      │
│                              ▼                                      │
│  Transport Layer:     [TCP/UDP][  DATA  ]                          │
│                       (Port numbers added)                          │
│                              │                                      │
│                              ▼                                      │
│  Internet Layer:      [IP][TCP][  DATA  ]                          │
│                       (IP addresses added)                          │
│                              │                                      │
│                              ▼                                      │
│  Link Layer:    [ETH][IP][TCP][  DATA  ][FCS]                      │
│                 (MAC addresses + frame check sequence)              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 8.2 IP Addressing

### IPv4 Addressing

An IPv4 address is a 32-bit number, written as four octets (bytes) separated by dots.

```
IPv4 Address Structure:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  Example: 192.168.1.100                                            │
│                                                                     │
│  Decimal:  192      .  168      .  1        .  100                 │
│  Binary:   11000000    10101000    00000001    01100100            │
│                                                                     │
│  Each octet: 0-255 (8 bits = 2^8 = 256 values)                     │
│  Total: 32 bits                                                     │
│                                                                     │
│  Range: 0.0.0.0 to 255.255.255.255                                 │
│  Total addresses: 2^32 = 4,294,967,296 (about 4.3 billion)         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

Network and Host Portions:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  An IP address has two parts:                                       │
│  1. Network portion: Identifies the network                         │
│  2. Host portion: Identifies the device on that network            │
│                                                                     │
│  Example: 192.168.1.100 with subnet mask 255.255.255.0 (/24)       │
│                                                                     │
│  IP Address:    192.168.1  .100                                    │
│                 └────┬────┘ └─┬─┘                                  │
│                  Network    Host                                    │
│                                                                     │
│  All devices with 192.168.1.x are on the same network              │
│  They can communicate directly (Layer 2)                            │
│  To reach 192.168.2.x, they need a router (Layer 3)                │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Subnet Masks and CIDR

```
Subnet Mask:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  A subnet mask identifies which bits are network vs host            │
│                                                                     │
│  Subnet Mask: 255.255.255.0                                        │
│  Binary:      11111111.11111111.11111111.00000000                  │
│               └────────── Network ─────────┘└─ Host ─┘              │
│                                                                     │
│  The 1s mark the network portion                                    │
│  The 0s mark the host portion                                       │
│                                                                     │
│  To find the network address:                                       │
│  IP Address AND Subnet Mask = Network Address                       │
│                                                                     │
│  Example:                                                           │
│  IP:      192.168.1.100  = 11000000.10101000.00000001.01100100     │
│  Mask:    255.255.255.0  = 11111111.11111111.11111111.00000000     │
│  AND:     192.168.1.0    = 11000000.10101000.00000001.00000000     │
│                                                                     │
│  Network address: 192.168.1.0                                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

CIDR Notation (Classless Inter-Domain Routing):
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  CIDR uses /N to indicate the number of network bits               │
│                                                                     │
│  /24 = 24 network bits = 255.255.255.0                             │
│  /16 = 16 network bits = 255.255.0.0                               │
│  /8  = 8 network bits  = 255.0.0.0                                 │
│                                                                     │
│  Common CIDR blocks:                                                │
│  ┌───────┬─────────────────┬──────────────┬──────────────────────┐ │
│  │ CIDR  │ Subnet Mask     │ # of IPs     │ Usable IPs           │ │
│  ├───────┼─────────────────┼──────────────┼──────────────────────┤ │
│  │ /32   │ 255.255.255.255 │ 1            │ 1 (single host)      │ │
│  │ /31   │ 255.255.255.254 │ 2            │ 2 (point-to-point)   │ │
│  │ /30   │ 255.255.255.252 │ 4            │ 2                    │ │
│  │ /29   │ 255.255.255.248 │ 8            │ 6                    │ │
│  │ /28   │ 255.255.255.240 │ 16           │ 14                   │ │
│  │ /27   │ 255.255.255.224 │ 32           │ 30                   │ │
│  │ /26   │ 255.255.255.192 │ 64           │ 62                   │ │
│  │ /25   │ 255.255.255.128 │ 128          │ 126                  │ │
│  │ /24   │ 255.255.255.0   │ 256          │ 254                  │ │
│  │ /23   │ 255.255.254.0   │ 512          │ 510                  │ │
│  │ /22   │ 255.255.252.0   │ 1,024        │ 1,022                │ │
│  │ /21   │ 255.255.248.0   │ 2,048        │ 2,046                │ │
│  │ /20   │ 255.255.240.0   │ 4,096        │ 4,094                │ │
│  │ /16   │ 255.255.0.0     │ 65,536       │ 65,534               │ │
│  │ /8    │ 255.0.0.0       │ 16,777,216   │ 16,777,214           │ │
│  └───────┴─────────────────┴──────────────┴──────────────────────┘ │
│                                                                     │
│  Why "Usable" is less: Network address and broadcast are reserved  │
│  - Network address: All host bits = 0 (e.g., 192.168.1.0)          │
│  - Broadcast: All host bits = 1 (e.g., 192.168.1.255)              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

Subnet Calculation Example:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  Given: 10.20.30.0/26                                              │
│                                                                     │
│  /26 = 26 network bits, 6 host bits                                │
│  Host addresses: 2^6 = 64 total, 62 usable                         │
│                                                                     │
│  Subnet mask: 255.255.255.192                                      │
│  (192 = 11000000 in binary, meaning first 2 bits of last octet)    │
│                                                                     │
│  Network:   10.20.30.0                                             │
│  First IP:  10.20.30.1                                             │
│  Last IP:   10.20.30.62                                            │
│  Broadcast: 10.20.30.63                                            │
│  Next network: 10.20.30.64                                         │
│                                                                     │
│  The /26 divides a /24 into 4 subnets:                             │
│  10.20.30.0/26   (0-63)                                            │
│  10.20.30.64/26  (64-127)                                          │
│  10.20.30.128/26 (128-191)                                         │
│  10.20.30.192/26 (192-255)                                         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Special IP Addresses

```
Reserved IP Address Ranges:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  Private IP Ranges (RFC 1918) - Not routable on internet:          │
│  ┌─────────────────────┬───────────────────────────────────────────┐
│  │ Range               │ CIDR Notation                             │
│  ├─────────────────────┼───────────────────────────────────────────┤
│  │ 10.0.0.0 -          │ 10.0.0.0/8                                │
│  │ 10.255.255.255      │ (16 million addresses)                    │
│  ├─────────────────────┼───────────────────────────────────────────┤
│  │ 172.16.0.0 -        │ 172.16.0.0/12                             │
│  │ 172.31.255.255      │ (1 million addresses)                     │
│  ├─────────────────────┼───────────────────────────────────────────┤
│  │ 192.168.0.0 -       │ 192.168.0.0/16                            │
│  │ 192.168.255.255     │ (65,536 addresses)                        │
│  └─────────────────────┴───────────────────────────────────────────┘
│                                                                     │
│  Loopback:                                                          │
│  127.0.0.0/8 (127.0.0.1 is localhost)                              │
│  Traffic never leaves the machine                                   │
│                                                                     │
│  Link-Local (APIPA):                                                │
│  169.254.0.0/16                                                     │
│  Auto-assigned when DHCP fails                                      │
│                                                                     │
│  Multicast:                                                         │
│  224.0.0.0/4 (224.0.0.0 - 239.255.255.255)                         │
│  One-to-many communication                                          │
│                                                                     │
│  Broadcast:                                                         │
│  255.255.255.255 - Local broadcast                                  │
│  x.x.x.255 (for /24) - Directed broadcast                          │
│                                                                     │
│  Documentation/Examples (RFC 5737):                                 │
│  192.0.2.0/24 (TEST-NET-1)                                         │
│  198.51.100.0/24 (TEST-NET-2)                                      │
│  203.0.113.0/24 (TEST-NET-3)                                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### IPv6 Basics

```
IPv6 Address Format:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  128-bit address, written as 8 groups of 4 hexadecimal digits      │
│                                                                     │
│  Full form: 2001:0db8:0000:0000:0000:ff00:0042:8329                 │
│                                                                     │
│  Compressed form (remove leading zeros, use :: for consecutive 0s):│
│  2001:db8::ff00:42:8329                                            │
│                                                                     │
│  Rules for compression:                                             │
│  1. Leading zeros in each group can be removed                      │
│     0db8 → db8, 0000 → 0, 0042 → 42                                │
│  2. One sequence of consecutive all-zero groups can be ::          │
│     :0000:0000:0000: → ::                                          │
│  3. :: can only be used once per address                           │
│                                                                     │
│  Special addresses:                                                 │
│  ::1         - Loopback (like 127.0.0.1)                           │
│  ::          - All zeros (unspecified)                             │
│  fe80::/10   - Link-local (auto-configured)                        │
│  fc00::/7    - Unique local (like private IPv4)                    │
│  2000::/3    - Global unicast (public internet)                    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 8.3 TCP and UDP

### TCP (Transmission Control Protocol)

TCP provides reliable, ordered delivery of data.

```
TCP Characteristics:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  ✓ Connection-oriented (establishes connection before sending)     │
│  ✓ Reliable (guarantees delivery, retransmits lost packets)        │
│  ✓ Ordered (data arrives in correct sequence)                      │
│  ✓ Error-checked (checksums verify data integrity)                 │
│  ✓ Flow control (receiver can slow down sender)                    │
│  ✓ Congestion control (adapts to network conditions)               │
│                                                                     │
│  Used for: HTTP/HTTPS, SSH, FTP, SMTP, databases, APIs             │
│  (Anything where reliability matters more than speed)              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

TCP Three-Way Handshake:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  Client                                              Server         │
│    │                                                    │           │
│    │  ──────────── SYN (seq=x) ────────────────────▶   │           │
│    │         "I want to connect"                        │           │
│    │                                                    │           │
│    │  ◀────────── SYN-ACK (seq=y, ack=x+1) ─────────   │           │
│    │         "OK, I'm ready"                            │           │
│    │                                                    │           │
│    │  ──────────── ACK (ack=y+1) ───────────────────▶  │           │
│    │         "Let's go"                                 │           │
│    │                                                    │           │
│    │  ◀═══════════ DATA TRANSFER ══════════════════▶   │           │
│    │                                                    │           │
│                                                                     │
│  SYN = Synchronize (initiate connection)                           │
│  ACK = Acknowledge (confirm receipt)                               │
│  seq = Sequence number                                             │
│  ack = Acknowledgment number                                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

TCP Connection Termination (Four-Way):
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  Client                                              Server         │
│    │                                                    │           │
│    │  ──────────── FIN ─────────────────────────────▶  │           │
│    │         "I'm done sending"                         │           │
│    │                                                    │           │
│    │  ◀────────── ACK ──────────────────────────────   │           │
│    │         "OK, got it"                               │           │
│    │                                                    │           │
│    │  ◀────────── FIN ──────────────────────────────   │           │
│    │         "I'm done too"                             │           │
│    │                                                    │           │
│    │  ──────────── ACK ─────────────────────────────▶  │           │
│    │         "Goodbye"                                  │           │
│    │                                                    │           │
│    │  [TIME_WAIT]                                       │           │
│    │         Waits to ensure final ACK received         │           │
│    │                                                    │           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

TCP States:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  LISTEN       : Server waiting for connections                      │
│  SYN_SENT     : Client sent SYN, waiting for SYN-ACK               │
│  SYN_RECEIVED : Server received SYN, sent SYN-ACK, waiting for ACK │
│  ESTABLISHED  : Connection active, data can flow                    │
│  FIN_WAIT_1   : Sent FIN, waiting for ACK                          │
│  FIN_WAIT_2   : Received ACK for FIN, waiting for FIN from peer    │
│  CLOSE_WAIT   : Received FIN, sent ACK, waiting to send FIN        │
│  CLOSING      : Both sides sent FIN simultaneously                 │
│  LAST_ACK     : Sent FIN, waiting for final ACK                    │
│  TIME_WAIT    : Waiting to ensure remote received final ACK        │
│  CLOSED       : Connection terminated                              │
│                                                                     │
│  Common issue: Many TIME_WAIT connections                          │
│  Solution: Enable tcp_tw_reuse in sysctl                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### UDP (User Datagram Protocol)

UDP provides fast, connectionless communication.

```
UDP Characteristics:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  ✓ Connectionless (no handshake, just sends)                       │
│  ✓ Fast (no overhead of connection management)                     │
│  ✓ Low latency (no waiting for acknowledgments)                    │
│  ✗ Unreliable (no guarantee of delivery)                           │
│  ✗ Unordered (packets may arrive out of order)                     │
│  ✗ No flow control                                                 │
│                                                                     │
│  Used for: DNS, DHCP, NTP, streaming, VoIP, gaming                 │
│  (Anything where speed matters more than reliability)              │
│                                                                     │
│  Note: Applications using UDP often implement their own            │
│  reliability mechanisms if needed                                   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

TCP vs UDP Summary:
┌────────────────────────────────────────────────────────────────────┐
│                                                                    │
│  Feature              TCP                  UDP                     │
│  ──────────────────────────────────────────────────────────────── │
│  Connection           Yes (3-way)          No                     │
│  Reliability          Guaranteed           Best-effort            │
│  Ordering             Yes                  No                     │
│  Speed                Slower               Faster                 │
│  Overhead             Higher               Lower                  │
│  Error recovery       Yes                  No                     │
│  Use case             Web, email, files    Streaming, DNS, games │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

## 8.4 Common Ports

Every network service listens on a port number. Knowing common ports is essential.

```
Well-Known Ports (0-1023) - Require root to bind:
┌──────┬─────────────────┬──────────────────────────────────────────┐
│ Port │ Service         │ Description                               │
├──────┼─────────────────┼──────────────────────────────────────────┤
│ 20   │ FTP-data        │ FTP data transfer                        │
│ 21   │ FTP             │ FTP control/commands                     │
│ 22   │ SSH             │ Secure Shell                             │
│ 23   │ Telnet          │ Unencrypted remote access (avoid!)       │
│ 25   │ SMTP            │ Simple Mail Transfer Protocol            │
│ 53   │ DNS             │ Domain Name System (TCP and UDP)         │
│ 67   │ DHCP server     │ Dynamic Host Configuration Protocol      │
│ 68   │ DHCP client     │ DHCP client                              │
│ 69   │ TFTP            │ Trivial File Transfer Protocol (UDP)     │
│ 80   │ HTTP            │ Hypertext Transfer Protocol              │
│ 110  │ POP3            │ Post Office Protocol v3                  │
│ 123  │ NTP             │ Network Time Protocol (UDP)              │
│ 143  │ IMAP            │ Internet Message Access Protocol         │
│ 161  │ SNMP            │ Simple Network Management Protocol       │
│ 162  │ SNMP Trap       │ SNMP notifications                       │
│ 389  │ LDAP            │ Lightweight Directory Access Protocol    │
│ 443  │ HTTPS           │ HTTP Secure (TLS)                        │
│ 445  │ SMB             │ Server Message Block (Windows shares)    │
│ 465  │ SMTPS           │ SMTP over SSL                            │
│ 514  │ Syslog          │ System logging (UDP)                     │
│ 587  │ SMTP            │ SMTP submission (with auth)              │
│ 636  │ LDAPS           │ LDAP over SSL                            │
│ 993  │ IMAPS           │ IMAP over SSL                            │
│ 995  │ POP3S           │ POP3 over SSL                            │
└──────┴─────────────────┴──────────────────────────────────────────┘

Registered Ports (1024-49151) - Common services:
┌──────┬─────────────────┬──────────────────────────────────────────┐
│ Port │ Service         │ Description                               │
├──────┼─────────────────┼──────────────────────────────────────────┤
│ 1433 │ MSSQL           │ Microsoft SQL Server                     │
│ 1521 │ Oracle          │ Oracle database                          │
│ 2049 │ NFS             │ Network File System                      │
│ 2379 │ etcd client     │ Kubernetes etcd                          │
│ 2380 │ etcd peer       │ etcd cluster communication               │
│ 3000 │ Various         │ Node.js apps, Grafana                    │
│ 3306 │ MySQL           │ MySQL/MariaDB database                   │
│ 3389 │ RDP             │ Remote Desktop Protocol                  │
│ 5432 │ PostgreSQL      │ PostgreSQL database                      │
│ 5672 │ AMQP            │ RabbitMQ                                 │
│ 5900 │ VNC             │ Virtual Network Computing                │
│ 6379 │ Redis           │ Redis in-memory database                 │
│ 6443 │ Kubernetes API  │ Kubernetes API server                    │
│ 8080 │ HTTP-alt        │ Alternative HTTP (Tomcat, proxy)         │
│ 8443 │ HTTPS-alt       │ Alternative HTTPS                        │
│ 9090 │ Prometheus      │ Prometheus server                        │
│ 9200 │ Elasticsearch   │ Elasticsearch HTTP                       │
│ 9300 │ Elasticsearch   │ Elasticsearch transport                  │
│ 10250│ Kubelet         │ Kubernetes Kubelet API                   │
│ 27017│ MongoDB         │ MongoDB database                         │
└──────┴─────────────────┴──────────────────────────────────────────┘

Ephemeral Ports (49152-65535):
Used for client-side connections (source port).
Range can be configured: /proc/sys/net/ipv4/ip_local_port_range
```

---

# Chapter 9: DNS (Domain Name System)

## 9.1 How DNS Works

DNS translates human-readable domain names into IP addresses.

```
DNS Resolution Process:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  You type: www.example.com                                         │
│                                                                     │
│  1. Browser Cache                                                   │
│     └── Check if domain was recently resolved                      │
│                                                                     │
│  2. Operating System Cache                                          │
│     └── Check /etc/hosts and OS DNS cache                          │
│                                                                     │
│  3. Resolver (Recursive DNS Server)                                 │
│     └── Usually your ISP or configured DNS (8.8.8.8)               │
│                                                                     │
│  4. Root Name Servers (.)                                          │
│     └── "I don't know www.example.com, but .com is at..."          │
│                                                                     │
│  5. TLD Name Servers (.com, .org, .net)                            │
│     └── "I don't know www.example.com, but example.com is at..."   │
│                                                                     │
│  6. Authoritative Name Server (example.com)                        │
│     └── "www.example.com is 93.184.216.34"                         │
│                                                                     │
│  7. Response cached at each level                                   │
│     └── TTL (Time To Live) determines cache duration               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

Visual Flow:
┌──────────┐      ┌──────────┐      ┌──────────┐      ┌──────────┐
│   Your   │      │ Resolver │      │   Root   │      │   TLD    │
│ Computer │─────▶│  (ISP)   │─────▶│    .     │─────▶│   .com   │
└──────────┘      └──────────┘      └──────────┘      └──────────┘
                       │                                    │
                       │                              ┌─────▼─────┐
                       │                              │   Auth    │
                       │◀─────────────────────────────│example.com│
                       │         93.184.216.34        └───────────┘
```

## 9.2 DNS Record Types

```
DNS Record Types:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  A Record (Address)                                                 │
│  ├── Maps hostname to IPv4 address                                  │
│  └── Example: www.example.com → 93.184.216.34                      │
│                                                                     │
│  AAAA Record (IPv6 Address)                                         │
│  ├── Maps hostname to IPv6 address                                  │
│  └── Example: www.example.com → 2606:2800:220:1:248:1893:25c8:1946 │
│                                                                     │
│  CNAME Record (Canonical Name)                                      │
│  ├── Alias that points to another hostname                          │
│  ├── Cannot coexist with other records for same name               │
│  └── Example: www.example.com → example.com                         │
│                                                                     │
│  MX Record (Mail Exchange)                                          │
│  ├── Specifies mail servers for domain                              │
│  ├── Has priority value (lower = higher priority)                   │
│  └── Example: example.com MX 10 mail1.example.com                   │
│              example.com MX 20 mail2.example.com                   │
│                                                                     │
│  TXT Record (Text)                                                  │
│  ├── Arbitrary text data                                            │
│  ├── Used for SPF, DKIM, domain verification                       │
│  └── Example: example.com TXT "v=spf1 include:_spf.google.com ~all"│
│                                                                     │
│  NS Record (Name Server)                                            │
│  ├── Delegates domain to name servers                               │
│  └── Example: example.com NS ns1.example.com                        │
│                                                                     │
│  SOA Record (Start of Authority)                                    │
│  ├── Contains administrative info about zone                        │
│  └── Serial number, refresh interval, TTL, etc.                     │
│                                                                     │
│  PTR Record (Pointer)                                               │
│  ├── Reverse DNS - IP to hostname                                   │
│  └── Example: 34.216.184.93.in-addr.arpa → www.example.com         │
│                                                                     │
│  SRV Record (Service)                                               │
│  ├── Specifies servers for specific services                        │
│  └── Example: _sip._tcp.example.com SRV 10 5 5060 sip.example.com  │
│      (priority, weight, port, target)                              │
│                                                                     │
│  CAA Record (Certificate Authority Authorization)                   │
│  ├── Specifies which CAs can issue certificates                     │
│  └── Example: example.com CAA 0 issue "letsencrypt.org"            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 9.3 DNS Commands

```bash
# nslookup - Basic DNS lookup
nslookup example.com                   # Default lookup
nslookup example.com 8.8.8.8           # Use specific DNS server
nslookup -type=mx example.com          # MX records
nslookup -type=txt example.com         # TXT records
nslookup -type=ns example.com          # NS records
nslookup -type=any example.com         # All records

# dig - Advanced DNS lookup (preferred)
dig example.com                        # A record
dig example.com A                      # Explicitly A record
dig example.com AAAA                   # IPv6 address
dig example.com MX                     # Mail servers
dig example.com TXT                    # TXT records
dig example.com NS                     # Name servers
dig example.com ANY                    # All records
dig example.com +short                 # Brief output
dig @8.8.8.8 example.com              # Use specific DNS server
dig +trace example.com                 # Show full resolution path
dig -x 93.184.216.34                  # Reverse lookup (IP to name)

# Understanding dig output:
# ;; ANSWER SECTION:
# example.com.        3600    IN    A    93.184.216.34
#     │                │       │    │         │
#     │                │       │    │         └── IP address
#     │                │       │    └── Record type
#     │                │       └── Class (IN = Internet)
#     │                └── TTL in seconds
#     └── Domain name

# host - Simple DNS lookup
host example.com                       # Basic lookup
host -t mx example.com                 # MX records
host 93.184.216.34                    # Reverse lookup

# Check DNS configuration
cat /etc/resolv.conf                   # DNS servers configured
resolvectl status                      # systemd-resolved status

# Clear DNS cache
# macOS
sudo dscacheutil -flushcache
# Linux (systemd-resolved)
sudo resolvectl flush-caches
# Windows
ipconfig /flushdns
```

## 9.4 DNS in Kubernetes

```
Kubernetes DNS (CoreDNS):
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  Every Service gets a DNS name:                                     │
│  <service-name>.<namespace>.svc.cluster.local                      │
│                                                                     │
│  Examples:                                                          │
│  nginx.default.svc.cluster.local                                   │
│  postgresql.database.svc.cluster.local                             │
│  api-gateway.production.svc.cluster.local                          │
│                                                                     │
│  Shorthand within same namespace:                                   │
│  nginx                    → Works if you're in default namespace   │
│  nginx.default            → Works from any namespace               │
│  nginx.default.svc        → Full service qualification              │
│                                                                     │
│  Pods also get DNS (but less commonly used):                        │
│  <pod-ip-dashed>.<namespace>.pod.cluster.local                     │
│  10-244-1-5.default.pod.cluster.local                              │
│                                                                     │
│  Headless Services (ClusterIP: None):                               │
│  Returns individual Pod IPs instead of Service IP                   │
│  Used for stateful applications (databases, etc.)                   │
│                                                                     │
│  External DNS:                                                       │
│  Pods can resolve external domains normally                         │
│  Uses /etc/resolv.conf configured by kubelet                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

DNS Debugging in Kubernetes:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  # Run a debugging pod                                              │
│  kubectl run dnstest --rm -it --image=busybox -- sh                │
│                                                                     │
│  # Inside the pod:                                                  │
│  nslookup kubernetes                                               │
│  nslookup nginx.default                                            │
│  nslookup google.com                                               │
│                                                                     │
│  # Check DNS configuration                                          │
│  cat /etc/resolv.conf                                              │
│  # Output:                                                          │
│  # nameserver 10.96.0.10                                           │
│  # search default.svc.cluster.local svc.cluster.local cluster.local│
│  # options ndots:5                                                  │
│                                                                     │
│  # Check CoreDNS pods                                               │
│  kubectl get pods -n kube-system -l k8s-app=kube-dns              │
│  kubectl logs -n kube-system -l k8s-app=kube-dns                  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

# Chapter 10: Linux Network Configuration

## 10.1 Network Interfaces

```bash
# View network interfaces
ip link show                          # List all interfaces
ip link show eth0                     # Specific interface
ip -s link show                       # With statistics
ip addr show                          # Show IP addresses
ip addr show eth0                     # Specific interface
ifconfig                              # Legacy command
ifconfig -a                           # All interfaces

# Understanding interface names:
# eth0, eth1      : Traditional Ethernet naming
# ens33, ens192   : BIOS/slot-based naming (ens = Ethernet, s = slot)
# enp0s3, enp0s8  : PCI-based naming (p = PCI bus, s = slot)
# eno1, eno2      : Onboard device naming
# wlan0, wlp2s0   : Wireless interfaces
# lo              : Loopback (127.0.0.1)
# docker0, br-xxx : Docker bridge networks
# veth*           : Virtual Ethernet (containers)

# Manage interface state
ip link set eth0 up                   # Bring interface up
ip link set eth0 down                 # Bring interface down

# View IP addresses
ip addr                               # All addresses
ip -4 addr                            # IPv4 only
ip -6 addr                            # IPv6 only

# Add/Remove IP addresses
ip addr add 192.168.1.10/24 dev eth0        # Add IP
ip addr add 192.168.1.10/24 dev eth0 label eth0:0  # Add alias
ip addr del 192.168.1.10/24 dev eth0        # Remove IP

# Routing
ip route show                         # Show routing table
ip route                              # Same as above
route -n                              # Legacy command

# Add/Remove routes
ip route add 10.0.0.0/8 via 192.168.1.1              # Add route
ip route add 10.0.0.0/8 via 192.168.1.1 dev eth0     # Via specific interface
ip route add default via 192.168.1.1                  # Default gateway
ip route del 10.0.0.0/8 via 192.168.1.1              # Remove route

# ARP table (IP to MAC mapping)
ip neigh show                         # Show ARP table
arp -n                                # Legacy command
ip neigh add 192.168.1.20 lladdr 00:11:22:33:44:55 dev eth0  # Add entry
ip neigh del 192.168.1.20 dev eth0   # Remove entry
ip neigh flush dev eth0              # Clear ARP cache for interface
```

## 10.2 Network Configuration Files

### Debian/Ubuntu (Netplan)

```yaml
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd  # or NetworkManager
  
  ethernets:
    eth0:
      dhcp4: true
    
    eth1:
      dhcp4: no
      addresses:
        - 192.168.1.10/24
        - 192.168.1.11/24  # Multiple IPs
      gateway4: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
        search:
          - example.com
      routes:
        - to: 10.0.0.0/8
          via: 192.168.1.254
      mtu: 1500
  
  vlans:
    vlan100:
      id: 100
      link: eth0
      addresses:
        - 10.100.0.10/24
  
  bonds:
    bond0:
      interfaces:
        - eth0
        - eth1
      addresses:
        - 192.168.1.10/24
      parameters:
        mode: 802.3ad
        lacp-rate: fast
```

```bash
# Apply netplan configuration
sudo netplan generate                  # Generate config
sudo netplan apply                     # Apply config
sudo netplan try                       # Apply with automatic rollback
```

### RHEL/CentOS (NetworkManager)

```bash
# Using nmcli (NetworkManager CLI)
nmcli device status                    # Show device status
nmcli connection show                  # Show connections
nmcli connection show "Wired connection 1"  # Connection details

# Create/Modify connections
nmcli connection add type ethernet con-name eth0 ifname eth0 \
    ipv4.addresses 192.168.1.10/24 \
    ipv4.gateway 192.168.1.1 \
    ipv4.dns "8.8.8.8 8.8.4.4" \
    ipv4.method manual

# Modify existing connection
nmcli connection modify eth0 ipv4.addresses 192.168.1.20/24
nmcli connection modify eth0 ipv4.dns "1.1.1.1"
nmcli connection modify eth0 +ipv4.dns "8.8.8.8"  # Add additional
nmcli connection modify eth0 ipv4.method manual

# Activate connection
nmcli connection up eth0
nmcli connection down eth0

# Connection files location
/etc/NetworkManager/system-connections/

# Or using config files
# /etc/sysconfig/network-scripts/ifcfg-eth0
TYPE=Ethernet
BOOTPROTO=static  # or dhcp
NAME=eth0
DEVICE=eth0
ONBOOT=yes
IPADDR=192.168.1.10
PREFIX=24
GATEWAY=192.168.1.1
DNS1=8.8.8.8
DNS2=8.8.4.4
```

## 10.3 Network Troubleshooting Commands

### Connectivity Testing

```bash
# ping - Test basic connectivity
ping 192.168.1.1                      # Ping IP address
ping google.com                        # Ping hostname (tests DNS too)
ping -c 4 192.168.1.1                 # Send 4 packets only
ping -i 0.2 192.168.1.1               # 0.2 second interval
ping -s 1472 192.168.1.1              # Specific packet size
ping -I eth0 192.168.1.1              # Use specific interface
ping -W 2 192.168.1.1                 # 2 second timeout
ping6 ::1                             # IPv6 ping

# traceroute - Trace packet path
traceroute google.com                  # UDP-based (default)
traceroute -I google.com               # ICMP-based
traceroute -T google.com               # TCP-based
traceroute -n google.com               # No DNS resolution
traceroute -p 80 google.com            # Use port 80

# mtr - Continuous traceroute
mtr google.com                         # Interactive mode
mtr -r google.com                      # Report mode
mtr -r -c 10 google.com               # 10 cycles, report

# Test specific port
nc -zv 192.168.1.1 22                 # TCP connect to port 22
nc -zv 192.168.1.1 20-30              # Range of ports
nc -zuv 192.168.1.1 53                # UDP port
telnet 192.168.1.1 22                 # Alternative TCP test

# Netcat (nc) for more testing
nc -l 8080                            # Listen on port 8080
nc 192.168.1.1 8080                   # Connect to port 8080
echo "hello" | nc 192.168.1.1 8080    # Send data
nc -l 8080 > received.txt             # Receive file
nc 192.168.1.1 8080 < file.txt        # Send file
```

### HTTP/HTTPS Testing

```bash
# curl - HTTP client
curl http://example.com               # GET request
curl -I http://example.com            # Headers only
curl -v http://example.com            # Verbose output
curl -o file.html http://example.com  # Save to file
curl -O http://example.com/file.zip   # Save with original name
curl -L http://example.com            # Follow redirects
curl -k https://example.com           # Ignore SSL errors
curl -X POST http://example.com       # POST request
curl -d "data" http://example.com     # Send data
curl -H "Content-Type: application/json" -d '{"key":"value"}' http://example.com
curl -u user:pass http://example.com  # Basic auth
curl --connect-timeout 5 http://example.com  # Connection timeout
curl -w "%{http_code}\n" -o /dev/null -s http://example.com  # Just status code

# wget - Download files
wget http://example.com/file.zip      # Download file
wget -q http://example.com/file.zip   # Quiet mode
wget -O output.zip http://example.com/file.zip  # Custom filename
wget -c http://example.com/file.zip   # Continue partial download
wget -r http://example.com            # Recursive download
wget --spider http://example.com      # Check if URL exists
```

### Socket and Connection Analysis

```bash
# ss - Socket statistics (modern)
ss -tuln                              # TCP/UDP listening ports
ss -tun                               # Active connections
ss -tunp                              # With process info
ss -s                                 # Summary statistics
ss -t state established              # Only established
ss -t dst 192.168.1.1                # Filter by destination
ss -t sport = :22                    # Filter by source port
ss -t dport = :80                    # Filter by dest port

# netstat - Socket statistics (legacy)
netstat -tuln                         # Listening ports
netstat -tunp                         # With process
netstat -s                            # Statistics
netstat -r                            # Routing table
netstat -i                            # Interface statistics

# lsof - List open files (including network)
lsof -i                               # All network files
lsof -i :80                           # Port 80
lsof -i tcp                           # TCP only
lsof -i @192.168.1.1                  # Connections to IP
lsof -i -n -P                         # No DNS/port resolution
lsof -u username -i                   # User's network connections

# fuser - Find process using port
fuser 80/tcp                          # PID using port 80
fuser -k 80/tcp                       # Kill process using port 80
```

### Packet Capture (tcpdump)

tcpdump is essential for network troubleshooting. It captures and displays network packets.

```bash
# Basic capture
tcpdump                               # Capture on default interface
tcpdump -i eth0                       # Capture on eth0
tcpdump -i any                        # Capture on all interfaces
tcpdump -c 100                        # Capture 100 packets only
tcpdump -n                            # Don't resolve hostnames
tcpdump -nn                           # Don't resolve hostnames or ports
tcpdump -v                            # Verbose
tcpdump -vv                           # More verbose
tcpdump -X                            # Show hex and ASCII
tcpdump -A                            # Show ASCII only

# Write/Read capture files
tcpdump -w capture.pcap               # Write to file
tcpdump -r capture.pcap               # Read from file
tcpdump -r capture.pcap -n            # Read without resolving

# Filter by host
tcpdump host 192.168.1.1              # To or from host
tcpdump src 192.168.1.1               # From host
tcpdump dst 192.168.1.1               # To host
tcpdump src host 192.168.1.1 and dst host 192.168.1.2

# Filter by network
tcpdump net 192.168.1.0/24            # Traffic to/from network

# Filter by port
tcpdump port 80                       # Port 80
tcpdump src port 80                   # Source port 80
tcpdump dst port 80                   # Destination port 80
tcpdump portrange 80-443              # Port range
tcpdump port 80 or port 443           # Multiple ports

# Filter by protocol
tcpdump tcp                           # TCP only
tcpdump udp                           # UDP only
tcpdump icmp                          # ICMP only
tcpdump arp                           # ARP only
tcpdump tcp port 80                   # TCP on port 80

# Complex filters
tcpdump 'tcp port 80 and host 192.168.1.1'
tcpdump 'tcp[tcpflags] & (tcp-syn) != 0'     # SYN packets
tcpdump 'tcp[tcpflags] & (tcp-rst) != 0'     # RST packets
tcpdump 'tcp[tcpflags] & (tcp-fin) != 0'     # FIN packets
tcpdump 'tcp[tcpflags] & (tcp-syn|tcp-fin) != 0'  # SYN or FIN

# Practical examples
# Capture HTTP traffic
tcpdump -i eth0 -n -A 'tcp port 80'

# Capture DNS traffic
tcpdump -i eth0 -n 'port 53'

# Capture traffic between two hosts
tcpdump -i eth0 -n 'host 192.168.1.1 and host 192.168.1.2'

# Capture SSH connection attempts
tcpdump -i eth0 -n 'tcp port 22 and tcp[tcpflags] & tcp-syn != 0'

# Save for later analysis in Wireshark
tcpdump -i eth0 -w /tmp/capture.pcap -c 10000
```

---

# Chapter 11: Firewalls and Security

## 11.1 iptables

iptables is the traditional Linux firewall. It's being replaced by nftables, but iptables knowledge is still essential.

### Understanding iptables Architecture

```
iptables Structure:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  Tables (contain chains):                                           │
│  ├── filter (default): Packet filtering (accept/drop)              │
│  ├── nat: Network Address Translation                              │
│  ├── mangle: Packet modification                                   │
│  └── raw: Connection tracking exemptions                           │
│                                                                     │
│  Chains in 'filter' table:                                          │
│  ├── INPUT: Packets destined for local system                      │
│  ├── OUTPUT: Packets originating from local system                 │
│  └── FORWARD: Packets being routed through system                  │
│                                                                     │
│  Chains in 'nat' table:                                             │
│  ├── PREROUTING: Before routing decision                           │
│  ├── POSTROUTING: After routing decision                           │
│  └── OUTPUT: Locally generated packets                             │
│                                                                     │
│  Targets (actions):                                                 │
│  ├── ACCEPT: Allow packet                                          │
│  ├── DROP: Silently discard packet                                 │
│  ├── REJECT: Discard with ICMP error                               │
│  ├── LOG: Log packet (then continue to next rule)                  │
│  ├── DNAT: Destination NAT                                         │
│  ├── SNAT: Source NAT                                              │
│  └── MASQUERADE: Dynamic SNAT                                      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

Packet Flow:
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│                         Incoming Packet                             │
│                              │                                      │
│                              ▼                                      │
│                    ┌──────────────────┐                             │
│                    │    PREROUTING    │ (nat, mangle, raw)          │
│                    └────────┬─────────┘                             │
│                             │                                       │
│                    ┌────────▼─────────┐                             │
│                    │ Routing Decision │                             │
│                    └────────┬─────────┘                             │
│                             │                                       │
│              ┌──────────────┴──────────────┐                        │
│              │                             │                        │
│              ▼                             ▼                        │
│     ┌──────────────┐              ┌──────────────┐                  │
│     │    INPUT     │              │   FORWARD    │                  │
│     │ (for local)  │              │ (for routing)│                  │
│     └──────┬───────┘              └──────┬───────┘                  │
│            │                             │                          │
│            ▼                             │                          │
│     Local Process                        │                          │
│            │                             │                          │
│            ▼                             │                          │
│     ┌──────────────┐                     │                          │
│     │    OUTPUT    │                     │                          │
│     └──────┬───────┘                     │                          │
│            │                             │                          │
│            └──────────────┬──────────────┘                          │
│                           │                                         │
│                           ▼                                         │
│                  ┌──────────────────┐                               │
│                  │   POSTROUTING    │ (nat, mangle)                 │
│                  └────────┬─────────┘                               │
│                           │                                         │
│                           ▼                                         │
│                    Outgoing Packet                                  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### iptables Commands

```bash
# View rules
iptables -L                           # List filter table rules
iptables -L -n                        # Without DNS resolution
iptables -L -n -v                     # Verbose (packet counts)
iptables -L -n --line-numbers         # With line numbers
iptables -S                           # Print rules as commands
iptables -t nat -L                    # List NAT table
iptables -t nat -L -n -v              # NAT table verbose

# Rule syntax:
# iptables -A CHAIN -p PROTOCOL --dport PORT -s SOURCE -d DEST -j TARGET

# Basic filtering rules
# Accept established connections (stateful firewall)
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Accept loopback
iptables -A INPUT -i lo -j ACCEPT

# Accept SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Accept HTTP and HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Accept from specific IP
iptables -A INPUT -s 192.168.1.100 -j ACCEPT

# Accept from subnet
iptables -A INPUT -s 192.168.1.0/24 -j ACCEPT

# Accept ICMP (ping)
iptables -A INPUT -p icmp -j ACCEPT

# Drop all other incoming
iptables -A INPUT -j DROP

# Common options:
# -A CHAIN      : Append rule to chain
# -I CHAIN N    : Insert rule at position N
# -D CHAIN N    : Delete rule at position N
# -D CHAIN ...  : Delete matching rule
# -F            : Flush (delete all rules)
# -P CHAIN TARGET : Set default policy
# -p PROTOCOL   : Protocol (tcp, udp, icmp)
# --dport PORT  : Destination port
# --sport PORT  : Source port
# -s SOURCE     : Source address
# -d DEST       : Destination address
# -i INTERFACE  : Input interface
# -o INTERFACE  : Output interface
# -j TARGET     : Jump to target (ACCEPT, DROP, etc.)
# -m MODULE     : Load module

# Insert rule at beginning
iptables -I INPUT 1 -p tcp --dport 22 -j ACCEPT

# Delete rule
iptables -D INPUT -p tcp --dport 22 -j ACCEPT
iptables -D INPUT 5                   # By line number

# Set default policy
iptables -P INPUT DROP                # Default drop for input
iptables -P FORWARD DROP              # Default drop for forward
iptables -P OUTPUT ACCEPT             # Default accept for output

# Flush all rules
iptables -F                           # Flush filter table
iptables -t nat -F                    # Flush NAT table
iptables -X                           # Delete user-defined chains

# NAT rules
# SNAT: Change source IP (outbound)
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth0 -j SNAT --to-source 203.0.113.1

# DNAT: Change destination IP (port forwarding)
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to-destination 192.168.1.10:8080

# Port forwarding example (public port 80 to internal port 8080)
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.10:8080
iptables -A FORWARD -p tcp -d 192.168.1.10 --dport 8080 -j ACCEPT

# Logging
iptables -A INPUT -j LOG --log-prefix "IPTables-Dropped: " --log-level 4
# Then check: dmesg or /var/log/kern.log

# Rate limiting
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -m limit --limit 3/min -j ACCEPT

# Save and restore
iptables-save > /etc/iptables.rules
iptables-restore < /etc/iptables.rules

# Persistent rules (Debian/Ubuntu)
apt install iptables-persistent
netfilter-persistent save
netfilter-persistent reload
```

### Practical Firewall Script

```bash
#!/bin/bash
# Basic firewall script

# Flush existing rules
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X

# Set default policies
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Allow loopback
iptables -A INPUT -i lo -j ACCEPT

# Allow established connections
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Allow SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP/HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow ping (optional)
iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT

# Log dropped packets (optional)
iptables -A INPUT -j LOG --log-prefix "IPT-DROP: " --log-level 4

# Save rules
iptables-save > /etc/iptables.rules
```

## 11.2 Firewalld (RHEL/CentOS)

Firewalld provides a dynamic, zone-based firewall.

```bash
# Check status
systemctl status firewalld
firewall-cmd --state

# Zones
firewall-cmd --get-zones              # List available zones
firewall-cmd --get-default-zone       # Show default zone
firewall-cmd --get-active-zones       # Show active zones
firewall-cmd --set-default-zone=public

# List rules
firewall-cmd --list-all               # Current zone
firewall-cmd --zone=public --list-all # Specific zone

# Add services
firewall-cmd --add-service=http       # Temporary
firewall-cmd --add-service=http --permanent  # Permanent
firewall-cmd --add-service=https --permanent
firewall-cmd --reload                 # Apply permanent rules

# Add ports
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --add-port=53/udp --permanent
firewall-cmd --reload

# Remove rules
firewall-cmd --remove-service=http --permanent
firewall-cmd --remove-port=8080/tcp --permanent
firewall-cmd --reload

# Rich rules (advanced)
firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" service name="ssh" accept' --permanent
firewall-cmd --add-rich-rule='rule family="ipv4" source address="10.0.0.0/8" port port="5432" protocol="tcp" accept' --permanent

# Port forwarding
firewall-cmd --add-forward-port=port=80:proto=tcp:toport=8080 --permanent
firewall-cmd --add-forward-port=port=80:proto=tcp:toaddr=192.168.1.10:toport=8080 --permanent

# Enable masquerading (NAT)
firewall-cmd --add-masquerade --permanent
```

## 11.3 UFW (Ubuntu)

UFW (Uncomplicated Firewall) is a user-friendly frontend for iptables.

```bash
# Enable/Disable
ufw enable
ufw disable
ufw status
ufw status verbose
ufw status numbered               # With rule numbers

# Default policies
ufw default deny incoming
ufw default allow outgoing

# Allow services
ufw allow ssh                     # Allow SSH (22/tcp)
ufw allow http                    # Allow HTTP (80/tcp)
ufw allow https                   # Allow HTTPS (443/tcp)

# Allow ports
ufw allow 8080                    # Allow 8080 (tcp and udp)
ufw allow 8080/tcp               # Allow 8080 (tcp only)
ufw allow 8080:8090/tcp          # Port range

# Allow from specific IP
ufw allow from 192.168.1.100
ufw allow from 192.168.1.100 to any port 22
ufw allow from 192.168.1.0/24

# Deny rules
ufw deny 23                       # Deny port 23
ufw deny from 10.0.0.1           # Deny from IP

# Delete rules
ufw delete allow 8080
ufw delete 5                      # Delete by number

# Logging
ufw logging on
ufw logging medium                # Levels: off, low, medium, high, full

# Reset
ufw reset                         # Reset to defaults
```

---

This concludes Part 3 covering networking in depth.

**Continue to Part 4 for:**
- Kubernetes Complete Deep Dive
- AKS (Azure Kubernetes Service) Mastery
- Azure Services Beyond AKS
