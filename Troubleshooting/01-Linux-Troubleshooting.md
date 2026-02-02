# Linux Troubleshooting - Complete Guide
## From Basic to Advanced - Every Scenario

---

# HOW TO USE THIS GUIDE

This guide covers THREE levels for each topic:
- **LEVEL 1 - DIRECT**: Cause is visible in error/output
- **LEVEL 2 - INTERMEDIATE**: Needs a few steps to find
- **LEVEL 3 - COMPLEX**: Hidden cause, requires deep investigation

---

# SECTION 1: CPU & PROCESS ISSUES

---

## 1.1 HIGH CPU USAGE

### LEVEL 1 - DIRECT: One Process Using 100% CPU

**Scenario**: System slow, you run `top` and immediately see one process at 100%.

```bash
$ top
  PID USER      %CPU %MEM    COMMAND
12345 appuser   99.9  5.0    java
```

**Cause is VISIBLE**: PID 12345 (java) is using all CPU.

**Solution**:
```bash
# Identify the process
ps aux | grep 12345

# Check what it's doing
strace -p 12345 -c  # System call summary

# If it's stuck in a loop, might need to restart
kill 12345  # Graceful
kill -9 12345  # Force
```

---

### LEVEL 1 - DIRECT: System Overloaded (Many Processes)

**Scenario**: Many processes each using moderate CPU, totaling > 100%.

```bash
$ top
%Cpu(s): 95.0 us,  3.0 sy,  0.0 ni,  2.0 id

  PID   %CPU   COMMAND
 1001   25.0   httpd
 1002   25.0   httpd
 1003   25.0   httpd
 1004   25.0   httpd
```

**Cause is VISIBLE**: Too many httpd workers consuming all CPU.

**Solution**:
```bash
# Check how many workers
ps aux | grep httpd | wc -l

# Reduce worker count in config
# Or add more CPU (scale up/out)
```

---

### LEVEL 2 - INTERMEDIATE: High CPU But Top Shows Nothing Obvious

**Scenario**: System feels slow. `top` shows high CPU but no single process stands out.

```bash
$ top
%Cpu(s): 85.0 us,  10.0 sy,  0.0 ni,  5.0 id
# Many processes at 5-10% each
```

**Investigation**:
```bash
# Sort by CPU to see cumulative
ps aux --sort=-%cpu | head -20

# Check if it's system (kernel) or user processes
# High %sy = kernel activity (might be I/O, network)

# Check for short-lived processes (top won't catch them)
pidstat 1 5  # Shows per-process stats every 1 second

# Or use atop for historical data
atop
```

**Cause**: Often many small processes, or short-lived processes spawning repeatedly.

---

### LEVEL 3 - COMPLEX: Low CPU Shown But System Slow

**Scenario**: `top` shows only 20% CPU used, but system is very slow.

```bash
$ top
%Cpu(s): 20.0 us,  5.0 sy,  0.0 ni, 15.0 id, 60.0 wa
#                                             ^^^^^^
#                                             I/O WAIT!
```

**Hidden Cause**: The 60% I/O wait means CPU is waiting for disk. It's not a CPU problem at all.

**Theory**:
```
CPU TIME BREAKDOWN:
┌─────────────────────────────────────────────────────────────────┐
│  %us (user)    - Running user applications                     │
│  %sy (system)  - Running kernel code                           │
│  %id (idle)    - Doing nothing, available                      │
│  %wa (iowait)  - Waiting for I/O (disk, network)  ← THE ISSUE  │
│  %st (steal)   - Time stolen by hypervisor (VMs)               │
│                                                                 │
│  High %wa = CPU is idle but waiting for slow disk              │
└─────────────────────────────────────────────────────────────────┘
```

**Deep Investigation**:
```bash
# Confirm I/O is the problem
iostat -x 1 5
# Look at: %util (disk busy), await (response time)

# Find which processes are doing I/O
iotop -o

# Check disk health
smartctl -a /dev/sda
dmesg | grep -i error
```

---

### LEVEL 3 - COMPLEX: High Load Average, All CPUs Idle

**Scenario**: `uptime` shows load 50, but 8 CPUs are all 95% idle.

