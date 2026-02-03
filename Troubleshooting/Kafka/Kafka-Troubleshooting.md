# Kafka Troubleshooting - Complete Guide
## Broker, Producer, Consumer, Performance Issues

---

# SECTION 1: BROKER ISSUES

---

## 1.1 BROKER WON'T START

### LEVEL 1 - DIRECT: Port Already in Use
```
Error: Address already in use
```

**Cause is VISIBLE**: Another process using Kafka port.

**Solution**:
```bash
# Find process on port
lsof -i :9092
netstat -tlnp | grep 9092

# Kill process or change Kafka port
# server.properties:
listeners=PLAINTEXT://:9093
```

---

### LEVEL 1 - DIRECT: ZooKeeper Connection Failed
```
Error: Unable to connect to ZooKeeper
```

**Solution**:
```bash
# Check ZooKeeper is running
echo "ruok" | nc localhost 2181

# Check connection string in server.properties
zookeeper.connect=zk1:2181,zk2:2181,zk3:2181
```

---

### LEVEL 2 - INTERMEDIATE: Broker Disk Full
```
Error: No space left on device
```

**Investigation**:
```bash
df -h /var/kafka-logs

# Check largest topics
du -sh /var/kafka-logs/* | sort -rh | head -10
```

**Solution**:
```bash
# Delete old segments (if retention allows)
# Or increase disk
# Or reduce retention:
kafka-configs.sh --bootstrap-server localhost:9092 \
  --alter --entity-type topics --entity-name my-topic \
  --add-config retention.ms=86400000  # 1 day
```

---

### LEVEL 3 - COMPLEX: Broker Slow / Unresponsive

**Scenario**: Broker running but very slow.

**Hidden Causes**:
- Disk I/O saturation
- Network congestion
- Too many partitions
- GC pauses
- Thread pool exhaustion

**Investigation**:
```bash
# Check I/O wait
iostat -x 1

# Check network
sar -n DEV 1

# Check GC
jstat -gc <kafka-pid> 1000

# Check request handler threads
kafka-metrics.sh --jmx-url service:jmx:rmi:///jndi/rmi://localhost:9999/jmxrmi \
  --metrics kafka.network:type=SocketServer,name=NetworkProcessorAvgIdlePercent

# Check partition count
kafka-topics.sh --bootstrap-server localhost:9092 --describe | wc -l
```

**Solutions**:
```bash
# Increase threads
# server.properties:
num.network.threads=8
num.io.threads=16

# Tune GC
# kafka-server-start.sh:
export KAFKA_HEAP_OPTS="-Xmx6g -Xms6g"
export KAFKA_JVM_PERFORMANCE_OPTS="-XX:+UseG1GC"

# Reduce partitions per broker (rebalance)
kafka-reassign-partitions.sh --bootstrap-server localhost:9092 \
  --reassignment-json-file reassign.json --execute
```

---

## 1.2 UNDER-REPLICATED PARTITIONS

### LEVEL 1 - DIRECT: Broker Down
```
Under-replicated partitions: 100
```

**Investigation**:
```bash
# Check which broker is affected
kafka-topics.sh --bootstrap-server localhost:9092 \
  --describe --under-replicated-partitions

# Check broker status
kafka-broker-api-versions.sh --bootstrap-server broker0:9092
```

**Solution**: Restart the failed broker.

---

### LEVEL 2 - INTERMEDIATE: Slow Follower

**Scenario**: All brokers running but replicas behind.

**Investigation**:
```bash
# Check ISR shrink rate
kafka-metrics.sh --metrics kafka.server:type=ReplicaManager,name=IsrShrinksPerSec

# Check replica lag
kafka-replica-verification.sh --broker-list localhost:9092 --topic-white-list ".*"
```

**Common Causes**:
- Slow disk on follower
- Network issues between brokers
- Follower CPU overloaded

**Solution**:
```bash
# Increase replica lag time (temporary)
kafka-configs.sh --bootstrap-server localhost:9092 \
  --alter --entity-type brokers --entity-default \
  --add-config replica.lag.time.max.ms=30000
```

---

### LEVEL 3 - COMPLEX: Leadership Imbalance

**Scenario**: One broker has most leaders, causing overload.

**Investigation**:
```bash
# Check leader distribution
kafka-topics.sh --bootstrap-server localhost:9092 --describe | \
  awk '/Leader:/ {print $4}' | sort | uniq -c | sort -rn
```

**Solution**:
```bash
# Trigger preferred leader election
kafka-leader-election.sh --bootstrap-server localhost:9092 \
  --election-type preferred --all-topic-partitions
```

---

# SECTION 2: PRODUCER ISSUES

---

## 2.1 PRODUCER CAN'T SEND

### LEVEL 1 - DIRECT: Connection Refused
```
Error: Connection to node -1 could not be established
```

