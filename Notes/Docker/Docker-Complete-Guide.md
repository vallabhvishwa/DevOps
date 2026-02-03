# Docker - Complete Deep Dive Guide
## Images, Containers, Dockerfile, Networking, Volumes, Security

---

# 1. UNDERSTANDING CONTAINERS

## What Is a Container?

```
CONTAINER = Isolated process with its own:
- Filesystem (from image)
- Network namespace
- Process namespace
- User namespace
- Resource limits (CPU, memory)

VM vs CONTAINER:
┌─────────────────────────────────────────────────────────────────┐
│ Virtual Machine:                                                │
│ ┌─────────┐ ┌─────────┐ ┌─────────┐                            │
│ │  App A  │ │  App B  │ │  App C  │                            │
│ ├─────────┤ ├─────────┤ ├─────────┤                            │
│ │Guest OS │ │Guest OS │ │Guest OS │  ← Each VM has full OS     │
│ └────┬────┘ └────┬────┘ └────┬────┘                            │
│      └──────────┬┴──────────┘                                   │
│           ┌─────┴─────┐                                         │
│           │ Hypervisor│                                         │
│           └─────┬─────┘                                         │
│           ┌─────┴─────┐                                         │
│           │  Host OS  │                                         │
│           └───────────┘                                         │
│                                                                 │
│ Container:                                                      │
│ ┌─────────┐ ┌─────────┐ ┌─────────┐                            │
│ │  App A  │ │  App B  │ │  App C  │                            │
│ └────┬────┘ └────┬────┘ └────┬────┘                            │
│      └──────────┬┴──────────┘                                   │
│           ┌─────┴─────┐                                         │
│           │  Docker   │  ← Containers share host kernel        │
│           └─────┬─────┘                                         │
│           ┌─────┴─────┐                                         │
│           │  Host OS  │                                         │
│           └───────────┘                                         │
└─────────────────────────────────────────────────────────────────┘
```

## Container Internals

```
LINUX FEATURES ENABLING CONTAINERS:

Namespaces (Isolation):
┌────────────────────────────────────────────────────────────────┐
│ PID namespace:   Isolated process tree                        │
│ NET namespace:   Isolated network stack                       │
│ MNT namespace:   Isolated filesystem mounts                   │
│ UTS namespace:   Isolated hostname                            │
│ IPC namespace:   Isolated inter-process communication         │
│ USER namespace:  Isolated user IDs                            │
└────────────────────────────────────────────────────────────────┘

Cgroups (Resource Limits):
┌────────────────────────────────────────────────────────────────┐
│ CPU:      Limit CPU usage (cores, shares)                     │
│ Memory:   Limit RAM usage                                     │
│ I/O:      Limit disk read/write                               │
│ Network:  Limit bandwidth                                     │
└────────────────────────────────────────────────────────────────┘

Union Filesystem (Layers):
┌────────────────────────────────────────────────────────────────┐
│ Container layer (writable)    ← Your changes                  │
│ App layer (read-only)         ← Application files             │
│ Dependencies layer            ← Libraries, runtime            │
│ Base image layer              ← OS files                      │
└────────────────────────────────────────────────────────────────┘
```

---

# 2. DOCKER IMAGES

## What Is an Image?

```
IMAGE = Read-only template for creating containers

IMAGE STRUCTURE:
┌─────────────────────────────────────────────────────────────────┐
│ Image Manifest:                                                 │
│ - List of layers                                               │
│ - Configuration (env vars, entrypoint, etc.)                   │
│ - Metadata (author, labels, etc.)                              │
│                                                                 │
│ Layers:                                                         │
│ - Each layer is a tarball of filesystem changes                │
│ - Layers are cached and shared                                 │
│ - Layers are immutable                                         │
└─────────────────────────────────────────────────────────────────┘
```

## Image Naming

