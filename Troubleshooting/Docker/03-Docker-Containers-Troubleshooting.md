# Docker & Container Troubleshooting - Complete Guide
## From Basic to Advanced - Every Scenario

---

# SECTION 1: CONTAINER WON'T START

---

## 1.1 IMAGE ISSUES

### LEVEL 1 - DIRECT: Image Not Found

**Scenario**: Container fails to start, image not found.

```bash
$ docker run myregistry/myapp:v1.0
Unable to find image 'myregistry/myapp:v1.0' locally
Error: pull access denied, repository does not exist
```

**Cause is VISIBLE**: Image doesn't exist or wrong name.

**Solution**:
```bash
# Check if image exists locally
docker images | grep myapp

# Try to pull it
docker pull myregistry/myapp:v1.0

# Check exact name and tag
# Fix typos in image name
```

---

### LEVEL 1 - DIRECT: Authentication Required

**Scenario**: Private registry needs login.

```bash
$ docker pull myregistry.azurecr.io/myapp:v1.0
Error: unauthorized: authentication required
```

**Cause is VISIBLE**: Not logged in to registry.

**Solution**:
```bash
# Login to registry
docker login myregistry.azurecr.io

# For Azure ACR
az acr login --name myregistry
```

---

### LEVEL 1 - DIRECT: Tag Not Found

**Scenario**: Image exists but tag doesn't.

```bash
$ docker pull myapp:v2.0
Error: manifest for myapp:v2.0 not found
```

**Cause is VISIBLE**: Tag v2.0 doesn't exist.

**Solution**:
```bash
# List available tags
# Docker Hub:
curl -s https://hub.docker.com/v2/repositories/library/myapp/tags | jq '.results[].name'

# Use correct tag
docker pull myapp:v1.9
```

---

## 1.2 CONTAINER EXITS IMMEDIATELY

### LEVEL 1 - DIRECT: Exit Code 0 (Command Completed)

**Scenario**: Container starts and exits with code 0.

```bash
$ docker run myapp
$ docker ps -a
CONTAINER ID   IMAGE   COMMAND      STATUS
abc123         myapp   "echo hi"    Exited (0)
```

**Cause is VISIBLE**: Command ran successfully and completed.

**Solution**:
```bash
# Check what command runs
docker inspect myapp --format='{{.Config.Cmd}}'
docker inspect myapp --format='{{.Config.Entrypoint}}'

# If command should be long-running, fix Dockerfile:
CMD ["node", "server.js"]  # Not: CMD ["echo", "hello"]

# Or keep container alive for debugging
docker run -it myapp /bin/sh
```

---

### LEVEL 1 - DIRECT: Exit Code 1 with Error in Logs

**Scenario**: Container exits with error visible in logs.

```bash
$ docker run myapp
Error: Config file not found at /app/config.yaml

$ docker ps -a
CONTAINER ID   STATUS
abc123         Exited (1)
```

**Cause is VISIBLE**: Missing config file.

**Solution**:
```bash
# Mount config file
docker run -v ./config.yaml:/app/config.yaml myapp

# Or set environment variable
docker run -e CONFIG_PATH=/alt/config.yaml myapp
```

---

### LEVEL 1 - DIRECT: Exit Code 127 - Command Not Found

**Scenario**: Container exits with code 127.

```bash
$ docker run myapp
/bin/sh: myapp: not found

$ echo $?
127
```

**Cause is VISIBLE**: Command/binary not found in container.

**Solution**:
```bash
# Check what's in the image
docker run -it myapp /bin/sh  # Or /bin/bash

# List files
ls -la /app/

# Check if binary has correct path in CMD
# Fix Dockerfile
```

---

### LEVEL 1 - DIRECT: Exit Code 126 - Permission Denied

**Scenario**: Container exits with code 126.

```bash
$ docker run myapp
/bin/sh: /app/start.sh: Permission denied
```

**Cause is VISIBLE**: Script not executable.

**Solution**:
```dockerfile
# In Dockerfile, add execute permission
RUN chmod +x /app/start.sh

# Or fix before building
chmod +x start.sh
docker build .
```

---

### LEVEL 2 - INTERMEDIATE: Exit Code 137 - OOMKilled

**Scenario**: Container killed with code 137.

```bash
$ docker run -m 128m myapp
Killed

$ docker inspect <container-id> --format='{{.State.OOMKilled}}'
true
```

**Investigation**:
- Exit 137 = 128 + 9 = SIGKILL
- OOMKilled = true confirms out of memory

**Solution**:
```bash
# Increase memory limit
docker run -m 512m myapp

# Or fix application memory usage
```

---

### LEVEL 2 - INTERMEDIATE: Exit Code 139 - Segfault

**Scenario**: Container exits with code 139.

```bash
$ docker run myapp
Segmentation fault

$ echo $?
139
```

**Investigation**:
- Exit 139 = 128 + 11 = SIGSEGV
- Usually binary compatibility issue

```bash
# Check if binary matches architecture
docker run myapp uname -m
# vs your build machine architecture

# Check library dependencies
docker run myapp ldd /app/myapp
```