**Solution**:
```bash
# Check bootstrap servers
telnet broker:9092

# Check advertised.listeners in broker config
# Must be reachable by producer
advertised.listeners=PLAINTEXT://broker.example.com:9092
```

---

### LEVEL 1 - DIRECT: Topic Not Found
```
Error: Unknown topic or partition
```

**Solution**:
```bash
# Check if topic exists
kafka-topics.sh --bootstrap-server localhost:9092 --list | grep my-topic

# Create topic
kafka-topics.sh --bootstrap-server localhost:9092 \
  --create --topic my-topic --partitions 6 --replication-factor 3
```

---

### LEVEL 2 - INTERMEDIATE: Not Enough Replicas
```
Error: NOT_ENOUGH_REPLICAS
```

**Cause**: acks=all but min.insync.replicas not satisfied.

**Investigation**:
```bash
# Check ISR for topic
kafka-topics.sh --bootstrap-server localhost:9092 \
  --describe --topic my-topic

# If ISR < min.insync.replicas, writes fail
```

**Solution**:
```bash
# Fix unhealthy brokers to restore ISR
# Or temporarily reduce min.insync.replicas (less safe)
kafka-configs.sh --bootstrap-server localhost:9092 \
  --alter --entity-type topics --entity-name my-topic \
  --add-config min.insync.replicas=1
```

---

### LEVEL 3 - COMPLEX: Producer Slow / Timeout

**Scenario**: Send takes seconds or times out.

**Hidden Causes**:
- Buffer full (buffer.memory exhausted)
- Batch waiting too long (linger.ms)
- Broker slow to respond
- Network latency

**Investigation**:
```bash
# Check producer metrics
# buffer-available-bytes → if low, buffer exhausted
# record-queue-time-avg → time in buffer
# request-latency-avg → broker response time

# In producer logs:
# "batch expired" → batch.size or linger.ms issue
# "buffer exhausted" → buffer.memory too small
```

**Solution**:
```properties
# Increase buffer
buffer.memory=67108864  # 64 MB

# Reduce batching (if latency critical)
linger.ms=0
batch.size=16384

# Increase timeout
request.timeout.ms=60000
delivery.timeout.ms=120000
```

---

# SECTION 3: CONSUMER ISSUES

---

## 3.1 CONSUMER NOT RECEIVING MESSAGES

### LEVEL 1 - DIRECT: Wrong Group Offset
```
Consumer starts but no messages
```

**Investigation**:
```bash
# Check consumer group offset
kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
  --describe --group my-group

# If CURRENT-OFFSET = LOG-END-OFFSET → all caught up
# Check auto.offset.reset setting
```

**Solution**:
```bash
# Reset to beginning
kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
  --group my-group --topic my-topic \
  --reset-offsets --to-earliest --execute
```

---

### LEVEL 1 - DIRECT: Consumer Group Conflict
```
Error: Consumer with same group.id already exists
```

**Cause**: Another consumer with same client.id in same group.

**Solution**:
```properties
# Use unique client.id
client.id=consumer-1-instance-a
```

---

### LEVEL 2 - INTERMEDIATE: Consumer Lag Growing

**Scenario**: Lag continuously increasing.

**Investigation**:
```bash
# Monitor lag
kafka-consumer-groups.sh --bootstrap-server localhost:9092 \
  --describe --group my-group

# Check consumer throughput
# records-consumed-rate metric

# Check processing time per message
```

**Solutions**:
```bash
# 1. Add more consumers (up to partition count)

# 2. Increase partitions (requires topic recreation or careful planning)
kafka-topics.sh --bootstrap-server localhost:9092 \
  --alter --topic my-topic --partitions 12

# 3. Optimize processing
# - Use async processing
# - Batch database writes
# - Reduce max.poll.records if processing slow
```

---

### LEVEL 3 - COMPLEX: Constant Rebalancing

**Scenario**: Consumers continuously leaving and rejoining.

**Hidden Causes**:
- Processing exceeds max.poll.interval.ms
- GC pauses exceed session.timeout.ms
- Network issues causing heartbeat failures
- Consumer crash/restart loop

**Investigation**:
```bash
# Check consumer logs for:
# "member failed to heartbeat"
# "consumer poll timeout expired"
# "revoking partitions"

# Check GC logs
jstat -gcutil <consumer-pid> 1000

# Check network
ping -c 100 broker
```

**Solutions**:
```properties
# Increase poll interval
max.poll.interval.ms=600000  # 10 minutes

# Reduce records per poll
max.poll.records=100

# Increase session timeout
session.timeout.ms=30000
heartbeat.interval.ms=10000

# Use static membership (prevents rebalance on restart)
group.instance.id=consumer-1

# Use cooperative rebalancing
partition.assignment.strategy=org.apache.kafka.clients.consumer.CooperativeStickyAssignor
```

---

## 3.2 DUPLICATE PROCESSING

### LEVEL 2 - INTERMEDIATE: Messages Processed Twice

