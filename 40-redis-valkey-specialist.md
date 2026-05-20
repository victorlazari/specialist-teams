# Redis and Valkey: Comprehensive Specialist Guide – Part 1

---

## Introduction to Redis and Valkey

In modern distributed systems, low-latency, high-throughput data storage solutions are critical for performance-sensitive applications. **Redis** is a leading in-memory data structure store, widely utilized for caching, real-time analytics, session management, and message brokering. It supports various data types, persistence models, and clustering options, making it a versatile choice in production environments.

**Valkey** is a specialized key-value store inspired by Redis, often optimized for specific high-performance use cases or extended with enterprise-level features. While sharing architectural similarities with Redis, Valkey introduces enhancements in scaling, persistence, or security tailored for certain deployments. Understanding both systems' internals helps tech support engineers anticipate failure cases and optimize operational workflows.

This guide focuses on Redis and Valkey’s architecture and high availability mechanisms (Sentinel and Clustering), emphasizing production-grade operations, troubleshooting, and client guidance.

---

## Core Architecture Deep Dive

### 1. Single-threaded Model

Redis operates primarily as a **single-threaded** process for command execution, leveraging an event-driven, multiplexed I/O model. This design choice simplifies concurrency control, avoiding complex locking mechanisms and ensuring predictable command execution order.

#### Why Single-threaded?

- **Deterministic command execution order:** Commands are executed sequentially, guaranteeing data consistency without race conditions.
- **Low context-switch overhead:** No need for threads to compete for CPU or locks.
- **Simplicity in debugging and profiling.**

**Technical Note:** Redis uses non-blocking I/O multiplexing (typically `epoll` on Linux, `kqueue` on BSD/macOS, or `select` on Windows) to handle multiple client connections concurrently.

### 2. Event Loop

The **event loop** is the core mechanism driving Redis. It continuously polls for:

- **Client requests**
- **Internal timers (e.g., eviction, replication timeouts)**
- **Persistence triggers (RDB/AOF writes)**
- **Background I/O (forked child processes for persistence)**

The loop processes events in batches, minimizing latency.

#### Key Components:

| Component       | Description                                                      |
|-----------------|------------------------------------------------------------------|
| File event      | Handles readable/writable events on client sockets              |
| Time event      | Scheduled tasks like key expiration, persistence, replication   |
| Command execution | Sequential processing of client commands                        |

### 3. Data Structures

Redis supports multiple data types, each implemented with efficient internal representations:

| Data Type | Internal Representation(s)                                   | Usage Notes                                |
|-----------|--------------------------------------------------------------|--------------------------------------------|
| String    | Simple dynamic string (SDS)                                  | Binary-safe, supports arbitrary binary data|
| List      | Linked list or compressed ziplist                            | Use ziplists for small lists for memory efficiency|
| Set       | Hash tables or intsets                                       | Intsets are optimized for small integer sets|
| Hash      | Hash tables or ziplist                                       | Ziplist for small hashes with few fields    |
| Sorted Set| Skip lists + Hash tables                                     | Enables O(log N) range queries             |

**Persistence and memory efficiency** are balanced by dynamic switching between representations based on size and complexity.

---

## High Availability: Sentinel

Redis Sentinel provides **monitoring, notification, and automated failover** to ensure Redis availability in production.

---

### Sentinel Architecture

Sentinel is a distributed system comprising multiple Sentinel nodes monitoring one or more Redis master-replica setups.

- **Sentinel Nodes:** Independent processes that discover master and replicas.
- **Quorum:** Minimum number of Sentinels that must agree on a master failure.
- **Leader Election:** Sentinels run a consensus algorithm (based on majority votes) to elect a leader Sentinel responsible for failover.
- **Failover:** On detecting master failure, the leader promotes a replica to master and reconfigures other replicas.

#### Key Sentinel Components:

| Component         | Role                                           |
|-------------------|------------------------------------------------|
| Sentinel Monitors | Continuously check Redis master and replicas  |
| Sentinel Quorum   | Ensures failover decisions are consensus-based|
| Failover Leader   | Coordinates promotion and reconfiguration     |

---

### Failover Process