---

### LEVEL 3 - COMPLEX: Exit Code 1, No Logs

**Scenario**: Container exits with code 1, logs are empty.

```bash
$ docker run myapp
# No output

$ docker logs abc123
# Empty
```

**Hidden Cause**: Process crashes before logging starts.

**Deep Investigation**:
```bash
# Override to keep alive
docker run -it --entrypoint /bin/sh myapp

# Inside container, try running the app
./myapp

# Check for missing libraries
ldd ./myapp

# Check if it's a glibc vs musl issue
# (Alpine uses musl, Ubuntu uses glibc)
```

---

## 1.3 PORT BINDING ISSUES

### LEVEL 1 - DIRECT: Port Already In Use

**Scenario**: Container fails to start, port conflict.

```bash
$ docker run -p 8080:80 nginx
Error: bind: address already in use
```

**Cause is VISIBLE**: Port 8080 is already used.

**Solution**:
```bash
# Find what's using the port
lsof -i :8080
ss -tuln | grep 8080

# Use different port
docker run -p 8081:80 nginx

# Or stop the conflicting process
```

---

### LEVEL 1 - DIRECT: Container Port Not Exposed

**Scenario**: Port mapping doesn't work.

```bash
$ docker run -d -p 8080:3000 myapp
$ curl http://localhost:8080
curl: (52) Empty reply from server
```

**Investigation**:
```bash
# Check if app is listening inside container
docker exec <container> ss -tuln
docker exec <container> netstat -tuln

# If listening on different port, fix mapping
docker run -d -p 8080:8000 myapp  # If app listens on 8000
```

---

# SECTION 2: CONTAINER RUNNING BUT NOT WORKING

---

## 2.1 APPLICATION ISSUES

### LEVEL 1 - DIRECT: App Logs Show Error

**Scenario**: Container running but app not working, logs show why.

```bash
$ docker logs myapp
ERROR: Database connection failed
Cannot connect to postgres:5432
```

**Cause is VISIBLE**: Can't connect to database.

**Solution**:
```bash
# Check if database container is running
docker ps | grep postgres

# Check if on same network
docker network inspect mynetwork

# Fix network
docker run --network mynetwork myapp
```

---

### LEVEL 1 - DIRECT: Environment Variable Missing

**Scenario**: App fails due to missing env var.

```bash
$ docker logs myapp
Error: DATABASE_URL environment variable is required
```

**Cause is VISIBLE**: Missing environment variable.

**Solution**:
```bash
docker run -e DATABASE_URL=postgres://... myapp

# Or use env file
docker run --env-file .env myapp
```

---

### LEVEL 2 - INTERMEDIATE: Works Locally, Fails in Container

**Scenario**: App runs on host, fails in container.

```bash
# On host
$ ./myapp
Running on port 3000...

# In container
$ docker run myapp
Error: ENOENT: no such file or directory '/home/user/config.json'
```

**Investigation**:
- Hardcoded paths don't exist in container
- Different filesystem layout
- Missing files not copied in Dockerfile

**Solution**:
```dockerfile
# Use relative paths or configurable paths
WORKDIR /app
COPY config.json .
ENV CONFIG_PATH=/app/config.json
```

---

## 2.2 NETWORKING ISSUES

### LEVEL 1 - DIRECT: Container Can't Reach Internet

**Scenario**: Container can't reach external services.

```bash
$ docker exec myapp ping google.com
ping: bad address 'google.com'
```

**Cause is VISIBLE**: DNS not working.

**Solution**:
```bash
# Test with IP
docker exec myapp ping 8.8.8.8

# If IP works, DNS issue
docker run --dns 8.8.8.8 myapp
```

---

### LEVEL 1 - DIRECT: Containers Can't Communicate

**Scenario**: Container A can't reach Container B.

```bash
$ docker exec app1 ping app2
ping: bad address 'app2'
```

**Investigation**:
```bash
# Check if on same network
docker network inspect bridge

# Default bridge doesn't have DNS!
```

**Solution**:
```bash
# Create user-defined network (has DNS)
docker network create mynet
docker run --network mynet --name app1 myapp1
docker run --network mynet --name app2 myapp2

# Now app1 can reach app2 by name
```

---

### LEVEL 2 - INTERMEDIATE: Port Mapping Works But Slow

**Scenario**: Container responds but very slowly.

```bash
$ curl http://localhost:8080
# Takes 5+ seconds for response
```

**Investigation**:
```bash
# Check docker-proxy (userland proxy)
ps aux | grep docker-proxy

# Check container resource usage
docker stats myapp

# Check if DNS is slow
docker exec myapp time nslookup somehost
```

---

# SECTION 3: BUILD ISSUES

---

## 3.1 BUILD FAILURES

### LEVEL 1 - DIRECT: Dockerfile Syntax Error

**Scenario**: Build fails with syntax error.

```bash
$ docker build .
Dockerfile parse error: unknown instruction: RRUN
```

**Cause is VISIBLE**: Typo in Dockerfile (RRUN instead of RUN).

