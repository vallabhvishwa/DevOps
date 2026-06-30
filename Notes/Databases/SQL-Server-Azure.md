# SQL Server on Azure — DevOps Reference

> **last_reviewed:** 2026-06-30  
> **See also:** [Databases Overview](DevOps-21-Databases.md) | [Azure Databases](../Azure/Azure-04-Database-Services.md) | [Troubleshooting](../../Troubleshooting/Databases/SQL-Server-Azure-Troubleshooting.md)

---

## 1. Deployment options

| Option | Use when | DevOps notes |
|--------|----------|--------------|
| **Azure SQL Database** | PaaS, managed patches | Private endpoint, firewall rules, elastic pool |
| **Azure SQL Managed Instance** | Near full SQL Server compatibility | VNet integration, linked servers |
| **SQL Server on Azure VM** | Legacy apps, full OS control | You patch OS + SQL; backup your problem |
| **SQL Server in K8s** | Rare; dev/test only | StatefulSet + PVC; not for prod billing systems |

---

## 2. Connectivity from AKS

```
┌─────────────┐     Private Link      ┌──────────────────┐
│  AKS Pod    │ ────────────────────▶│  Azure SQL       │
│  (app/ldr)  │   privatelink.database │  *.database.     │
└─────────────┘   .windows.net         │  windows.net     │
                                       └──────────────────┘
```

**Connection string pattern:**

```
Server=tcp:myserver.database.windows.net,1433;
Database=mydb;
User ID=<user>@myserver;
Password=<password>;
Encrypt=True;
TrustServerCertificate=False;
Connection Timeout=30;
```

**From private endpoint:**

```
Server=tcp:myserver.privatelink.database.windows.net,1433;...
```

---

## 3. Terraform sketch (Azure SQL + private endpoint)

```hcl
resource "azurerm_mssql_server" "main" {
  name                         = "sql-${var.environment}"
  resource_group_name          = azurerm_resource_group.main.name
  location                     = var.location
  version                      = "12.0"
  administrator_login          = var.sql_admin_login
  administrator_login_password = var.sql_admin_password
  minimum_tls_version          = "1.2"
}

resource "azurerm_mssql_database" "app" {
  name      = "petclinic"
  server_id = azurerm_mssql_server.main.id
  sku_name  = "S0"
}

resource "azurerm_private_endpoint" "sql" {
  name                = "pe-sql"
  location            = var.location
  resource_group_name = azurerm_resource_group.main.name
  subnet_id           = azurerm_subnet.private_endpoints.id

  private_service_connection {
    name                           = "psc-sql"
    private_connection_resource_id = azurerm_mssql_server.main.id
    subresource_names              = ["sqlServer"]
    is_manual_connection           = false
  }

  private_dns_zone_group {
    name                 = "sql-dns"
    private_dns_zone_ids = [azurerm_private_dns_zone.sql.id]
  }
}
```

---

## 4. Secrets in Kubernetes

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: sql-credentials
type: Opaque
stringData:
  connection-string: "Server=tcp:..."
```

**Better:** Azure Key Vault + CSI driver or External Secrets Operator. See [Private Link & Key Vault](../Azure/Azure-Private-Link-KeyVault-CICD.md).

```yaml
# External Secrets (conceptual)
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: sql-credentials
spec:
  secretStoreRef:
    name: azure-keyvault
    kind: SecretStore
  target:
    name: sql-credentials
  data:
    - secretKey: connection-string
      remoteRef:
        key: sql-connection-string
```

---

## 5. Flyway migrations in K8s

Pattern: Helm pre-install/pre-upgrade Job runs Flyway before app pods start.

```yaml
# Job spec (simplified)
containers:
  - name: flyway
    image: flyway/flyway:10
    args:
      - migrate
    env:
      - name: FLYWAY_URL
        value: "jdbc:sqlserver://myserver.database.windows.net:1433;databaseName=petclinic"
      - name: FLYWAY_USER
        valueFrom:
          secretKeyRef:
            name: sql-credentials
            key: username
      - name: FLYWAY_PASSWORD
        valueFrom:
          secretKeyRef:
            name: sql-credentials
            key: password
    volumeMounts:
      - name: migrations
        mountPath: /flyway/sql
volumes:
  - name: migrations
    configMap:
      name: flyway-scripts
```

**Helm hook:** See [Helm Enterprise Patterns](../Helm/Helm-Enterprise-Patterns.md#6-hooks--db-migration-job).

---

## 6. Failover and DR

| Feature | Azure SQL Database | SQL MI |
|---------|-------------------|--------|
| Active geo-replication | Yes | Yes |
| Failover groups | Auto-failover DNS | Yes |
| RPO/RTO | SLA-driven | SLA-driven |

**DevOps checklist:**
- Document failover runbook (manual vs automatic)
- Test failover quarterly
- Update connection strings / DNS after failover
- Jenkins pipeline: parameter for primary vs DR SQL host

---

## 7. Monitoring

```sql
-- Active connections
SELECT COUNT(*) AS active_connections
FROM sys.dm_exec_sessions
WHERE is_user_process = 1;

-- Blocking
SELECT * FROM sys.dm_exec_requests WHERE blocking_session_id <> 0;

-- Top CPU queries
SELECT TOP 10
  total_worker_time/execution_count AS avg_cpu,
  SUBSTRING(st.text, (qs.statement_start_offset/2)+1, ...) AS query_text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) st
ORDER BY avg_cpu DESC;
```

**Azure Monitor:** DTU/vCore %, deadlocks, failed connections — alert at 80% DTU sustained.

---

## 8. Operational commands

```bash
# Test connectivity from jump box or debug pod
kubectl run sqltest --rm -it --image=mcr.microsoft.com/mssql-tools -- \
  /opt/mssql-tools/bin/sqlcmd -S myserver.database.windows.net -U user -P pass -Q "SELECT 1"

# Azure CLI
az sql server list --resource-group myRG
az sql db list --server myserver --resource-group myRG
az sql db show-connection-string --client ado.net --name petclinic --server myserver
```

---

## Related

- [SQL Server Troubleshooting](../../Troubleshooting/Databases/SQL-Server-Azure-Troubleshooting.md)
- [Database Troubleshooting (general)](../../Troubleshooting/Databases/08-Database-Troubleshooting.md)
- [PetClinic project](../../Projects/PetClinic.md)