1. **Subjective Down (SDOWN):** Individual Sentinel marks the master as down based on health checks (e.g., ping timeout).
2. **Objective Down (ODOWN):** When a quorum of Sentinels agree the master is down.
3. **Failover Initiation:** Leader Sentinel selects the best replica to promote as new master.
4. **Replication Reconfiguration:** Replicas are updated to replicate from the new master.
5. **Client Notification:** Sentinels publish events clients can subscribe to for updated master info.

---

### Split-Brain Scenario

**Split-brain** occurs when network partitions lead to multiple Sentinels or clients believing different nodes are master simultaneously, causing data inconsistency.

#### Causes:

- Network partitions isolating Sentinels from each other.
- Insufficient Sentinel quorum.
- Misconfiguration of Sentinel parameters.

#### Mitigation Strategies:

- Deploy Sentinels across multiple, reliable network segments.
- Configure **`sentinel down-after-milliseconds`** conservatively to avoid false positives.
- Ensure quorum size and number of Sentinels are sufficient (minimum 3 recommended).
- Use **`sentinel parallel-syncs`** to limit replication overload during failover.

---

### Sentinel Configuration Example

```conf
# sentinel.conf snippet
sentinel monitor mymaster 192.168.1.100 6379 3
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1
```

- Monitor a master at IP 192.168.1.100 port 6379 with a quorum of 3.
- Mark as down if unresponsive for 5000ms.
- Failover timeout set to 60 seconds.
- Only one replica sync at a time during failover.

---

### Sentinel Troubleshooting

| Issue                          | Symptoms                             | Diagnostic Commands                       | Resolution Steps                                  |
|--------------------------------|------------------------------------|------------------------------------------|--------------------------------------------------|
| False failover trigger          | Frequent failovers or master switch | `SENTINEL ckquorum mymaster`             | Check network latency, increase `down-after-milliseconds` |
| Failover does not start         | Master down but no failover         | `SENTINEL masters` and `SENTINEL slaves` | Verify quorum, check Sentinel logs for errors    |
| Multiple masters (split-brain)  | Clients connected to different masters | `SENTINEL is-master-down-by-addr`        | Review network partitions, increase Sentinel count |
| Replica lag causing promotion delay | Failover stalls                   | `INFO replication` on replicas            | Tune replication, check I/O and CPU load          |

---

## High Availability: Clustering

Redis Cluster enables **automatic sharding and scalability** by partitioning the keyspace across multiple nodes.

---

### Hash Slots

Redis Cluster divides the keyspace into **16,384 hash slots**. Each key is hashed to one of these slots using CRC16 modulo 16384.

- Each node in the cluster is assigned a subset of hash slots.
- Clients send commands directly to the node owning the hash slot of the key.
- This provides linear scalability by distributing keys and load.

---

### Gossip Protocol

Cluster nodes communicate via a **gossip protocol** to:

- Exchange node status.
- Propagate cluster state changes.
- Detect failures.

Nodes periodically send PING/PONG messages with cluster state info.

---

### Resharding

Resharding is the process to **rebalance hash slots** when nodes are added or removed.

- Slots can be moved incrementally between nodes.
- Clients automatically redirect commands to the new owner via **MOVED** or **ASK** responses.
- Resharding requires careful coordination to avoid data loss and downtime.

---

### Cluster Configuration Example

```conf
# redis.conf snippet for cluster node
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 15000
appendonly yes
```

- Enables clustering.
- Specifies cluster state persistence file.
- Sets node timeout to 15 seconds.
- Enables AOF persistence for durability.

---

### Cluster Troubleshooting

| Issue                             | Symptoms                              | Diagnostic Commands                     | Resolution Steps                               |
|----------------------------------|-------------------------------------|---------------------------------------|------------------------------------------------|
| Node failure and slot unavailability | Cluster reports FAIL state          | `CLUSTER NODES`, `CLUSTER INFO`       | Check network connectivity, replace failed node|
| Client redirected in loops         | Clients receive multiple MOVED errors| Enable cluster client support, check client library versions | Upgrade clients, review cluster slot assignments|
| Split-brain in cluster            | Multiple masters for same slots      | `CLUSTER FAILOVER`, `CLUSTER KEYSLOT`| Review cluster configuration, network partitions|
| Resharding stuck or slow          | Keys not moving, high latency        | `CLUSTER COUNT-FAILURE-REPORTS`       | Check node loads, increase `cluster-node-timeout`|