**Solution**:
```dockerfile
# Fix the typo
RUN apt-get update
```

---

### LEVEL 1 - DIRECT: Base Image Not Found

**Scenario**: Build fails on FROM.

```bash
$ docker build .
Step 1/5 : FROM mybase:latest
Error: pull access denied
```

**Cause is VISIBLE**: Base image doesn't exist.

**Solution**:
```dockerfile
# Use correct base image
FROM node:18-alpine
```

---

### LEVEL 1 - DIRECT: COPY File Not Found

**Scenario**: COPY can't find file.

```bash
$ docker build .
Step 3/5 : COPY package.json .
COPY failed: file not found in build context
```

**Cause is VISIBLE**: File not in build context.

**Solution**:
```bash
# Check file exists
ls -la package.json

# Check .dockerignore isn't excluding it
cat .dockerignore

# Make sure building from correct directory
docker build -f Dockerfile .
```

---

### LEVEL 2 - INTERMEDIATE: Build Hangs

**Scenario**: Build hangs at a step, never completes.

**Investigation**:
```bash
# Check which step
# Likely causes:
# - Package install waiting for input
# - Infinite loop in script
# - Network timeout

# Make commands non-interactive
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y package

# Add timeouts
RUN timeout 300 npm install
```

---

### LEVEL 2 - INTERMEDIATE: Works Locally, Fails in CI

**Scenario**: Build works on laptop, fails in CI/CD.

**Investigation**:
```bash
# Common differences:
# - Network (proxy needed?)
# - Cache (CI has no cache)
# - Architecture (laptop arm64, CI amd64?)

# Add build args for proxy
docker build --build-arg HTTP_PROXY=$HTTP_PROXY .

# Check architecture
docker build --platform linux/amd64 .
```

---

## 3.2 IMAGE SIZE ISSUES

### LEVEL 1 - DIRECT: Image Too Large

**Scenario**: Simple app has 2GB image.

```bash
$ docker images
REPOSITORY   TAG       SIZE
myapp        latest    2.1GB
```

**Investigation**:
```bash
# Analyze layers
docker history myapp

# Find large layers
docker history myapp --format "{{.Size}}\t{{.CreatedBy}}" | sort -hr | head

# Use dive tool
dive myapp
```

**Solution**:
```dockerfile
# Multi-stage build
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production image
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/index.js"]
```

---

# SECTION 4: RESOURCE ISSUES

---

## 4.1 MEMORY ISSUES

### LEVEL 1 - DIRECT: Container OOMKilled

**Scenario**: Container keeps getting killed.

```bash
$ docker inspect myapp --format='{{.State.OOMKilled}}'
true
```

**Cause is VISIBLE**: Container exceeded memory limit.

**Solution**:
```bash
# Check current limit
docker inspect myapp --format='{{.HostConfig.Memory}}'

# Run with higher limit
docker run -m 1g myapp

# Or fix memory leak in application
```

---

### LEVEL 2 - INTERMEDIATE: Memory High But Not OOMKilled

**Scenario**: Container using lots of memory, running slow.

```bash
$ docker stats myapp
CONTAINER   MEM USAGE / LIMIT
myapp       490MiB / 512MiB
```

**Investigation**:
- Near limit = might be swapping or thrashing
- Application might be struggling

**Solution**:
```bash
# Increase limit
docker run -m 1g myapp

# Disable swap for container
docker run --memory-swap=-1 myapp
```

---

# QUICK REFERENCE

## Exit Codes

```
0   → Success (but shouldn't exit if long-running!)
1   → Generic application error
2   → Misuse of shell command
126 → Permission denied (can't execute)
127 → Command not found
128+N → Fatal signal N (137=SIGKILL, 139=SIGSEGV, 143=SIGTERM)
137 → Usually OOMKilled (SIGKILL)
139 → Segmentation fault (often binary compatibility)
143 → SIGTERM (graceful shutdown)
255 → Exit status out of range
```

## Quick Debug Commands

```bash
# Container status
docker ps -a                           # All containers
docker logs <container>                # Container logs
docker logs --tail 100 -f <container>  # Follow last 100 lines

# Container details
docker inspect <container>             # Full details
docker stats <container>               # Resource usage

# Interactive debug
docker exec -it <container> /bin/sh    # Shell into running container
docker run -it --entrypoint /bin/sh <image>  # Override entrypoint

# Image analysis
docker history <image>                 # Layer history
docker inspect <image>                 # Image details

# Network
docker network ls                      # List networks
docker network inspect <network>       # Network details

# Cleanup
docker system prune -a                 # Remove unused everything
docker volume prune                    # Remove unused volumes
```

## Common Errors → Solutions

```
"address already in use"      → Port conflict, use different port
"permission denied"           → chmod +x or run as root
"command not found"           → Binary missing or wrong PATH
"no such file or directory"   → File not in image or wrong path
"connection refused"          → Service not running or wrong port
"unauthorized"                → docker login required
"manifest not found"          → Wrong image tag
"OOMKilled"                   → Increase memory limit
```
