# Azure Database Services - Complete Guide
## SQL, PostgreSQL, MySQL, Cosmos DB, Redis

---

# 1. AZURE SQL DATABASE

## What It Is
Fully managed relational database with SQL Server engine. Automatic patching, backups, and high availability.

## Purchasing Models
```
PURCHASING MODELS:
┌─────────────────────────────────────────────────────────────────┐
│ DTU MODEL (Database Transaction Units):                        │
│ - Bundled compute, storage, I/O                                │
│ - Simple pricing                                               │
│ - Basic, Standard, Premium tiers                               │
│                                                                 │
│ vCORE MODEL:                                                    │
│ - Choose compute and storage separately                        │
│ - General Purpose, Business Critical, Hyperscale               │
│ - Better for migration from SQL Server                         │
└─────────────────────────────────────────────────────────────────┘
```

## Deployment Options
```
DEPLOYMENT OPTIONS:
┌─────────────────────────────────────────────────────────────────┐
│ SINGLE DATABASE:                                                │
│ - Isolated database                                            │
│ - Own resources                                                │
│                                                                 │
│ ELASTIC POOL:                                                   │
│ - Multiple databases share resources                           │
│ - Cost-effective for variable workloads                        │
│                                                                 │
│ MANAGED INSTANCE:                                               │
│ - Near 100% SQL Server compatibility                           │
│ - VNet integration                                             │
│ - SQL Agent, CLR, cross-database queries                       │
└─────────────────────────────────────────────────────────────────┘
```

## CLI Commands

```bash
# Create SQL Server
az sql server create \
  --name mysqlserver \
  --resource-group myRG \
  --location eastus \
  --admin-user myadmin \
  --admin-password 'MyP@ssw0rd!'

# Create database
az sql db create \
  --resource-group myRG \
  --server mysqlserver \
  --name mydb \
  --service-objective S0

# Create elastic pool
az sql elastic-pool create \
  --resource-group myRG \
  --server mysqlserver \
  --name mypool \
  --edition Standard \
  --dtu 100

# Add database to pool
az sql db create \
  --resource-group myRG \
  --server mysqlserver \
  --name mydb2 \
  --elastic-pool mypool

# Configure firewall rule
az sql server firewall-rule create \
  --resource-group myRG \
  --server mysqlserver \
  --name AllowMyIP \
  --start-ip-address 1.2.3.4 \
  --end-ip-address 1.2.3.4

# Allow Azure services
az sql server firewall-rule create \
  --resource-group myRG \
  --server mysqlserver \
  --name AllowAzure \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 0.0.0.0

# Create private endpoint
az network private-endpoint create \
  --name sqlPrivateEndpoint \
  --resource-group myRG \
  --vnet-name myVNet \
  --subnet PrivateEndpointSubnet \
  --private-connection-resource-id $(az sql server show -g myRG -n mysqlserver --query id -o tsv) \
  --group-id sqlServer \
  --connection-name sqlConnection
```

---

# 2. AZURE DATABASE FOR POSTGRESQL

## What It Is
Fully managed PostgreSQL database service. Automatic backups, patching, and high availability.

## Deployment Options
```
DEPLOYMENT OPTIONS:
┌─────────────────────────────────────────────────────────────────┐
│ FLEXIBLE SERVER (Recommended):                                  │
│ - Zone-redundant HA                                            │
│ - Configurable maintenance window                              │
│ - Stop/start capability                                        │
│ - PgBouncer built-in                                           │
│                                                                 │
│ SINGLE SERVER (Legacy):                                         │
│ - Being deprecated                                             │
│ - Migrate to Flexible Server                                   │
└─────────────────────────────────────────────────────────────────┘
```

## CLI Commands

```bash
# Create Flexible Server
az postgres flexible-server create \
  --resource-group myRG \
  --name mypgserver \
  --location eastus \
  --admin-user myadmin \
  --admin-password 'MyP@ssw0rd!' \
  --sku-name Standard_D2s_v3 \
  --tier GeneralPurpose \
  --storage-size 128 \
  --version 15

# Create database
az postgres flexible-server db create \
  --resource-group myRG \
  --server-name mypgserver \
  --database-name mydb

# Configure firewall
az postgres flexible-server firewall-rule create \
  --resource-group myRG \
  --name mypgserver \
  --rule-name AllowMyIP \
  --start-ip-address 1.2.3.4 \
  --end-ip-address 1.2.3.4

# Enable extensions
az postgres flexible-server parameter set \
  --resource-group myRG \
  --server-name mypgserver \
  --name azure.extensions \
  --value "pg_stat_statements,uuid-ossp"

# Enable high availability
az postgres flexible-server update \
  --resource-group myRG \
  --name mypgserver \
  --high-availability ZoneRedundant

# Connect with psql
psql "host=mypgserver.postgres.database.azure.com port=5432 dbname=mydb user=myadmin sslmode=require"
```

---

# 3. AZURE DATABASE FOR MYSQL

## What It Is
Fully managed MySQL database service. Supports MySQL 5.7 and 8.0.

## CLI Commands