---

# Summary

This first part of the guide has provided an in-depth exploration of Redis and Valkey core architecture, focusing on their single-threaded event-driven design and data structures. We have covered high availability mechanisms—Sentinel and Cluster—with detailed explanations of their architectures, failover and partition handling, configuration samples, and critical troubleshooting workflows.

In production operations, understanding these internals and failure modes is vital for minimizing downtime and ensuring seamless client experiences. Part 2 will continue with persistence mechanisms, eviction policies, tech support operations, and client-facing best practices.

# Specialist Guide to Tech Support Operations for Redis and Valkey  
## Part 2: Persistence Mechanisms & Memory Management

---

## 1. Persistence Mechanisms in Redis and Valkey

Persistence is a critical aspect of Redis and Valkey operations, ensuring data durability and recovery after restarts or crashes. This section covers the three main persistence modes: RDB (Redis Database Backup), AOF (Append Only File), and Hybrid persistence. We will deep dive into their configurations, operational pros and cons, corruption recovery strategies, and common pitfalls such as fork issues.

---

### 1.1 RDB Persistence (Snapshotting)

#### 1.1.1 Overview  
RDB persistence creates point-in-time snapshots of the dataset at configured intervals. It serializes the in-memory dataset into a compact binary dump file (`dump.rdb`), which can be loaded on restart.

#### 1.1.2 Configuration Essentials  
Key configuration directives in `redis.conf` or Valkey equivalent:

```
save 900 1        # Save after 900 seconds if at least 1 key changed
save 300 10       # Save after 300 seconds if at least 10 keys changed
save 60 10000     # Save after 60 seconds if at least 10,000 keys changed
stop-writes-on-bgsave-error yes  # Avoid writes if snapshotting fails
rdbcompression yes               # Compress RDB files using LZF
rdbchecksum yes                  # Enable checksum for corruption detection
dir /var/lib/redis               # Directory for dump.rdb
dbfilename dump.rdb             # RDB filename
```

#### 1.1.3 Pros  
- **Low latency during normal operations**: Snapshots happen asynchronously.
- **Compact, compressed file**: Minimum disk footprint.
- **Fast startup**: Loading RDB snapshots is quicker than AOF replay.
- **Good for backups and disaster recovery**.

#### 1.1.4 Cons  
- **Potential data loss**: Changes since last snapshot are lost on crash.
- **Fork overhead**: RDB snapshot forks a child process which duplicates memory pages (COW). On large datasets or memory fragmentation, this can cause latency spikes or failures.
- **Not suitable for high durability requirements**.

#### 1.1.5 Fork and COW Issues  
- **Fork failures**: When memory is fragmented or under heavy pressure, `fork()` may fail or result in long pauses.
- Monitor fork-related latency with Redis slowlog (`SLOWLOG GET`) and OS metrics (`pidstat`, `vmstat`).
- Tune `vm.overcommit_memory=1` to allow fork despite overcommit.
- Use `RDB compression` and memory defragmentation to reduce memory footprint.

#### 1.1.6 Corruption Recovery  
- On startup, Redis validates checksum; if corrupted, it refuses to load.
- Recovery steps:
  1. Use backups if available.
  2. Attempt to recover partial RDB files using `redis-check-rdb --fix /path/to/dump.rdb`.
  3. Restore from AOF if hybrid persistence is enabled.
  4. If no backup, data loss is inevitable; consider configuring more frequent snapshots or AOF.

---

### 1.2 AOF Persistence (Append-Only File)

#### 1.2.1 Overview  
AOF logs every write operation received by Redis in an append-only log file that can be replayed to reconstruct the dataset.

#### 1.2.2 Configuration Essentials  
Key directives in `redis.conf`:

```
appendonly yes                  # Enable AOF
appendfilename "appendonly.aof" # AOF filename
appendfsync everysec            # Options: always, everysec, no
no-appendfsync-on-rewrite no    # Avoid fsync during rewrite
auto-aof-rewrite-percentage 100 # Rewrite AOF when file size doubles
auto-aof-rewrite-min-size 64mb # Minimum size to trigger rewrite
```

