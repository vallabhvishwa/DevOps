# Network Troubleshooting - Complete Guide
## From Basic to Advanced - Every Scenario

> **Learning reference:** [Networking](../../Notes/Networking/DevOps-03-Networking.md) | [SSL/TLS](../../Notes/SSL-TLS/SSL-TLS-Certificates-Guide.md)

---

# SECTION 1: CONNECTIVITY ISSUES

---

## 1.1 CANNOT REACH HOST

### LEVEL 1 - DIRECT: Network Unreachable

**Scenario**: Can't ping a host.

```bash
$ ping 10.0.0.50
connect: Network is unreachable
```

**Cause is VISIBLE**: No route to that network.

**Solution**:
```bash
# Check routing table
ip route show

# Check if you have route to 10.0.0.0/24
ip route get 10.0.0.50

# Add route if missing
sudo ip route add 10.0.0.0/24 via <gateway>
```

---

### LEVEL 1 - DIRECT: Host Unreachable

**Scenario**: Network reachable but host isn't.

```bash
$ ping 10.0.0.50
From 10.0.0.1 icmp_seq=1 Destination Host Unreachable
```

**Cause is VISIBLE**: Host is down or doesn't exist.

**Solution**:
```bash
# Check if host is up
# Try from different machine

# Check ARP table
arp -a | grep 10.0.0.50

# If no ARP entry, host isn't responding
# Check if IP is correct
# Check if host is running
```

---

### LEVEL 1 - DIRECT: No Route to Host

**Scenario**: Route exists but packets fail.

```bash
$ ping 10.0.0.50
ping: sendmsg: No route to host
```

**Cause is VISIBLE**: Firewall blocking on target or path.

**Solution**:
```bash
# This usually means:
# - Target firewall is REJECTING packets
# - Or intermediate firewall

# Check if ICMP is allowed
# On target: iptables -L INPUT | grep icmp
```

---

### LEVEL 2 - INTERMEDIATE: Ping Works, Service Doesn't

**Scenario**: Can ping but can't connect to service.

```bash
$ ping 10.0.0.50
64 bytes from 10.0.0.50: icmp_seq=1 time=1ms  # Works!

$ curl http://10.0.0.50:8080
curl: (7) Connection refused
```

**Investigation**:
```bash
# ICMP works, TCP doesn't = port-specific issue

# Check if port is listening (from target)
ss -tuln | grep 8080

# Check firewall rules
iptables -L INPUT -n | grep 8080

# Check if service is running
systemctl status myapp
```

---

### LEVEL 3 - COMPLEX: Timeout (Packets Disappear)

**Scenario**: Connection hangs, never completes.

```bash
$ curl --connect-timeout 5 http://10.0.0.50:8080
curl: (28) Connection timed out
```

**Hidden Causes**:
```
TIMEOUT CAUSES:
┌─────────────────────────────────────────────────────────────────┐
│  1. Firewall DROP rule (not REJECT)                             │
│  2. Routing black hole                                          │
│  3. Asymmetric routing (reply goes different path)              │
│  4. SNAT/DNAT misconfiguration                                  │
│  5. MTU issues (packet too big)                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Deep Investigation**:
```bash
# Trace the path
traceroute 10.0.0.50
mtr 10.0.0.50

# Capture packets at source
tcpdump -i any host 10.0.0.50 and port 8080

# If you see SYN going out but no SYN-ACK:
# - Packet not reaching target, OR
# - Reply not coming back

# Capture at target (if possible)
# If SYN arrives but no SYN-ACK goes out = target firewall
# If SYN-ACK goes out = reply path problem
```

---

## 1.2 DNS ISSUES

### LEVEL 1 - DIRECT: DNS Not Resolving

**Scenario**: Hostname doesn't resolve.

```bash
$ nslookup myserver.example.com
** server can't find myserver.example.com: NXDOMAIN
```

**Cause is VISIBLE**: Name doesn't exist in DNS.

**Solution**:
```bash
# Check if name is correct (typo?)

# Try different DNS server
nslookup myserver.example.com 8.8.8.8

# Check /etc/hosts
grep myserver /etc/hosts

# If internal name, check internal DNS
nslookup myserver.example.com 10.0.0.2
```

---

### LEVEL 1 - DIRECT: DNS Timeout

**Scenario**: DNS query times out.

```bash
$ nslookup google.com
;; connection timed out; no servers could be reached
```

**Cause is VISIBLE**: Can't reach DNS server.

**Solution**:
```bash
# Check resolv.conf
cat /etc/resolv.conf

# Test DNS server connectivity
ping 10.0.0.2  # Your DNS server
nc -zvu 10.0.0.2 53  # UDP 53

