# Java Application Troubleshooting - Complete Guide
## From Basic to Advanced - Every Scenario

> **Learning reference:** [Observability & Java Apps](../../Notes/Observability/DevOps-07-Observability-Apps.md)

---

# SECTION 1: STARTUP ISSUES

---

## 1.1 APPLICATION WON'T START

### LEVEL 1 - DIRECT: Class Not Found

**Scenario**: App fails with ClassNotFoundException.

```bash
$ java -jar myapp.jar
Exception in thread "main" java.lang.ClassNotFoundException: com.myapp.Main
```

**Cause is VISIBLE**: Main class not found.

**Solution**:
```bash
# Check JAR contents
jar tf myapp.jar | grep Main

# Check MANIFEST.MF for Main-Class
unzip -p myapp.jar META-INF/MANIFEST.MF

# Fix build to include Main-Class in manifest
```

---

### LEVEL 1 - DIRECT: No Main Manifest Attribute

**Scenario**: JAR has no main class defined.

```bash
$ java -jar myapp.jar
no main manifest attribute, in myapp.jar
```

**Cause is VISIBLE**: MANIFEST.MF doesn't specify main class.

**Solution**:
```bash
# Run with explicit class
java -cp myapp.jar com.myapp.Main

# Or fix build configuration
# Maven:
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-jar-plugin</artifactId>
  <configuration>
    <archive>
      <manifest>
        <mainClass>com.myapp.Main</mainClass>
      </manifest>
    </archive>
  </configuration>
</plugin>
```

---

### LEVEL 1 - DIRECT: Port Already In Use

**Scenario**: App fails because port is taken.

```bash
$ java -jar myapp.jar
Error: Address already in use (Bind failed) :8080
```

**Cause is VISIBLE**: Port 8080 is already in use.

**Solution**:
```bash
# Find what's using the port
lsof -i :8080
ss -tuln | grep 8080

# Kill it or use different port
java -jar myapp.jar --server.port=8081
```

---

### LEVEL 1 - DIRECT: Missing Dependency

**Scenario**: App fails with ClassNotFoundException for library.

```bash
$ java -jar myapp.jar
java.lang.NoClassDefFoundError: org/apache/commons/lang3/StringUtils
```

**Cause is VISIBLE**: Commons Lang library not included.

**Solution**:
```bash
# Check if dependency is in JAR
jar tf myapp.jar | grep commons

# Build fat JAR with dependencies
# Maven with shade plugin
# Or Spring Boot maven plugin
```

---

### LEVEL 2 - INTERMEDIATE: Config File Not Found

**Scenario**: App starts but fails reading config.

```bash
$ java -jar myapp.jar
java.io.FileNotFoundException: /etc/myapp/config.properties (No such file)
```

**Investigation**:
```bash
# App expects specific config location
# Either create file or override location
java -jar myapp.jar --spring.config.location=/path/to/config.properties
```

---

### LEVEL 2 - INTERMEDIATE: Database Connection Failed

**Scenario**: App starts but can't connect to DB.

```bash
$ java -jar myapp.jar
org.postgresql.util.PSQLException: Connection refused
Check that the hostname and port are correct
```

**Investigation**:
```bash
# Check if database is accessible
nc -zv dbhost 5432

# Check connection string
# Host? Port? Database name? Credentials?

# Check firewall
```

---

# SECTION 2: MEMORY ISSUES

---

## 2.1 OUT OF MEMORY

### LEVEL 1 - DIRECT: Heap Space Error

**Scenario**: App crashes with OutOfMemoryError.

```bash
java.lang.OutOfMemoryError: Java heap space
```

**Cause is VISIBLE**: Heap is full.

**Solution**:
```bash
# Increase heap size
java -Xmx2g -Xms1g -jar myapp.jar

# Check current settings
jcmd <pid> VM.flags | grep -i heap
```

---

### LEVEL 1 - DIRECT: Metaspace Error

**Scenario**: OutOfMemoryError for Metaspace.

```bash
java.lang.OutOfMemoryError: Metaspace
```

**Cause is VISIBLE**: Metaspace (class metadata) is full.