#### 1.2.3 Pros  
- **High durability**: Can persist every write or every second.
- **Incremental write**: No need to snapshot entire dataset.
- **Safer against crashes**: Reduces data loss window.

#### 1.2.4 Cons  
- **Larger disk usage**: Logs all commands.
- **Slower startup**: Replaying AOF takes longer than loading RDB.
- **Rewrite overhead**: AOF rewrite forks a child process that compacts the file but can cause CPU and memory spikes.
- **Potential for AOF corruption** if system crashes during writes.

#### 1.2.5 Fork and Rewrite Issues  
- Similar fork issues as RDB apply during AOF rewrites.
- Monitor rewrite duration and CPU load.
- Tune rewrite thresholds appropriately.
- Disable `no-appendfsync-on-rewrite` only if experiencing fsync spikes.

#### 1.2.6 Corruption Recovery  
- Redis attempts to auto-repair AOF on startup by truncating corrupted parts.
- Use `redis-check-aof --fix /path/to/appendonly.aof` for manual repair.
- Always keep backups of AOF files.
- In Valkey, ensure equivalent AOF durability features are enabled and monitored.

---

### 1.3 Hybrid Persistence (Redis 7+ Feature)

#### 1.3.1 Overview  
Hybrid persistence combines snapshotting with AOF. It creates a base RDB snapshot and appends incremental changes in AOF format. On restart, the snapshot and incremental logs are loaded for fast recovery.

#### 1.3.2 Configuration Example  

```
aof-use-rdb-preamble yes
appendonly yes
appendfilename "appendonly.aof"
rdbcompression yes
save 900 1
```

#### 1.3.3 Pros  
- **Fast restarts**: Load snapshot then replay incremental commands.
- **Good durability**: Near AOF-level durability with snapshot speed.
- **Reduced disk I/O**: Less frequent full rewrites.

#### 1.3.4 Cons  
- **Complexity**: More components to monitor (RDB + AOF).
- **Potential for hybrid file corruption**: Both RDB snapshot and AOF parts must be intact.
- **Limited backward compatibility**.

#### 1.3.5 Troubleshooting  
- Monitor hybrid file sizes and rewriting frequency.
- Validate no fork failures during rewrites.
- Use `redis-check-rdb` and `redis-check-aof` tools on hybrid files if loading fails.
- Keep backups of both files.

---

## 2. Memory Management and Eviction Policies

Redis and Valkey provide advanced memory management to operate efficiently under memory constraints. Understanding and tuning these mechanisms is essential to prevent OOM (Out Of Memory) crashes and performance degradation.

---

### 2.1 Maxmemory Configuration

#### 2.1.1 Overview  
`maxmemory` defines the upper limit of memory Redis/Valkey will use for dataset storage. Exceeding this triggers eviction policies or write rejections.

```
maxmemory 4gb
```

- Monitor actual memory usage with `INFO memory` and OS tools (`top`, `ps`, `smem`).
- Consider total memory footprint, including buffers, clients, and replication overhead.

#### 2.1.2 Memory Overhead Considerations  
- Redis memory usage ≠ dataset size. Includes fragmentation, internal data structures, client buffers.
- Use `MEMORY USAGE key` for detailed per-key memory.
- Use `MEMORY STATS` for global internal fragmentation.

---

### 2.2 Eviction Policies: LRU vs LFU and Others

#### 2.2.1 Eviction Policy Options

| Policy           | Description                                                                                 |
|------------------|---------------------------------------------------------------------------------------------|
| noeviction       | Return errors when maxmemory is reached                                                    |
| allkeys-lru      | Evict least recently used keys regardless of TTL                                           |
| volatile-lru     | Evict least recently used keys with TTL only                                               |
| allkeys-lfu      | Evict least frequently used keys                                                          |
| volatile-lfu     | Evict least frequently used keys with TTL only                                            |
| volatile-ttl     | Evict keys with shortest TTL                                                              |
| volatile-random  | Evict random keys with TTL only                                                            |
| allkeys-random   | Evict random keys regardless of TTL                                                        |

