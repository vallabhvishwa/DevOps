# SQL Server / Azure SQL Troubleshooting

> **Learning reference:** [SQL Server on Azure](../../Notes/Databases/SQL-Server-Azure.md) | [Databases Overview](../../Notes/Databases/DevOps-21-Databases.md) | [Azure Databases](../../Notes/Azure/Azure-04-Database-Services.md)

---

## Methodology

```
1. Can you reach the server?     → telnet/sqlcmd, DNS, private endpoint
2. Can you authenticate?         → login, firewall, AAD vs SQL auth
3. Can you open the database?    → permissions, DB online/offline
4. Is the app connection pool OK? → timeouts, max connections
5. Is migration/schema correct?  → Flyway logs, schema version table
```

---

## 1. Login failed for user

### LEVEL 1 — Wrong password or user

```
Login failed for user 'appuser'.
```

```bash
# Verify credentials from secret
kubectl get secret sql-credentials -o jsonpath='{.data.username}' | base64 -d

# Test from pod
sqlcmd -S myserver.database.windows.net -U appuser -P "$DB_PASS" -Q "SELECT 1"
```

**Fix:** Rotate secret in Key Vault; `kubectl rollout restart deployment/myapp`

### LEVEL 2 — Firewall / private endpoint

```
Cannot open server 'myserver' requested by the login. Client with IP address 'x.x.x.x' is not allowed...
```

```bash
# Check if using public endpoint from private-only network
nslookup myserver.database.windows.net

# Azure: allow AKS egress IP or use private endpoint only
az sql server firewall-rule list --server myserver -g myRG
```

**Fix:** Add firewall rule for AKS outbound IP, or route via private endpoint + private DNS zone.

---

## 2. DNS / Private Link failures

### Symptom

```
A network-related or instance-specific error occurred while establishing a connection to SQL Server
```

```bash
# From AKS debug pod
nslookup myserver.privatelink.database.windows.net
nslookup myserver.database.windows.net

# Should resolve to private IP (10.x) when using Private Link
dig +short myserver.privatelink.database.windows.net
```

**Causes:**
- Private DNS zone not linked to AKS VNet
- Wrong hostname in connection string (public vs privatelink)
- `no_proxy` missing privatelink suffix in Jenkins/agent

**Fix:** Link `privatelink.database.windows.net` zone to VNet; update connection string and proxy bypass list.

---

## 3. Connection timeouts / pool exhaustion

### Symptom

App logs: `Connection timed out`, `HikariPool - Connection is not available`

```sql
-- On SQL Server
SELECT DB_NAME(dbid) AS db, COUNT(*) AS connections
FROM sys.sysprocesses
WHERE dbid > 0
GROUP BY dbid
ORDER BY connections DESC;
```

```bash
# App pod — count open connections to SQL port
kubectl exec -it myapp-pod -- ss -tn | grep 1433 | wc -l
```

**Fix:**
- Reduce pool size per replica: `spring.datasource.hikari.maximum-pool-size=10`
- Scale app replicas carefully (each replica = N connections)
- Use connection pooling at gateway if many microservices hit same DB

---

## 4. Flyway / migration job failures

### Symptom

Helm pre-upgrade hook job `Failed`; pod logs show Flyway error.

```bash
kubectl logs job/petclinic-migrate -n petclinic
kubectl describe job petclinic-migrate
```

| Error | Cause | Fix |
|-------|-------|-----|
| `Validate failed: Migration checksum mismatch` | Script changed after apply | Repair or baseline in dev only |
| `Found non-empty schema without metadata table` | Existing DB, no flyway_schema_history | `baselineOnMigrate=true` (careful in prod) |
| `Login failed` | Job secret wrong | Fix ExternalSecret sync |

```bash
# Re-run migration job manually
kubectl delete job petclinic-migrate
helm upgrade petclinic ./helm/petclinic --reuse-values
```

---

## 5. DTU / performance degradation

### Symptom

Slow queries, app timeouts, Azure alert on DTU > 80%

```sql
-- Current waits
SELECT wait_type, waiting_tasks_count, wait_time_ms
FROM sys.dm_os_wait_stats
ORDER BY wait_time_ms DESC;

-- Blocking chain
SELECT blocking.session_id AS blocker, blocked.session_id AS blocked
FROM sys.dm_exec_requests blocked
JOIN sys.dm_exec_requests blocking ON blocked.blocking_session_id = blocking.session_id;
```

**Fix:**
- Scale up SKU or elastic pool
- Add indexes (dev team)
- Kill long-running blocker (emergency only): `KILL <session_id>`

---

## 6. Failover / DR connection failures

After geo-failover, apps still point to old server name.

**Check:**
```bash
az sql failover-group show --name myfg --server myserver -g myRG
```

**Fix:** Update connection string / Key Vault secret to failover group listener FQDN; restart deployments.

---

## Quick reference

| Symptom | First command |
|---------|---------------|
| Can't connect | `sqlcmd -S server -Q "SELECT 1"` |
| DNS issue | `nslookup server.privatelink.database.windows.net` |
| Auth issue | Check K8s secret vs Key Vault |
| Migration fail | `kubectl logs job/<migrate-job>` |
| Slow DB | Azure Portal → Metrics → DTU % |
| Blocking | `sys.dm_exec_requests` blocking chain |

---

## Related

- [General DB Troubleshooting](08-Database-Troubleshooting.md)
- [Networking Troubleshooting](../Networking/04-Networking-Troubleshooting.md)
- [Azure Services Troubleshooting](../Azure/11-Azure-Services-Troubleshooting.md)
