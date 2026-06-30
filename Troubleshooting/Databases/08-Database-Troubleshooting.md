# Database Troubleshooting - Complete Guide
## PostgreSQL, MySQL, Redis - From Basic to Advanced

---

# SECTION 1: CONNECTION ISSUES

---

## 1.1 CAN'T CONNECT

### LEVEL 1 - DIRECT: Connection Refused

**Scenario**: Database not accepting connections.

```bash
$ psql -h localhost -U postgres
psql: connection refused
Is the server running on host "localhost"?
```

**Cause is VISIBLE**: PostgreSQL not running.

**Solution**:
```bash
# Start the service
systemctl start postgresql

# Check if running
systemctl status postgresql
```

---

### LEVEL 1 - DIRECT: Authentication Failed

**Scenario**: Wrong password.

```bash
$ psql -h localhost -U myuser
FATAL: password authentication failed for user "myuser"
```

**Cause is VISIBLE**: Wrong password.

**Solution**:
```bash
# Reset password as postgres user
sudo -u postgres psql
ALTER USER myuser WITH PASSWORD 'newpassword';
```

---

### LEVEL 1 - DIRECT: Database Not Found

**Scenario**: Database doesn't exist.

```bash
$ psql -h localhost -U myuser -d myapp
FATAL: database "myapp" does not exist
```

**Cause is VISIBLE**: Database doesn't exist.

**Solution**:
```bash
# Create database
sudo -u postgres createdb myapp

# Or in SQL
CREATE DATABASE myapp;
```

---

### LEVEL 1 - DIRECT: Host Not Allowed

**Scenario**: Remote connection rejected.

```bash
$ psql -h 10.0.0.50 -U myuser
FATAL: no pg_hba.conf entry for host "10.0.0.1"
```

**Cause is VISIBLE**: pg_hba.conf doesn't allow this host.

**Solution**:
```bash
# Edit pg_hba.conf
# Add line:
host    all    all    10.0.0.0/24    md5

# Reload config
sudo systemctl reload postgresql
```

---

### LEVEL 2 - INTERMEDIATE: Too Many Connections

**Scenario**: Connection limit reached.

```bash
$ psql -h localhost -U myuser
FATAL: too many connections for role "myuser"
```

**Investigation**:
```sql
-- Check current connections
SELECT count(*) FROM pg_stat_activity;

-- Check limit
SHOW max_connections;

-- Who's connected?
SELECT usename, client_addr, state, count(*)
FROM pg_stat_activity
GROUP BY usename, client_addr, state;
```

**Solution**:
```sql
-- Kill idle connections
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE state = 'idle'
AND query_start < now() - interval '10 minutes';

-- Or increase limit (requires restart)
ALTER SYSTEM SET max_connections = 200;
```

---

### LEVEL 3 - COMPLEX: Connection Works Then Dies

**Scenario**: Connection established, then dropped.

**Hidden Causes**:
```
CONNECTION DROPS:
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  1. Firewall/NAT timeout                                        â”‚
â”‚  2. Connection pool timeout                                     â”‚
â”‚  3. Database killed idle connection                             â”‚
â”‚  4. Network issue                                               â”‚
â”‚  5. OOMKilled database                                          â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

**Deep Investigation**:
```sql
-- Check if connection was terminated
SELECT * FROM pg_stat_activity WHERE state = 'idle';

-- Check PostgreSQL logs
tail -f /var/log/postgresql/postgresql-*.log
```

**Solution**:
```bash
# Enable TCP keepalive
# postgresql.conf:
tcp_keepalives_idle = 60
tcp_keepalives_interval = 10
tcp_keepalives_count = 6
```

---

# SECTION 2: PERFORMANCE ISSUES

---

## 2.1 SLOW QUERIES

### LEVEL 1 - DIRECT: Missing Index

**Scenario**: Query slow on large table.

```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';
-- Seq Scan on users (cost=0.00..50000.00 rows=1)
-- Actual time: 500ms
```

**Cause is VISIBLE**: Seq Scan on large table = no index.

**Solution**:
```sql
-- Create index
CREATE INDEX idx_users_email ON users(email);