# If DNS unreachable, fix network or use different DNS
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
```

---

### LEVEL 2 - INTERMEDIATE: DNS Returns Wrong IP

**Scenario**: DNS resolves but to wrong IP.

```bash
$ nslookup myapp.example.com
Address: 10.0.0.99    # Expected 10.0.0.50!
```

**Investigation**:
```bash
# Check where answer comes from
dig myapp.example.com +trace

# Check if cached
dig myapp.example.com @8.8.8.8
dig myapp.example.com @your-dns

# Might be stale cache, or wrong DNS record
```

---

## 1.3 PORT CONNECTIVITY

### LEVEL 1 - DIRECT: Connection Refused

**Scenario**: Port explicitly refused.

```bash
$ telnet 10.0.0.50 8080
Connection refused
```

**Cause is VISIBLE**: Nothing listening on port 8080.

**Solution**:
```bash
# On target server, check listening
ss -tuln | grep 8080

# If not listening, start the service
# If listening on 127.0.0.1, it's localhost only
ss -tuln | grep 8080
# Shows: 127.0.0.1:8080 → Only local connections!
# Fix: Bind to 0.0.0.0
```

---

### LEVEL 1 - DIRECT: Connection Closed

**Scenario**: Connection closes immediately.

```bash
$ telnet 10.0.0.50 8080
Connection closed by foreign host.
```

**Cause is VISIBLE**: Server actively closes connection.

**Investigation**:
```bash
# Server might be:
# - Checking authentication and rejecting
# - Protocol mismatch (expecting HTTPS?)
# - Application error

# Check server logs
journalctl -u myapp
```

---

### LEVEL 2 - INTERMEDIATE: Intermittent Connection Issues

**Scenario**: Sometimes works, sometimes doesn't.

```bash
$ curl http://10.0.0.50:8080
# Success!

$ curl http://10.0.0.50:8080  
# Connection refused

$ curl http://10.0.0.50:8080
# Success!
```

**Investigation**:
```bash
# Multiple backend servers, one is down?
# DNS returns multiple IPs?
dig myserver.example.com +short

# Load balancer with unhealthy backend?

# Check if same IP each time
curl -v http://myserver 2>&1 | grep "Connected to"
```

---

# SECTION 2: NETWORK PERFORMANCE

---

## 2.1 SLOW NETWORK

### LEVEL 1 - DIRECT: High Latency Visible

**Scenario**: Ping shows high latency.

```bash
$ ping 10.0.0.50
64 bytes from 10.0.0.50: icmp_seq=1 time=500 ms  # High!
```

**Cause is VISIBLE**: 500ms latency (should be <10ms on LAN).

**Solution**:
```bash
# Find where latency is introduced
mtr 10.0.0.50

# Each hop shows latency
# Find the hop with jump in latency
```

---

### LEVEL 1 - DIRECT: Packet Loss

**Scenario**: Ping shows packet loss.

```bash
$ ping -c 100 10.0.0.50
100 packets transmitted, 85 received, 15% packet loss
```

**Cause is VISIBLE**: 15% packet loss.

**Solution**:
```bash
# Find where loss occurs
mtr -r -c 100 10.0.0.50

# Check interface errors
ip -s link show eth0
# Look for: errors, dropped, overruns
```

---

### LEVEL 2 - INTERMEDIATE: Slow Transfer, No Packet Loss

**Scenario**: Large transfer slow, but ping is fine.

```bash
$ ping 10.0.0.50
time=1ms, 0% loss

$ scp bigfile.tar 10.0.0.50:
# Very slow: 1MB/s instead of 100MB/s
```

**Investigation**:
```bash
# Test bandwidth
iperf3 -c 10.0.0.50

# Check for MTU issues
ping -M do -s 1472 10.0.0.50
# If this fails, MTU mismatch

# Check TCP settings
cat /proc/sys/net/ipv4/tcp_window_scaling
```

---

### LEVEL 3 - COMPLEX: Network Slow After Some Time

**Scenario**: Network fast initially, slows down over time.

**Hidden Causes**:
```
PERFORMANCE DEGRADATION:
┌─────────────────────────────────────────────────────────────────┐
│  1. Conntrack table filling up                                  │
│  2. Interface buffer overflows                                  │
│  3. iptables rules accumulating                                 │
│  4. Time-based rate limiting                                    │
│  5. Thermal throttling                                          │
└─────────────────────────────────────────────────────────────────┘
```

**Deep Investigation**:
```bash
# Check conntrack
cat /proc/sys/net/nf_conntrack_count
cat /proc/sys/net/nf_conntrack_max
# If count near max, that's the problem

# Check interface drops
ip -s link show eth0
# Look for RX/TX dropped

# Check iptables rules
iptables -L -n | wc -l
```

---

# SECTION 3: FIREWALL ISSUES

---

## 3.1 IPTABLES/FIREWALLD

### LEVEL 1 - DIRECT: Firewall Blocking

**Scenario**: Firewall explicitly blocking traffic.

```bash
$ sudo iptables -L INPUT -n -v
Chain INPUT (policy DROP)
pkts  target  prot  dport
5000  DROP    tcp   8080  # Explicitly dropped!
```

**Cause is VISIBLE**: Rule dropping port 8080.

**Solution**:
```bash
# Add allow rule before drop
iptables -I INPUT -p tcp --dport 8080 -j ACCEPT

