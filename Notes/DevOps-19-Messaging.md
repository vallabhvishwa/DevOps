# DevOps Engineer's Complete Reference Guide
# Part 19: Message Queues & Event Streaming

---

## Table of Contents

1. [Messaging Concepts](#1-messaging-concepts)
2. [Apache Kafka](#2-apache-kafka)
3. [RabbitMQ](#3-rabbitmq)
4. [Azure Service Bus](#4-azure-service-bus)
5. [Azure Event Hubs](#5-azure-event-hubs)
6. [Comparison & Selection](#6-comparison--selection)
7. [Monitoring & Operations](#7-monitoring--operations)

---

## 1. Messaging Concepts

```
Message Queue vs Event Stream:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  MESSAGE QUEUE (Point-to-Point):                                │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐                      │
│  │Producer │───►│  Queue  │───►│Consumer │                      │
│  └─────────┘    └─────────┘    └─────────┘                      │
│                                                                 │
│  • Message consumed once, then deleted                          │
│  • Work distribution pattern                                    │
│  • Examples: RabbitMQ, Azure Service Bus Queue                  │
│                                                                 │
│  EVENT STREAM (Pub/Sub + Replay):                               │
│  ┌─────────┐    ┌─────────────────┐    ┌─────────┐              │
│  │Producer │───►│  Topic/Stream   │───►│Consumer1│              │
│  └─────────┘    │  [0][1][2][3]...│───►│Consumer2│              │
│                 └─────────────────┘───►│Consumer3│              │
│                                        └─────────┘              │
│  • Events retained (configurable period)                        │
│  • Multiple consumers read same events                          │
│  • Replayable from any offset                                   │
│  • Examples: Kafka, Azure Event Hubs                            │
│                                                                 │
│  USE CASES:                                                     │
│  Queue: Task processing, RPC, work distribution                 │
│  Stream: Event sourcing, analytics, audit logs, ETL             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Apache Kafka

### 2.1 Kafka Architecture

```
Kafka Cluster:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    Kafka Cluster                         │    │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐            │    │
│  │  │ Broker 0  │  │ Broker 1  │  │ Broker 2  │            │    │
│  │  │           │  │           │  │           │            │    │
│  │  │ Topic A   │  │ Topic A   │  │ Topic A   │            │    │
│  │  │ Part 0(L) │  │ Part 1(L) │  │ Part 2(L) │            │    │
│  │  │ Part 1(F) │  │ Part 2(F) │  │ Part 0(F) │            │    │
│  │  └───────────┘  └───────────┘  └───────────┘            │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  Components:                                                    │
│  ├── Broker: Kafka server, stores messages                      │
│  ├── Topic: Category/feed of messages                           │
│  ├── Partition: Ordered, immutable sequence (parallelism)       │
│  ├── Leader (L): Handles reads/writes for partition             │
│  ├── Follower (F): Replicates leader data                       │
│  ├── Consumer Group: Set of consumers sharing workload          │
│  └── Offset: Position of message in partition                   │
│                                                                 │
│  Key Concepts:                                                  │
│  ├── Replication Factor: Number of copies (typically 3)         │
│  ├── ISR: In-Sync Replicas (caught up followers)                │
│  ├── Retention: How long to keep messages (time or size)        │
│  └── Compaction: Keep only latest value per key                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Kafka Commands

```bash
# Topic management
kafka-topics.sh --bootstrap-server localhost:9092 --list
kafka-topics.sh --bootstrap-server localhost:9092 --create \
  --topic my-topic --partitions 6 --replication-factor 3

kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic my-topic

# Produce messages
kafka-console-producer.sh --bootstrap-server localhost:9092 --topic my-topic
# Type messages, Ctrl+D to exit

# Consume messages
kafka-console-consumer.sh --bootstrap-server localhost:9092 \
  --topic my-topic --from-beginning

# Consumer groups
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
  --describe --group my-group

# Reset offsets
kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
  --group my-group --topic my-topic --reset-offsets --to-earliest --execute
```

### 2.3 Kafka in Kubernetes (Strimzi)

```yaml
# Kafka cluster with Strimzi operator
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: my-cluster
spec:
  kafka:
    version: 3.6.0
    replicas: 3
    listeners:
      - name: plain
        port: 9092
        type: internal
        tls: false
      - name: tls
        port: 9093
        type: internal
        tls: true
    config:
      offsets.topic.replication.factor: 3
      transaction.state.log.replication.factor: 3
      transaction.state.log.min.isr: 2
      default.replication.factor: 3
      min.insync.replicas: 2
    storage:
      type: persistent-claim
      size: 100Gi
      class: managed-premium
  zookeeper:
    replicas: 3
    storage:
      type: persistent-claim
      size: 10Gi
---
# Topic
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaTopic
metadata:
  name: orders
  labels:
    strimzi.io/cluster: my-cluster
spec:
  partitions: 12
  replicas: 3
  config:
    retention.ms: 604800000  # 7 days
    segment.bytes: 1073741824
```

---

## 3. RabbitMQ

### 3.1 RabbitMQ Concepts

```
RabbitMQ Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Producer ──► Exchange ──► Queue ──► Consumer                   │
│                                                                 │
│  EXCHANGE TYPES:                                                │
│  ├── Direct:  Route by exact routing key match                  │
│  ├── Fanout:  Broadcast to all bound queues                     │
│  ├── Topic:   Route by pattern (*.error, logs.#)                │
│  └── Headers: Route by message headers                          │
│                                                                 │
│  Example - Direct Exchange:                                     │
│  ┌─────────┐       ┌──────────┐       ┌─────────┐              │
│  │Producer │──────►│ Exchange │──────►│ Queue A │──►Consumer   │
│  │key=error│       │ (direct) │       │ (error) │              │
│  └─────────┘       │          │       └─────────┘              │
│                    │          │       ┌─────────┐              │
│                    │          │──────►│ Queue B │──►Consumer   │
│                    │          │       │ (info)  │              │
│                    └──────────┘       └─────────┘              │
│                                                                 │
│  Example - Fanout Exchange (Pub/Sub):                           │
│  ┌─────────┐       ┌──────────┐       ┌─────────┐              │
│  │Producer │──────►│ Exchange │──────►│ Queue A │──►Email      │
│  │         │       │ (fanout) │──────►│ Queue B │──►SMS        │
│  └─────────┘       └──────────┘──────►│ Queue C │──►Push       │
│                                       └─────────┘              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 RabbitMQ in Kubernetes

```yaml
# RabbitMQ Cluster Operator
apiVersion: rabbitmq.com/v1beta1
kind: RabbitmqCluster
metadata:
  name: rabbitmq
spec:
  replicas: 3
  resources:
    requests:
      cpu: 500m
      memory: 1Gi
    limits:
      cpu: 1
      memory: 2Gi
  persistence:
    storageClassName: managed-premium
    storage: 50Gi
  rabbitmq:
    additionalConfig: |
      cluster_partition_handling = pause_minority
      vm_memory_high_watermark.relative = 0.8
      disk_free_limit.relative = 1.5
```

---

## 4. Azure Service Bus

```
Azure Service Bus:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  QUEUES (Point-to-Point):                                       │
│  Sender ──► Queue ──► Receiver                                  │
│  • FIFO delivery                                                │
│  • At-least-once or at-most-once                                │
│  • Dead-letter queue for failed messages                        │
│                                                                 │
│  TOPICS & SUBSCRIPTIONS (Pub/Sub):                              │
│  Publisher ──► Topic ──► Subscription A ──► Subscriber          │
│                      ──► Subscription B ──► Subscriber          │
│  • Filters on subscriptions (SQL-like)                          │
│  • Each subscription gets a copy                                │
│                                                                 │
│  FEATURES:                                                      │
│  ├── Sessions: Ordered, grouped messages                        │
│  ├── Duplicate detection                                        │
│  ├── Scheduled messages                                         │
│  ├── Auto-forwarding                                            │
│  ├── Dead-lettering                                             │
│  └── Transactions                                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```bash
# Azure CLI commands
az servicebus namespace create --name mybus --resource-group myRG --sku Standard
az servicebus queue create --namespace-name mybus --name orders --resource-group myRG
az servicebus topic create --namespace-name mybus --name events --resource-group myRG
az servicebus topic subscription create --namespace-name mybus \
  --topic-name events --name email-sub --resource-group myRG
```

---

## 5. Azure Event Hubs

```
Event Hubs (Kafka-compatible):
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    Event Hub                             │    │
│  │  ┌───────────┐ ┌───────────┐ ┌───────────┐              │    │
│  │  │Partition 0│ │Partition 1│ │Partition 2│              │    │
│  │  │[0][1][2]..│ │[0][1][2]..│ │[0][1][2]..│              │    │
│  │  └───────────┘ └───────────┘ └───────────┘              │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
│  • Kafka protocol compatible (use Kafka client)                 │
│  • Millions of events/second                                    │
│  • Capture to Azure Storage/Data Lake                           │
│  • Schema Registry available                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```bash
# Create Event Hub
az eventhubs namespace create --name myeventhub --resource-group myRG --sku Standard
az eventhubs eventhub create --namespace-name myeventhub --name events \
  --resource-group myRG --partition-count 4 --message-retention 7
```

---

## 6. Comparison & Selection

```
When to Use What:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  KAFKA / EVENT HUBS:                                            │
│  ├── High throughput (millions/sec)                             │
│  ├── Event sourcing, replay needed                              │
│  ├── Stream processing (Kafka Streams, Flink)                   │
│  ├── Long retention (days/weeks)                                │
│  └── Multiple consumers same data                               │
│                                                                 │
│  RABBITMQ / SERVICE BUS QUEUES:                                 │
│  ├── Complex routing (exchanges, filters)                       │
│  ├── RPC patterns                                               │
│  ├── Message acknowledgment important                           │
│  ├── Priority queues                                            │
│  └── Transactional messaging                                    │
│                                                                 │
│  QUICK DECISION:                                                │
│  ├── Need replay? → Kafka/Event Hubs                            │
│  ├── Task queue? → RabbitMQ/Service Bus                         │
│  ├── Already on Azure? → Service Bus/Event Hubs                 │
│  └── Need Kafka ecosystem? → Kafka or Event Hubs (Kafka mode)   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. Monitoring & Operations

```
Key Metrics to Monitor:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  KAFKA:                                                         │
│  ├── Consumer lag (messages behind)                             │
│  ├── Under-replicated partitions                                │
│  ├── Request latency (produce/fetch)                            │
│  ├── Disk usage                                                 │
│  └── ISR shrink/expand rate                                     │
│                                                                 │
│  RABBITMQ:                                                      │
│  ├── Queue depth (messages ready)                               │
│  ├── Consumer count                                             │
│  ├── Message rates (publish/deliver)                            │
│  ├── Memory/disk usage                                          │
│  └── Connection count                                           │
│                                                                 │
│  ALERTS:                                                        │
│  ├── Consumer lag > threshold                                   │
│  ├── Dead letter queue growing                                  │
│  ├── Disk > 80%                                                 │
│  └── No consumers on queue                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Summary

Message queues and event streaming are critical infrastructure:

1. **Kafka**: High-throughput event streaming, replay, multiple consumers
2. **RabbitMQ**: Flexible routing, traditional message queue patterns
3. **Azure Service Bus**: Managed queues/topics with enterprise features
4. **Azure Event Hubs**: Managed Kafka-compatible streaming

Choose based on: throughput needs, replay requirements, routing complexity, and existing ecosystem.

---

**Next Part**: [Part 20: Helm Deep Dive](./DevOps-Complete-Reference-Guide-Part20-Helm.md)