#### 2.2.2 LRU vs LFU Deep Dive

- **LRU (Least Recently Used)**  
  Tracks usage recency; evicts keys not accessed recently. Good for caching workloads with temporal locality.
- **LFU (Least Frequently Used)**  
  Tracks usage frequency; evicts keys with lowest access frequency. Better for workloads with uneven access patterns.

#### 2.2.3 Configuration Example

```
maxmemory-policy allkeys-lru
maxmemory-samples 5      # Number of keys sampled for eviction candidate
```

- `maxmemory-samples` higher values improve eviction accuracy but increase CPU.

#### 2.2.4 Monitoring and Tuning  
- Use `INFO memory` to observe `evicted_keys` count.
- Use `INFO stats` for `keyspace_hits` and `keyspace_misses`.
- Tune eviction policy based on workload characteristics.
- Monitor latency spikes during eviction.

---

### 2.3 Memory Fragmentation and Defragmentation

#### 2.3.1 Understanding Fragmentation  
- **Memory fragmentation** occurs when free memory is split into small blocks, causing inefficient allocation.
- Redis fragmentation ratio: `used_memory_rss / used_memory` (from `INFO memory`).
- High fragmentation (>1.5) indicates memory overhead and potential OOM risk.

#### 2.3.2 Causes of Fragmentation  
- Large keys frequently modified.
- Small allocations and deallocations.
- Forking during persistence.

#### 2.3.3 Active Defragmentation

Introduced in Redis 4.0+, active defragmentation helps reduce fragmentation by relocating memory blocks during runtime.

**Configuration:**

```
activedefrag yes
activedefrag-ignore-bytes 100mb
activedefrag-threshold-lower 10
activedefrag-threshold-upper 100
```

- `activedefrag` enables the feature.
- Thresholds control when defrag starts/stops based on fragmentation ratio.
- Monitor defrag stats with `INFO memory` → `active_defrag_hits`, `active_defrag_misses`, `active_defrag_key_hits`.

#### 2.3.4 Troubleshooting Fragmentation

- If fragmentation is high and defrag ineffective:
  - Consider upgrading jemalloc or switching allocator.
  - Review dataset for large objects or inefficient data types.
  - Restart Redis during maintenance windows.

---

### 2.4 Troubleshooting Out Of Memory (OOM) Issues

#### 2.4.1 Symptoms  
- Redis crashes with OOM error.
- Writes rejected with `OOM command not allowed`.
- Latency spikes due to eviction.

#### 2.4.2 Diagnostic Steps

1. **Check memory usage**:

```
INFO memory
```

2. **Check maxmemory and eviction policy**:

```
CONFIG GET maxmemory
CONFIG GET maxmemory-policy
```

3. **Check eviction stats**:

```
INFO stats
```

Look for `evicted_keys` and `expired_keys`.

4. **Analyze fragmentation**:

```
INFO memory
```

Check `mem_fragmentation_ratio`.

5. **Check slowlog and latency spikes**:

```
SLOWLOG GET 10
```

6. **Monitor OS memory**:

Use `top`, `vmstat`, `free -m`.

#### 2.4.3 Mitigation Strategies

- Increase `maxmemory` if possible.
- Change eviction policy to `allkeys-lru` or `allkeys-lfu` for aggressive eviction.
- Optimize dataset:
  - Compress large values.
  - Use more memory-efficient data structures.
- Enable active defragmentation.
- Reduce client buffer sizes (`client-output-buffer-limit`).
- Review and limit long-running Lua scripts or commands.
- Use Redis memory optimization commands like `MEMORY PURGE` (Redis 4.0+).

#### 2.4.4 Emergency Recovery

- If Redis refuses writes and eviction is not possible (`noeviction` policy):
  1. Increase maxmemory temporarily.
  2. Manually delete large or unnecessary keys.
  3. Restart Redis after clearing memory pressure.

---

## Summary Table: Persistence and Memory Management Quick Reference