-- Query now uses index
-- Index Scan on users (cost=0.00..8.00 rows=1)
-- Actual time: 0.05ms
```

---

### LEVEL 1 - DIRECT: Query Returns Too Much Data

**Scenario**: Query slow because returning millions of rows.

```sql
SELECT * FROM events;
-- Returns 10 million rows
-- Takes 30 seconds
```

**Cause is VISIBLE**: Selecting too much data.

**Solution**:
```sql
-- Add filters
SELECT * FROM events 
WHERE created_at > NOW() - INTERVAL '1 day'
LIMIT 100;
```

---

### LEVEL 2 - INTERMEDIATE: Query Fast Then Slow

**Scenario**: Same query sometimes fast, sometimes slow.

```sql
-- First run: 10ms
-- Second run: 10ms
-- After 30 min: 5000ms
```

**Investigation**:
```sql
-- Check if table is bloated
SELECT 
    schemaname, relname,
    n_live_tup, n_dead_tup,
    last_vacuum, last_autovacuum
FROM pg_stat_user_tables
WHERE n_dead_tup > 10000;
```

**Causes**:
- Table bloat (dead tuples)
- Cache cleared
- Lock contention
- Statistics outdated

**Solution**:
```sql
-- Run vacuum
VACUUM ANALYZE tablename;

-- Update statistics
ANALYZE tablename;
```

---

### LEVEL 3 - COMPLEX: Database Randomly Slow

**Scenario**: Everything slow sometimes, no pattern.

**Hidden Causes**:
```
RANDOM SLOWNESS:
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  1. Autovacuum running                                          â”‚
â”‚  2. Checkpoint happening                                        â”‚
â”‚  3. Lock contention                                             â”‚
â”‚  4. Disk I/O saturation                                         â”‚
â”‚  5. Memory pressure (swapping)                                  â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

**Deep Investigation**:
```sql
-- Check for locks
SELECT 
    blocked.pid AS blocked_pid,
    blocked.query AS blocked_query,
    blocking.pid AS blocking_pid,
    blocking.query AS blocking_query
FROM pg_stat_activity blocked
JOIN pg_locks bl ON blocked.pid = bl.pid
JOIN pg_locks l ON bl.relation = l.relation AND bl.pid != l.pid
JOIN pg_stat_activity blocking ON l.pid = blocking.pid
WHERE NOT bl.granted;

-- Check for autovacuum
SELECT * FROM pg_stat_progress_vacuum;

-- Check checkpoint activity
SELECT * FROM pg_stat_bgwriter;
```

---

## 2.2 DISK SPACE

### LEVEL 1 - DIRECT: Database Full

**Scenario**: Insert fails, disk full.

```sql
INSERT INTO logs VALUES (...);
-- ERROR: could not extend file: No space left on device
```

**Cause is VISIBLE**: Disk is full.

**Solution**:
```bash
# Check disk
df -h

# Find large tables
SELECT 
    relname,
    pg_size_pretty(pg_relation_size(relid))
FROM pg_stat_user_tables
ORDER BY pg_relation_size(relid) DESC
LIMIT 10;

# Cleanup
VACUUM FULL tablename;  # Reclaims space but locks table
DELETE FROM logs WHERE created_at < NOW() - INTERVAL '30 days';
```

---

### LEVEL 2 - INTERMEDIATE: Table Bloat

**Scenario**: Table uses 10GB but data is only 2GB.

**Investigation**:
```sql
-- Check bloat
SELECT 
    schemaname, relname,
    pg_size_pretty(pg_relation_size(relid)) AS table_size,
    n_live_tup,
    n_dead_tup
FROM pg_stat_user_tables
WHERE n_dead_tup > n_live_tup * 0.2;  -- >20% dead tuples
```

**Solution**:
```sql
-- Regular vacuum (doesn't lock)
VACUUM ANALYZE tablename;

-- Full vacuum (reclaims space, locks table)
VACUUM FULL tablename;

-- Better: Set up autovacuum properly
ALTER TABLE high_traffic_table SET (
    autovacuum_vacuum_scale_factor = 0.05,
    autovacuum_analyze_scale_factor = 0.02
);
```

---

# SECTION 3: REDIS ISSUES

---

## 3.1 REDIS CONNECTION

### LEVEL 1 - DIRECT: Redis Not Running

**Scenario**: Can't connect to Redis.

```bash
$ redis-cli ping
Could not connect to Redis at 127.0.0.1:6379: Connection refused
```