```bash
$ uptime
load average: 50.23, 48.15, 45.00

$ top
# All 8 CPUs showing 95%+ idle
```

**Hidden Cause**: Load average counts processes in "uninterruptible sleep" (D state) waiting for I/O. These aren't using CPU but inflate load average.

**Theory**:
```
LOAD AVERAGE includes:
- Running processes (using CPU)
- Runnable processes (waiting for CPU)  
- Uninterruptible sleep (D state - waiting for I/O) ← HIDDEN!
```

**Deep Investigation**:
```bash
# Count D state processes
ps aux | awk '$8 ~ /D/ { print }'

# What are they waiting for?
cat /proc/<pid>/wchan
# Common: nfs_wait_bit_killable = NFS hung

# Check if NFS/mount is stuck
mount | grep nfs
ls /path/to/nfs/mount  # Will hang if NFS is stuck
```

---

## 1.2 PROCESS ISSUES

### LEVEL 1 - DIRECT: Process Not Running

**Scenario**: Service isn't working. Check if it's running.

```bash
$ systemctl status myservice
● myservice.service - My Service
   Active: inactive (dead)

# Or
$ ps aux | grep myprocess
# No output
```

**Cause is VISIBLE**: Service is not running.

**Solution**:
```bash
# Start it
systemctl start myservice

# Check why it stopped
journalctl -u myservice --since "1 hour ago"
```

---

### LEVEL 1 - DIRECT: Process Crashed with Clear Error

**Scenario**: Service failed to start with error message.

```bash
$ systemctl status myservice
● myservice.service - My Service
   Active: failed (Result: exit-code)
   
$ journalctl -u myservice
ERROR: Cannot bind to port 8080: Address already in use
```

**Cause is VISIBLE**: Port 8080 is already in use.

**Solution**:
```bash
# Find what's using the port
lsof -i :8080
ss -tuln | grep 8080

# Kill it or use different port
```

---

### LEVEL 2 - INTERMEDIATE: Process Keeps Crashing (Restart Loop)

**Scenario**: Service starts but keeps crashing.

```bash
$ systemctl status myservice
● myservice.service
   Active: activating (auto-restart)
   # Restarted 15 times in last hour
```

**Investigation**:
```bash
# Check logs for pattern
journalctl -u myservice | tail -100

# Look for the crash reason
journalctl -u myservice | grep -i "error\|fail\|exception"

# Check if it's resource related
journalctl -u myservice | grep -i "memory\|oom\|killed"
```

**Common Causes**:
- Configuration error
- Missing dependency
- Resource exhaustion
- Startup race condition

---

### LEVEL 3 - COMPLEX: Process Running But Not Working

**Scenario**: `systemctl status` shows "active (running)" but service doesn't respond.

```bash
$ systemctl status myservice
● myservice.service
   Active: active (running)   # Says it's running!

$ curl http://localhost:8080
curl: (7) Connection refused   # But it's not responding!
```

**Hidden Cause**: Main process is alive but worker thread died, or app is in error state.

**Deep Investigation**:
```bash
# Is it listening?
ss -tuln | grep 8080
# Nothing = app started but didn't bind to port

# Check application logs (not just systemd logs)
cat /var/log/myservice/application.log

# Check process state
cat /proc/$(pgrep myservice)/status | grep State

# Check if stuck in syscall
strace -p $(pgrep myservice) 2>&1 | head -20
```

---

## 1.3 MEMORY ISSUES

### LEVEL 1 - DIRECT: Out of Memory Error

**Scenario**: Application fails with OOM error.

```bash
$ dmesg | tail
[12345.678] Out of memory: Kill process 1234 (java)
```

**Cause is VISIBLE**: System ran out of memory, killed java process.

**Solution**:
```bash
# Check current memory
free -h

# Find memory hogs
ps aux --sort=-%mem | head -10

# Add swap as safety net
fallocate -l 4G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

---

### LEVEL 1 - DIRECT: High Memory Usage Visible

**Scenario**: `top` or `free` shows high memory usage.

```bash
$ free -h
              total        used        free      buff/cache   available
Mem:           16G         15G        200M           800M         500M
```

**Cause is VISIBLE**: 15GB used, almost no free memory.

**Solution**:
```bash
# Find what's using memory
ps aux --sort=-%mem | head -10

