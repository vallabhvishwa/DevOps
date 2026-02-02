# Troubleshooting Documentation - Master Index
## Complete Coverage: Basic to Advanced

---

## Guide Structure

Every guide now covers THREE levels for each topic:

| Level | Description | Example |
|-------|-------------|---------|
| **LEVEL 1 - DIRECT** | Cause is visible in error/output | "Connection refused" → Port not listening |
| **LEVEL 2 - INTERMEDIATE** | Needs a few steps to find | Works locally, fails in CI → Environment differences |
| **LEVEL 3 - COMPLEX** | Hidden cause, deep investigation | High load but CPUs idle → NFS hung mount |

---

## Available Guides

| # | Guide | Coverage |
|---|-------|----------|
| 1 | [Linux Troubleshooting](./01-Linux-Troubleshooting.md) | CPU, Memory, Disk, Network, Services, Permissions |
| 2 | [Kubernetes Troubleshooting](./02-Kubernetes-Troubleshooting.md) | Pod States, Scheduling, Networking, Storage, Workloads |
| 3 | [Docker/Container Troubleshooting](./03-Docker-Containers-Troubleshooting.md) | Startup, Images, Networking, Builds, Resources |
| 4 | [Networking Troubleshooting](./04-Networking-Troubleshooting.md) | Connectivity, DNS, Firewall, SSL, Performance |
| 5 | [Java Application Troubleshooting](./05-Java-Application-Troubleshooting.md) | Startup, Memory, GC, Performance, Containers |
| 6 | [Node.js Application Troubleshooting](./06-NodeJS-Application-Troubleshooting.md) | Startup, Memory, Event Loop, Containers |
| 7 | [Jenkins Troubleshooting](./07-Jenkins-Troubleshooting.md) | Builds, Pipelines, K8s Agents, Performance |
| 8 | [Database Troubleshooting](./08-Database-Troubleshooting.md) | PostgreSQL, MySQL, Redis |

---

## Master Error → Cause → Solution Reference

### Linux Errors
```
"Permission denied"           → Check all 6 permission layers
"No space left on device"     → Check df -h AND df -i (inodes!)
"Connection refused"          → Port not listening or firewall REJECT
"Connection timed out"        → Firewall DROP or routing issue
"Cannot allocate memory"      → Check free -h, check ulimits
"Too many open files"         → ulimit -n, /proc/<pid>/limits
"Name or service not known"   → DNS issue, check /etc/resolv.conf
```

### Kubernetes Errors
```
"Pending" (no events)         → Scheduler issue or wrong schedulerName
"Pending" (Insufficient cpu)  → Reduce requests or add nodes
"Pending" (taint)             → Add toleration to pod
"CrashLoopBackOff"            → Check logs with --previous flag
"ImagePullBackOff"            → Check image name, tag, credentials
"0/1 Ready"                   → Readiness probe failing
"No endpoints"                → Service selector doesn't match pods
```

### Docker Errors
```
Exit 0                        → Command completed (not error!)
Exit 1                        → Application error (check logs)
Exit 126                      → Permission denied (can't execute)
Exit 127                      → Command not found
Exit 137                      → OOMKilled (SIGKILL)
Exit 139                      → Segfault (binary compatibility)
Exit 143                      → SIGTERM (graceful shutdown)
"Address already in use"      → Port conflict
```

### Network Errors
```
"Network unreachable"         → No route, check ip route
"Host unreachable"            → Host down or ARP issue
"Connection refused"          → Nothing listening
"Connection timed out"        → Firewall DROP (silent block)
"Name not resolved"           → DNS failure
"Certificate expired"         → Renew certificate
```

### Java Errors
```
"ClassNotFoundException"      → Class not in classpath
"OutOfMemoryError: heap"      → Increase -Xmx
"OutOfMemoryError: Metaspace" → Increase -XX:MaxMetaspaceSize
"OutOfMemoryError: threads"   → Reduce stack or thread count
"GC overhead limit exceeded"  → Memory leak or need more heap
```

### Node.js Errors
```
"Cannot find module"          → npm install
"Heap out of memory"          → --max-old-space-size
"EADDRINUSE"                  → Port already in use
"ECONNREFUSED"                → Service not running
"ENOENT"                      → File not found
```

### Database Errors
```
"Connection refused"          → Database not running
"Authentication failed"       → Wrong credentials
"Too many connections"        → Connection leak or limit too low
"No space left"               → VACUUM or add disk
"Lock wait timeout"           → Query blocking others
```

---

## Systematic Troubleshooting Approach

```
1. IDENTIFY THE SYMPTOM
   What exactly is happening?
   What should be happening?

2. GATHER EVIDENCE
   Logs, metrics, error messages
   When did it start? What changed?

3. FORM HYPOTHESIS
   Based on evidence, what could cause this?
   Start with most likely causes

4. TEST HYPOTHESIS
   Use diagnostic commands
   Eliminate possibilities

5. FIX AND VERIFY
   Apply the fix
   Confirm issue is resolved
   Monitor for recurrence

6. DOCUMENT
   What was the issue?
   What fixed it?
   How to prevent in future?
```

---

## Quick Diagnostic Commands

### Linux
```bash
top / htop                 # Process overview
vmstat 1 5                 # System stats
iostat -x 1 5              # Disk I/O
free -h                    # Memory
df -h / df -i              # Disk space/inodes
ss -tuln                   # Listening ports
journalctl -u svc          # Service logs
dmesg | tail               # Kernel messages
```

### Kubernetes
```bash
kubectl get pods -o wide
kubectl describe pod <pod>
kubectl logs <pod> --previous
kubectl get events --sort-by='.lastTimestamp'
kubectl top pods
kubectl exec <pod> -- <command>
```

### Docker
```bash
docker ps -a
docker logs <container>
docker inspect <container>
docker stats
docker exec -it <container> sh
```

### Network
```bash
ping / traceroute / mtr
ss -tuln / netstat -tuln
tcpdump -i any port X
dig / nslookup
curl -v
```

### Java
```bash
jcmd <pid> Thread.print
jcmd <pid> GC.heap_dump /tmp/heap.hprof
jstat -gcutil <pid> 1000
jcmd <pid> VM.native_memory summary
```

### Database
```sql
-- PostgreSQL
SELECT * FROM pg_stat_activity;
EXPLAIN ANALYZE <query>;

-- Redis
INFO
SLOWLOG GET 10
```

---

## Learning Path

**Recommended Order:**

1. **Linux** - Foundation for everything else
2. **Networking** - Critical for distributed systems
3. **Docker** - Container fundamentals before K8s
4. **Kubernetes** - Orchestration platform
5. **Java / Node.js** - Application layer issues
6. **Jenkins** - CI/CD specific problems
7. **Database** - Data layer troubleshooting

---

## Key Principles

```
1. DON'T ASSUME
   "The network is fine" → Prove it with evidence

2. LAYER BY LAYER
   Physical → Network → Transport → Application

3. SIMPLE FIRST
   Is it running? Is it listening? Are logs helpful?

4. ONE CHANGE AT A TIME
   Otherwise you won't know what fixed it

5. CONSIDER WHAT CHANGED
   It was working yesterday - what's different?

6. THE SYMPTOM ISN'T THE CAUSE
   Pod OOM → Real cause: memory leak
   Slow query → Real cause: missing index
```