```
IMAGE NAME STRUCTURE:
[registry/][namespace/]repository[:tag][@digest]

EXAMPLES:
nginx                              → library/nginx:latest from Docker Hub
nginx:1.24                         → Specific version
myregistry.azurecr.io/myapp:v1     → From Azure Container Registry
gcr.io/google-containers/pause    → From Google Container Registry

BEST PRACTICES:
✗ myapp:latest     → Ambiguous, changes over time
✓ myapp:v1.2.3     → Semantic versioning
✓ myapp:abc123     → Git commit SHA
✓ myapp:20240115   → Date-based
```

## Common Commands

```bash
# List images
docker images
docker image ls

# Pull image
docker pull nginx:1.24

# Build image
docker build -t myapp:v1 .

# Tag image
docker tag myapp:v1 myregistry.azurecr.io/myapp:v1

# Push image
docker push myregistry.azurecr.io/myapp:v1

# Remove image
docker rmi myapp:v1
docker image prune -a  # Remove all unused images

# Inspect image
docker image inspect nginx:1.24

# View image history (layers)
docker history nginx:1.24

# Save/Load images (for transfer)
docker save -o myapp.tar myapp:v1
docker load -i myapp.tar
```

---

# 3. DOCKERFILE - COMPLETE REFERENCE

## Basic Structure

```dockerfile
# Base image
FROM node:18-alpine

# Metadata
LABEL maintainer="devops@company.com"
LABEL version="1.0"

# Set working directory
WORKDIR /app

# Copy files
COPY package*.json ./
COPY src/ ./src/

# Run commands
RUN npm ci --only=production

# Environment variables
ENV NODE_ENV=production
ENV PORT=3000

# Expose port (documentation)
EXPOSE 3000

# Define user
USER node

# Health check
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# Entrypoint and Command
ENTRYPOINT ["node"]
CMD ["src/server.js"]
```

## Instructions Deep Dive

### FROM
```dockerfile
# Official image
FROM node:18-alpine

# Specific digest (immutable)
FROM node@sha256:abc123...

# Multi-stage build
FROM node:18 AS builder
# ... build steps ...
FROM node:18-alpine AS runtime
COPY --from=builder /app/dist /app
```

### COPY vs ADD
```dockerfile
# COPY - Simple file copy (preferred)
COPY file.txt /app/
COPY src/ /app/src/
COPY --chown=node:node . /app/

# ADD - Has extra features (use sparingly)
ADD archive.tar.gz /app/        # Auto-extracts archives
ADD https://example.com/file /  # Downloads from URL (not recommended)

# Best Practice: Use COPY unless you need ADD features
```

### RUN
```dockerfile
# Shell form (uses /bin/sh -c)
RUN apt-get update && apt-get install -y curl

# Exec form (no shell processing)
RUN ["apt-get", "update"]

# Best Practice: Chain commands to reduce layers
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      curl \
      ca-certificates && \
    rm -rf /var/lib/apt/lists/*  # Clean up to reduce size
```

### ENV vs ARG
```dockerfile
# ARG - Build-time variable (not in final image)
ARG VERSION=1.0
ARG BUILD_DATE

# ENV - Runtime environment variable (in final image)
ENV APP_VERSION=${VERSION}
ENV NODE_ENV=production

# Usage
docker build --build-arg VERSION=2.0 .
```

### ENTRYPOINT vs CMD
```dockerfile
# CMD only - Can be fully overridden
CMD ["npm", "start"]
# docker run myapp npm test  → runs "npm test"

# ENTRYPOINT only - Always runs
ENTRYPOINT ["npm"]
# docker run myapp start     → runs "npm start"
# docker run myapp test      → runs "npm test"

# ENTRYPOINT + CMD (Best Practice)
ENTRYPOINT ["node"]
CMD ["server.js"]
# docker run myapp           → runs "node server.js"
# docker run myapp app.js    → runs "node app.js"

# Exec form vs Shell form
ENTRYPOINT ["node", "app.js"]  # Exec form (PID 1 = node)
ENTRYPOINT node app.js         # Shell form (PID 1 = /bin/sh)
# Always use exec form for proper signal handling
```

### HEALTHCHECK
```dockerfile
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1

# Disable health check
HEALTHCHECK NONE

# Health check exit codes:
# 0 = healthy
# 1 = unhealthy
```

