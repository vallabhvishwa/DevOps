# DevOps Engineer's Complete Reference Guide
# Part 21: Database Operations

---

## Table of Contents

1. [Database Fundamentals](#1-database-fundamentals)
2. [PostgreSQL Operations](#2-postgresql-operations)
3. [MySQL Operations](#3-mysql-operations)
4. [Redis Operations](#4-redis-operations)
5. [Backup & Recovery](#5-backup--recovery)
6. [High Availability](#6-high-availability)
7. [Performance Tuning](#7-performance-tuning)
8. [Azure Database Services](#8-azure-database-services)

---

## 1. Database Fundamentals

```
Database Types:
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                                                                 â”‚
â”‚  RELATIONAL (SQL):                                              â”‚
â”‚  â”œâ”€â”€ PostgreSQL  - Feature-rich, ACID compliant                 â”‚
â”‚  â”œâ”€â”€ MySQL       - Popular, web applications                    â”‚
â”‚  â””â”€â”€ SQL Server  - Enterprise, Microsoft ecosystem              â”‚
â”‚                                                                 â”‚
â”‚  KEY-VALUE:                                                     â”‚
â”‚  â”œâ”€â”€ Redis       - In-memory, caching, sessions                 â”‚
â”‚  â””â”€â”€ Memcached   - Simple caching                               â”‚
â”‚                                                                 â”‚
â”‚  DOCUMENT:                                                      â”‚
â”‚  â”œâ”€â”€ MongoDB     - JSON documents                               â”‚
â”‚  â””â”€â”€ Cosmos DB   - Multi-model, global distribution             â”‚
â”‚                                                                 â”‚
â”‚  DevOps Responsibilities:                                       â”‚
â”‚  â”œâ”€â”€ Installation & configuration                               â”‚
â”‚  â”œâ”€â”€ Backup & recovery                                          â”‚
â”‚  â”œâ”€â”€ High availability setup                                    â”‚
â”‚  â”œâ”€â”€ Performance monitoring                                     â”‚
â”‚  â”œâ”€â”€ Security & access control                                  â”‚
â”‚  â””â”€â”€ Upgrades & patching                                        â”‚
â”‚                                                                 â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

---

## 2. PostgreSQL Operations

### 2.1 Essential Commands

```bash
# Connection
psql -h localhost -U postgres -d mydb
psql "postgresql://user:pass@host:5432/db?sslmode=require"

# Database management
CREATE DATABASE myapp;
DROP DATABASE myapp;
\l                     # List databases
\c myapp               # Connect to database
\dt                    # List tables
\d+ tablename          # Describe table

# User management
CREATE USER appuser WITH PASSWORD 'secret';
GRANT ALL PRIVILEGES ON DATABASE myapp TO appuser;
GRANT SELECT, INSERT, UPDATE ON ALL TABLES IN SCHEMA public TO appuser;
ALTER USER appuser WITH PASSWORD 'newpass';

# Useful queries
SELECT pg_size_pretty(pg_database_size('myapp'));  -- DB size
SELECT * FROM pg_stat_activity;                     -- Active connections
SELECT * FROM pg_stat_user_tables;                  -- Table stats
```

### 2.2 PostgreSQL Configuration

```ini
# postgresql.conf - Key settings

# Connections
max_connections = 200
shared_buffers = 256MB          # 25% of RAM

# Memory
work_mem = 64MB                 # Per-operation memory
maintenance_work_mem = 512MB    # For VACUUM, CREATE INDEX
effective_cache_size = 1GB      # OS cache estimate

# WAL & Replication
wal_level = replica
max_wal_senders = 5
wal_keep_size = 1GB

# Logging
log_statement = 'all'
log_duration = on
log_min_duration_statement = 1000  # Log slow queries > 1s
```

---

## 3. MySQL Operations

```bash
# Connection
mysql -h localhost -u root -p
mysql -h host -u user -p dbname < script.sql

# Database management
CREATE DATABASE myapp CHARACTER SET utf8mb4;
SHOW DATABASES;
USE myapp;
SHOW TABLES;
DESCRIBE tablename;

# User management
CREATE USER 'appuser'@'%' IDENTIFIED BY 'password';
GRANT SELECT, INSERT, UPDATE ON myapp.* TO 'appuser'@'%';
FLUSH PRIVILEGES;

# Status queries
SHOW PROCESSLIST;
SHOW STATUS LIKE 'Threads%';
SHOW VARIABLES LIKE 'max_connections';
```

---

## 4. Redis Operations

```bash
# Connection
redis-cli -h localhost -p 6379
redis-cli -h host -p 6379 -a password

# Basic commands
SET key "value"
GET key
DEL key
KEYS pattern*
TTL key
EXPIRE key 3600

# Data structures
LPUSH list value          # List
SADD set member           # Set
HSET hash field value     # Hash
ZADD sorted key score member  # Sorted set

# Admin
INFO                      # Server info
DBSIZE                    # Key count
FLUSHDB                   # Clear current DB
FLUSHALL                  # Clear all DBs
BGSAVE                    # Background save
```

---

## 5. Backup & Recovery

### 5.1 PostgreSQL Backup

```bash
# Logical backup (pg_dump)
pg_dump -h host -U user dbname > backup.sql
pg_dump -Fc dbname > backup.dump          # Custom format (compressed)
pg_dumpall > all_databases.sql            # All databases

# Restore
psql -h host -U user dbname < backup.sql
pg_restore -d dbname backup.dump

# Physical backup (base backup)
pg_basebackup -h host -D /backup/data -Fp -Xs -P

# Point-in-time recovery (PITR)
# 1. Have continuous WAL archiving configured
# 2. Restore base backup
# 3. Configure recovery.conf with target time
```

### 5.2 MySQL Backup

```bash
# mysqldump
mysqldump -h host -u root -p dbname > backup.sql
mysqldump --all-databases > all_dbs.sql
mysqldump --single-transaction dbname > backup.sql  # InnoDB consistent

# Restore
mysql -h host -u root -p dbname < backup.sql
```

### 5.3 Automated Backup Script

```bash
#!/bin/bash
# backup-postgres.sh

set -e

DB_HOST="${DB_HOST:-localhost}"
DB_NAME="${DB_NAME:-myapp}"
BACKUP_DIR="/backups"
RETENTION_DAYS=7
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="${BACKUP_DIR}/${DB_NAME}_${DATE}.dump"

# Create backup
pg_dump -h "$DB_HOST" -Fc "$DB_NAME" > "$BACKUP_FILE"

# Upload to Azure Blob
az storage blob upload \
  --account-name backups \
  --container-name db-backups \
  --file "$BACKUP_FILE" \
  --name "$(basename $BACKUP_FILE)"

# Cleanup old backups
find "$BACKUP_DIR" -name "*.dump" -mtime +$RETENTION_DAYS -delete

echo "Backup completed: $BACKUP_FILE"
```

---

## 6. High Availability

```
PostgreSQL HA Options:
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                                                                 â”‚
â”‚  STREAMING REPLICATION:                                         â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”    WAL Stream    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”                       â”‚
â”‚  â”‚ Primary â”‚â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â–ºâ”‚ Replica â”‚                       â”‚
â”‚  â”‚  (R/W)  â”‚                  â”‚  (R/O)  â”‚                       â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜                  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜                       â”‚
â”‚                                                                 â”‚
â”‚  WITH PATRONI (Recommended):                                    â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”                         â”‚
â”‚  â”‚ Node 1  â”‚  â”‚ Node 2  â”‚  â”‚ Node 3  â”‚                         â”‚
â”‚  â”‚ Primary â”‚  â”‚ Replica â”‚  â”‚ Replica â”‚                         â”‚
â”‚  â””â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”˜                         â”‚
â”‚       â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜                               â”‚
â”‚                    â–¼                                            â”‚
â”‚              â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”                                       â”‚
â”‚              â”‚   etcd   â”‚ (Distributed consensus)               â”‚
â”‚              â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜                                       â”‚
â”‚                                                                 â”‚
â”‚  Automatic failover, leader election                            â”‚
â”‚                                                                 â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

---

## 7. Performance Tuning

```
Performance Monitoring Queries:
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                                                                 â”‚
â”‚  POSTGRESQL:                                                    â”‚
â”‚  -- Slow queries                                                â”‚
â”‚  SELECT * FROM pg_stat_statements                               â”‚
â”‚    ORDER BY total_time DESC LIMIT 10;                           â”‚
â”‚                                                                 â”‚
â”‚  -- Table bloat                                                 â”‚
â”‚  SELECT relname, n_dead_tup, n_live_tup                         â”‚
â”‚    FROM pg_stat_user_tables                                     â”‚
â”‚    WHERE n_dead_tup > 1000;                                     â”‚
â”‚                                                                 â”‚
â”‚  -- Index usage                                                 â”‚
â”‚  SELECT relname, indexrelname, idx_scan, idx_tup_read           â”‚
â”‚    FROM pg_stat_user_indexes;                                   â”‚
â”‚                                                                 â”‚
â”‚  -- Connection usage                                            â”‚
â”‚  SELECT count(*) FROM pg_stat_activity;                         â”‚
â”‚                                                                 â”‚
â”‚  MYSQL:                                                         â”‚
â”‚  SHOW FULL PROCESSLIST;                                         â”‚
â”‚  SHOW STATUS LIKE 'Slow_queries';                               â”‚
â”‚  SHOW ENGINE INNODB STATUS;                                     â”‚
â”‚                                                                 â”‚
â”‚  Key Maintenance:                                               â”‚
â”‚  â”œâ”€â”€ VACUUM ANALYZE (PostgreSQL)                                â”‚
â”‚  â”œâ”€â”€ OPTIMIZE TABLE (MySQL)                                     â”‚
â”‚  â”œâ”€â”€ REINDEX (PostgreSQL)                                       â”‚
â”‚  â””â”€â”€ Update statistics regularly                                â”‚
â”‚                                                                 â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

---

## 8. Azure Database Services

```bash
# Azure Database for PostgreSQL
az postgres flexible-server create \
  --name mypostgres \
  --resource-group myRG \
  --location eastus \
  --admin-user admin \
  --admin-password SecurePass123! \
  --sku-name Standard_D2s_v3 \
  --storage-size 128 \
  --version 15

# Connection
az postgres flexible-server show-connection-string \
  --server-name mypostgres

# Azure Cache for Redis
az redis create \
  --name myredis \
  --resource-group myRG \
  --location eastus \
  --sku Standard \
  --vm-size c1

# Get connection info
az redis list-keys --name myredis --resource-group myRG
```

---

## Summary

Database operations for DevOps:

1. **PostgreSQL/MySQL**: User management, configuration, optimization
2. **Redis**: Caching, session storage
3. **Backup**: Automated, tested, offsite storage
4. **High Availability**: Replication, Patroni, managed services
5. **Performance**: Monitoring, slow query analysis, maintenance
6. **Azure**: Managed services reduce operational burden

Use managed database services when possible to reduce operational overhead.

---

## Complete Guide Summary

This completes the DevOps Engineer's Complete Reference Guide with **21 parts**:

| Parts 1-7 | Foundation |
|-----------|------------|
| 1-2 | Linux |
| 3 | Networking |
| 4-5 | Kubernetes & Azure |
| 6-7 | CI/CD & Observability |

| Parts 8-17 | FAANG-Level |
|------------|-------------|
| 8 | Terraform |
| 9 | System Design |
| 10 | SRE |
| 11-12 | GitOps & Service Mesh |
| 13-14 | Security & Chaos |
| 15-17 | Programming, Multi-Cloud, FinOps |

| Parts 18-21 | Complete Coverage |
|-------------|-------------------|
| 18 | DSA for Interviews |
| 19 | Message Queues |
| 20 | Helm |
| 21 | Databases |

**Total: ~25,000+ lines of comprehensive documentation**
---

## Related

- [SQL Server on Azure](./SQL-Server-Azure.md)
- [SQL Server Troubleshooting](../../Troubleshooting/Databases/SQL-Server-Azure-Troubleshooting.md)
- [General Database Troubleshooting](../../Troubleshooting/Databases/08-Database-Troubleshooting.md)