# Azure Storage Services - Complete Guide
## Blob, Files, Queue, Table, Disks, Data Lake

---

# 1. AZURE STORAGE ACCOUNT

## What It Is
Storage account is the container for all Azure Storage data objects. Provides a unique namespace accessible from anywhere via HTTP/HTTPS.

## Types
```
STORAGE ACCOUNT TYPES:
┌──────────────────────┬─────────────────────────────────────────┐
│ Type                 │ Supported Services                      │
├──────────────────────┼─────────────────────────────────────────┤
│ Standard GPv2        │ Blob, Files, Queue, Table              │
│ Premium Block Blobs  │ Block blobs (high transaction)         │
│ Premium File Shares  │ Azure Files only                       │
│ Premium Page Blobs   │ Page blobs only (unmanaged disks)      │
└──────────────────────┴─────────────────────────────────────────┘
```

## Redundancy Options
```
REDUNDANCY:
┌──────────────────────┬──────────────┬───────────────┬──────────┐
│ Option               │ Copies       │ Location      │ Durability│
├──────────────────────┼──────────────┼───────────────┼──────────┤
│ LRS (Locally Redund.)│ 3            │ Single DC     │ 11 nines │
│ ZRS (Zone Redundant) │ 3            │ 3 zones       │ 12 nines │
│ GRS (Geo Redundant)  │ 6            │ 2 regions     │ 16 nines │
│ GZRS (Geo-Zone)      │ 6            │ 3 zones + 2nd │ 16 nines │
│ RA-GRS (Read Access) │ 6            │ 2 regions     │ 16 nines │
│ RA-GZRS              │ 6            │ 3 zones + 2nd │ 16 nines │
└──────────────────────┴──────────────┴───────────────┴──────────┘
```

## CLI Commands

```bash
# Create storage account
az storage account create \
  --name mystorageaccount \
  --resource-group myRG \
  --location eastus \
  --sku Standard_ZRS \
  --kind StorageV2

# Get connection string
az storage account show-connection-string \
  --name mystorageaccount \
  --resource-group myRG

# Get keys
az storage account keys list \
  --name mystorageaccount \
  --resource-group myRG

# Enable blob versioning
az storage account blob-service-properties update \
  --account-name mystorageaccount \
  --resource-group myRG \
  --enable-versioning true

# Configure lifecycle management
az storage account management-policy create \
  --account-name mystorageaccount \
  --resource-group myRG \
  --policy @policy.json
```

---

# 2. BLOB STORAGE

## What It Is
Object storage for unstructured data. Optimized for storing massive amounts of data like images, videos, backups, logs.

## Blob Types
```
BLOB TYPES:
┌─────────────────┬─────────────────────────────────────────────┐
│ Block Blobs     │ Text and binary data (files, media)        │
│                 │ Up to 190.7 TB                              │
│                 │ Best for: Documents, images, videos         │
├─────────────────┼─────────────────────────────────────────────┤
│ Append Blobs    │ Optimized for append operations            │
│                 │ Up to 195 GB                                │
│                 │ Best for: Logging, audit trails            │
├─────────────────┼─────────────────────────────────────────────┤
│ Page Blobs      │ Random read/write access                   │
│                 │ Up to 8 TB                                  │
│                 │ Best for: VHD disks, databases             │
└─────────────────┴─────────────────────────────────────────────┘
```

## Access Tiers
```
ACCESS TIERS:
┌─────────────────┬───────────────┬────────────────┬─────────────┐
│ Tier            │ Storage Cost  │ Access Cost    │ Min Days    │
├─────────────────┼───────────────┼────────────────┼─────────────┤
│ Hot             │ Highest       │ Lowest         │ None        │
│ Cool            │ Lower         │ Higher         │ 30 days     │
│ Cold            │ Even lower    │ Even higher    │ 90 days     │
│ Archive         │ Lowest        │ Highest        │ 180 days    │
└─────────────────┴───────────────┴────────────────┴─────────────┘

Note: Archive tier requires rehydration (hours to retrieve)
```