### USER
```dockerfile
# Create non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Switch to non-root user
USER appuser

# Or use numeric IDs
USER 1000:1000
```

---

# 4. MULTI-STAGE BUILDS

## Why Multi-Stage?

```
PROBLEM: Build tools inflate image size

Single stage:
┌────────────────────────────────────────────────────────────────┐
│ Final image contains:                                          │
│ - Source code                                                  │
│ - Build tools (gcc, npm dev deps, etc.)                       │
│ - Compiled application                                         │
│ = Large image (500MB+)                                         │
└────────────────────────────────────────────────────────────────┘

Multi-stage:
┌────────────────────────────────────────────────────────────────┐
│ Build stage: Has build tools, compiles code                   │
│ Runtime stage: Only has runtime + compiled code               │
│ = Small image (50-100MB)                                       │
└────────────────────────────────────────────────────────────────┘
```

## Examples

### Node.js Multi-Stage
```dockerfile
# Stage 1: Build
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Runtime
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package.json ./
USER node
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

### Java Multi-Stage
```dockerfile
# Stage 1: Build
FROM maven:3.9-eclipse-temurin-17 AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests

# Stage 2: Runtime
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
RUN addgroup -S spring && adduser -S spring -G spring
USER spring
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Go Multi-Stage (Scratch Image)
```dockerfile
# Stage 1: Build
FROM golang:1.21 AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o main .

# Stage 2: Minimal runtime
FROM scratch
COPY --from=builder /app/main /main
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
ENTRYPOINT ["/main"]
# Result: ~10MB image!
```

---

# 5. DOCKER NETWORKING

## Network Types

```
DOCKER NETWORK DRIVERS:

bridge (default):
┌────────────────────────────────────────────────────────────────┐
│ - Containers on same host can communicate                     │
│ - Isolated from host network                                  │
│ - NAT for external access                                     │
│ - Default network: docker0 bridge                             │
└────────────────────────────────────────────────────────────────┘

host:
┌────────────────────────────────────────────────────────────────┐
│ - Container uses host's network directly                      │
│ - No network isolation                                        │
│ - Best performance                                            │
│ - Port conflicts with host                                    │
└────────────────────────────────────────────────────────────────┘

none:
┌────────────────────────────────────────────────────────────────┐
│ - No network connectivity                                     │
│ - Complete isolation                                          │
│ - For security-sensitive workloads                            │
└────────────────────────────────────────────────────────────────┘

overlay:
┌────────────────────────────────────────────────────────────────┐
│ - Multi-host networking                                       │
│ - Used in Swarm/Kubernetes                                    │
│ - Encrypted by default in Swarm                               │
└────────────────────────────────────────────────────────────────┘
```

## Bridge Networking Deep Dive

```
DEFAULT BRIDGE vs USER-DEFINED BRIDGE:

Default bridge (docker0):
- Containers communicate via IP only
- No automatic DNS
- All containers connected by default

User-defined bridge (recommended):
- Automatic DNS (container name resolution)
- Better isolation
- Connect/disconnect without restart

# Create user-defined network
docker network create mynetwork

# Run containers on network
docker run -d --name web --network mynetwork nginx
docker run -d --name api --network mynetwork myapi

# 'web' can now reach 'api' by name:
# curl http://api:8080
```

## Port Mapping

```bash
# Map host port to container port
docker run -p 8080:80 nginx
# Host:8080 → Container:80

# Map to specific interface
docker run -p 127.0.0.1:8080:80 nginx
# Only accessible from localhost

# Map random host port
docker run -p 80 nginx
docker port <container>  # See assigned port

# Map all exposed ports
docker run -P nginx

# UDP port
docker run -p 53:53/udp dns-server
```

## DNS and Service Discovery