**Solution**:
```bash
# Increase metaspace
java -XX:MaxMetaspaceSize=512m -jar myapp.jar

# Often caused by too many classes loaded
# Check for classloader leaks
```

---

### LEVEL 1 - DIRECT: Unable to Create Native Thread

**Scenario**: Can't create new threads.

```bash
java.lang.OutOfMemoryError: unable to create native thread
```

**Cause is VISIBLE**: Thread limit reached.

**Solution**:
```bash
# Check current threads
jcmd <pid> Thread.print | grep "^\"" | wc -l

# Check system limit
ulimit -u

# Reduce thread stack size
java -Xss256k -jar myapp.jar

# Or reduce number of threads in app
```

---

### LEVEL 2 - INTERMEDIATE: Container OOMKilled But Heap OK

**Scenario**: Container killed, but heap wasn't full.

```bash
# Container exit code 137, OOMKilled=true
# But heap dump shows plenty of free heap
```

**Investigation**:
```bash
# JVM uses more than just heap!
# Enable Native Memory Tracking
java -XX:NativeMemoryTracking=summary -jar myapp.jar

# Check total usage
jcmd <pid> VM.native_memory summary

# Common non-heap memory:
# - Metaspace
# - Thread stacks (1MB each!)
# - Direct buffers
# - Code cache
```

**Solution**:
```bash
# Limit all memory regions
java \
  -Xmx1536m \
  -XX:MaxMetaspaceSize=256m \
  -XX:MaxDirectMemorySize=256m \
  -XX:ReservedCodeCacheSize=128m \
  -Xss256k \
  -jar myapp.jar
```

---

### LEVEL 3 - COMPLEX: Memory Leak Over Time

**Scenario**: Memory grows slowly until OOM after days.

**Hidden Cause**: Memory leak - objects held longer than needed.

**Deep Investigation**:
```bash
# Take heap dump
jcmd <pid> GC.heap_dump /tmp/heap.hprof

# Analyze with Eclipse MAT
# Look for:
# - Biggest objects
# - Retained heap
# - GC roots

# Check class histogram
jcmd <pid> GC.class_histogram | head -30
```

**Common Leak Sources**:
- Collections that grow without cleanup
- Event listeners not removed
- Caches without eviction
- Static references

---

## 2.2 GARBAGE COLLECTION ISSUES

### LEVEL 1 - DIRECT: GC Overhead Limit Exceeded

**Scenario**: App spends too much time in GC.

```bash
java.lang.OutOfMemoryError: GC overhead limit exceeded
```

**Cause is VISIBLE**: Spending >98% time in GC, recovering <2% heap.

**Solution**:
```bash
# Increase heap (short term)
java -Xmx4g -jar myapp.jar

# Check for memory leak (long term)
# Find what's consuming memory
```

---

### LEVEL 2 - INTERMEDIATE: Application Pauses

**Scenario**: App freezes periodically.

```bash
# Response times spike every few minutes
# 50ms normally, 3 seconds during pause
```

**Investigation**:
```bash
# Enable GC logging
java -Xlog:gc*:file=/tmp/gc.log:time,level,tags -jar myapp.jar

# Check for long GC pauses
grep "GC pause" /tmp/gc.log | awk '{print $NF}'
```

**Solution**:
```bash
# Use low-pause GC
java -XX:+UseG1GC -XX:MaxGCPauseMillis=100 -jar myapp.jar

# Or use ZGC for sub-millisecond pauses (Java 11+)
java -XX:+UseZGC -jar myapp.jar
```

---

# SECTION 3: PERFORMANCE ISSUES

---

## 3.1 HIGH CPU

### LEVEL 1 - DIRECT: One Thread Using 100% CPU

**Scenario**: Application consuming 100% CPU.

```bash
$ top -H -p <pid>
  PID   %CPU   COMMAND
12345   99.0   java
```

**Investigation**:
```bash
# Get thread dump
jcmd <pid> Thread.print > thread.txt

# Convert Linux TID to Java thread
# Linux shows 12345, convert to hex: 0x3039
printf "%x\n" 12345

# Search in thread dump
grep -A 20 "nid=0x3039" thread.txt
```

---

### LEVEL 2 - INTERMEDIATE: High CPU, Thread Dump Shows Nothing

