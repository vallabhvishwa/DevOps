# Apache Kafka - Complete Deep Dive Guide
## Architecture, Operations, Streaming, Troubleshooting

---

# 1. KAFKA FUNDAMENTALS

## What Is Kafka?

```
KAFKA = Distributed event streaming platform

CORE CAPABILITIES:
┌─────────────────────────────────────────────────────────────────┐
│ PUBLISH/SUBSCRIBE:  Producers write, consumers read            │
│ STORAGE:            Messages stored durably on disk            │
│ PROCESSING:         Stream processing with Kafka Streams       │
│ CONNECT:            Integration with external systems          │
└─────────────────────────────────────────────────────────────────┘

USE CASES:
- Event sourcing
- Log aggregation
- Stream processing
- Messaging
- Activity tracking
- Metrics collection
- Commit log
```

## Architecture Deep Dive

```
KAFKA CLUSTER ARCHITECTURE:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│     PRODUCERS                    KAFKA CLUSTER                  │
│   ┌──────────┐                                                  │
│   │Producer 1│─────┐         ┌─────────────────────────────┐   │
│   └──────────┘     │         │     BROKER 0                │   │
│   ┌──────────┐     │         │  ┌─────────────────────┐    │   │
│   │Producer 2│─────┼────────►│  │ Topic: orders       │    │   │
│   └──────────┘     │         │  │ Partition 0 (L)     │    │   │
│   ┌──────────┐     │         │  │ Partition 3 (F)     │    │   │
│   │Producer N│─────┘         │  └─────────────────────┘    │   │
│   └──────────┘               └─────────────────────────────┘   │
│                                                                 │
│                              ┌─────────────────────────────┐   │
│                              │     BROKER 1                │   │
│                              │  ┌─────────────────────┐    │   │
│                              │  │ Topic: orders       │    │   │
│                              │  │ Partition 1 (L)     │    │   │
│                              │  │ Partition 0 (F)     │    │   │
│                              │  └─────────────────────┘    │   │
│                              └─────────────────────────────┘   │
│                                                                 │
│                              ┌─────────────────────────────┐   │
│     CONSUMERS                │     BROKER 2                │   │
│   ┌──────────┐               │  ┌─────────────────────┐    │   │
│   │Consumer 1│◄──────────────│  │ Topic: orders       │    │   │
│   └──────────┘               │  │ Partition 2 (L)     │    │   │
│   ┌──────────┐               │  │ Partition 1 (F)     │    │   │
│   │Consumer 2│◄──────────────│  └─────────────────────┘    │   │
│   └──────────┘               └─────────────────────────────┘   │
│                                                                 │
│                              ┌─────────────────────────────┐   │
│                              │     ZOOKEEPER / KRAFT       │   │
│                              │  - Cluster metadata          │   │
│                              │  - Leader election           │   │
│                              │  - Configuration             │   │
│                              └─────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Core Concepts

### Topics and Partitions
```
TOPIC = Category/stream of records
PARTITION = Ordered, immutable sequence within topic

PARTITION STRUCTURE:
┌─────────────────────────────────────────────────────────────────┐
│ Partition 0:                                                    │
│ ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐        │
│ │  0  │  1  │  2  │  3  │  4  │  5  │  6  │  7  │  8  │ ───►   │
│ └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘        │
│   ▲                                               ▲             │
│   │                                               │             │
│   Oldest                                        Newest          │
│   (Offset 0)                                   (Offset 8)       │
│                                                                 │
│ KEY PROPERTIES:                                                 │
│ ├── Messages ordered WITHIN partition only                     │
│ ├── Each message has unique offset                             │
│ ├── Messages immutable once written                            │
│ ├── Retention based on time or size                            │
│ └── Partitions distributed across brokers                      │
└─────────────────────────────────────────────────────────────────┘