```
CONTAINER DNS:

User-defined networks:
┌────────────────────────────────────────────────────────────────┐
│ Container name → Container IP                                 │
│ Network alias  → Container IP                                 │
│                                                                │
│ # Create with alias                                           │
│ docker run --network mynet --network-alias db postgres        │
│                                                                │
│ # Multiple containers with same alias (round-robin)           │
│ docker run --network mynet --network-alias api myapi:v1       │
│ docker run --network mynet --network-alias api myapi:v2       │
└────────────────────────────────────────────────────────────────┘

DNS resolution order:
1. /etc/hosts entries
2. Docker's embedded DNS (127.0.0.11)
3. Host's DNS servers
```

---

# 6. DOCKER VOLUMES

## Volume Types

```
STORAGE OPTIONS:

Volumes (Managed by Docker):
┌────────────────────────────────────────────────────────────────┐
│ - Stored in Docker-managed directory                          │
│ - Best for persistent data                                    │
│ - Portable across hosts                                       │
│ - Can use volume drivers (NFS, cloud, etc.)                   │
└────────────────────────────────────────────────────────────────┘

Bind Mounts (Host directory):
┌────────────────────────────────────────────────────────────────┐
│ - Maps host directory into container                          │
│ - Good for development (live code reload)                     │
│ - Host-dependent                                              │
│ - Full host path required                                     │
└────────────────────────────────────────────────────────────────┘

tmpfs Mounts (Memory):
┌────────────────────────────────────────────────────────────────┐
│ - Stored in host memory only                                  │
│ - Fast but temporary                                          │
│ - Good for sensitive data                                     │
│ - Linux only                                                  │
└────────────────────────────────────────────────────────────────┘
```

## Commands

```bash
# Create volume
docker volume create mydata

# List volumes
docker volume ls

# Inspect volume
docker volume inspect mydata

# Use volume
docker run -v mydata:/app/data myapp
docker run --mount source=mydata,target=/app/data myapp

# Bind mount
docker run -v /host/path:/container/path myapp
docker run --mount type=bind,source=/host/path,target=/container/path myapp

# Read-only mount
docker run -v mydata:/app/data:ro myapp

# tmpfs mount
docker run --tmpfs /app/temp:size=100m myapp

# Remove unused volumes
docker volume prune
```

## Volume Best Practices

```dockerfile
# Declare volume in Dockerfile
VOLUME ["/data"]

# Best Practices:
# 1. Use named volumes for persistence
# 2. Use bind mounts for development
# 3. Use tmpfs for sensitive temporary data
# 4. Use :ro for read-only when possible
# 5. Backup volumes regularly
```

---

# 7. DOCKER SECURITY

## Security Best Practices

### Run as Non-Root
```dockerfile
# Create non-root user
RUN addgroup -g 1000 appgroup && \
    adduser -u 1000 -G appgroup -D appuser

# Set ownership
COPY --chown=appuser:appgroup . /app

# Switch to non-root
USER appuser
```

### Use Minimal Base Images
```dockerfile
# ❌ Full OS (large attack surface)
FROM ubuntu:22.04

# ✓ Minimal images
FROM alpine:3.18           # ~5MB
FROM gcr.io/distroless/base  # No shell, minimal
FROM scratch               # Empty (for static binaries)
```

### Don't Store Secrets in Images
```dockerfile
# ❌ Bad - secret in image layer
COPY credentials.json /app/
ENV API_KEY=secret123

# ✓ Good - use secrets at runtime
# docker run -e API_KEY=$API_KEY myapp
# docker run --secret id=mykey myapp
```

### Scan for Vulnerabilities
```bash
# Docker Scout (built-in)
docker scout cve myimage:tag

# Trivy
trivy image myimage:tag

# Snyk
snyk container test myimage:tag
```

### Read-Only Filesystem
```bash
# Run with read-only filesystem
docker run --read-only myapp

# Allow specific writable directories
docker run --read-only --tmpfs /tmp myapp
```

### Resource Limits
```bash
# Limit memory
docker run --memory=512m myapp

# Limit CPU
docker run --cpus=1.5 myapp

# Prevent fork bombs
docker run --pids-limit=100 myapp
```