# Or with firewalld
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --reload
```

---

### LEVEL 1 - DIRECT: Default Policy DROP

**Scenario**: No specific rule, but default policy blocks.

```bash
$ iptables -L INPUT
Chain INPUT (policy DROP)  # Default DROP!
```

**Cause is VISIBLE**: Default policy is DROP, no allow rule.

**Solution**:
```bash
# Add specific allow rule
iptables -A INPUT -p tcp --dport 8080 -j ACCEPT
```

---

### LEVEL 2 - INTERMEDIATE: Rule Order Problem

**Scenario**: Allow rule exists but still blocked.

```bash
$ iptables -L INPUT -n --line-numbers
1   DROP    all  --  0.0.0.0/0   0.0.0.0/0
2   ACCEPT  tcp  --  0.0.0.0/0   0.0.0.0/0  dport 8080
```

**Investigation**: Rule 2 (ACCEPT) comes AFTER rule 1 (DROP all). Never evaluated.

**Solution**:
```bash
# Insert at beginning
iptables -I INPUT 1 -p tcp --dport 8080 -j ACCEPT

# Or delete and reorder
iptables -D INPUT 2
iptables -I INPUT 1 -p tcp --dport 8080 -j ACCEPT
```

---

# SECTION 4: SSL/TLS ISSUES

---

## 4.1 CERTIFICATE ISSUES

### LEVEL 1 - DIRECT: Certificate Expired

**Scenario**: SSL connection fails, certificate expired.

```bash
$ curl https://myserver.com
curl: (60) SSL certificate problem: certificate has expired
```

**Cause is VISIBLE**: Certificate expired.

**Solution**:
```bash
# Check expiry
openssl s_client -connect myserver.com:443 -servername myserver.com 2>/dev/null | openssl x509 -noout -dates

# Renew certificate
# For Let's Encrypt:
certbot renew
```

---

### LEVEL 1 - DIRECT: Certificate Name Mismatch

**Scenario**: SSL fails, hostname mismatch.

```bash
$ curl https://10.0.0.50
curl: (60) SSL: certificate subject name 'myserver.com' does not match target host name '10.0.0.50'
```

**Cause is VISIBLE**: Accessing by IP but cert is for hostname.

**Solution**:
```bash
# Use hostname instead of IP
curl https://myserver.com

# Or add to /etc/hosts
echo "10.0.0.50 myserver.com" >> /etc/hosts
```

---

### LEVEL 2 - INTERMEDIATE: Untrusted CA

**Scenario**: Certificate not trusted.

```bash
$ curl https://myserver.com
curl: (60) SSL certificate problem: unable to get local issuer certificate
```

**Investigation**:
```bash
# Check the certificate chain
openssl s_client -connect myserver.com:443 -showcerts

# Check if intermediate cert is missing
```

**Solution**:
```bash
# Server needs to send full chain
# Update server config to include intermediate cert

# Or add CA to client trust store
update-ca-certificates
```

---

# QUICK REFERENCE

## Diagnostic Commands

```bash
# Layer 1-2: Physical/Link
ip link show                    # Interface status
ethtool eth0                    # Link speed, duplex
ip -s link show eth0            # Interface statistics

# Layer 3: Network
ping <host>                     # Basic connectivity
traceroute <host>               # Path tracing
mtr <host>                      # Continuous traceroute
ip route show                   # Routing table
ip addr show                    # IP addresses

# Layer 4: Transport
ss -tuln                        # Listening ports
ss -tan                         # Active connections
nc -zv <host> <port>            # Port check
telnet <host> <port>            # Port test

# DNS
nslookup <hostname>             # DNS lookup
dig <hostname>                  # Detailed DNS
dig <hostname> +trace           # Full resolution path

# Packet capture
tcpdump -i any port 80          # Capture packets
tcpdump -w capture.pcap         # Save to file

# Firewall
iptables -L -n -v               # List iptables rules
firewall-cmd --list-all         # List firewalld rules

# SSL/TLS
openssl s_client -connect h:443 # Test SSL connection
curl -vI https://host           # Verbose HTTPS test
```

## Error → Cause → Solution

```
"Network unreachable"      → No route → Add route or fix gateway
"Host unreachable"         → Host down → Check host or ARP
"Connection refused"       → Port not listening → Start service
"Connection timed out"     → Firewall DROP → Check firewall rules
"Name not resolved"        → DNS issue → Check /etc/resolv.conf
"Certificate expired"      → Cert expired → Renew certificate
"Unable to get issuer"     → Missing CA → Add intermediate cert
```
