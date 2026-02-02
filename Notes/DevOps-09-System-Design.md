# DevOps Engineer's Complete Reference Guide
# Part 9: System Design for Infrastructure

---

## Table of Contents

1. [Introduction to System Design](#1-introduction-to-system-design)
2. [Core Design Principles](#2-core-design-principles)
3. [Scalability Patterns](#3-scalability-patterns)
4. [High Availability Design](#4-high-availability-design)
5. [Load Balancing Strategies](#5-load-balancing-strategies)
6. [Caching Strategies](#6-caching-strategies)
7. [Database Design Patterns](#7-database-design-patterns)
8. [Microservices Architecture](#8-microservices-architecture)
9. [Event-Driven Architecture](#9-event-driven-architecture)
10. [CDN and Edge Computing](#10-cdn-and-edge-computing)
11. [Disaster Recovery](#11-disaster-recovery)
12. [Capacity Planning](#12-capacity-planning)
13. [System Design Interview Problems](#13-system-design-interview-problems)
14. [Real-World Architecture Examples](#14-real-world-architecture-examples)

---

## 1. Introduction to System Design

### 1.1 What is System Design?

System design is the process of defining the architecture, components, modules, interfaces, and data flow of a system to satisfy specified requirements.

```
System Design Scope:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Functional Requirements:                                       │
│  ├── What the system should do                                  │
│  ├── Features and capabilities                                  │
│  └── User interactions                                          │
│                                                                 │
│  Non-Functional Requirements:                                   │
│  ├── Scalability      - Handle growth                           │
│  ├── Availability     - Uptime and reliability                  │
│  ├── Performance      - Response time, throughput               │
│  ├── Durability       - Data persistence                        │
│  ├── Consistency      - Data correctness                        │
│  ├── Security         - Protection and compliance               │
│  ├── Maintainability  - Ease of updates                         │
│  └── Cost             - Budget constraints                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 The System Design Process

```
System Design Process:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. UNDERSTAND REQUIREMENTS                                     │
│     ├── Clarify functional requirements                         │
│     ├── Identify non-functional requirements                    │
│     ├── Define scope and constraints                            │
│     └── Identify key metrics (users, requests, data size)       │
│                                                                 │
│  2. ESTIMATE SCALE                                              │
│     ├── Number of users (DAU, MAU)                              │
│     ├── Read/Write ratio                                        │
│     ├── Storage requirements                                    │
│     ├── Bandwidth requirements                                  │
│     └── QPS (Queries Per Second)                                │
│                                                                 │
│  3. DEFINE HIGH-LEVEL DESIGN                                    │
│     ├── Core components                                         │
│     ├── Data flow                                               │
│     ├── API design                                              │
│     └── Database schema                                         │
│                                                                 │
│  4. DEEP DIVE INTO COMPONENTS                                   │
│     ├── Database design                                         │
│     ├── Caching strategy                                        │
│     ├── Load balancing                                          │
│     └── Message queues                                          │
│                                                                 │
│  5. ADDRESS BOTTLENECKS                                         │
│     ├── Identify single points of failure                       │
│     ├── Address scaling challenges                              │
│     ├── Optimize for performance                                │
│     └── Plan for failures                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.3 Back-of-the-Envelope Calculations

Essential numbers to memorize:

```
Power of 2:
┌─────────────────────────────────────────────────────────────────┐
│  2^10 = 1 Thousand     = 1 KB                                   │
│  2^20 = 1 Million      = 1 MB                                   │
│  2^30 = 1 Billion      = 1 GB                                   │
│  2^40 = 1 Trillion     = 1 TB                                   │
│  2^50 = 1 Quadrillion  = 1 PB                                   │
└─────────────────────────────────────────────────────────────────┘

Latency Numbers (Approximate):
┌─────────────────────────────────────────────────────────────────┐
│  L1 cache reference             =     0.5 ns                    │
│  L2 cache reference             =       7 ns                    │
│  Main memory reference          =     100 ns                    │
│  SSD random read                =  16,000 ns = 16 μs            │
│  HDD seek                       = 2,000,000 ns = 2 ms           │
│  Round trip within data center  =   500,000 ns = 0.5 ms         │
│  Round trip CA to Netherlands   = 150,000,000 ns = 150 ms       │
└─────────────────────────────────────────────────────────────────┘

Common Throughput:
┌─────────────────────────────────────────────────────────────────┐
│  Sequential read from SSD        = 1 GB/s                       │
│  Sequential read from HDD        = 100 MB/s                     │
│  Network within data center      = 10 Gbps                      │
│  Typical database query          = 1-10 ms                      │
│  Web API call                    = 50-200 ms                    │
└─────────────────────────────────────────────────────────────────┘

Time Conversions:
┌─────────────────────────────────────────────────────────────────┐
│  1 day     = 86,400 seconds ≈ 100,000 seconds                   │
│  1 week    = 604,800 seconds ≈ 600,000 seconds                  │
│  1 month   = 2,592,000 seconds ≈ 2.5 million seconds            │
│  1 year    = 31,536,000 seconds ≈ 30 million seconds            │
└─────────────────────────────────────────────────────────────────┘
```

**Example Calculation:**

```
Scenario: Design Twitter

Given:
- 300 million monthly active users (MAU)
- 50% are daily active (DAU) = 150 million
- Each user makes 2 tweets per day (write)
- Each user reads 100 tweets per day (read)

Calculations:

Write QPS:
- Tweets per day = 150M × 2 = 300M tweets/day
- Write QPS = 300M / 86,400 = ~3,500 QPS
- Peak (2x average) = 7,000 QPS

Read QPS:
- Reads per day = 150M × 100 = 15B reads/day
- Read QPS = 15B / 86,400 = ~175,000 QPS
- Peak = 350,000 QPS

Storage (per year):
- Tweet size = 280 chars × 2 bytes = 560 bytes
- Metadata = 200 bytes
- Media (30% have image) = 0.3 × 500KB = 150KB average
- Average tweet = 560 + 200 + 150,000 = ~150KB
- Daily storage = 300M × 150KB = 45 TB/day
- Yearly storage = 45 TB × 365 = 16.4 PB/year

Bandwidth:
- Write = 3,500 QPS × 150KB = 525 MB/s = 4.2 Gbps
- Read = 175,000 QPS × 150KB = 26 GB/s = 208 Gbps
```

---

## 2. Core Design Principles

### 2.1 CAP Theorem

The CAP theorem states that a distributed system can only guarantee two of three properties:

```
CAP Theorem:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│                      Consistency                                │
│                          /\                                     │
│                         /  \                                    │
│                        /    \                                   │
│                       / CP   \                                  │
│                      /  Zone  \                                 │
│                     /          \                                │
│                    /     CA     \                               │
│                   /    (Single   \                              │
│                  /     Node)      \                             │
│                 /                  \                            │
│                /        AP          \                           │
│               /        Zone          \                          │
│              /__________________________\                       │
│         Availability              Partition                     │
│                                   Tolerance                     │
│                                                                 │
│  Consistency: All nodes see the same data at the same time     │
│  Availability: Every request receives a response               │
│  Partition Tolerance: System works despite network failures    │
│                                                                 │
│  Examples:                                                      │
│  CP: MongoDB, HBase, Redis (cluster) - May reject writes       │
│  AP: Cassandra, DynamoDB, CouchDB - May return stale data      │
│  CA: Single-node RDBMS - No partition tolerance                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 ACID vs BASE

```
ACID (Traditional RDBMS):
┌─────────────────────────────────────────────────────────────────┐
│  Atomicity    - All or nothing transactions                    │
│  Consistency  - Database always in valid state                 │
│  Isolation    - Concurrent transactions don't interfere        │
│  Durability   - Committed data is permanent                    │
│                                                                 │
│  Use when: Financial transactions, critical data               │
│  Examples: PostgreSQL, MySQL, SQL Server                        │
└─────────────────────────────────────────────────────────────────┘

BASE (NoSQL Systems):
┌─────────────────────────────────────────────────────────────────┐
│  Basically Available - System always responds                  │
│  Soft state          - State may change without input          │
│  Eventually consistent - Data converges over time              │
│                                                                 │
│  Use when: High availability, massive scale, eventual OK       │
│  Examples: Cassandra, DynamoDB, Couchbase                       │
└─────────────────────────────────────────────────────────────────┘
```

### 2.3 Consistency Models

```
Consistency Spectrum:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Strong ◄──────────────────────────────────────────────► Weak  │
│                                                                 │
│  Linearizability                                                │
│  │ └── Real-time ordering, most strict                         │
│  ▼                                                              │
│  Sequential Consistency                                         │
│  │ └── Operations appear in same order to all                  │
│  ▼                                                              │
│  Causal Consistency                                             │
│  │ └── Causally related operations are ordered                 │
│  ▼                                                              │
│  Read-Your-Writes                                               │
│  │ └── User sees their own writes immediately                  │
│  ▼                                                              │
│  Monotonic Reads                                                │
│  │ └── Never see older data after seeing newer                 │
│  ▼                                                              │
│  Eventual Consistency                                           │
│    └── All replicas converge eventually                        │
│                                                                 │
│  Trade-off: Stronger consistency = Lower availability/perf     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2.4 Single Point of Failure (SPOF)

```
Identifying and Eliminating SPOFs:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Common SPOFs:                    Solutions:                    │
│  ├── Single database server    → Primary/Replica + Failover    │
│  ├── Single web server         → Load balancer + Multiple      │
│  ├── Single data center        → Multi-region deployment       │
│  ├── Single DNS provider       → Multiple DNS providers        │
│  ├── Single load balancer      → Active-passive LB pair        │
│  ├── Single network path       → Redundant network paths       │
│  └── Single admin              → Team with shared access       │
│                                                                 │
│  Design Pattern: N+1 Redundancy                                │
│  └── Always have at least one extra of critical components     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Scalability Patterns

### 3.1 Vertical vs Horizontal Scaling

```
Vertical Scaling (Scale Up):
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Before:              After:                                    │
│  ┌─────────┐          ┌─────────────┐                           │
│  │ 4 CPU   │    →     │ 32 CPU      │                           │
│  │ 16 GB   │          │ 256 GB      │                           │
│  │ 500 GB  │          │ 4 TB SSD    │                           │
│  └─────────┘          └─────────────┘                           │
│                                                                 │
│  Pros:                                                          │
│  ├── Simple - no code changes                                   │
│  ├── No distributed system complexity                           │
│  └── Works for databases (easier than sharding)                 │
│                                                                 │
│  Cons:                                                          │
│  ├── Hardware limits (can't scale infinitely)                   │
│  ├── Single point of failure                                    │
│  ├── Expensive at high end                                      │
│  └── Downtime for upgrades                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

Horizontal Scaling (Scale Out):
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Before:              After:                                    │
│  ┌─────────┐          ┌─────────┐ ┌─────────┐ ┌─────────┐       │
│  │ Server  │    →     │ Server  │ │ Server  │ │ Server  │       │
│  └─────────┘          └─────────┘ └─────────┘ └─────────┘       │
│                              ▲                                  │
│                       Load Balancer                             │
│                                                                 │
│  Pros:                                                          │
│  ├── Near-infinite scalability                                  │
│  ├── No single point of failure                                 │
│  ├── Cost-effective (commodity hardware)                        │
│  └── Can scale incrementally                                    │
│                                                                 │
│  Cons:                                                          │
│  ├── Distributed system complexity                              │
│  ├── Data consistency challenges                                │
│  ├── Requires stateless design                                  │
│  └── Load balancing complexity                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 Stateless vs Stateful Services

```
Stateless Design (Preferred for Scaling):
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Characteristics:                                               │
│  ├── No local state between requests                            │
│  ├── All state stored externally (database, cache)              │
│  ├── Any instance can handle any request                        │
│  └── Easy to scale horizontally                                 │
│                                                                 │
│  Implementation:                                                │
│  ├── Store sessions in Redis/database                           │
│  ├── Use JWT for authentication                                 │
│  ├── Store uploads in object storage (S3, Blob)                 │
│  └── Use external cache (Redis, Memcached)                      │
│                                                                 │
│  Example Architecture:                                          │
│                                                                 │
│       ┌─────────────────────────────────────────┐               │
│       │            Load Balancer                │               │
│       └─────────────────────────────────────────┘               │
│                        │                                        │
│       ┌────────────────┼────────────────┐                       │
│       ▼                ▼                ▼                       │
│  ┌─────────┐      ┌─────────┐      ┌─────────┐                  │
│  │ App 1   │      │ App 2   │      │ App 3   │  ← Stateless     │
│  └─────────┘      └─────────┘      └─────────┘                  │
│       │                │                │                       │
│       └────────────────┼────────────────┘                       │
│                        ▼                                        │
│       ┌─────────────────────────────────────────┐               │
│       │  Redis (Sessions) │ Database │ S3      │  ← State       │
│       └─────────────────────────────────────────┘               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 3.3 Database Scaling Patterns

```
Database Scaling Strategies:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. READ REPLICAS                                               │
│     ┌─────────┐                                                 │
│     │ Primary │───Write──────────────────┐                      │
│     └────┬────┘                          │                      │
│          │ Replication                   │                      │
│          ▼                               ▼                      │
│     ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐             │
│     │Replica 1│ │Replica 2│ │Replica 3│ │ App     │             │
│     └────┬────┘ └────┬────┘ └────┬────┘ └─────────┘             │
│          └───────────┼───────────┘                              │
│                      │                                          │
│          ◄───────────┴───── Read ──────────────────             │
│                                                                 │
│  2. SHARDING (Horizontal Partitioning)                          │
│                                                                 │
│     Users 1-1M    Users 1M-2M    Users 2M-3M                    │
│     ┌─────────┐   ┌─────────┐    ┌─────────┐                    │
│     │ Shard 1 │   │ Shard 2 │    │ Shard 3 │                    │
│     └─────────┘   └─────────┘    └─────────┘                    │
│                                                                 │
│     Sharding Strategies:                                        │
│     ├── Range-based: user_id 1-1M, 1M-2M, etc.                  │
│     ├── Hash-based: hash(user_id) % num_shards                  │
│     ├── Directory-based: Lookup table for shard                 │
│     └── Geo-based: By user location                             │
│                                                                 │
│  3. VERTICAL PARTITIONING                                       │
│     Split by feature/table:                                     │
│     ├── User DB: profiles, auth                                 │
│     ├── Product DB: catalog, inventory                          │
│     └── Order DB: orders, payments                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 3.4 Auto-Scaling

```
Auto-Scaling in Kubernetes (HPA + VPA + Cluster Autoscaler):
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Horizontal Pod Autoscaler (HPA):                               │
│  ├── Scales number of pods                                      │
│  ├── Based on CPU, memory, custom metrics                       │
│  └── Best for stateless workloads                               │
│                                                                 │
│  apiVersion: autoscaling/v2                                     │
│  kind: HorizontalPodAutoscaler                                  │
│  spec:                                                          │
│    scaleTargetRef:                                              │
│      apiVersion: apps/v1                                        │
│      kind: Deployment                                           │
│      name: myapp                                                │
│    minReplicas: 2                                               │
│    maxReplicas: 20                                              │
│    metrics:                                                     │
│    - type: Resource                                             │
│      resource:                                                  │
│        name: cpu                                                │
│        target:                                                  │
│          type: Utilization                                      │
│          averageUtilization: 70                                 │
│    - type: Pods                                                 │
│      pods:                                                      │
│        metric:                                                  │
│          name: requests_per_second                              │
│        target:                                                  │
│          type: AverageValue                                     │
│          averageValue: "1000"                                   │
│                                                                 │
│  Vertical Pod Autoscaler (VPA):                                 │
│  ├── Adjusts CPU/memory requests                                │
│  ├── Learns from actual usage                                   │
│  └── Can restart pods to apply changes                          │
│                                                                 │
│  Cluster Autoscaler:                                            │
│  ├── Adds/removes nodes                                         │
│  ├── Works with cloud node pools                                │
│  └── Based on pending pods                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. High Availability Design

### 4.1 Availability Calculation

```
Availability Levels:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Availability   Downtime/Year    Downtime/Month   Downtime/Week│
│  ───────────────────────────────────────────────────────────────│
│  90%            36.5 days        3 days           16.8 hours   │
│  99%            3.65 days        7.2 hours        1.68 hours   │
│  99.9%          8.76 hours       43.2 minutes     10.1 minutes │
│  99.99%         52.6 minutes     4.32 minutes     1.01 minutes │
│  99.999%        5.26 minutes     26 seconds       6 seconds    │
│                                                                 │
│  Formula: Availability = Uptime / (Uptime + Downtime)          │
│                                                                 │
│  Series Components (all must work):                             │
│  A_total = A1 × A2 × A3                                        │
│  Example: 99.9% × 99.9% × 99.9% = 99.7%                        │
│                                                                 │
│  Parallel Components (any can work):                            │
│  A_total = 1 - (1-A1) × (1-A2)                                 │
│  Example: 1 - (0.001 × 0.001) = 99.9999%                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 High Availability Patterns

```
HA Pattern: Active-Active
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│           ┌─────────────────────────────────────────┐           │
│           │         Global Load Balancer            │           │
│           └─────────────────────────────────────────┘           │
│                    │                    │                       │
│           ┌────────┴────────┐  ┌────────┴────────┐              │
│           │   Region A      │  │   Region B      │              │
│           │   (Active)      │  │   (Active)      │              │
│           │                 │  │                 │              │
│           │  ┌───────────┐  │  │  ┌───────────┐  │              │
│           │  │    App    │  │  │  │    App    │  │              │
│           │  └───────────┘  │  │  └───────────┘  │              │
│           │       │         │  │       │         │              │
│           │  ┌───────────┐  │  │  ┌───────────┐  │              │
│           │  │ Database  │◄─┼──┼──│ Database  │  │              │
│           │  │ (Primary) │──┼──┼─►│ (Primary) │  │              │
│           │  └───────────┘  │  │  └───────────┘  │              │
│           └─────────────────┘  └─────────────────┘              │
│                                                                 │
│  Pros: Full capacity in both regions, instant failover          │
│  Cons: Data sync complexity, conflict resolution needed         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

HA Pattern: Active-Passive
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│           ┌─────────────────────────────────────────┐           │
│           │         Global Load Balancer            │           │
│           └─────────────────────────────────────────┘           │
│                    │                                            │
│           ┌────────┴────────┐  ┌─────────────────────┐          │
│           │   Region A      │  │   Region B          │          │
│           │   (Active)      │  │   (Passive/Standby) │          │
│           │                 │  │                     │          │
│           │  ┌───────────┐  │  │  ┌───────────────┐  │          │
│           │  │    App    │  │  │  │  App (idle)   │  │          │
│           │  └───────────┘  │  │  └───────────────┘  │          │
│           │       │         │  │         │           │          │
│           │  ┌───────────┐  │  │  ┌───────────────┐  │          │
│           │  │ Database  │──┼──┼─►│   Database    │  │          │
│           │  │ (Primary) │  │  │  │   (Replica)   │  │          │
│           │  └───────────┘  │  │  └───────────────┘  │          │
│           └─────────────────┘  └─────────────────────┘          │
│                                                                 │
│  Pros: Simpler data sync, no conflicts                          │
│  Cons: Wasted capacity, failover delay                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.3 AKS High Availability

```
AKS High Availability Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Multi-Zone AKS Cluster:                                        │
│                                                                 │
│           ┌─────────────────────────────────────────┐           │
│           │          Azure Load Balancer            │           │
│           └─────────────────────────────────────────┘           │
│                    │          │          │                      │
│           ┌────────┼──────────┼──────────┼────────┐             │
│           │        ▼          ▼          ▼        │             │
│           │   Zone 1     Zone 2     Zone 3        │             │
│           │   ┌─────┐    ┌─────┐    ┌─────┐       │             │
│           │   │Node1│    │Node3│    │Node5│       │             │
│           │   │Node2│    │Node4│    │Node6│       │             │
│           │   └─────┘    └─────┘    └─────┘       │             │
│           │                                       │             │
│           │   AKS Cluster (zone-redundant)        │             │
│           └───────────────────────────────────────┘             │
│                                                                 │
│  Pod Anti-Affinity for zone distribution:                       │
│                                                                 │
│  affinity:                                                      │
│    podAntiAffinity:                                             │
│      requiredDuringSchedulingIgnoredDuringExecution:            │
│      - labelSelector:                                           │
│          matchLabels:                                           │
│            app: myapp                                           │
│        topologyKey: topology.kubernetes.io/zone                 │
│                                                                 │
│  Zone-Redundant Storage:                                        │
│  storageClassName: managed-premium-zrs                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.4 Health Checks and Failover

```
Health Check Types:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. LIVENESS PROBE                                              │
│     └── Is the container alive?                                 │
│     └── Failure: Restart container                              │
│                                                                 │
│     livenessProbe:                                              │
│       httpGet:                                                  │
│         path: /healthz                                          │
│         port: 8080                                              │
│       initialDelaySeconds: 30                                   │
│       periodSeconds: 10                                         │
│       failureThreshold: 3                                       │
│                                                                 │
│  2. READINESS PROBE                                             │
│     └── Is the container ready for traffic?                     │
│     └── Failure: Remove from service endpoints                  │
│                                                                 │
│     readinessProbe:                                             │
│       httpGet:                                                  │
│         path: /ready                                            │
│         port: 8080                                              │
│       initialDelaySeconds: 5                                    │
│       periodSeconds: 5                                          │
│       failureThreshold: 3                                       │
│                                                                 │
│  3. STARTUP PROBE                                               │
│     └── For slow-starting containers                            │
│     └── Disables other probes until success                     │
│                                                                 │
│     startupProbe:                                               │
│       httpGet:                                                  │
│         path: /healthz                                          │
│         port: 8080                                              │
│       failureThreshold: 30                                      │
│       periodSeconds: 10  # 30 × 10s = 5 min startup             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Load Balancing Strategies

### 5.1 Load Balancing Layers

```
Load Balancing at Different Layers:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Layer 4 (Transport - TCP/UDP):                                 │
│  ├── Fast, simple                                               │
│  ├── Based on IP and port                                       │
│  ├── No content inspection                                      │
│  └── Examples: Azure LB, AWS NLB, HAProxy (TCP mode)            │
│                                                                 │
│  Layer 7 (Application - HTTP/HTTPS):                            │
│  ├── Content-aware routing                                      │
│  ├── URL path routing, host routing                             │
│  ├── SSL termination                                            │
│  ├── Request manipulation                                       │
│  └── Examples: Azure App Gateway, AWS ALB, NGINX, Istio         │
│                                                                 │
│  DNS Load Balancing:                                            │
│  ├── Geographic routing                                         │
│  ├── Latency-based routing                                      │
│  ├── Health-check based                                         │
│  └── Examples: Azure Traffic Manager, AWS Route 53              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 Load Balancing Algorithms

```
Load Balancing Algorithms:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. ROUND ROBIN                                                 │
│     Requests: 1→A, 2→B, 3→C, 4→A, 5→B, 6→C...                  │
│     Pros: Simple, even distribution                             │
│     Cons: Ignores server capacity and current load              │
│                                                                 │
│  2. WEIGHTED ROUND ROBIN                                        │
│     A(weight=3), B(weight=2), C(weight=1)                       │
│     Requests: A,A,A,B,B,C, A,A,A,B,B,C...                       │
│     Pros: Accounts for different server capacities              │
│                                                                 │
│  3. LEAST CONNECTIONS                                           │
│     Route to server with fewest active connections              │
│     Pros: Better for long-lived connections                     │
│     Cons: Doesn't account for connection weight                 │
│                                                                 │
│  4. WEIGHTED LEAST CONNECTIONS                                  │
│     Combines weight and connection count                        │
│     Score = Connections / Weight                                │
│                                                                 │
│  5. IP HASH                                                     │
│     Server = hash(client_ip) % num_servers                      │
│     Pros: Session stickiness without cookies                    │
│     Cons: Uneven distribution if many clients behind NAT        │
│                                                                 │
│  6. LEAST RESPONSE TIME                                         │
│     Route to server with lowest average response time           │
│     Pros: Optimizes user experience                             │
│     Cons: Requires monitoring, cold start issues                │
│                                                                 │
│  7. RANDOM                                                      │
│     Randomly select a server                                    │
│     Pros: No state needed                                       │
│     Cons: Can create hotspots with few requests                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.3 Session Persistence

```
Session Persistence Strategies:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. CLIENT IP AFFINITY                                          │
│     └── Same IP always goes to same server                      │
│     └── Problem: NAT, mobile networks                           │
│                                                                 │
│  2. COOKIE-BASED AFFINITY                                       │
│     └── LB sets cookie with server identifier                   │
│     └── Subsequent requests use cookie for routing              │
│                                                                 │
│  3. SESSION REPLICATION                                         │
│     └── Sessions copied to all servers                          │
│     └── Any server can handle any request                       │
│     └── High overhead, limited scalability                      │
│                                                                 │
│  4. CENTRALIZED SESSION STORE (Recommended)                     │
│     └── Store sessions in Redis/database                        │
│     └── Stateless servers, sticky sessions not needed           │
│                                                                 │
│     ┌─────────┐   ┌─────────┐   ┌─────────┐                     │
│     │ App 1   │   │ App 2   │   │ App 3   │                     │
│     └────┬────┘   └────┬────┘   └────┬────┘                     │
│          └─────────────┼─────────────┘                          │
│                        ▼                                        │
│                  ┌─────────┐                                    │
│                  │  Redis  │ ← Session Store                    │
│                  └─────────┘                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. Caching Strategies

### 6.1 Cache Placement

```
Cache Locations:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Client ──▶ CDN ──▶ Load Balancer ──▶ App ──▶ Cache ──▶ DB     │
│                                                                 │
│  1. CLIENT-SIDE CACHE                                           │
│     ├── Browser cache (HTTP headers)                            │
│     ├── Service Worker cache                                    │
│     └── Mobile app cache                                        │
│                                                                 │
│  2. CDN (Edge Cache)                                            │
│     ├── Static assets                                           │
│     ├── API responses (with care)                               │
│     └── Geographic distribution                                 │
│                                                                 │
│  3. REVERSE PROXY CACHE                                         │
│     ├── NGINX, Varnish                                          │
│     └── Full page caching                                       │
│                                                                 │
│  4. APPLICATION CACHE                                           │
│     ├── In-process (local memory)                               │
│     └── Distributed (Redis, Memcached)                          │
│                                                                 │
│  5. DATABASE CACHE                                              │
│     ├── Query cache                                             │
│     └── Buffer pool                                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 6.2 Caching Patterns

```
Cache-Aside (Lazy Loading):
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Read:                                                          │
│  1. App checks cache                                            │
│  2. If miss, app reads from DB                                  │
│  3. App writes to cache                                         │
│  4. App returns data                                            │
│                                                                 │
│  ┌─────┐     1. Get     ┌───────┐                               │
│  │ App │ ────────────►  │ Cache │                               │
│  │     │ ◄──── Miss ─── │       │                               │
│  └──┬──┘                └───────┘                               │
│     │ 2. Get from DB                                            │
│     ▼                                                           │
│  ┌──────┐   3. Write to Cache                                   │
│  │  DB  │ ───────────────────────►                              │
│  └──────┘                                                       │
│                                                                 │
│  Pros: Only cache what's needed, resilient to cache failure     │
│  Cons: Cache miss penalty, potential stale data                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

Write-Through:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Write:                                                         │
│  1. App writes to cache                                         │
│  2. Cache writes to DB (synchronously)                          │
│  3. Return success                                              │
│                                                                 │
│  ┌─────┐    Write     ┌───────┐    Write    ┌──────┐            │
│  │ App │ ───────────► │ Cache │ ──────────► │  DB  │            │
│  │     │ ◄── ACK ──── │       │ ◄── ACK ─── │      │            │
│  └─────┘              └───────┘             └──────┘            │
│                                                                 │
│  Pros: Cache always consistent with DB                          │
│  Cons: Write latency, cache fills with unread data              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

Write-Behind (Write-Back):
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Write:                                                         │
│  1. App writes to cache                                         │
│  2. Cache acknowledges immediately                              │
│  3. Cache writes to DB asynchronously                           │
│                                                                 │
│  ┌─────┐    Write     ┌───────┐   Async    ┌──────┐             │
│  │ App │ ───────────► │ Cache │ ─ ─ ─ ─ ─► │  DB  │             │
│  │     │ ◄── ACK ──── │       │            │      │             │
│  └─────┘              └───────┘            └──────┘             │
│                                                                 │
│  Pros: Fast writes, batching possible                           │
│  Cons: Data loss risk if cache fails, complexity                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 6.3 Cache Invalidation

```
Cache Invalidation Strategies:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. TIME-TO-LIVE (TTL)                                          │
│     └── Automatic expiration after set time                     │
│     └── Simple but may serve stale data                         │
│                                                                 │
│     redis.setex("user:123", 3600, user_data)  # 1 hour TTL      │
│                                                                 │
│  2. EVENT-BASED INVALIDATION                                    │
│     └── Invalidate on data change                               │
│     └── Accurate but complex                                    │
│                                                                 │
│     def update_user(user_id, data):                             │
│         db.update(user_id, data)                                │
│         redis.delete(f"user:{user_id}")                         │
│                                                                 │
│  3. VERSION-BASED                                               │
│     └── Include version in cache key                            │
│     └── Change version to invalidate                            │
│                                                                 │
│     cache_key = f"config:v{config_version}"                     │
│                                                                 │
│  4. CACHE TAGS                                                  │
│     └── Group related cache entries                             │
│     └── Invalidate by tag                                       │
│                                                                 │
│     cache.set("product:123", data, tags=["catalog"])            │
│     cache.invalidate_tag("catalog")                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 6.4 Redis Caching Example

```python
# Python Redis Caching Example

import redis
import json
from functools import wraps

redis_client = redis.Redis(host='redis', port=6379, decode_responses=True)

def cache(ttl=3600):
    """Decorator for caching function results"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            # Create cache key from function name and arguments
            cache_key = f"{func.__name__}:{hash(str(args) + str(kwargs))}"
            
            # Try to get from cache
            cached = redis_client.get(cache_key)
            if cached:
                return json.loads(cached)
            
            # Execute function and cache result
            result = func(*args, **kwargs)
            redis_client.setex(cache_key, ttl, json.dumps(result))
            return result
        return wrapper
    return decorator

@cache(ttl=300)  # Cache for 5 minutes
def get_user(user_id):
    """Expensive database query"""
    return db.query(f"SELECT * FROM users WHERE id = {user_id}")

def update_user(user_id, data):
    """Update user and invalidate cache"""
    db.update(user_id, data)
    # Invalidate all user-related cache
    for key in redis_client.scan_iter(f"get_user:*"):
        redis_client.delete(key)
```

---

## 7. Database Design Patterns

### 7.1 Database Selection

```
Database Selection Guide:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  RELATIONAL (SQL):                                              │
│  Use when: ACID needed, complex queries, structured data        │
│  Examples: PostgreSQL, MySQL, SQL Server                        │
│  Use cases: Financial, ERP, traditional apps                    │
│                                                                 │
│  DOCUMENT:                                                      │
│  Use when: Flexible schema, hierarchical data                   │
│  Examples: MongoDB, Cosmos DB                                   │
│  Use cases: CMS, catalogs, user profiles                        │
│                                                                 │
│  KEY-VALUE:                                                     │
│  Use when: Simple lookups, caching, sessions                    │
│  Examples: Redis, Memcached, DynamoDB                           │
│  Use cases: Caching, sessions, leaderboards                     │
│                                                                 │
│  WIDE-COLUMN:                                                   │
│  Use when: Time-series, high write throughput                   │
│  Examples: Cassandra, HBase, Bigtable                           │
│  Use cases: IoT, logs, analytics                                │
│                                                                 │
│  GRAPH:                                                         │
│  Use when: Complex relationships, traversals                    │
│  Examples: Neo4j, Amazon Neptune, Cosmos DB                     │
│  Use cases: Social networks, fraud detection, recommendations   │
│                                                                 │
│  TIME-SERIES:                                                   │
│  Use when: Time-stamped data, metrics                           │
│  Examples: InfluxDB, TimescaleDB, Prometheus                    │
│  Use cases: Monitoring, IoT, financial data                     │
│                                                                 │
│  SEARCH:                                                        │
│  Use when: Full-text search, analytics                          │
│  Examples: Elasticsearch, Azure Cognitive Search                │
│  Use cases: Log analysis, product search                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 7.2 Replication Patterns

```
Database Replication:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  SINGLE-LEADER (Primary-Replica):                               │
│                                                                 │
│        Writes                Reads                              │
│          │                     │                                │
│          ▼                     ▼                                │
│     ┌─────────┐          ┌─────────┐                            │
│     │ Primary │──Sync───►│ Replica │                            │
│     └─────────┘    │     └─────────┘                            │
│                    │     ┌─────────┐                            │
│                    └────►│ Replica │                            │
│                          └─────────┘                            │
│                                                                 │
│  MULTI-LEADER:                                                  │
│                                                                 │
│     ┌─────────┐◄────────►┌─────────┐                            │
│     │Leader A │          │Leader B │                            │
│     └─────────┘          └─────────┘                            │
│         │                    │                                  │
│     Writes (Region A)    Writes (Region B)                      │
│                                                                 │
│  LEADERLESS:                                                    │
│                                                                 │
│     Client writes to multiple nodes (quorum)                    │
│     ┌─────────┐ ┌─────────┐ ┌─────────┐                         │
│     │ Node 1  │ │ Node 2  │ │ Node 3  │                         │
│     └─────────┘ └─────────┘ └─────────┘                         │
│         ▲           ▲           ▲                               │
│         └───────────┼───────────┘                               │
│                Write to W nodes, Read from R nodes              │
│                W + R > N for consistency                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 7.3 Partitioning/Sharding Strategies

```
Sharding Strategies:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. RANGE-BASED PARTITIONING                                    │
│     └── Key ranges map to shards                                │
│                                                                 │
│     user_id 1-1M     → Shard 1                                  │
│     user_id 1M-2M    → Shard 2                                  │
│     user_id 2M-3M    → Shard 3                                  │
│                                                                 │
│     Pros: Range queries efficient                               │
│     Cons: Hot spots if data skewed                              │
│                                                                 │
│  2. HASH-BASED PARTITIONING                                     │
│     └── hash(key) % num_shards                                  │
│                                                                 │
│     hash("user123") % 3 = 1 → Shard 1                           │
│     hash("user456") % 3 = 2 → Shard 2                           │
│                                                                 │
│     Pros: Even distribution                                     │
│     Cons: Range queries require scatter-gather                  │
│                                                                 │
│  3. CONSISTENT HASHING                                          │
│     └── Minimizes redistribution when adding/removing shards    │
│                                                                 │
│          Shard A                                                │
│            ┌─┐                                                  │
│           /   \                                                 │
│     Shard D     Shard B     ← Ring                              │
│           \   /                                                 │
│            └─┘                                                  │
│          Shard C                                                │
│                                                                 │
│  4. DIRECTORY-BASED PARTITIONING                                │
│     └── Lookup service maps keys to shards                      │
│                                                                 │
│     Pros: Flexible, can rebalance easily                        │
│     Cons: Lookup service is SPOF                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

Shard Key Selection:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Good Shard Keys:                                               │
│  ├── High cardinality (many unique values)                      │
│  ├── Even distribution                                          │
│  ├── Query isolation (most queries hit single shard)            │
│  └── Examples: user_id, tenant_id, order_id                     │
│                                                                 │
│  Bad Shard Keys:                                                │
│  ├── Low cardinality (status: active/inactive)                  │
│  ├── Monotonically increasing (timestamp, auto-increment)       │
│  ├── Skewed distribution (country: 80% from one country)        │
│  └── Frequently changing values                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 7.4 Database Connection Pooling

```
Connection Pooling:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Without Pooling:                                               │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ Request 1 → Connect → Query → Disconnect                │    │
│  │ Request 2 → Connect → Query → Disconnect                │    │
│  │ Request 3 → Connect → Query → Disconnect                │    │
│  │                                                          │    │
│  │ Problem: Connection overhead 50-100ms each time          │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  With Pooling:                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                                                          │    │
│  │  ┌─────────────────────┐                                │    │
│  │  │   Connection Pool   │                                │    │
│  │  │  ┌───┐┌───┐┌───┐   │                                │    │
│  │  │  │ C ││ C ││ C │   │  ← Pre-established connections  │    │
│  │  │  └───┘└───┘└───┘   │                                │    │
│  │  └─────────────────────┘                                │    │
│  │        │   │   │                                        │    │
│  │        ▼   ▼   ▼                                        │    │
│  │     Request 1, 2, 3 → Borrow → Query → Return           │    │
│  │                                                          │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  Pool Configuration:                                            │
│  ├── Min connections: 5-10                                      │
│  ├── Max connections: Based on DB capacity                      │
│  ├── Idle timeout: 30 seconds                                   │
│  ├── Max lifetime: 30 minutes                                   │
│  └── Connection validation: Test before use                     │
│                                                                 │
│  Formula: Max connections = (CPU cores × 2) + disk spindles     │
│           For SSD: ~50-100 connections per database             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 8. Microservices Architecture

### 8.1 Microservices vs Monolith

```
Monolith vs Microservices:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  MONOLITH:                                                      │
│  ┌──────────────────────────────────────────┐                   │
│  │              Single Application          │                   │
│  │  ┌────────┐ ┌────────┐ ┌────────┐        │                   │
│  │  │ Users  │ │ Orders │ │Payment │        │                   │
│  │  └────────┘ └────────┘ └────────┘        │                   │
│  │  ┌────────────────────────────────┐      │                   │
│  │  │        Shared Database         │      │                   │
│  │  └────────────────────────────────┘      │                   │
│  └──────────────────────────────────────────┘                   │
│                                                                 │
│  Pros: Simple development, easy debugging, no network overhead  │
│  Cons: Scaling, tech lock-in, deployment risk, team coupling    │
│                                                                 │
│  MICROSERVICES:                                                 │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐                      │
│  │  Users  │    │ Orders  │    │ Payment │                      │
│  │ Service │    │ Service │    │ Service │                      │
│  └────┬────┘    └────┬────┘    └────┬────┘                      │
│       │              │              │                           │
│  ┌────┴────┐    ┌────┴────┐    ┌────┴────┐                      │
│  │ User DB │    │Order DB │    │  Stripe │                      │
│  └─────────┘    └─────────┘    └─────────┘                      │
│                                                                 │
│  Pros: Independent scaling, tech flexibility, team autonomy     │
│  Cons: Distributed system complexity, operational overhead      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 8.2 Microservices Communication Patterns

```
Communication Patterns:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  SYNCHRONOUS:                                                   │
│                                                                 │
│  1. HTTP/REST                                                   │
│     Service A ──HTTP──► Service B                               │
│              ◄── Response ──                                    │
│                                                                 │
│  2. gRPC                                                        │
│     Service A ──gRPC──► Service B                               │
│     Pros: Binary, fast, type-safe, streaming                    │
│                                                                 │
│  ASYNCHRONOUS:                                                  │
│                                                                 │
│  3. Message Queue                                               │
│     Service A ──► [Queue] ──► Service B                         │
│     Examples: RabbitMQ, Azure Service Bus                       │
│                                                                 │
│  4. Event Streaming                                             │
│     Service A ──► [Kafka/Event Hub] ──► Service B, C, D         │
│     Events stored, replayable                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

Service Mesh (Istio/Linkerd):
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  ┌─────────────────┐           ┌─────────────────┐              │
│  │   Service A     │           │   Service B     │              │
│  │  ┌───────────┐  │  mTLS     │  ┌───────────┐  │              │
│  │  │    App    │  │◄─────────►│  │    App    │  │              │
│  │  └───────────┘  │           │  └───────────┘  │              │
│  │  ┌───────────┐  │           │  ┌───────────┐  │              │
│  │  │  Sidecar  │  │           │  │  Sidecar  │  │              │
│  │  │  (Envoy)  │  │           │  │  (Envoy)  │  │              │
│  │  └───────────┘  │           │  └───────────┘  │              │
│  └─────────────────┘           └─────────────────┘              │
│                                                                 │
│  Features:                                                      │
│  ├── Automatic mTLS                                             │
│  ├── Traffic management (canary, circuit breaking)              │
│  ├── Observability (traces, metrics)                            │
│  └── Policy enforcement                                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 8.3 API Gateway Pattern

```
API Gateway:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│                        Clients                                  │
│                 Web / Mobile / IoT                              │
│                          │                                      │
│                          ▼                                      │
│       ┌──────────────────────────────────────┐                  │
│       │            API Gateway               │                  │
│       │  ┌────────────────────────────────┐  │                  │
│       │  │ • Authentication/Authorization │  │                  │
│       │  │ • Rate Limiting                │  │                  │
│       │  │ • Request/Response Transform  │  │                  │
│       │  │ • SSL Termination              │  │                  │
│       │  │ • Caching                      │  │                  │
│       │  │ • Request Routing              │  │                  │
│       │  │ • Load Balancing               │  │                  │
│       │  │ • Circuit Breaking             │  │                  │
│       │  │ • Logging/Monitoring           │  │                  │
│       │  └────────────────────────────────┘  │                  │
│       └──────────────────────────────────────┘                  │
│              │           │           │                          │
│              ▼           ▼           ▼                          │
│         ┌────────┐  ┌────────┐  ┌────────┐                      │
│         │Users   │  │Orders  │  │Payments│                      │
│         │Service │  │Service │  │Service │                      │
│         └────────┘  └────────┘  └────────┘                      │
│                                                                 │
│  Examples: Kong, Azure API Management, AWS API Gateway          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 8.4 Saga Pattern for Distributed Transactions

```
Saga Pattern:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Problem: ACID transactions don't work across services          │
│                                                                 │
│  Order Process:                                                 │
│  1. Create Order    → Order Service                             │
│  2. Reserve Stock   → Inventory Service                         │
│  3. Process Payment → Payment Service                           │
│  4. Ship Order      → Shipping Service                          │
│                                                                 │
│  CHOREOGRAPHY (Event-Driven):                                   │
│  ┌─────────┐ OrderCreated ┌─────────┐ StockReserved ┌─────────┐ │
│  │ Order   │─────────────►│Inventory│───────────────►│ Payment │ │
│  │ Service │              │ Service │               │ Service │ │
│  └─────────┘              └─────────┘               └─────────┘ │
│       ▲                        │                         │      │
│       │                  ReserveFailed              PaymentFailed│
│       │                        │                         │      │
│       └────────────────────────┴─────────────────────────┘      │
│                    Compensating Events                          │
│                                                                 │
│  ORCHESTRATION (Central Coordinator):                           │
│                  ┌───────────────┐                              │
│                  │     Saga      │                              │
│                  │ Orchestrator  │                              │
│                  └───────┬───────┘                              │
│              ┌───────────┼───────────┐                          │
│              ▼           ▼           ▼                          │
│         ┌────────┐  ┌────────┐  ┌────────┐                      │
│         │ Order  │  │Inventory│  │Payment │                     │
│         │Service │  │Service │  │Service │                      │
│         └────────┘  └────────┘  └────────┘                      │
│                                                                 │
│  Compensating Actions (Rollback):                               │
│  ├── Order: Cancel order                                        │
│  ├── Inventory: Release reserved stock                          │
│  └── Payment: Refund payment                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 8.5 Circuit Breaker Pattern

```
Circuit Breaker States:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│      ┌────────────────┐                                         │
│      │     CLOSED     │ ← Normal operation                      │
│      │ (Requests pass)│                                         │
│      └───────┬────────┘                                         │
│              │                                                  │
│              │ Failure threshold exceeded                       │
│              ▼                                                  │
│      ┌────────────────┐                                         │
│      │      OPEN      │ ← Fail fast, reject requests            │
│      │(Requests fail) │                                         │
│      └───────┬────────┘                                         │
│              │                                                  │
│              │ Timeout expires                                  │
│              ▼                                                  │
│      ┌────────────────┐                                         │
│      │   HALF-OPEN    │ ← Test with limited requests            │
│      │(Limited requests)                                        │
│      └───────┬────────┘                                         │
│              │                                                  │
│    ┌─────────┴─────────┐                                        │
│    ▼                   ▼                                        │
│ Success            Failure                                      │
│    │                   │                                        │
│    ▼                   ▼                                        │
│ CLOSED              OPEN                                        │
│                                                                 │
│  Configuration:                                                 │
│  ├── Failure threshold: 5 failures                              │
│  ├── Success threshold: 3 successes to close                    │
│  ├── Timeout: 30 seconds                                        │
│  └── Half-open requests: 3                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 9. Event-Driven Architecture

### 9.1 Event-Driven Patterns

```
Event-Driven Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  EVENT NOTIFICATION:                                            │
│  ┌─────────┐    Event     ┌─────────┐                           │
│  │ Service │─────────────►│ Queue   │──────►│ Consumer │        │
│  │    A    │ "UserCreated"│         │       │          │        │
│  └─────────┘              └─────────┘       └──────────┘        │
│                                                                 │
│  EVENT-CARRIED STATE TRANSFER:                                  │
│  Event contains all data needed:                                │
│  {                                                              │
│    "event": "UserCreated",                                      │
│    "data": {                                                    │
│      "id": 123,                                                 │
│      "name": "John",                                            │
│      "email": "john@example.com"                                │
│    }                                                            │
│  }                                                              │
│                                                                 │
│  EVENT SOURCING:                                                │
│  Store events as source of truth, derive state                  │
│                                                                 │
│  Event Store:                                                   │
│  ┌──────────────────────────────────────────────┐               │
│  │ 1. AccountCreated { id: 123, balance: 0 }    │               │
│  │ 2. MoneyDeposited { id: 123, amount: 100 }   │               │
│  │ 3. MoneyWithdrawn { id: 123, amount: 30 }    │               │
│  │ 4. MoneyDeposited { id: 123, amount: 50 }    │               │
│  └──────────────────────────────────────────────┘               │
│                    │                                            │
│                    ▼ Replay/Aggregate                           │
│            Current State: balance = 120                         │
│                                                                 │
│  CQRS (Command Query Responsibility Segregation):               │
│                                                                 │
│       Commands                    Queries                       │
│          │                           │                          │
│          ▼                           ▼                          │
│  ┌───────────────┐           ┌───────────────┐                  │
│  │ Write Model   │──Events──►│  Read Model   │                  │
│  │ (Normalized)  │           │ (Denormalized)│                  │
│  └───────────────┘           └───────────────┘                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 9.2 Message Queues vs Event Streams

```
Message Queue vs Event Stream:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  MESSAGE QUEUE (RabbitMQ, Azure Service Bus):                   │
│                                                                 │
│  Producer ──► [Queue] ──► Consumer                              │
│                                                                 │
│  Characteristics:                                               │
│  ├── Messages deleted after consumption                         │
│  ├── Point-to-point or pub/sub                                  │
│  ├── Message ordering per queue                                 │
│  ├── Push model (broker pushes to consumers)                    │
│  └── Good for task distribution                                 │
│                                                                 │
│  EVENT STREAM (Kafka, Azure Event Hubs):                        │
│                                                                 │
│  Producer ──► [Topic/Partition] ◄── Consumer Group              │
│                                                                 │
│  Characteristics:                                               │
│  ├── Events retained for configured period                      │
│  ├── Multiple consumer groups can read same events              │
│  ├── Ordering guaranteed within partition                       │
│  ├── Pull model (consumers pull events)                         │
│  ├── Replayable (consumers track offset)                        │
│  └── Good for event sourcing, audit logs                        │
│                                                                 │
│  When to use:                                                   │
│  ├── Queue: Task processing, work distribution, RPC             │
│  └── Stream: Event sourcing, analytics, audit, replay needed    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 9.3 Kafka Architecture

```
Kafka Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Producers        Kafka Cluster              Consumers          │
│                                                                 │
│  ┌─────────┐    ┌──────────────────────────┐   ┌─────────┐      │
│  │Producer │───►│  Topic: orders           │──►│Consumer │      │
│  │         │    │  ┌────────────────────┐  │   │ Group A │      │
│  └─────────┘    │  │ Partition 0        │  │   └─────────┘      │
│                 │  │ [0][1][2][3][4]... │  │                    │
│  ┌─────────┐    │  └────────────────────┘  │   ┌─────────┐      │
│  │Producer │───►│  ┌────────────────────┐  │──►│Consumer │      │
│  │         │    │  │ Partition 1        │  │   │ Group A │      │
│  └─────────┘    │  │ [0][1][2][3]...    │  │   └─────────┘      │
│                 │  └────────────────────┘  │                    │
│                 │  ┌────────────────────┐  │   ┌─────────┐      │
│                 │  │ Partition 2        │  │──►│Consumer │      │
│                 │  │ [0][1][2]...       │  │   │ Group B │      │
│                 │  └────────────────────┘  │   └─────────┘      │
│                 └──────────────────────────┘                    │
│                                                                 │
│  Key Concepts:                                                  │
│  ├── Topic: Category of messages                                │
│  ├── Partition: Ordered, immutable sequence                     │
│  ├── Offset: Position in partition                              │
│  ├── Consumer Group: Set of consumers sharing workload          │
│  └── Replication: Each partition replicated across brokers      │
│                                                                 │
│  Partitioning Strategy:                                         │
│  ├── Key-based: hash(key) % num_partitions                      │
│  ├── Round-robin: Distribute evenly (no key)                    │
│  └── Custom: Implement custom partitioner                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 10. CDN and Edge Computing

### 10.1 CDN Architecture

```
CDN (Content Delivery Network):
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│                         User Request                            │
│                              │                                  │
│                              ▼                                  │
│                    ┌──────────────────┐                         │
│                    │   DNS Resolver   │                         │
│                    └────────┬─────────┘                         │
│                             │ Returns nearest edge              │
│                             ▼                                   │
│     ┌─────────────────────────────────────────────────┐         │
│     │                   Edge Locations                │         │
│     │                                                 │         │
│     │  ┌───────┐    ┌───────┐    ┌───────┐           │         │
│     │  │ Edge  │    │ Edge  │    │ Edge  │           │         │
│     │  │  NYC  │    │  LON  │    │  TOK  │           │         │
│     │  └───┬───┘    └───┬───┘    └───┬───┘           │         │
│     │      │            │            │               │         │
│     │      │    Cache Hit? Return    │               │         │
│     │      │            │            │               │         │
│     │      └────────────┼────────────┘               │         │
│     │                   │ Cache Miss                 │         │
│     └───────────────────┼─────────────────────────────┘         │
│                         ▼                                       │
│               ┌──────────────────┐                              │
│               │  Origin Server   │                              │
│               │  (Your backend)  │                              │
│               └──────────────────┘                              │
│                                                                 │
│  What to cache:                                                 │
│  ├── Static assets (JS, CSS, images)                            │
│  ├── API responses (with short TTL)                             │
│  ├── HTML pages (for static sites)                              │
│  └── Video/Audio streams                                        │
│                                                                 │
│  Cache headers:                                                 │
│  Cache-Control: public, max-age=31536000   # 1 year             │
│  Cache-Control: private, max-age=0          # No cache          │
│  Cache-Control: public, max-age=300, stale-while-revalidate=60  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 10.2 CDN Cache Invalidation

```
Cache Invalidation Strategies:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. TTL-BASED                                                   │
│     └── Content expires after set time                          │
│     └── Simple but may serve stale content                      │
│                                                                 │
│  2. VERSIONED URLS                                              │
│     └── /assets/app.v1.2.3.js                                   │
│     └── Change version = new URL = cache miss                   │
│     └── Recommended for static assets                           │
│                                                                 │
│  3. CONTENT HASH                                                │
│     └── /assets/app.a1b2c3d4.js                                 │
│     └── Hash of content in filename                             │
│     └── Only changes when content changes                       │
│                                                                 │
│  4. MANUAL PURGE                                                │
│     └── API call to CDN to invalidate                           │
│     └── az cdn endpoint purge --content-paths "/*"              │
│     └── Expensive, use sparingly                                │
│                                                                 │
│  5. CACHE TAGS                                                  │
│     └── Tag content with categories                             │
│     └── Purge by tag                                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 11. Disaster Recovery

### 11.1 DR Terminology

```
Disaster Recovery Metrics:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  RPO (Recovery Point Objective):                                │
│  └── Maximum acceptable data loss                               │
│  └── "How much data can we afford to lose?"                     │
│                                                                 │
│       RPO = 1 hour                                              │
│       ◄────────────────────►                                    │
│  ──────┬───────────────────┬────►                               │
│     Last                Disaster                                │
│     Backup                                                      │
│                                                                 │
│  RTO (Recovery Time Objective):                                 │
│  └── Maximum acceptable downtime                                │
│  └── "How long can we be down?"                                 │
│                                                                 │
│                    RTO = 4 hours                                │
│                  ◄─────────────────►                            │
│  ────────────────┬─────────────────┬────►                       │
│              Disaster          Recovery                         │
│                                Complete                         │
│                                                                 │
│  DR Strategies by RPO/RTO:                                      │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │Strategy        │ RPO      │ RTO      │ Cost                ││
│  │─────────────────────────────────────────────────────────────││
│  │Backup/Restore  │ Hours    │ Hours    │ $                   ││
│  │Pilot Light     │ Minutes  │ Hours    │ $$                  ││
│  │Warm Standby    │ Minutes  │ Minutes  │ $$$                 ││
│  │Multi-Site      │ Seconds  │ Seconds  │ $$$$                ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 11.2 DR Strategies

```
DR Strategy: Backup and Restore
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Normal:              Disaster:             Recovery:           │
│  ┌─────────┐          ┌─────────┐           ┌─────────┐         │
│  │ Region A│          │ Region A│    ┌─────►│ Region B│         │
│  │ (Active)│          │  (Down) │    │      │ (Active)│         │
│  └────┬────┘          └─────────┘    │      └─────────┘         │
│       │ Backup                       │            ▲             │
│       ▼                              │            │             │
│  ┌─────────┐                    ┌────┴────┐      │             │
│  │  Blob   │                    │  Blob   │ ─────┘             │
│  │ Storage │                    │ Storage │  Restore           │
│  └─────────┘                    └─────────┘                     │
│                                                                 │
│  RPO: Hours (last backup)                                       │
│  RTO: Hours (provision + restore)                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

DR Strategy: Pilot Light
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Normal:                            Disaster:                   │
│  ┌─────────────────┐                ┌─────────────────┐         │
│  │    Region A     │                │    Region A     │         │
│  │    (Active)     │                │    (Down)       │         │
│  │  ┌───┐ ┌───┐   │                └─────────────────┘         │
│  │  │App│ │App│   │                                             │
│  │  └───┘ └───┘   │                ┌─────────────────┐         │
│  │  ┌───────────┐ │   Replication  │    Region B     │         │
│  │  │    DB     │◄├───────────────►│   (Activated)   │         │
│  │  └───────────┘ │                │  ┌───┐ ┌───┐   │         │
│  └─────────────────┘                │  │App│ │App│   │ Scale up │
│                                     │  └───┘ └───┘   │         │
│  ┌─────────────────┐                │  ┌───────────┐ │         │
│  │    Region B     │                │  │    DB     │ │         │
│  │   (Pilot Light) │                │  └───────────┘ │         │
│  │  ┌───────────┐ │                └─────────────────┘         │
│  │  │ DB Replica│ │ ← Minimal                                   │
│  │  └───────────┘ │   running                                   │
│  └─────────────────┘                                            │
│                                                                 │
│  RPO: Minutes (replication lag)                                 │
│  RTO: Hours (scale up infrastructure)                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

DR Strategy: Multi-Site Active-Active
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│           ┌─────────────────────────────────────────┐           │
│           │       Global Load Balancer (DNS)        │           │
│           └─────────────────────────────────────────┘           │
│                    │                    │                       │
│           ┌────────┴────────┐  ┌────────┴────────┐              │
│           │   Region A      │  │   Region B      │              │
│           │   (Active)      │  │   (Active)      │              │
│           │  ┌───┐ ┌───┐   │  │  ┌───┐ ┌───┐   │              │
│           │  │App│ │App│   │  │  │App│ │App│   │              │
│           │  └───┘ └───┘   │  │  └───┘ └───┘   │              │
│           │  ┌───────────┐ │  │  ┌───────────┐ │              │
│           │  │    DB     │◄├──┼─►│    DB     │ │              │
│           │  └───────────┘ │  │  └───────────┘ │              │
│           └─────────────────┘  └─────────────────┘              │
│                                                                 │
│  Normal: Both regions serve traffic                             │
│  Disaster: Healthy region handles all traffic                   │
│                                                                 │
│  RPO: Near-zero (sync replication)                              │
│  RTO: Seconds (DNS failover)                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 12. Capacity Planning

### 12.1 Capacity Planning Process

```
Capacity Planning Steps:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. MEASURE CURRENT USAGE                                       │
│     ├── CPU utilization                                         │
│     ├── Memory usage                                            │
│     ├── Disk I/O                                                │
│     ├── Network throughput                                      │
│     └── Request rate                                            │
│                                                                 │
│  2. UNDERSTAND WORKLOAD PATTERNS                                │
│     ├── Peak hours/days                                         │
│     ├── Seasonal patterns                                       │
│     ├── Growth trends                                           │
│     └── Special events                                          │
│                                                                 │
│  3. FORECAST GROWTH                                             │
│     ├── Historical growth rate                                  │
│     ├── Business projections                                    │
│     └── New features impact                                     │
│                                                                 │
│  4. CALCULATE REQUIREMENTS                                      │
│     ├── Current capacity × Growth factor                        │
│     ├── Add headroom (20-30%)                                   │
│     └── Consider redundancy                                     │
│                                                                 │
│  5. PLAN SCALING STRATEGY                                       │
│     ├── When to scale (thresholds)                              │
│     ├── How to scale (vertical/horizontal)                      │
│     └── Cost optimization                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 12.2 Capacity Estimation Examples

```
Example: E-commerce Platform Capacity Planning

Given:
- 10 million daily active users
- Average 5 page views per user per day
- 2% conversion rate (purchases)
- Peak traffic = 3× average
- Target response time < 200ms

Calculations:

Page Views:
- Daily page views = 10M × 5 = 50M/day
- Average QPS = 50M / 86,400 = 580 QPS
- Peak QPS = 580 × 3 = 1,740 QPS

Orders:
- Daily orders = 10M × 0.02 = 200,000/day
- Peak orders/hour = 200,000 / 24 × 3 = 25,000/hour

Server Sizing (assuming 1000 QPS per server):
- Base servers = 580 / 1000 = 1 (round up)
- Peak servers = 1,740 / 1000 = 2
- With 50% headroom = 3 servers
- With redundancy (N+1) = 4 servers

Database Sizing:
- Read QPS = 1,740 (mostly reads)
- Write QPS = 25,000/3600 = 7 QPS (orders)
- 1 primary + 2 read replicas

Cache Sizing:
- Cache hit rate target: 95%
- Hot data = 100,000 products × 10KB = 1 GB
- With overhead = 4 GB Redis cluster

Storage:
- Order size = 5 KB
- Daily storage = 200,000 × 5 KB = 1 GB/day
- Yearly = 365 GB
- With images/logs = 5 TB/year
```

---

## 13. System Design Interview Problems

### 13.1 Design a URL Shortener (like bit.ly)

```
URL Shortener Design:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Requirements:                                                  │
│  ├── Shorten long URLs to short aliases                         │
│  ├── Redirect short URL to original                             │
│  ├── Custom aliases (optional)                                  │
│  ├── Analytics (click count, location)                          │
│  └── Scale: 100M URLs/day, 10B redirects/day                    │
│                                                                 │
│  API:                                                           │
│  POST /shorten { "url": "https://very-long-url.com/..." }       │
│  → { "short_url": "https://short.ly/abc123" }                   │
│                                                                 │
│  GET /abc123 → 302 Redirect to original URL                     │
│                                                                 │
│  Architecture:                                                  │
│                                                                 │
│        ┌─────────────────────────────────────────┐              │
│        │            Load Balancer               │              │
│        └─────────────────────────────────────────┘              │
│                         │                                       │
│        ┌────────────────┼────────────────┐                      │
│        ▼                ▼                ▼                      │
│   ┌─────────┐      ┌─────────┐      ┌─────────┐                 │
│   │ App 1   │      │ App 2   │      │ App 3   │                 │
│   └────┬────┘      └────┬────┘      └────┬────┘                 │
│        │                │                │                      │
│        └────────────────┼────────────────┘                      │
│                         │                                       │
│        ┌────────────────┼────────────────┐                      │
│        ▼                ▼                ▼                      │
│   ┌─────────┐      ┌─────────────┐   ┌─────────┐               │
│   │  Redis  │      │   Database  │   │  Kafka  │               │
│   │ (Cache) │      │ (PostgreSQL)│   │(Analytics)              │
│   └─────────┘      └─────────────┘   └─────────┘               │
│                                                                 │
│  Short URL Generation:                                          │
│  Option 1: Base62 encoding of auto-increment ID                 │
│            ID 12345 → Base62 → "dnh"                            │
│                                                                 │
│  Option 2: Hash + collision handling                            │
│            MD5(url)[:6] → check if exists → retry               │
│                                                                 │
│  Option 3: Pre-generated keys in database                       │
│            Key Generation Service → pre-generate millions       │
│                                                                 │
│  Database Schema:                                               │
│  urls:                                                          │
│    short_code: VARCHAR(10) PRIMARY KEY                          │
│    original_url: TEXT                                           │
│    created_at: TIMESTAMP                                        │
│    user_id: BIGINT (optional)                                   │
│    expires_at: TIMESTAMP (optional)                             │
│                                                                 │
│  Scaling:                                                       │
│  ├── Read-heavy (100:1) → Heavy caching                         │
│  ├── Cache popular URLs in Redis                                │
│  ├── Database sharding by short_code hash                       │
│  └── CDN for global distribution                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 13.2 Design a Rate Limiter

```
Rate Limiter Design:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Algorithms:                                                    │
│                                                                 │
│  1. TOKEN BUCKET                                                │
│     ┌─────────────────────────────────────────────────────────┐ │
│     │  Bucket capacity: 10 tokens                             │ │
│     │  Refill rate: 2 tokens/second                           │ │
│     │                                                          │ │
│     │  [●][●][●][●][●][●][ ][ ][ ][ ]  ← 6 tokens available   │ │
│     │                                                          │ │
│     │  Request arrives:                                        │ │
│     │  - If tokens > 0: Allow, remove 1 token                  │ │
│     │  - If tokens = 0: Reject (429 Too Many Requests)         │ │
│     └─────────────────────────────────────────────────────────┘ │
│                                                                 │
│  2. SLIDING WINDOW LOG                                          │
│     ┌─────────────────────────────────────────────────────────┐ │
│     │  Window: 1 minute, Limit: 100 requests                  │ │
│     │                                                          │ │
│     │  Request log: [12:00:05, 12:00:15, 12:00:45, ...]       │ │
│     │                                                          │ │
│     │  On request at 12:01:00:                                 │ │
│     │  1. Remove entries older than 12:00:00                   │ │
│     │  2. Count remaining entries                              │ │
│     │  3. If count < 100: Allow, add timestamp                 │ │
│     │  4. Else: Reject                                         │ │
│     └─────────────────────────────────────────────────────────┘ │
│                                                                 │
│  3. SLIDING WINDOW COUNTER                                      │
│     Weighted average of current and previous window             │
│     More memory-efficient than log                              │
│                                                                 │
│  Implementation with Redis:                                     │
│                                                                 │
│  # Token Bucket in Redis                                        │
│  SCRIPT rate_limit(key, capacity, rate, now)                    │
│    tokens = GET key.tokens                                      │
│    last_update = GET key.timestamp                              │
│                                                                 │
│    # Refill tokens                                              │
│    elapsed = now - last_update                                  │
│    tokens = min(capacity, tokens + elapsed * rate)              │
│                                                                 │
│    if tokens >= 1:                                              │
│      SET key.tokens (tokens - 1)                                │
│      SET key.timestamp now                                      │
│      return ALLOWED                                             │
│    else:                                                        │
│      return DENIED                                              │
│                                                                 │
│  Architecture:                                                  │
│                                                                 │
│       Client                                                    │
│          │                                                      │
│          ▼                                                      │
│  ┌───────────────┐                                              │
│  │ Rate Limiter  │◄────────► Redis (distributed counter)       │
│  │  Middleware   │                                              │
│  └───────┬───────┘                                              │
│          │ If allowed                                           │
│          ▼                                                      │
│  ┌───────────────┐                                              │
│  │    API        │                                              │
│  └───────────────┘                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 13.3 Design a Distributed Cache

```
Distributed Cache Design:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Requirements:                                                  │
│  ├── Low latency (<10ms)                                        │
│  ├── High throughput (millions QPS)                             │
│  ├── High availability                                          │
│  ├── Scalability (add/remove nodes)                             │
│  └── Consistent hashing for distribution                        │
│                                                                 │
│  Consistent Hashing:                                            │
│                                                                 │
│              Node A                                             │
│                ●                                                │
│            /      \                                             │
│       ●───●        ●───● Key hash falls here                    │
│   Node D  │        │   → Route to next node (A)                 │
│           │  Ring  │                                            │
│       ●───●        ●───●                                        │
│            \      /   Node B                                    │
│                ●                                                │
│              Node C                                             │
│                                                                 │
│  Virtual Nodes:                                                 │
│  Each physical node = multiple points on ring                   │
│  Improves distribution evenness                                 │
│                                                                 │
│  Architecture:                                                  │
│                                                                 │
│       ┌─────────────────────────────────────────┐               │
│       │            Cache Clients                │               │
│       │  (with consistent hashing logic)        │               │
│       └─────────────────────────────────────────┘               │
│                    │   │   │                                    │
│       ┌────────────┼───┼───┼────────────┐                       │
│       ▼            ▼   ▼   ▼            ▼                       │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐             │
│  │ Cache 1 │  │ Cache 2 │  │ Cache 3 │  │ Cache 4 │             │
│  │ Replica │  │ Replica │  │ Replica │  │ Replica │             │
│  │ Cache 2 │  │ Cache 3 │  │ Cache 4 │  │ Cache 1 │             │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘             │
│                                                                 │
│  Eviction Policies:                                             │
│  ├── LRU (Least Recently Used)                                  │
│  ├── LFU (Least Frequently Used)                                │
│  ├── TTL (Time To Live)                                         │
│  └── Random                                                     │
│                                                                 │
│  Cache Coherence:                                               │
│  ├── Write-through: Sync with source on write                   │
│  ├── Write-around: Write to source, invalidate cache            │
│  └── Write-back: Async write to source                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 14. Real-World Architecture Examples

### 14.1 E-Commerce Platform Architecture

```
E-Commerce Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│                          Users                                  │
│                            │                                    │
│                            ▼                                    │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                         CDN                               │   │
│  │              (Static assets, images)                      │   │
│  └──────────────────────────────────────────────────────────┘   │
│                            │                                    │
│                            ▼                                    │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   API Gateway                             │   │
│  │        (Auth, Rate Limiting, Routing)                     │   │
│  └──────────────────────────────────────────────────────────┘   │
│                            │                                    │
│       ┌────────────────────┼────────────────────┐               │
│       ▼                    ▼                    ▼               │
│  ┌─────────┐         ┌─────────┐         ┌─────────┐            │
│  │ Product │         │  Order  │         │  User   │            │
│  │ Service │         │ Service │         │ Service │            │
│  └────┬────┘         └────┬────┘         └────┬────┘            │
│       │                   │                   │                 │
│       ▼                   ▼                   ▼                 │
│  ┌─────────┐         ┌─────────┐         ┌─────────┐            │
│  │Product  │         │ Order   │         │ User    │            │
│  │   DB    │         │   DB    │         │   DB    │            │
│  └─────────┘         └─────────┘         └─────────┘            │
│       │                   │                   │                 │
│       └───────────────────┴───────────────────┘                 │
│                           │                                     │
│                           ▼                                     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    Redis Cluster                          │   │
│  │      (Sessions, Cart, Product Cache, Rate Limits)         │   │
│  └──────────────────────────────────────────────────────────┘   │
│                           │                                     │
│                           ▼                                     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   Elasticsearch                           │   │
│  │                  (Product Search)                         │   │
│  └──────────────────────────────────────────────────────────┘   │
│                           │                                     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   Message Queue                           │   │
│  │    (Order Events, Inventory Updates, Notifications)       │   │
│  └──────────────────────────────────────────────────────────┘   │
│                           │                                     │
│       ┌───────────────────┼───────────────────┐                 │
│       ▼                   ▼                   ▼                 │
│  ┌─────────┐         ┌─────────┐         ┌─────────┐            │
│  │ Payment │         │Inventory│         │ Email   │            │
│  │ Service │         │ Service │         │ Service │            │
│  └─────────┘         └─────────┘         └─────────┘            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 14.2 Video Streaming Platform Architecture

```
Video Streaming Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Upload Flow:                                                   │
│                                                                 │
│  Creator                                                        │
│     │                                                           │
│     ▼                                                           │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────┐      │
│  │   Upload    │───►│   Object    │───►│   Transcoding   │      │
│  │   Service   │    │   Storage   │    │   Pipeline      │      │
│  └─────────────┘    └─────────────┘    └────────┬────────┘      │
│                                                  │               │
│                      Multiple quality levels     ▼               │
│                      ┌─────────────────────────────────────┐     │
│                      │        CDN Edge Servers             │     │
│                      │  ┌───────┐ ┌───────┐ ┌───────┐      │     │
│                      │  │1080p  │ │ 720p  │ │ 480p  │      │     │
│                      │  └───────┘ └───────┘ └───────┘      │     │
│                      └─────────────────────────────────────┘     │
│                                        │                         │
│  Playback Flow:                        │                         │
│                                        ▼                         │
│  Viewer ─────────────────────────► Edge Server                   │
│                                                                 │
│  Adaptive Bitrate Streaming (HLS/DASH):                         │
│  ├── Client requests manifest (.m3u8/.mpd)                      │
│  ├── Manifest lists available qualities                         │
│  ├── Client chooses based on bandwidth                          │
│  └── Can switch quality mid-stream                              │
│                                                                 │
│  Key Components:                                                │
│  ├── Video Encoding Service (FFmpeg)                            │
│  ├── Object Storage (S3, Azure Blob)                            │
│  ├── CDN (Akamai, CloudFront, Azure CDN)                        │
│  ├── Metadata Service (video info, thumbnails)                  │
│  ├── Recommendation Engine (ML-based)                           │
│  └── Analytics Pipeline (Kafka → Spark → Data Lake)             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Summary

System design for infrastructure requires understanding:

1. **Core Principles**: CAP theorem, ACID/BASE, consistency models
2. **Scalability**: Horizontal vs vertical, stateless design, database sharding
3. **High Availability**: Redundancy, failover, health checks
4. **Load Balancing**: Algorithms, session persistence, layer 4 vs 7
5. **Caching**: Cache placement, patterns, invalidation strategies
6. **Databases**: Selection criteria, replication, partitioning
7. **Microservices**: Communication patterns, API gateway, saga pattern
8. **Event-Driven**: Queues vs streams, Kafka, event sourcing
9. **DR Planning**: RPO/RTO, strategies from backup to multi-site
10. **Capacity Planning**: Estimation, growth forecasting

The key to system design is understanding trade-offs - there's no perfect solution, only trade-offs appropriate for specific requirements.

---

**Next Part**: [Part 10: SRE Practices](./DevOps-Complete-Reference-Guide-Part10-SRE.md)