**Scenario**: CPU high but no thread stuck in one place.

**Investigation**:
```bash
# Take multiple thread dumps
for i in 1 2 3 4 5; do
  jcmd <pid> Thread.print > dump_$i.txt
  sleep 2
done

# Compare - if threads in different places each time,
# might be processing load, not a bug

# Profile with async-profiler
./profiler.sh -d 30 -f profile.html <pid>
```

---

## 3.2 SLOW APPLICATION

### LEVEL 1 - DIRECT: Obvious Slow Query in Logs

**Scenario**: App slow, logs show slow database query.

```bash
WARN: Query took 5000ms: SELECT * FROM users WHERE...
```

**Cause is VISIBLE**: Slow SQL query.

**Solution**:
```bash
# Add index
# Optimize query
# Add caching
```

---

### LEVEL 2 - INTERMEDIATE: App Hangs, No CPU

**Scenario**: Application not responding, CPU is low.

```bash
# No response to requests
# CPU near 0%
```

**Investigation**:
```bash
# Get thread dump
jcmd <pid> Thread.print

# Look for BLOCKED threads
grep -A 10 "BLOCKED" thread_dump.txt

# Look for deadlock
# jstack auto-detects deadlocks
jstack <pid> | grep -i deadlock
```

**Common Causes**:
- Deadlock
- Thread pool exhausted
- Waiting for external resource
- Database connection pool empty

---

# SECTION 4: CONTAINER-SPECIFIC ISSUES

---

## 4.1 JAVA IN CONTAINERS

### LEVEL 1 - DIRECT: JVM Doesn't See Container Limits

**Scenario**: JVM uses more memory than container limit.

```bash
# Container has 512MB limit
# JVM tries to use 1GB, gets killed
```

**Cause is VISIBLE**: Old JVM doesn't respect container limits.

**Solution**:
```bash
# Java 8u191+ / Java 11+
# Container limits respected by default

# Or explicitly set
java -XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0 -jar myapp.jar
```

---

### LEVEL 2 - INTERMEDIATE: Slow Startup in Container

**Scenario**: App starts in 30s locally, 5 minutes in container.

**Investigation**:
```bash
# Check for entropy starvation
cat /proc/sys/kernel/random/entropy_avail
# < 200 = SecureRandom might block

# Check CPU limits
cat /sys/fs/cgroup/cpu/cpu.cfs_quota_us
cat /sys/fs/cgroup/cpu/cpu.cfs_period_us
```

**Solution**:
```bash
# Fix entropy
java -Djava.security.egd=file:/dev/./urandom -jar myapp.jar

# Increase startup resources
# Or remove CPU limit during startup
```

---

# QUICK REFERENCE

## Diagnostic Commands

```bash
# Process info
jps                              # List Java processes
jinfo <pid>                      # JVM info

# Memory
jstat -gcutil <pid> 1000         # GC statistics
jcmd <pid> GC.heap_info          # Heap summary
jcmd <pid> GC.heap_dump /tmp/heap.hprof  # Heap dump
jcmd <pid> VM.native_memory summary      # Native memory

# Threads
jcmd <pid> Thread.print          # Thread dump
jstack <pid>                     # Thread dump (older)

# CPU profiling
jcmd <pid> JFR.start duration=60s filename=/tmp/profile.jfr

# Flags
jcmd <pid> VM.flags              # Current flags
```

## Common Errors → Solutions

```
"ClassNotFoundException"         → Class missing from classpath
"NoClassDefFoundError"           → Dependency missing
"OutOfMemoryError: Java heap"    → Increase -Xmx
"OutOfMemoryError: Metaspace"    → Increase -XX:MaxMetaspaceSize
"OutOfMemoryError: native thread"→ Reduce stack size or threads
"GC overhead limit exceeded"     → Memory leak or need more heap
"Address already in use"         → Port conflict
```

## JVM Flags for Containers

```bash
java \
  -XX:+UseContainerSupport \
  -XX:MaxRAMPercentage=75.0 \
  -XX:InitialRAMPercentage=50.0 \
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=100 \
  -XX:+ExitOnOutOfMemoryError \
  -Djava.security.egd=file:/dev/./urandom \
  -jar myapp.jar
```