# Kill or restart the memory hog
# Or increase system memory
```

---

### LEVEL 2 - INTERMEDIATE: Free Shows Low But Apps Say OOM

**Scenario**: `free` shows memory available, but apps fail with OOM.

```bash
$ free -h
              total        used        free      buff/cache   available
Mem:           16G          8G         100M          7.9G          7.5G
#                                     ^^^^                       ^^^^^
#                                     Low free      But available is OK!
```

**Understanding**:
```
Linux Memory Model:
- "free" = truly unused (Linux minimizes this intentionally)
- "buff/cache" = used for caching but CAN be freed
- "available" = what's actually available for apps

If "available" is high but app says OOM:
- App might have its own memory limit (container, JVM -Xmx)
- Check container/cgroup limits
```

**Investigation**:
```bash
# Check container limits
cat /sys/fs/cgroup/memory/memory.limit_in_bytes

# Check JVM settings
jcmd <pid> VM.flags | grep -i heap
```

---

### LEVEL 3 - COMPLEX: Memory Leak - Gradual OOM Over Days

**Scenario**: System works fine for days, then OOM. Restarts fix it temporarily.

**Hidden Cause**: Memory leak - application slowly accumulates memory.

**Deep Investigation**:
```bash
# Track memory over time
while true; do
    echo "$(date): $(ps -p $(pgrep myapp) -o rss=)"
    sleep 60
done >> memory_log.txt

# For Java, take heap dumps at different times
jcmd <pid> GC.heap_dump /tmp/heap1.hprof
# Wait an hour
jcmd <pid> GC.heap_dump /tmp/heap2.hprof
# Compare with Eclipse MAT