| Aspect                  | RDB                          | AOF                            | Hybrid                       | Memory Management                          |
|-------------------------|------------------------------|-------------------------------|------------------------------|--------------------------------------------|
| Durability              | Medium (snapshot intervals)  | High (append every command)   | High (snapshot + append)     | Configurable maxmemory + eviction policies |
| Disk Usage              | Low (compressed snapshot)    | High (append logs)             | Medium                      | N/A                                        |
| Startup Time            | Fast                        | Slow (replay commands)         | Medium (snapshot + replay)  | N/A                                        |
| Fork Overhead           | High (snapshot fork)         | High (rewrite fork)            | High (both)                 | N/A                                        |
| Corruption Recovery     | Checksum + redis-check-rdb   | Auto repair + redis-check-aof  | Both tools applicable       | N/A                                        |
| Eviction Policies       | N/A                         | N/A                           | N/A                         | LRU, LFU, TTL, Random                      |
| Fragmentation Handling  | N/A                         | N/A                           | N/A                         | Active defragmentation available            |
| OOM Handling            | N/A                         | N/A                           | N/A                         | Reject writes or evict keys per policy     |

---

## Final Notes for Tech Support Engineers

- **Always monitor persistence fork times and memory fragmentation ratios in production.** Sudden spikes often presage outages.
- **Maintain regular backups of RDB and AOF files.** Corruption is rare but catastrophic.
- **Eviction policy tuning is workload-specific.** Analyze access patterns before changing policies.
- **Use Redis and Valkey CLI tools extensively for diagnostics (`INFO`, `MONITOR`, `SLOWLOG`, `MEMORY`).**
- **Document configuration

# Part 3: Tech Support Operations & Troubleshooting for Redis and Valkey

---

## 1. Tech Support Operations & Troubleshooting

This section addresses the most critical and complex scenarios encountered in production environments running Redis and Valkey. It focuses on diagnosing, troubleshooting, and resolving issues related to performance degradation, latency spikes, CPU anomalies, and network instabilities.

---

### 1.1 Worst-Case Scenarios in Production

Worst-case scenarios often manifest as critical outages or severe performance degradation, disrupting business continuity. Key examples include:

- **Complete Redis or Valkey unresponsiveness:** Clients experience connection timeouts or failures.
- **Persistent high latency:** Operations take significantly longer than expected, impacting SLAs.
- **CPU spikes leading to node instability or crashes.**
- **Network partitions or flapping causing cluster state inconsistencies.**
- **Memory exhaustion and eviction storms** that cause thrashing and increased latency.

**Proactive readiness:** Always ensure monitoring, alerting, and logging are in place to detect early signs of these scenarios.

---

### 1.2 Debugging Latency Issues

Latency problems can be multifactorial. The following systematic approach is recommended:

#### Step 1: Use the Redis `LATENCY` Command Suite

Redis provides a set of tools to analyze latency spikes:

- **`LATENCY DOCTOR`**: Diagnoses potential latency sources.
- **`LATENCY GRAPH <event>`**: Visualizes latency spikes for specific events.
- **`LATENCY LATEST`**: Shows the most recent latency spikes.

Example:

```
127.0.0.1:6379> LATENCY DOCTOR
```

This command outputs an analysis report identifying latency causes such as slow commands, blocking operations, or network issues.

#### Step 2: Analyze Slowlog

The Redis slowlog tracks commands exceeding a configured execution time threshold.

- **Check slowlog length:**

```
127.0.0.1:6379> SLOWLOG LEN
```

- **Retrieve slowlog entries:**

```
127.0.0.1:6379> SLOWLOG GET 10
```

Examine the command names, durations (in microseconds), and timestamps. Look for patterns or specific commands disproportionately contributing to latency.

##### Configuration example to enable slowlog with a threshold of 1000 microseconds (1 ms):

```
slowlog-log-slower-than 1000
slowlog-max-len 128
```

#### Step 3: Monitor Latency Over Time

Use Redis latency monitor tools or integrate with external APMs (Application Performance Monitoring) to track latency trends.

---

### 1.3 CPU Spike Troubleshooting

High CPU usage can indicate:

- Excessive command volume or expensive commands (e.g., `KEYS`, `SCAN` with large datasets).
- Inefficient Lua scripts or blocking commands.
- Garbage collection or background save (RDB/AOF) operations.

#### Step 1: Identify CPU Usage Using Redis CLI and OS Tools