**Scenario**: Same message processed multiple times.

**Causes**:
- Auto-commit before processing complete
- Consumer crash after process but before commit
- Rebalance during processing

**Solution**:
```java
// Manual commit after processing
props.put("enable.auto.commit", "false");

while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord<String, String> record : records) {
        process(record);  // Process first
    }
    consumer.commitSync();  // Then commit
}

// Better: Make processing idempotent
// Store processed message IDs in database
// Check before processing
```

---

# SECTION 4: DATA ISSUES

---

## 4.1 MESSAGES LOST

### LEVEL 2 - INTERMEDIATE: Messages Disappear

**Investigation**:
```bash
# Check retention settings
kafka-configs.sh --bootstrap-server localhost:9092 \
  --describe --entity-type topics --entity-name my-topic

# retention.ms or retention.bytes might be too low

# Check for log compaction
# If cleanup.policy=compact, old versions deleted
```

**Solution**:
```bash
# Increase retention
kafka-configs.sh --bootstrap-server localhost:9092 \
  --alter --entity-type topics --entity-name my-topic \
  --add-config retention.ms=604800000  # 7 days
```

---

### LEVEL 3 - COMPLEX: Data Loss During Broker Failure

**Scenario**: Messages acknowledged but lost.

**Hidden Causes**:
- acks=1 and leader failed before replicating
- unclean.leader.election.enable=true (out-of-sync replica became leader)

**Prevention**:
```properties
# Producer:
acks=all
enable.idempotence=true

# Broker:
unclean.leader.election.enable=false
min.insync.replicas=2
default.replication.factor=3
```

---

# SECTION 5: PERFORMANCE ISSUES

---

## 5.1 HIGH LATENCY

### LEVEL 2 - INTERMEDIATE: End-to-End Latency High

**Investigation**:
```bash
# Check each component:

# 1. Producer latency
# record-send-rate, request-latency-avg metrics

# 2. Broker latency
# Log flush time, disk I/O

# 3. Consumer latency
# fetch-latency-avg, processing time
```

**Solutions**:
```properties
# Producer - reduce batching for low latency
linger.ms=0
batch.size=0

# Consumer - fetch immediately
fetch.min.bytes=1
fetch.max.wait.ms=0

# Broker - faster disk, more memory
```

---

## 5.2 LOW THROUGHPUT

### LEVEL 2 - INTERMEDIATE: Can't Achieve Target Throughput

**Investigation**:
```bash
# Check partition count
kafka-topics.sh --describe --topic my-topic | grep PartitionCount

# Check consumer count vs partitions
kafka-consumer-groups.sh --describe --group my-group
```

**Solutions**:
```properties
# Producer - batch more
batch.size=65536
linger.ms=20
compression.type=lz4

# Consumer - fetch more
fetch.min.bytes=100000
max.poll.records=1000

# Add partitions for parallelism
```

---

# QUICK REFERENCE

```
COMMON ERRORS → SOLUTIONS:

PRODUCER:
┌────────────────────────────────────────────────────────────────┐
│ Connection refused      → Check bootstrap.servers, firewall   │
│ Topic not found         → Create topic or enable auto-create  │
│ NOT_ENOUGH_REPLICAS     → Fix broker issues, check ISR        │
│ Buffer exhausted        → Increase buffer.memory              │
│ Timeout                 → Increase timeouts, check broker     │
└────────────────────────────────────────────────────────────────┘

CONSUMER:
┌────────────────────────────────────────────────────────────────┐
│ No messages             → Check offset, reset if needed       │
│ Lag growing             → Add consumers, optimize processing  │
│ Constant rebalance      → Increase timeouts, reduce poll      │
│ Duplicates              → Manual commit, idempotent consumer  │
└────────────────────────────────────────────────────────────────┘

BROKER:
┌────────────────────────────────────────────────────────────────┐
│ Under-replicated        → Check broker health, network        │
│ Disk full               → Reduce retention, add disk          │
│ High CPU/memory         → Tune threads, add brokers           │
│ Won't start             → Check logs, ZK connection, port     │
└────────────────────────────────────────────────────────────────┘
```

---

# DIAGNOSTIC COMMANDS

```bash
# Cluster health
kafka-metadata.sh --snapshot /var/kafka-logs/__cluster_metadata-0/00000000000000000000.log --command "describe"

# Consumer group lag
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group <group>

# Topic details
kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic <topic>

# Under-replicated partitions
kafka-topics.sh --bootstrap-server localhost:9092 --describe --under-replicated-partitions

# Offline partitions
kafka-topics.sh --bootstrap-server localhost:9092 --describe --unavailable-partitions

# Broker configs
kafka-configs.sh --bootstrap-server localhost:9092 --describe --entity-type brokers --all

# Reset consumer offset
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group <group> --topic <topic> --reset-offsets --to-earliest --execute
```