```bash
# Create Flexible Server
az mysql flexible-server create \
  --resource-group myRG \
  --name mymysqlserver \
  --location eastus \
  --admin-user myadmin \
  --admin-password 'MyP@ssw0rd!' \
  --sku-name Standard_D2ds_v4 \
  --tier GeneralPurpose \
  --storage-size 128 \
  --version 8.0.21

# Create database
az mysql flexible-server db create \
  --resource-group myRG \
  --server-name mymysqlserver \
  --database-name mydb

# Configure firewall
az mysql flexible-server firewall-rule create \
  --resource-group myRG \
  --name mymysqlserver \
  --rule-name AllowMyIP \
  --start-ip-address 1.2.3.4 \
  --end-ip-address 1.2.3.4

# Connect with mysql client
mysql -h mymysqlserver.mysql.database.azure.com -u myadmin -p mydb --ssl
```

---

# 4. AZURE COSMOS DB

## What It Is
Globally distributed, multi-model NoSQL database. Single-digit millisecond latency at any scale.

## APIs
```
SUPPORTED APIs:
┌─────────────────────────────────────────────────────────────────┐
│ NoSQL (Core/SQL):  Document database, SQL-like queries        │
│ MongoDB:           MongoDB wire protocol compatible            │
│ Cassandra:         Column-family database                      │
│ Gremlin:           Graph database                              │
│ Table:             Key-value (Azure Table compatible)          │
│ PostgreSQL:        Distributed PostgreSQL (Citus)              │
└─────────────────────────────────────────────────────────────────┘
```

## Key Concepts
```
CONCEPTS:
┌─────────────────────────────────────────────────────────────────┐
│ ACCOUNT:           Top-level resource                          │
│ DATABASE:          Container for collections                   │
│ CONTAINER:         Collection of items                         │
│ ITEM:              Document (JSON)                             │
│ PARTITION KEY:     Determines data distribution                │
│ REQUEST UNITS (RU): Throughput measurement                     │
│                                                                 │
│ CONSISTENCY LEVELS:                                             │
│ - Strong: Guaranteed latest                                    │
│ - Bounded Staleness: Lag within bounds                         │
│ - Session: Consistent within session (default)                 │
│ - Consistent Prefix: Read in order                             │
│ - Eventual: Lowest latency                                     │
└─────────────────────────────────────────────────────────────────┘
```

## CLI Commands

```bash
# Create Cosmos DB account
az cosmosdb create \
  --name mycosmosaccount \
  --resource-group myRG \
  --kind GlobalDocumentDB \
  --default-consistency-level Session \
  --locations regionName=eastus failoverPriority=0 \
  --locations regionName=westus failoverPriority=1

# Create database
az cosmosdb sql database create \
  --account-name mycosmosaccount \
  --resource-group myRG \
  --name mydb

# Create container
az cosmosdb sql container create \
  --account-name mycosmosaccount \
  --resource-group myRG \
  --database-name mydb \
  --name mycontainer \
  --partition-key-path "/partitionKey" \
  --throughput 400

# Get connection string
az cosmosdb keys list \
  --name mycosmosaccount \
  --resource-group myRG \
  --type connection-strings
```

---

# 5. AZURE CACHE FOR REDIS

## What It Is
Fully managed Redis cache. In-memory data store for high-performance scenarios.

## Use Cases
- Session store
- Response caching
- Message broker
- Real-time analytics

## Tiers
```
TIERS:
┌─────────────────┬─────────────────────────────────────────────┐
│ Basic           │ Dev/test, single node, no SLA              │
│ Standard        │ Replicated, 99.9% SLA                      │
│ Premium         │ Clustering, persistence, VNet              │
│ Enterprise      │ Redis Enterprise, RediSearch, RedisJSON    │
│ Enterprise Flash│ Large datasets on flash storage            │
└─────────────────┴─────────────────────────────────────────────┘
```

## CLI Commands

```bash
# Create Redis cache
az redis create \
  --name myredis \
  --resource-group myRG \
  --location eastus \
  --sku Standard \
  --vm-size C1

# Get keys
az redis list-keys \
  --name myredis \
  --resource-group myRG

# Get connection string
az redis show \
  --name myredis \
  --resource-group myRG \
  --query hostName -o tsv

# Create firewall rule
az redis firewall-rules create \
  --name myredis \
  --resource-group myRG \
  --rule-name AllowMyIP \
  --start-ip 1.2.3.4 \
  --end-ip 1.2.3.4

# Connect with redis-cli
redis-cli -h myredis.redis.cache.windows.net -p 6380 -a <access-key> --tls
```

---

# DATABASE COMPARISON

| Feature | SQL DB | PostgreSQL | MySQL | Cosmos DB | Redis |
|---------|--------|------------|-------|-----------|-------|
| Type | Relational | Relational | Relational | NoSQL | Cache |
| Model | SQL Server | PostgreSQL | MySQL | Multi-model | Key-Value |
| Global Dist. | Geo-repl | Read replicas | Read replicas | Native | Geo-repl |
| Best For | Enterprise | Open source | Web apps | Planet scale | Caching |