- On Linux, use `top`, `htop`, or `pidstat` to identify process CPU consumption.
- Within Redis, use:

```
INFO CPU
```

to get CPU usage statistics:

| Field               | Description                                  |
|---------------------|----------------------------------------------|
| `used_cpu_sys`      | System CPU consumed by Redis (seconds)        |
| `used_cpu_user`     | User CPU consumed by Redis (seconds)          |
| `used_cpu_sys_children` | System CPU by child processes (forked saves) |
| `used_cpu_user_children`| User CPU by child processes (forked saves)   |

#### Step 2: Correlate CPU Spikes with Commands

Enable Redis command stats:

```
CONFIG SET latency-monitor-threshold 100
```

or

```
INFO commandstats
```

to identify commands with high cumulative CPU time.

#### Step 3: Mitigate CPU Spikes

- Avoid or optimize expensive commands.
- Use pipelining to reduce roundtrips.
- Schedule background saves (RDB/AOF rewrites) during off-peak times.
- Profile Lua scripts for optimization.
- Upgrade hardware or vertically scale if CPU capacity is insufficient.

---

### 1.4 Network Issues and Their Impact

Network instability can cause:

- Increased command latency.
- Connection drops and retries.
- Cluster failover storms.

#### Step 1: Diagnose Network Connectivity

- Use tools like `ping`, `traceroute`, or `mtr` to verify latency and packet loss.
- Check TCP retransmissions with `netstat -s` or `ss`.

#### Step 2: Redis-Specific Checks

- Monitor Redis client connection counts and errors:

```
INFO clients
```

Check for large numbers of `rejected_connections` or `blocked_clients`.

- Verify cluster node connectivity:

```
redis-cli -c CLUSTER NODES
```

Look for nodes in `fail` or `handshake` state.

---

### 1.5 Valkey-Specific Troubleshooting Considerations

Valkey, as a key-value store often integrated with Redis or used in specialized scenarios, may also exhibit:

- **Stale or inconsistent key states.**
- **Integration latency due to syncing or replication delays.**

Operationally:

- Verify Valkey daemon logs for errors.
- Use Valkey-specific CLI tools or APIs to check key-state consistency.
- Monitor Valkey's interaction latency with Redis.

---

## 2. Client-Facing Guidance

Providing clients with best practices reduces support overhead and improves system stability.

---

### 2.1 Best Practices for Connecting to Redis/Valkey

- **Use connection pooling:** Avoid creating new connections per request; reuse pooled connections to reduce overhead.
- **Set appropriate timeouts:** Configure `connect_timeout` and `read_timeout` to detect and recover from network issues.
- **Prefer pipelining:** Batch multiple commands to reduce network roundtrips.
- **Handle redirects gracefully:** Implement proper logic for Redis Cluster `MOVED` and `ASK` replies.
- **Use retries with exponential backoff** for transient errors.
- **Monitor client-side metrics** to detect early signs of issues.

---

### 2.2 Connection Pooling Configuration Example (Python redis-py)

```python
import redis
from redis.connection import ConnectionPool

pool = ConnectionPool(host='redis-host', port=6379, max_connections=50, socket_timeout=5)
client = redis.Redis(connection_pool=pool)

# Use client normally
client.set('key', 'value')
```

---

### 2.3 Timeouts and Their Importance

Set **connect_timeout** and **socket_timeout** to avoid indefinite blocking:

| Timeout Type     | Description                                  | Recommended Value           |
|------------------|----------------------------------------------|----------------------------|
| `connect_timeout`| Time to establish TCP connection             | 1-3 seconds                |
| `socket_timeout` | Time to wait for a response from Redis       | 2-5 seconds                |

Example configuration for a .NET StackExchange.Redis client:

```csharp
ConfigurationOptions options = new ConfigurationOptions
{
    EndPoints = { "redis-host:6379" },
    ConnectTimeout = 2000,  // 2 seconds
    SyncTimeout = 5000      // 5 seconds
};
var redis = ConnectionMultiplexer.Connect(options);
```

---

### 2.4 Pipelining to Reduce Latency

Pipelining enables sending multiple commands without waiting for each response:

```python
pipe = client.pipeline()
pipe.set('key1', 'value1')
pipe.set('key2', 'value2')
pipe.get('key1')
responses = pipe.execute()
```

Benefits:

- Reduces network RTT.
- Improves throughput.

---

### 2.5 Handling MOVED and ASK Redirects in Redis Cluster

Redis Cluster uses **MOVED** and **ASK** redirects when keys reside on different slots.

- **MOVED:** Permanent slot migration; clients should update slot cache.
- **ASK:** Temporary redirection during slot migration.

**Client Implementation Guidance:**

- Use client libraries that support cluster mode and auto-handle redirects.
- On receiving a MOVED response, update the client’s slot cache before retrying.
- On ASK, send `ASKING` command before retrying the redirected command.

Example snippet (pseudo-code):

```
if response == MOVED:
    update_slot_cache()
    retry_command()

elif response == ASK:
    send("ASKING")
    retry_command()
```

---

## 3. Integration with Other Specialist Files

---

### 3.1 Overview of the Specialist-Teams Repository

The Redis/Valkey specialist guide is part of a comprehensive repository consisting of **7 specialist files**, each covering critical aspects of system support and operations:

| File Number | File Name                     | Focus Area                                |
|-------------|-------------------------------|-------------------------------------------|
| Part 1      | Redis/Valkey Architecture     | System design and infrastructure overview |
| Part 2      | Redis/Valkey Configuration    | Configuration best practices and tuning   |
| **Part 3**  | Redis/Valkey Operations (this file) | Production troubleshooting and client guidance |
| Part 4      | Redis Security & Compliance   | Security hardening and compliance          |
| Part 5      | Redis Backup & Disaster Recovery | Data safety and failover procedures       |
| Part 6      | Valkey Integration Patterns   | Application integration and API usage      |
| Part 7      | Monitoring & Alerting          | Metrics, dashboards, and automation        |

---

### 3.2 How Part 3 Relates to Other Files

- **Dependency on Part 1 & 2:**  
  Understanding the architecture and configuration (Parts 1 & 2) is foundational to effective troubleshooting. For example, knowing cluster topology and configured timeouts is essential when diagnosing latency or connection issues.

- **Operational Context for Security (Part 4):**  
  Troubleshooting may surface security-related causes (e.g., ACL misconfigurations or network firewall blocks). Part 3 advises on detection while Part 4 provides remediation.

- **Backup & DR (Part 5) Coordination:**  
  CPU spikes or unresponsiveness may be caused by background save operations covered in Part 5. Tech support engineers should cross-reference backup schedules when analyzing resource usage.

- **Valkey Integration (Part 6):**  
  Part 3 covers Valkey-specific troubleshooting, but detailed integration patterns and API usage are elaborated in Part 6. Client guidance around connection pooling and timeouts complements these integration patterns.

- **Monitoring & Alerting (Part 7) Synergy:**  
  Latency monitoring and CPU spike detection rely heavily on metrics and alerting strategies defined in Part 7. Alerts can trigger the operational workflows outlined in Part 3.

---

### 3.3 Recommended Operational Workflow Across Files

| Operational Step           | Associated Specialist File(s)                 | Description                                      |
|----------------------------|-----------------------------------------------|-------------------------------------------------|
| Understand system design   | Part 1                                        | Review architecture to orient troubleshooting.  |
| Confirm configuration      | Part 2                                        | Validate configuration settings impacting ops.  |
| Detect and diagnose issues | Part 3                                        | Use CLI commands, logs, and client-side checks. |
| Validate security posture  | Part 4                                        | Check ACLs, firewall rules, and encryption.     |
| Review backup status       | Part 5                                        | Confirm backups are successful and recent.      |
| Verify application integration | Part 6                                    | Ensure client usage follows recommended patterns.|
| Monitor & alert            | Part 7                                        | Utilize dashboards and alerts for proactive ops.|

---

## Conclusion

Part 3 of this Redis/Valkey specialist guide equips technical support engineers with deep, practical knowledge to tackle worst-case production issues, optimize client interactions, and integrate seamlessly with the broader operational framework. Mastery of these troubleshooting techniques and client best practices is essential for maintaining high availability and performance in critical Redis and Valkey deployments.