WHY PARTITIONS?
┌─────────────────────────────────────────────────────────────────┐
│ PARALLELISM:    More partitions = more consumers in parallel   │
│ THROUGHPUT:     Each partition on different broker             │
│ ORDERING:       Guaranteed within partition                    │
│ SCALABILITY:    Add partitions to scale (but can't decrease)   │
└─────────────────────────────────────────────────────────────────┘
```

### Partition Assignment
```
HOW MESSAGES ARE ASSIGNED TO PARTITIONS:

1. KEY-BASED (Recommended):
   partition = hash(key) % num_partitions
   
   Same key → Same partition → Ordered processing
   
   Example:
   Key: "user-123" → Always goes to Partition 2
   Key: "user-456" → Always goes to Partition 5

2. ROUND-ROBIN (No Key):
   Messages distributed evenly across partitions
   No ordering guarantee across messages

3. CUSTOM PARTITIONER:
   Implement your own logic
   
   Example: Route by region
   "US-*" → Partitions 0-3
   "EU-*" → Partitions 4-7
```

### Replication
```
REPLICATION FOR FAULT TOLERANCE:
┌─────────────────────────────────────────────────────────────────┐
│ Topic: orders, Partition: 0, Replication Factor: 3             │
│                                                                 │
│   Broker 0          Broker 1          Broker 2                 │
│ ┌───────────┐     ┌───────────┐     ┌───────────┐             │
│ │ Partition │     │ Partition │     │ Partition │             │
│ │    0      │     │    0      │     │    0      │             │
│ │  LEADER   │────►│ FOLLOWER  │     │ FOLLOWER  │             │
│ └───────────┘     └───────────┘     └───────────┘             │
│      │                                    ▲                    │
│      └────────────────────────────────────┘                    │
│                                                                 │
│ LEADER:                                                         │
│ ├── Handles all reads and writes                               │
│ ├── Replicates to followers                                    │
│                                                                 │
│ FOLLOWER:                                                       │
│ ├── Fetches data from leader                                   │
│ ├── Takes over if leader fails                                 │
│                                                                 │
│ ISR (In-Sync Replicas):                                        │
│ ├── Followers caught up with leader                            │
│ ├── Only ISR members can become leader                         │
│ ├── min.insync.replicas = minimum ISR for writes               │
└─────────────────────────────────────────────────────────────────┘
```

---

# 2. PRODUCERS

## Producer Architecture

```
PRODUCER INTERNALS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   Application                                                   │
│       │                                                         │
│       ▼                                                         │
│   ┌───────────────┐                                            │
│   │   Serializer  │  Convert to bytes                          │
│   └───────┬───────┘                                            │
│           ▼                                                     │
│   ┌───────────────┐                                            │
│   │  Partitioner  │  Determine target partition                │
│   └───────┬───────┘                                            │
│           ▼                                                     │
│   ┌───────────────────────────────────────────────────────┐    │
│   │              Record Accumulator (Buffer)              │    │
│   │  ┌─────────┐  ┌─────────┐  ┌─────────┐              │    │
│   │  │ Batch 0 │  │ Batch 1 │  │ Batch 2 │  ...         │    │
│   │  │ (Part0) │  │ (Part1) │  │ (Part2) │              │    │
│   │  └─────────┘  └─────────┘  └─────────┘              │    │
│   └───────────────────────┬───────────────────────────────┘    │
│                           ▼                                     │
│   ┌───────────────────────────────────────────────────────┐    │
│   │                   Sender Thread                        │    │
│   │    Sends batches when: batch.size reached              │    │
│   │                   OR: linger.ms elapsed                │    │
│   └───────────────────────┬───────────────────────────────┘    │
│                           ▼                                     │
│                    Kafka Brokers                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Producer Configuration

```properties
# Essential Producer Configs

# Broker list
bootstrap.servers=broker1:9092,broker2:9092,broker3:9092

# Serializers
key.serializer=org.apache.kafka.common.serialization.StringSerializer
value.serializer=org.apache.kafka.common.serialization.StringSerializer

# Acknowledgment (Durability)
# acks=0  : No ack (fastest, data loss possible)
# acks=1  : Leader ack (balanced)
# acks=all: All ISR ack (safest, slowest)
acks=all

# Retries
retries=3
retry.backoff.ms=100

# Batching (Throughput)
batch.size=16384          # 16 KB per batch
linger.ms=5               # Wait up to 5ms to fill batch
buffer.memory=33554432    # 32 MB total buffer

# Compression
compression.type=snappy   # none, gzip, snappy, lz4, zstd

# Idempotence (Exactly-once)
enable.idempotence=true   # Prevents duplicates on retry

# Ordering
max.in.flight.requests.per.connection=5  # With idempotence, safe for ordering
```

## Producer Code Example

```java
// Java Producer Example
Properties props = new Properties();
props.put("bootstrap.servers", "broker1:9092,broker2:9092");
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("acks", "all");
props.put("enable.idempotence", "true");

Producer<String, String> producer = new KafkaProducer<>(props);

// Send with key (ordering guaranteed for same key)
ProducerRecord<String, String> record = new ProducerRecord<>(
    "orders",           // topic
    "user-123",         // key
    "{\"orderId\": 1}"  // value
);

// Async send with callback
producer.send(record, (metadata, exception) -> {
    if (exception != null) {
        exception.printStackTrace();
    } else {
        System.out.printf("Sent to partition %d, offset %d%n",
            metadata.partition(), metadata.offset());
    }
});

producer.close();
```

---

# 3. CONSUMERS

## Consumer Groups

```
CONSUMER GROUP = Set of consumers sharing topic workload

HOW IT WORKS:
┌─────────────────────────────────────────────────────────────────┐
│ Topic: orders (6 partitions)                                   │
│                                                                 │
│ Consumer Group: order-processor                                 │
│                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │ Consumer 1  │  │ Consumer 2  │  │ Consumer 3  │            │
│  │ P0, P1      │  │ P2, P3      │  │ P4, P5      │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
│                                                                 │
│ RULES:                                                          │
│ ├── Each partition assigned to ONE consumer in group           │
│ ├── One consumer can have multiple partitions                  │
│ ├── Consumers > Partitions = idle consumers                    │
│ ├── Adding consumers triggers rebalance                        │
│ └── Different groups get ALL messages independently            │
└─────────────────────────────────────────────────────────────────┘

MULTIPLE CONSUMER GROUPS (Independent):
┌─────────────────────────────────────────────────────────────────┐
│ Topic: orders                                                   │
│                                                                 │
│ Group: order-processor     Group: analytics      Group: audit  │
│ ┌──────────────────┐      ┌───────────────┐    ┌────────────┐ │
│ │ C1: P0,P1        │      │ C1: P0,P1,P2  │    │ C1: ALL    │ │
│ │ C2: P2,P3        │      │ C2: P3,P4,P5  │    │            │ │
│ │ C3: P4,P5        │      │               │    │            │ │
│ └──────────────────┘      └───────────────┘    └────────────┘ │
│                                                                 │
│ Each group reads ALL messages independently                    │
└─────────────────────────────────────────────────────────────────┘
```

## Rebalancing

```
REBALANCE = Redistribution of partitions among consumers

TRIGGERS:
┌─────────────────────────────────────────────────────────────────┐
│ • Consumer joins group                                         │
│ • Consumer leaves group (crash or graceful shutdown)           │
│ • Consumer heartbeat timeout (session.timeout.ms)              │
│ • Consumer poll timeout (max.poll.interval.ms)                 │
│ • Partitions added to topic                                    │
│ • Consumer subscribes to new topic matching pattern            │
└─────────────────────────────────────────────────────────────────┘

REBALANCE STRATEGIES:

1. EAGER (Default - older versions):
   - All consumers stop
   - All partitions revoked
   - Partitions reassigned
   - Stop-the-world pause!

2. COOPERATIVE (Recommended):
   - Incremental rebalancing
   - Only affected partitions revoked
   - Minimal disruption
   
   partition.assignment.strategy=
     org.apache.kafka.clients.consumer.CooperativeStickyAssignor

REBALANCE TIMELINE:
┌─────────────────────────────────────────────────────────────────┐
│ Consumer C3 dies                                                │
│        │                                                        │
│        ▼                                                        │
│ Wait session.timeout.ms (10s default)                          │
│        │                                                        │
│        ▼                                                        │
│ Coordinator detects failure                                     │
│        │                                                        │
│        ▼                                                        │
│ Trigger rebalance                                               │
│        │                                                        │
│        ▼                                                        │
│ C1, C2 get new partition assignments                           │
│        │                                                        │
│        ▼                                                        │
│ Resume processing                                               │
└─────────────────────────────────────────────────────────────────┘
```

## Consumer Configuration

```properties
# Essential Consumer Configs

bootstrap.servers=broker1:9092,broker2:9092,broker3:9092
group.id=my-consumer-group

# Deserializers
key.deserializer=org.apache.kafka.common.serialization.StringDeserializer
value.deserializer=org.apache.kafka.common.serialization.StringDeserializer

# Offset management
auto.offset.reset=earliest    # earliest, latest, none
enable.auto.commit=true       # Auto-commit offsets
auto.commit.interval.ms=5000  # Commit every 5 seconds

# Session management
session.timeout.ms=10000      # Heartbeat timeout
heartbeat.interval.ms=3000    # Heartbeat frequency
max.poll.interval.ms=300000   # Max time between polls

# Fetching
fetch.min.bytes=1             # Min data per fetch
fetch.max.wait.ms=500         # Max wait for min bytes
max.poll.records=500          # Max records per poll

# Rebalance strategy
partition.assignment.strategy=org.apache.kafka.clients.consumer.CooperativeStickyAssignor
```

## Offset Management

```
OFFSET COMMIT STRATEGIES:
┌─────────────────────────────────────────────────────────────────┐
│ AUTO-COMMIT (enable.auto.commit=true):                         │
│ ├── Simplest approach                                          │
│ ├── Commits periodically (auto.commit.interval.ms)             │
│ ├── Risk: Process fails after commit but before processing     │
│ └── Result: Message lost                                       │
│                                                                 │
│ MANUAL COMMIT (enable.auto.commit=false):                      │
│                                                                 │
│ commitSync():                                                   │
│ ├── Blocks until commit succeeds                               │
│ ├── Retries on failure                                         │
│ └── Safest but slower                                          │
│                                                                 │
│ commitAsync():                                                  │
│ ├── Non-blocking                                               │
│ ├── No retry (offset might be stale)                           │
│ └── Faster but less safe                                       │
│                                                                 │
│ BEST PRACTICE:                                                  │
│ ├── Use commitAsync() during processing                        │
│ └── Use commitSync() on shutdown                               │
└─────────────────────────────────────────────────────────────────┘
```

---

# 4. EXACTLY-ONCE SEMANTICS

## Delivery Guarantees

```
MESSAGE DELIVERY SEMANTICS:
┌─────────────────────────────────────────────────────────────────┐
│ AT-MOST-ONCE:                                                   │
│ ├── Message may be lost                                        │
│ ├── Never delivered twice                                      │
│ └── acks=0, auto-commit before processing                      │
│                                                                 │
│ AT-LEAST-ONCE:                                                  │
│ ├── Message never lost                                         │
│ ├── May be delivered multiple times                            │
│ └── acks=all, commit after processing, idempotent consumer     │
│                                                                 │
│ EXACTLY-ONCE:                                                   │
│ ├── Message delivered exactly once                             │
│ ├── No loss, no duplicates                                     │
│ └── Requires idempotent producer + transactions                │
└─────────────────────────────────────────────────────────────────┘
```

## Idempotent Producer

```
IDEMPOTENT PRODUCER:
┌─────────────────────────────────────────────────────────────────┐
│ enable.idempotence=true                                        │
│                                                                 │
│ HOW IT WORKS:                                                   │
│                                                                 │
│ Producer assigns:                                               │
│ ├── Producer ID (PID): Unique per producer session             │
│ └── Sequence Number: Per partition, incrementing               │
│                                                                 │
│ Broker:                                                         │
│ ├── Tracks last 5 sequence numbers per PID/partition           │
│ ├── Rejects duplicates (same PID + sequence)                   │
│ └── Detects out-of-order (gaps in sequence)                    │
│                                                                 │
│ SCENARIO:                                                       │
│ 1. Producer sends message (PID=1, seq=0)                       │
│ 2. Broker receives, writes, sends ack                          │
│ 3. Ack lost in network                                         │
│ 4. Producer retries (PID=1, seq=0)                             │
│ 5. Broker detects duplicate, returns success without writing   │
│                                                                 │
│ RESULT: Exactly once write to Kafka                            │
└─────────────────────────────────────────────────────────────────┘
```

## Transactions

```java
// Transactional Producer (Read-Process-Write)
Properties props = new Properties();
props.put("bootstrap.servers", "broker:9092");
props.put("transactional.id", "order-processor-1"); // Required
props.put("enable.idempotence", "true");

Producer<String, String> producer = new KafkaProducer<>(props);
producer.initTransactions();

while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    
    producer.beginTransaction();
    try {
        for (ConsumerRecord<String, String> record : records) {
            // Process and produce to output topic
            producer.send(new ProducerRecord<>("output-topic", 
                record.key(), 
                process(record.value())));
        }
        
        // Commit consumer offsets as part of transaction
        producer.sendOffsetsToTransaction(
            getOffsetsToCommit(records),
            consumer.groupMetadata());
        
        producer.commitTransaction();
    } catch (Exception e) {
        producer.abortTransaction();
    }
}
```

---

# 5. SCHEMA REGISTRY

## What Is Schema Registry?

```
SCHEMA REGISTRY = Central schema repository

WHY NEEDED:
┌─────────────────────────────────────────────────────────────────┐
│ WITHOUT SCHEMA REGISTRY:                                       │
│ ├── Producers/consumers must agree on format                   │
│ ├── Schema changes break consumers                             │
│ └── No validation of data format                               │
│                                                                 │
│ WITH SCHEMA REGISTRY:                                          │
│ ├── Central schema storage                                     │
│ ├── Schema versioning                                          │
│ ├── Compatibility checking                                     │
│ └── Automatic serialization/deserialization                    │
└─────────────────────────────────────────────────────────────────┘

ARCHITECTURE:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   Producer                    Schema Registry                   │
│ ┌──────────┐                 ┌──────────────┐                  │
│ │ Avro     │───Register─────►│ Schema Store │                  │
│ │ Serializer│◄──Schema ID────│              │                  │
│ └──────────┘                 └──────────────┘                  │
│      │                              ▲                          │
│      │                              │                          │
│      ▼                              │                          │
│ ┌──────────┐                        │                          │
│ │  Kafka   │                        │                          │
│ │  Broker  │                        │                          │
│ └──────────┘                        │                          │
│      │                              │                          │
│      ▼                              │                          │
│ ┌──────────┐                 ┌──────────────┐                  │
│ │ Avro     │───Get Schema───►│ Schema Store │                  │
│ │ Deserial │◄──Schema────────│              │                  │
│ └──────────┘                 └──────────────┘                  │
│   Consumer                                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Compatibility Modes

```
COMPATIBILITY TYPES:
┌─────────────────────────────────────────────────────────────────┐
│ BACKWARD (Default):                                            │
│ ├── New schema can read OLD data                               │
│ ├── Safe to upgrade consumers first                            │
│ └── Can: Add optional fields, remove fields                   │
│                                                                 │
│ FORWARD:                                                        │
│ ├── OLD schema can read new data                               │
│ ├── Safe to upgrade producers first                            │
│ └── Can: Remove optional fields, add fields                   │
│                                                                 │
│ FULL:                                                           │
│ ├── Both backward and forward compatible                       │
│ ├── Safest, most restrictive                                   │
│ └── Can: Add/remove optional fields only                      │
│                                                                 │
│ NONE:                                                           │
│ ├── No compatibility checking                                  │
│ └── Not recommended for production                             │
└─────────────────────────────────────────────────────────────────┘
```

## Avro Schema Example

```json
// User schema - version 1
{
  "type": "record",
  "name": "User",
  "namespace": "com.example",
  "fields": [
    {"name": "id", "type": "long"},
    {"name": "name", "type": "string"},
    {"name": "email", "type": "string"}
  ]
}

// User schema - version 2 (backward compatible)
{
  "type": "record",
  "name": "User",
  "namespace": "com.example",
  "fields": [
    {"name": "id", "type": "long"},
    {"name": "name", "type": "string"},
    {"name": "email", "type": "string"},
    {"name": "phone", "type": ["null", "string"], "default": null}  // New optional field
  ]
}
```

---

# 6. KAFKA STREAMS

## What Is Kafka Streams?

```
KAFKA STREAMS = Client library for stream processing

FEATURES:
┌─────────────────────────────────────────────────────────────────┐
│ • No separate cluster (just a library)                         │
│ • Exactly-once processing                                       │
│ • Stateful operations (aggregations, joins)                    │
│ • Interactive queries (query state stores)                     │
│ • High availability via state store replication                │
└─────────────────────────────────────────────────────────────────┘

CORE CONCEPTS:
┌─────────────────────────────────────────────────────────────────┐
│ KStream:   Unbounded sequence of records (event stream)        │
│ KTable:    Changelog stream (latest value per key)             │
│ GlobalKTable: Like KTable but fully replicated                 │
│ State Store: Local storage for aggregations                    │
└─────────────────────────────────────────────────────────────────┘
```

## Stream Processing Example

```java
// Word count example
StreamsBuilder builder = new StreamsBuilder();

// Source stream
KStream<String, String> textLines = builder.stream("text-input");

// Processing
KTable<String, Long> wordCounts = textLines
    .flatMapValues(line -> Arrays.asList(line.toLowerCase().split("\\W+")))
    .groupBy((key, word) -> word)
    .count(Materialized.as("word-counts-store"));

// Sink
wordCounts.toStream().to("word-counts-output", 
    Produced.with(Serdes.String(), Serdes.Long()));

// Build and start
KafkaStreams streams = new KafkaStreams(builder.build(), props);
streams.start();
```

## Joins

```
STREAM-TABLE JOIN:
┌─────────────────────────────────────────────────────────────────┐
│ KStream<UserId, PageView>  +  KTable<UserId, UserProfile>      │
│                                                                 │
│ Page View Event:           User Profile:                       │
│ {userId: 123,              {userId: 123,                       │
│  page: "/products"}         name: "Alice"}                      │
│                                                                 │
│ Result:                                                         │
│ {userId: 123,                                                   │
│  page: "/products",                                             │
│  name: "Alice"}                                                 │
│                                                                 │
│ pageViews.join(userProfiles,                                   │
│     (pageView, profile) -> enrich(pageView, profile))          │
└─────────────────────────────────────────────────────────────────┘
```

---

# 7. KAFKA CONNECT

## What Is Kafka Connect?

```
KAFKA CONNECT = Framework for data integration

ARCHITECTURE:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   External Systems           Kafka Connect          Kafka       │
│                                                                 │
│   ┌──────────┐              ┌────────────┐       ┌─────────┐  │
│   │ Database │──Source─────►│  Connect   │──────►│  Topic  │  │
│   │  (MySQL) │  Connector   │  Worker    │       │         │  │
│   └──────────┘              └────────────┘       └─────────┘  │
│                                                                 │
│   ┌──────────┐              ┌────────────┐       ┌─────────┐  │
│   │   S3     │◄──Sink──────│  Connect   │◄──────│  Topic  │  │
│   │          │  Connector   │  Worker    │       │         │  │
│   └──────────┘              └────────────┘       └─────────┘  │
│                                                                 │
│ SOURCE CONNECTORS:                                              │
│ ├── JDBC, Debezium (CDC), File, S3, etc.                       │
│                                                                 │
│ SINK CONNECTORS:                                                │
│ ├── JDBC, Elasticsearch, S3, HDFS, etc.                        │
└─────────────────────────────────────────────────────────────────┘
```

## Connector Configuration

```json
// JDBC Source Connector
{
  "name": "mysql-source",
  "config": {
    "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
    "connection.url": "jdbc:mysql://localhost:3306/mydb",
    "connection.user": "user",
    "connection.password": "password",
    "table.whitelist": "users,orders",
    "mode": "incrementing",
    "incrementing.column.name": "id",
    "topic.prefix": "mysql-"
  }
}

// Elasticsearch Sink Connector
{
  "name": "elasticsearch-sink",
  "config": {
    "connector.class": "io.confluent.connect.elasticsearch.ElasticsearchSinkConnector",
    "connection.url": "http://elasticsearch:9200",
    "topics": "orders",
    "key.ignore": "true",
    "type.name": "_doc"
  }
}
```

---

# 8. OPERATIONS & MONITORING

## Key Metrics

```
CRITICAL KAFKA METRICS:
┌─────────────────────────────────────────────────────────────────┐
│ BROKER METRICS:                                                 │
│ ├── UnderReplicatedPartitions: Should be 0                     │
│ ├── IsrShrinksPerSec: ISR shrinking rate                       │
│ ├── ActiveControllerCount: Should be 1 in cluster              │
│ ├── OfflinePartitionsCount: Should be 0                        │
│ ├── RequestHandlerAvgIdlePercent: >0.3 healthy                 │
│ └── NetworkProcessorAvgIdlePercent: >0.3 healthy               │
│                                                                 │
│ PRODUCER METRICS:                                               │
│ ├── record-send-rate: Messages/second                          │
│ ├── record-error-rate: Failed sends/second                     │
│ ├── request-latency-avg: Average send latency                  │
│ └── batch-size-avg: Average batch size                         │
│                                                                 │
│ CONSUMER METRICS:                                               │
│ ├── records-lag-max: Max lag across partitions                 │
│ ├── records-consumed-rate: Consumption rate                    │
│ ├── fetch-latency-avg: Average fetch latency                   │
│ └── commit-latency-avg: Offset commit latency                  │
│                                                                 │
│ CONSUMER LAG (CRITICAL):                                        │
│ ├── Lag = Latest Offset - Consumer Offset                      │
│ ├── Growing lag = Consumer can't keep up                       │
│ └── Alert if lag > threshold                                   │
└─────────────────────────────────────────────────────────────────┘
```

## Monitoring Commands

```bash
# Check consumer lag
kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
  --describe --group my-consumer-group

# Output:
# GROUP    TOPIC    PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
# mygroup  orders   0          1000            1050            50
# mygroup  orders   1          2000            2000            0

# Check under-replicated partitions
kafka-topics.sh --bootstrap-server localhost:9092 \
  --describe --under-replicated-partitions

# Check offline partitions
kafka-topics.sh --bootstrap-server localhost:9092 \
  --describe --unavailable-partitions

# Check topic configuration
kafka-configs.sh --bootstrap-server localhost:9092 \
  --entity-type topics --entity-name my-topic --describe

# Check broker configuration
kafka-configs.sh --bootstrap-server localhost:9092 \
  --entity-type brokers --entity-name 0 --describe
```

---

# 9. PERFORMANCE TUNING

## Producer Tuning

```
PRODUCER PERFORMANCE:
┌─────────────────────────────────────────────────────────────────┐
│ FOR THROUGHPUT:                                                 │
│ ├── batch.size: 32768-65536 (larger batches)                   │
│ ├── linger.ms: 10-100 (wait to fill batch)                     │
│ ├── compression.type: lz4 or zstd                              │
│ ├── buffer.memory: Increase for high throughput                │
│ └── acks: 1 (if durability not critical)                       │
│                                                                 │
│ FOR LATENCY:                                                    │
│ ├── batch.size: 0 (no batching)                                │
│ ├── linger.ms: 0 (send immediately)                            │
│ └── acks: 1                                                    │
│                                                                 │
│ FOR DURABILITY:                                                 │
│ ├── acks: all                                                  │
│ ├── enable.idempotence: true                                   │
│ ├── retries: MAX_INT                                           │
│ └── min.insync.replicas: 2 (on topic)                          │
└─────────────────────────────────────────────────────────────────┘
```

## Consumer Tuning

```
CONSUMER PERFORMANCE:
┌─────────────────────────────────────────────────────────────────┐
│ FOR THROUGHPUT:                                                 │
│ ├── fetch.min.bytes: 10000-100000                              │
│ ├── fetch.max.wait.ms: 500                                     │
│ ├── max.poll.records: 1000-5000                                │
│ └── Increase partition count                                   │
│                                                                 │
│ FOR LATENCY:                                                    │
│ ├── fetch.min.bytes: 1                                         │
│ ├── fetch.max.wait.ms: 0                                       │
│ └── Fewer partitions per consumer                              │
│                                                                 │
│ AVOID REBALANCE:                                                │
│ ├── session.timeout.ms: 30000-60000                            │
│ ├── heartbeat.interval.ms: 10000                               │
│ ├── max.poll.interval.ms: Increase if processing slow          │
│ ├── max.poll.records: Decrease if processing slow              │
│ └── Use CooperativeStickyAssignor                              │
└─────────────────────────────────────────────────────────────────┘
```

---

# 10. TROUBLESHOOTING

## Common Issues

### Consumer Lag Growing

```
SYMPTOM: Consumer lag continuously increasing

INVESTIGATION:
┌─────────────────────────────────────────────────────────────────┐
│ 1. Check consumer group status:                                │
│    kafka-consumer-groups.sh --describe --group mygroup         │
│                                                                 │
│ 2. Check consumer throughput:                                  │
│    - records-consumed-rate metric                              │
│    - Compare with producer rate                                │
│                                                                 │
│ 3. Check for rebalancing:                                      │
│    - Consumer logs for "Revoking" / "Assigned"                 │
│    - Indicates instability                                     │
│                                                                 │
│ 4. Check processing time:                                      │
│    - If > max.poll.interval.ms → rebalance                     │
└─────────────────────────────────────────────────────────────────┘

SOLUTIONS:
┌─────────────────────────────────────────────────────────────────┐
│ • Add more consumers (up to partition count)                   │
│ • Increase partitions (requires topic recreation)              │
│ • Optimize processing logic                                    │
│ • Reduce max.poll.records                                      │
│ • Increase max.poll.interval.ms (if processing slow)           │
│ • Use async processing                                         │
└─────────────────────────────────────────────────────────────────┘
```

### Under-Replicated Partitions

```
SYMPTOM: kafka.server:type=ReplicaManager,name=UnderReplicatedPartitions > 0

INVESTIGATION:
┌─────────────────────────────────────────────────────────────────┐
│ 1. Find affected partitions:                                   │
│    kafka-topics.sh --describe --under-replicated-partitions    │
│                                                                 │
│ 2. Check broker health:                                        │
│    - Is a broker down?                                         │
│    - CPU/memory/disk issues?                                   │
│                                                                 │
│ 3. Check network:                                              │
│    - Network partition between brokers?                        │
│    - Slow network causing replica lag?                         │
│                                                                 │
│ 4. Check disk I/O:                                             │
│    - Slow disk causing replication delay                       │
└─────────────────────────────────────────────────────────────────┘

SOLUTIONS:
┌─────────────────────────────────────────────────────────────────┐
│ • Restart unhealthy broker                                     │
│ • Fix network issues                                           │
│ • Add disk capacity / faster disks                             │
│ • Reduce partition leadership on overloaded broker             │
│ • Increase replica.lag.time.max.ms (temporary)                 │
└─────────────────────────────────────────────────────────────────┘
```

### Producer Timeout

```
SYMPTOM: TimeoutException sending messages

INVESTIGATION:
┌─────────────────────────────────────────────────────────────────┐
│ 1. Check broker availability:                                  │
│    - Can producer connect to bootstrap servers?                │
│                                                                 │
│ 2. Check leader availability:                                  │
│    - Is partition leader alive?                                │
│    - kafka-topics.sh --describe --topic mytopic                │
│                                                                 │
│ 3. Check producer buffer:                                      │
│    - buffer.memory full?                                       │
│    - Producing faster than sending                             │
│                                                                 │
│ 4. Check acks setting:                                         │
│    - acks=all with min.insync.replicas=2                       │
│    - But only 1 replica in ISR?                                │
└─────────────────────────────────────────────────────────────────┘

SOLUTIONS:
┌─────────────────────────────────────────────────────────────────┐
│ • Increase request.timeout.ms                                  │
│ • Increase buffer.memory                                       │
│ • Add batch.size to increase throughput                        │
│ • Fix broker / replica issues                                  │
│ • Reduce min.insync.replicas (trade durability)                │
└─────────────────────────────────────────────────────────────────┘
```

### Consumer Rebalancing Continuously

```
SYMPTOM: Consumers constantly leaving and rejoining group

INVESTIGATION:
┌─────────────────────────────────────────────────────────────────┐
│ 1. Check max.poll.interval.ms:                                 │
│    - Processing taking too long?                               │
│                                                                 │
│ 2. Check session.timeout.ms:                                   │
│    - GC pauses causing missed heartbeats?                      │
│                                                                 │
│ 3. Check consumer logs:                                        │
│    - "Member failed to heartbeat"                              │
│    - "Rebalance triggered"                                     │
│                                                                 │
│ 4. Check for frequent deployments:                             │
│    - Rolling restarts causing rebalance storm                  │
└─────────────────────────────────────────────────────────────────┘

SOLUTIONS:
┌─────────────────────────────────────────────────────────────────┐
│ • Increase max.poll.interval.ms                                │
│ • Decrease max.poll.records                                    │
│ • Increase session.timeout.ms                                  │
│ • Use CooperativeStickyAssignor                                │
│ • Use static membership (group.instance.id)                    │
│ • Tune GC settings                                             │
└─────────────────────────────────────────────────────────────────┘
```

---

# 11. KRAFT MODE (ZooKeeper-less)

```
KRAFT = Kafka Raft Metadata Mode

WHY KRAFT:
┌─────────────────────────────────────────────────────────────────┐
│ • Removes ZooKeeper dependency                                 │
│ • Simpler architecture                                         │
│ • Faster metadata operations                                   │
│ • Better scalability                                           │
│ • Fewer moving parts                                           │
└─────────────────────────────────────────────────────────────────┘

KRAFT ARCHITECTURE:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │              Kafka Cluster (KRaft)                       │  │
│   │  ┌───────────┐  ┌───────────┐  ┌───────────┐           │  │
│   │  │ Broker 0  │  │ Broker 1  │  │ Broker 2  │           │  │
│   │  │ (Ctrl+Brk)│  │ (Ctrl+Brk)│  │ (Ctrl+Brk)│           │  │
│   │  └───────────┘  └───────────┘  └───────────┘           │  │
│   │        ▲              ▲              ▲                  │  │
│   │        └──────────────┼──────────────┘                  │  │
│   │                       │                                  │  │
│   │              Raft Consensus                              │  │
│   │          (Built-in metadata)                             │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                 │
│   Combined mode: Same nodes are controllers + brokers          │
│   Separated mode: Dedicated controller nodes                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

# 12. BEST PRACTICES

```
KAFKA BEST PRACTICES:
┌─────────────────────────────────────────────────────────────────┐
│ TOPIC DESIGN:                                                   │
│ ✓ Partition count = expected max consumer parallelism          │
│ ✓ Use keys for ordering requirements                           │
│ ✓ Plan partition count upfront (can't decrease)                │
│ ✓ Use meaningful topic names (domain.entity.action)            │
│                                                                 │
│ PRODUCER:                                                       │
│ ✓ Enable idempotence for exactly-once                          │
│ ✓ Use acks=all for durability                                  │
│ ✓ Use compression for high throughput                          │
│ ✓ Implement proper error handling and retries                  │
│                                                                 │
│ CONSUMER:                                                       │
│ ✓ Use CooperativeStickyAssignor                                │
│ ✓ Make processing idempotent                                   │
│ ✓ Monitor consumer lag                                         │
│ ✓ Tune poll settings to avoid rebalance                        │
│                                                                 │
│ OPERATIONS:                                                     │
│ ✓ Replication factor >= 3                                      │
│ ✓ min.insync.replicas = 2                                      │
│ ✓ Monitor under-replicated partitions                          │
│ ✓ Use Schema Registry for schema evolution                     │
│ ✓ Plan for data retention                                      │
│ ✓ Regular backup of configurations                             │
└─────────────────────────────────────────────────────────────────┘
```