## CLI Commands

```bash
# Create container
az storage container create \
  --name mycontainer \
  --account-name mystorageaccount \
  --public-access off

# Upload blob
az storage blob upload \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myfile.txt \
  --file ./localfile.txt

# Download blob
az storage blob download \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myfile.txt \
  --file ./downloaded.txt

# List blobs
az storage blob list \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --output table

# Change access tier
az storage blob set-tier \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myfile.txt \
  --tier Cool

# Generate SAS token
az storage blob generate-sas \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myfile.txt \
  --permissions r \
  --expiry 2024-12-31T23:59:59Z

# Copy blob
az storage blob copy start \
  --account-name destaccount \
  --destination-container destcontainer \
  --destination-blob destblob.txt \
  --source-uri "https://sourceaccount.blob.core.windows.net/container/blob.txt"
```

---

# 3. AZURE FILES

## What It Is
Fully managed file shares accessible via SMB and NFS protocols. Can be mounted by cloud or on-premises deployments.

## Key Features
```
FEATURES:
- SMB 3.0/3.1 and NFS 4.1 protocols
- Shared across multiple VMs
- Supports Azure AD authentication
- Snapshots
- Soft delete
- Azure File Sync (hybrid)
```

## Tiers
```
FILE SHARE TIERS:
┌─────────────────┬───────────────────────────────────────────────┐
│ Premium         │ SSD, low latency, high IOPS                  │
│ Transaction Opt.│ HDD, balanced cost                           │
│ Hot             │ HDD, general purpose                         │
│ Cool            │ HDD, cost-optimized storage                  │
└─────────────────┴───────────────────────────────────────────────┘
```

## CLI Commands

```bash
# Create file share
az storage share create \
  --name myshare \
  --account-name mystorageaccount \
  --quota 100  # GB

# Upload file
az storage file upload \
  --share-name myshare \
  --source ./localfile.txt \
  --path myfile.txt \
  --account-name mystorageaccount

# List files
az storage file list \
  --share-name myshare \
  --account-name mystorageaccount \
  --output table

# Create directory
az storage directory create \
  --share-name myshare \
  --name mydir \
  --account-name mystorageaccount

# Mount on Linux
sudo mount -t cifs //mystorageaccount.file.core.windows.net/myshare /mnt/myshare \
  -o vers=3.0,username=mystorageaccount,password=<storage-key>,dir_mode=0777,file_mode=0777

# Mount on Windows
net use Z: \\mystorageaccount.file.core.windows.net\myshare /u:mystorageaccount <storage-key>
```

---

# 4. QUEUE STORAGE

## What It Is
Simple message queue service for asynchronous communication between components. Each message up to 64KB.

## Use Cases
- Decoupling application components
- Work queue for background processing
- Task scheduling

## CLI Commands

```bash
# Create queue
az storage queue create \
  --name myqueue \
  --account-name mystorageaccount

# Add message
az storage message put \
  --queue-name myqueue \
  --content "Hello World" \
  --account-name mystorageaccount

# Peek message (don't delete)
az storage message peek \
  --queue-name myqueue \
  --account-name mystorageaccount

# Get message (makes invisible for processing)
az storage message get \
  --queue-name myqueue \
  --account-name mystorageaccount

# Delete message
az storage message delete \
  --queue-name myqueue \
  --id <message-id> \
  --pop-receipt <pop-receipt> \
  --account-name mystorageaccount
```

---

# 5. TABLE STORAGE

## What It Is
NoSQL key-value store for semi-structured data. Schemaless design with fast access.

## When to Use
- Simple structured data
- Flexible schema
- Fast key-based lookup
- Cost-sensitive scenarios

## CLI Commands