### Security Options
```bash
# Drop all capabilities, add only needed
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myapp

# No new privileges
docker run --security-opt=no-new-privileges myapp

# Seccomp profile
docker run --security-opt seccomp=profile.json myapp

# AppArmor profile
docker run --security-opt apparmor=docker-default myapp
```

---

# 8. IMAGE OPTIMIZATION

## Reduce Image Size

```dockerfile
# 1. Use multi-stage builds
FROM node:18 AS builder
# ... build ...
FROM node:18-alpine AS runtime

# 2. Use smaller base images
FROM alpine:3.18           # 5MB vs Ubuntu 80MB
FROM node:18-alpine        # 170MB vs node:18 1GB

# 3. Minimize layers
# ❌ Bad
RUN apt-get update
RUN apt-get install curl
RUN apt-get install vim

# ✓ Good
RUN apt-get update && \
    apt-get install -y curl vim && \
    rm -rf /var/lib/apt/lists/*

# 4. Use .dockerignore
# .dockerignore file:
node_modules
.git
*.md
Dockerfile
.dockerignore

# 5. Order layers by change frequency
COPY package.json .         # Changes rarely
RUN npm install             # Cached if package.json unchanged
COPY src/ ./src/           # Changes often (rebuild from here)

# 6. Clean up in same layer
RUN apt-get update && \
    apt-get install -y build-essential && \
    make && \
    apt-get purge -y build-essential && \
    rm -rf /var/lib/apt/lists/*
```

## Layer Caching

```
DOCKER BUILD CACHE:

Layer 1: FROM node:18           ← Cached
Layer 2: COPY package.json      ← Cached (if unchanged)
Layer 3: RUN npm install        ← Cached (if Layer 2 cached)
Layer 4: COPY src/ ./src/       ← REBUILD (file changed)
Layer 5: RUN npm build          ← REBUILD (Layer 4 rebuilt)

CACHE BUSTING:
- Any change invalidates that layer and all following
- Order instructions from least to most frequently changed
```

---

# 9. DOCKER COMPOSE

## Basic Structure

```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build: ./web
    ports:
      - "80:80"
    depends_on:
      - api
    environment:
      - API_URL=http://api:8080
    networks:
      - frontend
      - backend

  api:
    image: myapi:latest
    environment:
      - DATABASE_URL=postgres://db:5432/mydb
    depends_on:
      db:
        condition: service_healthy
    networks:
      - backend

  db:
    image: postgres:15
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    secrets:
      - db_password
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend

volumes:
  db-data:

networks:
  frontend:
  backend:

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

## Commands

```bash
# Start services
docker compose up -d

# View logs
docker compose logs -f

# Stop services
docker compose down

# Stop and remove volumes
docker compose down -v

# Rebuild images
docker compose build
docker compose up -d --build

# Scale service
docker compose up -d --scale api=3

# Execute command in service
docker compose exec api sh
```

---

# 10. COMMON COMMANDS REFERENCE

```bash
# CONTAINER LIFECYCLE
docker run -d --name mycontainer myimage     # Create and start
docker start mycontainer                      # Start stopped
docker stop mycontainer                       # Stop gracefully
docker kill mycontainer                       # Force stop
docker restart mycontainer                    # Restart
docker rm mycontainer                         # Remove
docker rm -f mycontainer                      # Force remove running

# INSPECTION
docker ps                                     # Running containers
docker ps -a                                  # All containers
docker logs mycontainer                       # View logs
docker logs -f --tail 100 mycontainer        # Follow last 100 lines
docker inspect mycontainer                    # Full details
docker stats                                  # Resource usage
docker top mycontainer                        # Running processes

# INTERACTION
docker exec -it mycontainer sh               # Shell into container
docker exec mycontainer cat /etc/hosts       # Run command
docker cp mycontainer:/app/log.txt ./        # Copy from container
docker cp ./file.txt mycontainer:/app/       # Copy to container

# CLEANUP
docker system prune                           # Remove unused data
docker system prune -a --volumes             # Remove everything unused
docker container prune                        # Remove stopped containers
docker image prune -a                         # Remove unused images
docker volume prune                           # Remove unused volumes
docker network prune                          # Remove unused networks
```
