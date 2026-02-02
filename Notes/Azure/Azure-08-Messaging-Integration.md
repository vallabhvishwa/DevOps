# Azure Messaging & Integration - Complete Guide
## Service Bus, Event Hubs, Event Grid, Logic Apps, API Management

---

# 1. AZURE SERVICE BUS

## What It Is
Enterprise messaging service for decoupling applications. Supports queues (point-to-point) and topics (publish-subscribe).

## Key Concepts
```
SERVICE BUS COMPONENTS:
┌─────────────────────────────────────────────────────────────────┐
│ NAMESPACE:         Container for messaging entities            │
│ QUEUE:             Point-to-point messaging                    │
│ TOPIC:             Publish-subscribe messaging                 │
│ SUBSCRIPTION:      Receiver on a topic                         │
│ DEAD-LETTER QUEUE: Failed message storage                      │
└─────────────────────────────────────────────────────────────────┘
```

## Tiers
```
SERVICE BUS TIERS:
┌──────────────┬───────────────────────────────────────────────────┐
│ Basic        │ Queues only, no topics                           │
│ Standard     │ Queues + Topics, variable throughput             │
│ Premium      │ Dedicated capacity, predictable performance       │
└──────────────┴───────────────────────────────────────────────────┘
```

## CLI Commands

```bash
# Create namespace
az servicebus namespace create \
  --resource-group myRG \
  --name myservicebus \
  --location eastus \
  --sku Standard

# Create queue
az servicebus queue create \
  --resource-group myRG \
  --namespace-name myservicebus \
  --name myqueue \
  --max-size 1024 \
  --default-message-time-to-live P14D

# Create topic
az servicebus topic create \
  --resource-group myRG \
  --namespace-name myservicebus \
  --name mytopic

# Create subscription
az servicebus topic subscription create \
  --resource-group myRG \
  --namespace-name myservicebus \
  --topic-name mytopic \
  --name mysubscription

# Get connection string
az servicebus namespace authorization-rule keys list \
  --resource-group myRG \
  --namespace-name myservicebus \
  --name RootManageSharedAccessKey \
  --query primaryConnectionString -o tsv
```

---

# 2. AZURE EVENT HUBS

## What It Is
Big data streaming platform. Ingests millions of events per second for real-time analytics.

## Key Concepts
```
EVENT HUBS COMPONENTS:
┌─────────────────────────────────────────────────────────────────┐
│ NAMESPACE:         Container for event hubs                    │
│ EVENT HUB:         Stream of events                            │
│ PARTITION:         Parallel stream segment                     │
│ CONSUMER GROUP:    Independent reader view                     │
│ CAPTURE:           Auto-save to storage                        │
└─────────────────────────────────────────────────────────────────┘
```

## CLI Commands

```bash
# Create namespace
az eventhubs namespace create \
  --resource-group myRG \
  --name myeventhubs \
  --location eastus \
  --sku Standard

# Create event hub
az eventhubs eventhub create \
  --resource-group myRG \
  --namespace-name myeventhubs \
  --name myeventhub \
  --partition-count 4 \
  --message-retention 7

# Create consumer group
az eventhubs eventhub consumer-group create \
  --resource-group myRG \
  --namespace-name myeventhubs \
  --eventhub-name myeventhub \
  --name myconsumergroup

# Get connection string
az eventhubs namespace authorization-rule keys list \
  --resource-group myRG \
  --namespace-name myeventhubs \
  --name RootManageSharedAccessKey \
  --query primaryConnectionString -o tsv
```

---

# 3. AZURE EVENT GRID

## What It Is
Event routing service for reactive programming. Routes events from sources to handlers.

## Key Concepts
```
EVENT GRID COMPONENTS:
┌─────────────────────────────────────────────────────────────────┐
│ EVENT SOURCE:      Origin (Blob Storage, Resource Groups, etc.)│
│ TOPIC:             Endpoint for event publishers               │
│ EVENT SUBSCRIPTION: Defines which events to receive            │
│ EVENT HANDLER:     Destination (Function, Webhook, Queue, etc.)│
└─────────────────────────────────────────────────────────────────┘
```

## Common Event Sources
```
BUILT-IN SOURCES:
- Azure Subscriptions (resource changes)
- Resource Groups (resource changes)
- Storage Accounts (blob created/deleted)
- Container Registry (image pushed)
- Event Hubs (capture file available)
- IoT Hub (device events)
- Custom Topics (your applications)
```

## CLI Commands

```bash
# Create custom topic
az eventgrid topic create \
  --resource-group myRG \
  --name mytopic \
  --location eastus

# Create event subscription (to webhook)
az eventgrid event-subscription create \
  --name mysubscription \
  --source-resource-id /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.EventGrid/topics/mytopic \
  --endpoint https://myfunction.azurewebsites.net/api/handler

# Create event subscription (to storage queue)
az eventgrid event-subscription create \
  --name mysubscription \
  --source-resource-id /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Storage/storageAccounts/mystorageaccount \
  --endpoint-type storagequeue \
  --endpoint /subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Storage/storageAccounts/mystorageaccount/queueServices/default/queues/myqueue
```

---

# 4. AZURE API MANAGEMENT

## What It Is
API gateway for publishing APIs to external and internal consumers. Provides security, throttling, analytics.

## Key Features
```
APIM CAPABILITIES:
┌─────────────────────────────────────────────────────────────────┐
│ API GATEWAY:       Single entry point for APIs                 │
│ DEVELOPER PORTAL:  API documentation and testing               │
│ POLICIES:          Transform requests/responses                │
│ PRODUCTS:          Package APIs for consumers                  │
│ SUBSCRIPTIONS:     API keys for access                         │
│ ANALYTICS:         Usage and health metrics                    │
└─────────────────────────────────────────────────────────────────┘
```

## CLI Commands

```bash
# Create APIM instance
az apim create \
  --resource-group myRG \
  --name myapim \
  --publisher-name "My Company" \
  --publisher-email "api@mycompany.com" \
  --sku-name Developer

# Import API from OpenAPI
az apim api import \
  --resource-group myRG \
  --service-name myapim \
  --path myapi \
  --specification-format OpenApi \
  --specification-path ./openapi.json
```

---

# 5. AZURE LOGIC APPS

## What It Is
Workflow automation service. Connect apps and data with little to no code using visual designer.

## Use Cases
- Integration workflows
- B2B integrations
- Data processing
- Scheduled tasks
- Event-driven automation

## CLI Commands

```bash
# Create Logic App
az logic workflow create \
  --resource-group myRG \
  --name mylogicapp \
  --location eastus \
  --definition ./workflow.json
```

---

# MESSAGING COMPARISON

| Feature | Service Bus | Event Hubs | Event Grid |
|---------|-------------|------------|------------|
| Pattern | Queue/PubSub | Streaming | Reactive |
| Ordering | FIFO | Per partition | No |
| Max Size | 100 MB | 1 MB | 1 MB |
| Retention | 14 days | 7 days | 24 hours |
| Throughput | Moderate | Very High | High |
| Best For | Enterprise | Big Data | Events |