```bash
# Create table
az storage table create \
  --name mytable \
  --account-name mystorageaccount

# Insert entity
az storage entity insert \
  --table-name mytable \
  --entity PartitionKey=pk1 RowKey=rk1 Name=John Age=30 \
  --account-name mystorageaccount

# Query entities
az storage entity query \
  --table-name mytable \
  --filter "PartitionKey eq 'pk1'" \
  --account-name mystorageaccount
```

---

# 6. MANAGED DISKS

## What It Is
Block-level storage volumes for Azure VMs. Managed by Azure (no storage account management needed).

## Disk Types
```
MANAGED DISK TYPES:
┌──────────────────┬───────────────┬─────────────┬────────────────┐
│ Type             │ Max IOPS      │ Max Thrpt   │ Use Case       │
├──────────────────┼───────────────┼─────────────┼────────────────┤
│ Standard HDD     │ 500           │ 60 MB/s     │ Dev/test       │
│ Standard SSD     │ 6,000         │ 750 MB/s    │ Web servers    │
│ Premium SSD      │ 20,000        │ 900 MB/s    │ Production     │
│ Premium SSD v2   │ 80,000        │ 1,200 MB/s  │ Critical apps  │
│ Ultra Disk       │ 160,000       │ 4,000 MB/s  │ SAP, databases │
└──────────────────┴───────────────┴─────────────┴────────────────┘
```

## CLI Commands

```bash
# Create managed disk
az disk create \
  --resource-group myRG \
  --name myDisk \
  --size-gb 128 \
  --sku Premium_LRS

# Attach disk to VM
az vm disk attach \
  --resource-group myRG \
  --vm-name myVM \
  --name myDisk

# Detach disk
az vm disk detach \
  --resource-group myRG \
  --vm-name myVM \
  --name myDisk

# Create snapshot
az snapshot create \
  --resource-group myRG \
  --name mySnapshot \
  --source myDisk
```

---

# 7. DATA LAKE STORAGE GEN2

## What It Is
Combines Azure Blob Storage with hierarchical namespace for big data analytics. Optimized for analytics workloads.

## Key Features
```
FEATURES:
- Hierarchical namespace (true directories)
- POSIX-compliant ACLs
- Optimized for analytics (Hadoop, Spark)
- Blob storage compatibility
- All blob features (tiers, lifecycle)
```

## CLI Commands

```bash
# Create storage account with hierarchical namespace
az storage account create \
  --name mydatalake \
  --resource-group myRG \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2 \
  --enable-hierarchical-namespace true

# Create file system (container)
az storage fs create \
  --name myfilesystem \
  --account-name mydatalake

# Create directory
az storage fs directory create \
  --name mydir \
  --file-system myfilesystem \
  --account-name mydatalake

# Upload file
az storage fs file upload \
  --file-system myfilesystem \
  --path mydir/myfile.txt \
  --source ./localfile.txt \
  --account-name mydatalake
```

---

# SECURITY BEST PRACTICES

```
STORAGE SECURITY:
┌─────────────────────────────────────────────────────────────────┐
│ 1. DISABLE PUBLIC ACCESS                                       │
│    az storage account update --allow-blob-public-access false   │
│                                                                 │
│ 2. USE PRIVATE ENDPOINTS                                        │
│    Access storage only from VNet                                │
│                                                                 │
│ 3. ENABLE FIREWALL                                              │
│    Restrict to specific VNets/IPs                               │
│                                                                 │
│ 4. USE MANAGED IDENTITIES                                       │
│    No keys in code                                              │
│                                                                 │
│ 5. ENABLE SOFT DELETE                                           │
│    Protect against accidental deletion                          │
│                                                                 │
│ 6. ENABLE VERSIONING                                            │
│    Keep previous versions of blobs                              │
│                                                                 │
│ 7. USE RBAC                                                     │
│    Storage Blob Data Reader/Contributor                         │
│                                                                 │
│ 8. AVOID STORAGE KEYS                                           │
│    Use SAS tokens or Azure AD                                   │
└─────────────────────────────────────────────────────────────────┘
```