# Check for growing caches, connection pools, etc.
```

---

# SECTION 2: DISK ISSUES

---

## 2.1 DISK SPACE

### LEVEL 1 - DIRECT: Disk Full Error

**Scenario**: Application error "No space left on device"

```bash
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1       100G  100G     0 100% /
```

**Cause is VISIBLE**: Disk is 100% full.

**Solution**:
```bash
# Find large files
du -sh /* | sort -hr | head -10

# Find large files recursively
find / -type f -size +100M -exec ls -lh {} \;

# Common cleanups:
rm -rf /var/log/*.gz  # Old compressed logs
docker system prune   # Docker cleanup
journalctl --vacuum-size=500M  # Limit journal
```

---

### LEVEL 1 - DIRECT: Large File Visible

**Scenario**: Disk full, `du` shows a giant file.

```bash
$ du -sh /var/log/*
50G    /var/log/application.log
```

**Cause is VISIBLE**: application.log is 50GB.

**Solution**:
```bash
# Truncate the file (keeps file open)
> /var/log/application.log

# Or configure log rotation
cat > /etc/logrotate.d/myapp << EOF
/var/log/application.log {
    daily
    rotate 7
    size 100M
    compress
    missingok
}
EOF
```

---

### LEVEL 2 - INTERMEDIATE: Disk Full, Can't Find Large Files

**Scenario**: `df` says full, `du` doesn't add up.

```bash
$ df -h /
Filesystem      Size  Used Avail Use%
/dev/sda1       100G   95G    5G  95%

$ du -sh /
50G    /
# 50G used but df says 95G!
```

**Investigation**:
```bash
# Check for deleted but open files
lsof +L1
# Shows files that are deleted but still held open by processes
# These use disk space but du doesn't see them!
```

**Cause**: Deleted files still held open by running processes.

**Solution**:
```bash
# Find the culprit
lsof +L1 | awk '$5 == "REG" { print $7, $9, $1 }' | sort -rn | head

# Restart the process holding the file
# Or truncate via /proc/<pid>/fd/<fd>
```

---

### LEVEL 3 - COMPLEX: "No Space" But Disk Shows Free

**Scenario**: `df -h` shows 50% free, but can't create files.

```bash
$ df -h
Filesystem      Size  Used Avail Use%
/dev/sda1       100G   50G   50G  50%

$ touch /tmp/test
touch: cannot touch '/tmp/test': No space left on device
```

**Hidden Cause**: Inodes exhausted (file metadata), not disk blocks.

**Theory**:
```
Disk has TWO resources:
1. Blocks (space for file content) - shown by df -h
2. Inodes (one per file/directory) - shown by df -i

Millions of tiny files = inodes exhausted while space available
```

**Deep Investigation**:
```bash
$ df -i
Filesystem      Inodes   IUsed   IFree IUse%
/dev/sda1       6553600 6553600       0  100%   # INODES FULL!

# Find directory with most files
find / -xdev -type d -exec sh -c 'echo "$(find "{}" -maxdepth 1 | wc -l) {}"' \; | sort -rn | head
```

---

## 2.2 DISK I/O

### LEVEL 1 - DIRECT: Disk Error in Logs

**Scenario**: System logs show disk errors.

```bash
$ dmesg | grep -i error
[12345.678] sd 0:0:0:0: [sda] Sense Key : Medium Error
[12345.679] blk_update_request: I/O error, dev sda, sector 12345678
```

**Cause is VISIBLE**: Disk has physical errors.

**Solution**:
```bash
# Check disk health
smartctl -a /dev/sda

# Look for reallocated sectors, pending sectors
smartctl -a /dev/sda | grep -E "Reallocated|Pending|Uncorrectable"

# Plan disk replacement if SMART shows issues
```

---

### LEVEL 1 - DIRECT: Disk 100% Utilized in iostat

**Scenario**: System slow, `iostat` shows disk busy.

```bash
$ iostat -x 1
Device    %util   await
sda       99.50   250.00
```

**Cause is VISIBLE**: Disk is at 100% utilization with high wait time.

**Solution**:
```bash
# Find what's doing I/O
iotop -o

# Reduce I/O load or upgrade to faster disk (SSD)
```

---

### LEVEL 2 - INTERMEDIATE: Slow I/O, No Obvious Cause

**Scenario**: Disk is slow but not 100% utilized.

```bash
$ iostat -x 1
Device    %util   await   r_await   w_await
sda       40.00   50.00    10.00    200.00
#                          Read fast, Write slow!
```

**Investigation**:
- High `w_await` vs `r_await` = slow writes
- Could be: write-through cache, RAID rebuild, failing disk

```bash
# Check if RAID is rebuilding
cat /proc/mdstat

# Check for write-through vs write-back cache
# (depends on RAID controller)
```

---

# SECTION 3: NETWORK ISSUES

---

## 3.1 CONNECTIVITY

### LEVEL 1 - DIRECT: Host Unreachable

**Scenario**: Can't connect to server.

```bash
$ ping 10.0.0.50
PING 10.0.0.50: Destination Host Unreachable
```

**Cause is VISIBLE**: No route to host (network/routing issue).

**Solution**:
```bash
# Check routing
ip route get 10.0.0.50

# Check if on same network
ip addr show

# Check gateway
ip route show default
ping <gateway>  # Can you reach gateway?
```

---

### LEVEL 1 - DIRECT: Connection Refused

**Scenario**: Network works but service won't connect.

```bash
$ curl http://10.0.0.50:8080
curl: (7) Connection refused
```

**Cause is VISIBLE**: Port 8080 is not listening (or firewall REJECT).

**Solution**:
```bash
# Check if port is listening (on target server)
ss -tuln | grep 8080

# If not listening, start the service
# If listening, check firewall
iptables -L -n | grep 8080
```

---

### LEVEL 1 - DIRECT: DNS Resolution Failed

**Scenario**: Can't resolve hostname.

```bash
$ ping myserver.example.com
ping: myserver.example.com: Name or service not known
```

**Cause is VISIBLE**: DNS can't resolve the name.

**Solution**:
```bash
# Check DNS config
cat /etc/resolv.conf

# Test with specific DNS server
dig @8.8.8.8 myserver.example.com

# If internal name, use internal DNS
dig @10.0.0.2 myserver.example.com
```

---

### LEVEL 2 - INTERMEDIATE: Timeout (No Response)

**Scenario**: Connection hangs, eventually times out.

```bash
$ curl --connect-timeout 5 http://10.0.0.50:8080
curl: (28) Connection timed out
```

**Investigation**:
```bash
# Timeout means packets dropped (firewall DROP, not REJECT)
# Or network path broken

# Test basic connectivity
ping 10.0.0.50

# If ping works, check port specifically
telnet 10.0.0.50 8080
nc -zv 10.0.0.50 8080

# Trace the path
traceroute 10.0.0.50

# Check local firewall
iptables -L OUTPUT -n
```

**Common causes**:
- Firewall DROP rule (not REJECT)
- Routing black hole
- Service listening on wrong interface

---

### LEVEL 3 - COMPLEX: Intermittent Connectivity

**Scenario**: Works sometimes, fails randomly.

**Hidden Causes**:
```
INTERMITTENT FAILURE CAUSES:
┌─────────────────────────────────────────────────────────────────┐
│  1. DNS Round-Robin                                             │
│     - Multiple IPs, one is dead                                 │
│                                                                 │
│  2. Load Balancer                                               │
│     - One backend unhealthy                                     │
│                                                                 │
│  3. Packet Loss                                                 │
│     - Bad cable, network congestion                             │
│                                                                 │
│  4. Connection Table Full                                       │
│     - conntrack table exhausted                                 │
│                                                                 │
│  5. Port Exhaustion                                             │
│     - Too many outgoing connections                             │
└─────────────────────────────────────────────────────────────────┘
```

**Deep Investigation**:
```bash
# Check for multiple DNS results
dig myserver.example.com +short
# If multiple IPs, test each one

# Check packet loss
mtr -r -c 100 10.0.0.50

# Check conntrack
cat /proc/sys/net/nf_conntrack_count
cat /proc/sys/net/nf_conntrack_max
dmesg | grep conntrack

# Check port exhaustion
ss -s | grep TIME-WAIT
```

---

## 3.2 NETWORK PERFORMANCE

### LEVEL 1 - DIRECT: Slow Network, Visible Packet Loss

**Scenario**: Network slow, ping shows packet loss.

```bash
$ ping -c 100 10.0.0.50
100 packets transmitted, 85 received, 15% packet loss
```

**Cause is VISIBLE**: 15% packet loss.

**Investigation**:
```bash
# Find where loss occurs
mtr -r 10.0.0.50

# Check interface errors
ip -s link show eth0
# Look for RX/TX errors, drops
```

---

### LEVEL 2 - INTERMEDIATE: Slow Transfer, No Packet Loss

**Scenario**: Large file transfer is slow, but ping is fine.

```bash
$ ping 10.0.0.50
# Fast, no loss

$ scp bigfile.tar 10.0.0.50:
# Very slow transfer
```

**Investigation**:
```bash
# Test bandwidth
iperf3 -c 10.0.0.50

# Check MTU issues
ping -M do -s 1472 10.0.0.50
# If this fails, MTU mismatch

# Check TCP window scaling
cat /proc/sys/net/ipv4/tcp_window_scaling
```

---

# SECTION 4: PERMISSIONS & ACCESS

---

## 4.1 FILE PERMISSIONS

### LEVEL 1 - DIRECT: Permission Denied

**Scenario**: Can't read/write/execute file.

```bash
$ cat /etc/secret
cat: /etc/secret: Permission denied

$ ls -l /etc/secret
-rw------- 1 root root 100 Jan 1 00:00 /etc/secret
```

**Cause is VISIBLE**: File only readable by root, you're not root.

**Solution**:
```bash
# Run as root
sudo cat /etc/secret

# Or change permissions
sudo chmod 644 /etc/secret
```

---

### LEVEL 1 - DIRECT: Cannot Execute Script

**Scenario**: Script won't run.

```bash
$ ./script.sh
bash: ./script.sh: Permission denied

$ ls -l script.sh
-rw-r--r-- 1 user user 100 Jan 1 00:00 script.sh
```

**Cause is VISIBLE**: No execute permission.

**Solution**:
```bash
chmod +x script.sh
```

---

### LEVEL 2 - INTERMEDIATE: Permission OK But Still Denied

**Scenario**: File permissions look fine but still denied.

```bash
$ ls -l /data/file.txt
-rw-r--r-- 1 appuser appgroup 100 Jan 1 00:00 /data/file.txt

$ sudo -u appuser cat /data/file.txt
cat: /data/file.txt: Permission denied
```

**Investigation**:
```bash
# Check ENTIRE path
namei -l /data/file.txt
# Every directory needs 'x' permission to traverse

# Check ACLs
getfacl /data/file.txt

# Check if filesystem is mounted read-only
mount | grep /data
```

---

### LEVEL 3 - COMPLEX: Permission Denied Despite Everything Looking OK

**Scenario**: All permissions, ACLs, path look correct. Still denied.

**Hidden Causes**:
```
PERMISSION LAYERS:
┌─────────────────────────────────────────────────────────────────┐
│  1. Traditional UNIX permissions (ls -l)                        │
│  2. ACLs (getfacl)                                              │
│  3. SELinux (ls -Z, getenforce)                                 │
│  4. AppArmor (aa-status)                                        │
│  5. Mount options (ro, noexec)                                  │
│  6. File attributes (lsattr - immutable flag)                   │
└─────────────────────────────────────────────────────────────────┘
```

**Deep Investigation**:
```bash
# Check SELinux
getenforce
ls -laZ /data/file.txt
ausearch -m AVC -ts recent

# Check AppArmor  
aa-status
journalctl | grep apparmor

# Check file attributes
lsattr /data/file.txt
# 'i' = immutable (can't modify even as root)

# Check mount options
mount | grep /data
# Look for: ro, noexec, nosuid
```

---

# SECTION 5: SERVICE MANAGEMENT

---

## 5.1 SYSTEMD SERVICES

### LEVEL 1 - DIRECT: Service Failed to Start

**Scenario**: Service won't start, error visible.

```bash
$ systemctl start myservice
Job for myservice.service failed.

$ systemctl status myservice
● myservice.service
   Active: failed (Result: exit-code)

$ journalctl -u myservice -n 20
ERROR: Configuration file not found: /etc/myservice/config.yaml
```

**Cause is VISIBLE**: Config file missing.

**Solution**:
```bash
# Create the config file
cp /etc/myservice/config.yaml.example /etc/myservice/config.yaml
systemctl start myservice
```

---

### LEVEL 1 - DIRECT: Service Dependency Not Met

**Scenario**: Service fails because dependency isn't running.

```bash
$ journalctl -u myservice
ERROR: Cannot connect to database at localhost:5432
```

**Cause is VISIBLE**: Database not running.

**Solution**:
```bash
# Start the dependency first
systemctl start postgresql
systemctl start myservice
```

---

### LEVEL 2 - INTERMEDIATE: Service Starts Then Stops

**Scenario**: Service starts, runs briefly, then exits.

```bash
$ systemctl status myservice
   Active: inactive (dead)
   # Was active 10 seconds ago
```

**Investigation**:
```bash
# Check full log for error before exit
journalctl -u myservice --since "5 minutes ago"

# Check if it's exiting cleanly (exit 0)
systemctl show myservice --property=ExecMainStatus
```

---

### LEVEL 3 - COMPLEX: Service Running But Not Functioning

**Scenario**: `systemctl status` shows running, but service doesn't work.

Already covered in Process section - see "Process Running But Not Working"

---

# QUICK REFERENCE

## Diagnostic Commands by Category

```bash
# CPU
top, htop, ps aux --sort=-%cpu, pidstat, mpstat

# Memory
free -h, ps aux --sort=-%mem, vmstat, /proc/meminfo

# Disk Space
df -h, df -i, du -sh, ncdu, lsof +L1

# Disk I/O
iostat -x, iotop, dstat, /proc/diskstats

# Network
ping, traceroute, mtr, ss, netstat, tcpdump, ip

# Process
ps, pstree, strace, ltrace, /proc/<pid>/

# Logs
journalctl, dmesg, /var/log/

# Permissions
ls -la, getfacl, namei -l, lsattr, getenforce
```

## Error → Likely Cause

```
"Connection refused"     → Port not listening or firewall REJECT
"Connection timed out"   → Firewall DROP or routing issue
"Permission denied"      → Check all 6 permission layers
"No space left"          → df -h (space) AND df -i (inodes)
"Cannot allocate memory" → Check free -h, check ulimits
"Too many open files"    → ulimit -n, /proc/<pid>/limits
"Address already in use" → Port already taken (ss -tuln)
"Name not resolved"      → DNS issue (/etc/resolv.conf)
```