**Cause is VISIBLE**: Redis not running.

**Solution**:
```bash
systemctl start redis
redis-cli ping
# PONG
```

---

### LEVEL 1 - DIRECT: Authentication Required

**Scenario**: Redis requires password.

```bash
$ redis-cli ping
(error) NOAUTH Authentication required.
```

**Cause is VISIBLE**: Need to authenticate.

**Solution**:
```bash
redis-cli -a mypassword ping
# Or
redis-cli
AUTH mypassword
```

---

## 3.2 REDIS PERFORMANCE

### LEVEL 1 - DIRECT: Memory Limit Reached

**Scenario**: Redis rejecting writes.

```bash
$ redis-cli SET foo bar
(error) OOM command not allowed when used memory > 'maxmemory'
```

**Cause is VISIBLE**: Memory limit reached.

**Solution**:
```bash
# Check memory
redis-cli INFO memory

# Set eviction policy
redis-cli CONFIG SET maxmemory-policy allkeys-lru

# Or increase limit
redis-cli CONFIG SET maxmemory 2gb
```

---

### LEVEL 2 - INTERMEDIATE: Slow Commands

**Scenario**: Redis commands taking long.

**Investigation**:
```bash
# Check slow log
redis-cli SLOWLOG GET 10

# Check big keys
redis-cli --bigkeys
```

**Common Issues**:
- KEYS * on large dataset (use SCAN instead)
- Large hash/list operations
- Blocking commands (BLPOP with long timeout)

**Solution**:
```bash
# Use SCAN instead of KEYS
redis-cli SCAN 0 MATCH "user:*" COUNT 100

# Break large operations into chunks
```

---

# SECTION 4: MYSQL SPECIFIC

---

## 4.1 MYSQL ISSUES

### LEVEL 1 - DIRECT: Access Denied

**Scenario**: Can't log in.

```bash
$ mysql -u root -p
ERROR 1045 (28000): Access denied for user 'root'@'localhost'
```

**Cause is VISIBLE**: Wrong password or user doesn't exist.

**Solution**:
```bash
# Reset root password
sudo systemctl stop mysql
sudo mysqld_safe --skip-grant-tables &
mysql -u root
UPDATE mysql.user SET authentication_string=PASSWORD('new') WHERE User='root';
FLUSH PRIVILEGES;
```

---

### LEVEL 1 - DIRECT: Table Locked

**Scenario**: Query hangs on locked table.

```sql
SELECT * FROM orders;
-- Hangs...
```

**Investigation**:
```sql
SHOW PROCESSLIST;
SHOW ENGINE INNODB STATUS;
```

**Solution**:
```sql
-- Kill blocking query
KILL <processlist_id>;

-- Or wait for lock timeout
```

---

# QUICK REFERENCE

## PostgreSQL Commands

```sql
-- Connections
SELECT * FROM pg_stat_activity;
SELECT pg_terminate_backend(pid);

-- Table stats
SELECT * FROM pg_stat_user_tables;

-- Indexes
SELECT * FROM pg_stat_user_indexes;

-- Locks
SELECT * FROM pg_locks;

-- Size
SELECT pg_size_pretty(pg_database_size('mydb'));
SELECT pg_size_pretty(pg_relation_size('mytable'));

-- Explain query
EXPLAIN ANALYZE SELECT ...;
```

## MySQL Commands

```sql
-- Connections
SHOW PROCESSLIST;
KILL <id>;

-- Table stats
SHOW TABLE STATUS;

-- Explain query
EXPLAIN SELECT ...;

-- Locks
SHOW ENGINE INNODB STATUS;
```

## Redis Commands

```bash
INFO                    # Server info
INFO memory             # Memory stats
INFO clients            # Client info
SLOWLOG GET 10          # Slow queries
MONITOR                 # Real-time commands
DEBUG SLEEP 0           # Check if responsive
CLIENT LIST             # Connected clients
CONFIG GET maxmemory    # Check settings
```

## Common Errors â†’ Solutions

```
"Connection refused"        â†’ Start database service
"Authentication failed"     â†’ Check credentials
"Too many connections"      â†’ Kill idle, increase limit
"No space left"             â†’ VACUUM or add disk
"Lock wait timeout"         â†’ Find and kill blocking query
"OOM"                       â†’ Increase memory limit
```
