# Redis and Valkey Configuration Tuning Recommendations

## 1. Introduction to Redis and Valkey Configuration

In modern high-performance architectures, in-memory data stores like Redis and its open-source fork, Valkey, serve as critical components for caching, session management, message brokering, and real-time analytics. While both systems are designed to be exceptionally fast out of the box, achieving optimal performance, reliability, and stability in production environments requires meticulous configuration tuning. 

This document serves as a comprehensive guide for tech support operations and systems engineers. It delves deep into the intricacies of configuration tuning for Redis and Valkey, focusing on memory thresholds, eviction policies (`maxmemory-policy`), snapshotting intervals (`save`), and append-only file synchronization settings (`appendfsync`). Furthermore, it addresses worst-case scenarios, providing actionable insights for troubleshooting and mitigating production incidents.

The recommendations provided herein are based on extensive production experience and are designed to help you build resilient, high-throughput, and low-latency data infrastructure. Whether you are managing a small caching layer or a massive, globally distributed data store, understanding these configuration schemas is paramount.

---

## 2. Memory Management and Thresholds

Memory is the most critical resource for Redis and Valkey. Improper memory management can lead to out-of-memory (OOM) errors, operating system swapping, and catastrophic performance degradation. 

### 2.1 Setting the `maxmemory` Directive

The `maxmemory` directive dictates the maximum amount of RAM that the instance is allowed to consume for data. By default, this is set to zero on 64-bit systems, meaning the system will attempt to use all available memory. In a production environment, this is highly dangerous.

**Recommendation:** Always set a hard limit for `maxmemory`. A general rule of thumb is to allocate no more than 70-80% of the total system RAM to the instance, leaving the remaining 20-30% for the operating system, background saving processes (BGSAVE), and network buffers.

```conf
# Example: Limit memory usage to 12 Gigabytes
maxmemory 12gb
```

### 2.2 Understanding Memory Overhead

When calculating memory thresholds, you must account for memory overhead. This includes:
- **Client Output Buffers:** Memory used to queue responses for connected clients. Slow clients can cause these buffers to grow rapidly.
- **Replication Backlog:** Memory used to buffer commands for replicas.
- **AOF Rewrite Buffers:** Memory consumed during the Append-Only File rewrite process.

**Tech Support Tip:** If an instance is unexpectedly hitting its `maxmemory` limit, investigate the `used_memory_rss` metric. A high RSS (Resident Set Size) compared to `used_memory` indicates memory fragmentation. You can trigger active defragmentation in Redis 4.0+ and Valkey using `activedefrag yes`.

### 2.3 The Dangers of Swapping

If the instance exceeds physical memory and the OS begins swapping pages to disk, latency will spike from microseconds to milliseconds or even seconds. This is often the root cause of application-level timeouts.

**Mitigation:** 
1. Ensure `maxmemory` is strictly enforced.
2. Disable swap on the host machine entirely, or set `vm.swappiness=1` in the Linux kernel to heavily discourage swapping.

---

## 3. Maxmemory Policies (Eviction Strategies)

When the `maxmemory` limit is reached, the system must decide how to handle new write requests. The `maxmemory-policy` directive controls this behavior. Choosing the right policy is crucial and depends entirely on your use case.

### 3.1 Available Policies

- **noeviction:** (Default) Returns an error when the memory limit is reached and the client attempts to execute commands that could result in more memory being used. Ideal for databases where data loss is unacceptable.
- **allkeys-lru:** Evicts the least recently used (LRU) keys out of all keys. Best for general caching workloads where recently accessed data is most likely to be accessed again.
- **volatile-lru:** Evicts the least recently used keys among those that have an expiration set. Useful when you mix persistent data and cache data in the same instance.
- **allkeys-random:** Evicts random keys. Rarely recommended unless access patterns are completely uniform.
- **volatile-random:** Evicts random keys among those with an expiration set.
- **volatile-ttl:** Evicts keys with an expiration set, prioritizing those with the shortest time-to-live (TTL).
- **allkeys-lfu:** (Available in Redis 4.0+) Evicts the least frequently used (LFU) keys out of all keys. Excellent for power-law access patterns where a small subset of keys is accessed very frequently.
- **volatile-lfu:** Evicts the least frequently used keys among those with an expiration set.

### 3.2 Configuration Recommendations

For a pure caching layer, `allkeys-lru` or `allkeys-lfu` are the most robust choices. LFU often provides a better hit rate for typical web workloads.

```conf
# Example: Use LFU eviction for a caching workload
maxmemory-policy allkeys-lfu
```

**Worst-Case Scenario:** A sudden influx of writes hits an instance configured with `noeviction`. The application begins receiving OOM errors, leading to widespread failures. 
**Resolution:** Temporarily increase `maxmemory` if physical RAM allows, or immediately switch the policy to `allkeys-lru` via the `CONFIG SET` command to shed load, acknowledging that some data will be lost.

---

## 4. Persistence Strategies: RDB and AOF

Redis and Valkey offer two primary persistence mechanisms: RDB (Redis Database) snapshots and AOF (Append-Only File). Understanding how to tune these is vital for balancing performance and data durability.

### 4.1 RDB (Snapshotting)

RDB creates point-in-time snapshots of your dataset at specified intervals. It is highly efficient for backups and disaster recovery because it produces a compact, single-file representation of the data. However, any data written after the last snapshot will be lost in the event of a crash.

### 4.2 AOF (Append-Only File)

AOF logs every write operation received by the server. Upon restart, these operations are replayed to reconstruct the dataset. AOF provides much stronger durability guarantees but can result in larger file sizes and slightly higher disk I/O overhead.

### 4.3 Hybrid Persistence

Modern deployments often use a combination of both: RDB for fast restarts and compact backups, and AOF for up-to-the-second durability. In Redis 4.0+ and Valkey, the AOF rewrite process can generate a hybrid file containing an RDB preamble followed by AOF logs, offering the best of both worlds.

```conf
# Enable AOF
appendonly yes
# Enable hybrid persistence
aof-use-rdb-preamble yes
```

---

## 5. Tuning Save Intervals (RDB)

The `save` directive configures the conditions under which an RDB snapshot is automatically triggered. The syntax is `save <seconds> <changes>`.

### 5.1 Default Configuration

The default configuration is often too aggressive for high-throughput environments:
```conf
save 3600 1      # Save after 1 hour if at least 1 key changed
save 300 100     # Save after 5 minutes if at least 100 keys changed
save 60 10000    # Save after 60 seconds if at least 10000 keys changed
```

### 5.2 Production Recommendations

Frequent snapshotting on a large dataset can cause severe latency spikes due to the `fork()` system call required to create the background saving process. The time it takes to fork is proportional to the size of the dataset.

**Recommendation for High-Throughput Caches:** If the instance is purely a cache and data loss is acceptable, disable RDB saving entirely to maximize performance.
```conf
# Disable automatic RDB saving
save ""
```

**Recommendation for Persistent Stores:** Relax the save intervals to reduce disk I/O and fork latency. Rely on AOF for durability.
```conf
# Less aggressive RDB saving
save 3600 1000
save 900 10000
```

**Tech Support Operations:** If you observe periodic latency spikes, check the `INFO persistence` output. Look at `latest_fork_usec`. If this value is high (e.g., > 100,000 microseconds), the fork process is blocking the main thread. Consider disabling transparent huge pages (THP) on the Linux host, as THP severely exacerbates fork latency.

---

## 6. Tuning Appendfsync Settings (AOF)

The `appendfsync` directive controls how often the operating system is instructed to flush the AOF buffer to disk. This is the primary knob for balancing data safety and write performance.

### 6.1 Available Options

- **always:** `fsync` is called after every single write command. This provides the highest level of data safety (zero data loss) but severely degrades performance, as disk I/O becomes the bottleneck.
- **everysec:** (Default) `fsync` is called once per second by a background thread. This is the recommended compromise. In the worst-case scenario (a sudden power loss or kernel panic), you will lose at most one second of data. Performance is generally excellent.
- **no:** The system relies on the operating system to flush the buffers when it sees fit (typically every 30 seconds on Linux). This offers the best performance but the lowest data safety.

### 6.2 Production Recommendations

For 99% of production workloads, `everysec` is the optimal setting.

```conf
appendfsync everysec
```

### 6.3 Handling Disk I/O Bottlenecks

In high-write environments, even `everysec` can cause issues if the underlying storage is slow. If the background `fsync` thread takes longer than one second to complete, the main thread will block on subsequent write operations to prevent the buffer from growing indefinitely.

**Tech Support Operations:** Monitor the `aof_delayed_fsync` metric in the `INFO` output. If this value is incrementing, it means the main thread is being blocked by slow disk I/O. 

**Mitigation:**
1. Upgrade to faster storage (NVMe SSDs).
2. Ensure `no-appendfsync-on-rewrite` is set to `yes`. This prevents `fsync` from being called while a heavy AOF rewrite or RDB save is in progress, reducing disk contention at the cost of slightly higher risk during the rewrite window.

```conf
no-appendfsync-on-rewrite yes
```

---

## 7. Network and Connection Tuning

While memory and persistence are critical, network configuration is equally important for maintaining stability under load.

### 7.1 Client Output Buffer Limits

Runaway clients (e.g., a client executing a massive `KEYS *` command or a slow consumer subscribing to a high-volume Pub/Sub channel) can consume vast amounts of memory in output buffers, leading to OOM.

Configure strict limits to disconnect misbehaving clients:
```conf
# Normal clients: no limit
client-output-buffer-limit normal 0 0 0
# Pub/Sub clients: disconnect if buffer exceeds 32mb, or stays above 8mb for 60s
client-output-buffer-limit pubsub 32mb 8mb 60
```

### 7.2 TCP Keepalive and Timeout

Ensure idle connections are aggressively pruned to free up file descriptors and memory.
```conf
# Close connections after 300 seconds of idleness
timeout 300
# Send TCP keepalive probes every 60 seconds
tcp-keepalive 60
```

---

## 8. Advanced Configuration and Worst-Case Scenarios

### 8.1 The "Thundering Herd" Problem

**Scenario:** A highly trafficked cache key expires, and hundreds of application threads simultaneously query the database and attempt to write the new value back to the cache. This causes a massive spike in database load and cache writes.
**Mitigation:** Implement cache stampede protection at the application layer (e.g., probabilistic early expiration or mutex locks). On the cache side, ensure `maxmemory-policy` is correctly configured to handle the sudden influx of writes without crashing.

### 8.2 Replication Storms

**Scenario:** A master node restarts or experiences a brief network partition. Multiple replicas attempt to perform a full synchronization simultaneously, saturating the master's network interface and CPU, causing it to become unresponsive.
**Mitigation:** 
1. Increase the `repl-backlog-size` to allow replicas to perform partial resynchronizations (PSYNC) even after longer disconnects.
```conf
repl-backlog-size 512mb
```
2. Use a cascading replication topology (Master -> Replica -> Sub-Replica) to reduce the direct load on the master.

### 8.3 Security Considerations

Never expose an instance to the public internet without authentication and encryption.
```conf
# Require a strong password
requirepass "YourSuperStrongPasswordHere"
# Bind to internal interfaces only
bind 10.0.0.5 127.0.0.1
# Rename dangerous commands
rename-command FLUSHALL ""
rename-command FLUSHDB ""
rename-command KEYS ""
```

---

## 9. Tech Support Operations Guide: Incident Response

When responding to a production incident involving Redis or Valkey, follow this diagnostic workflow:

1. **Check Latency:** Use `redis-cli --latency` to measure network and processing latency. If latency is high, the issue is likely server-side.
2. **Inspect Memory:** Run `INFO memory`. Check `used_memory`, `used_memory_rss`, and `mem_fragmentation_ratio`. If fragmentation is > 1.5, consider active defragmentation or a restart.
3. **Analyze Persistence:** Run `INFO persistence`. Look for `aof_delayed_fsync` > 0 or high `latest_fork_usec`. These indicate disk I/O or CPU bottlenecks during background saves.
4. **Identify Slow Queries:** Use the `SLOWLOG GET` command to identify commands that are blocking the main thread. Look for `KEYS`, `SMEMBERS` on large sets, or massive `MGET` operations.
5. **Monitor Connections:** Run `INFO clients`. A sudden spike in `connected_clients` or `client_recent_max_output_buffer` indicates application-side connection leaks or slow consumers.

By adhering to these configuration schemas and operational guidelines, tech support teams can ensure that Redis and Valkey deployments remain robust, performant, and resilient against the most demanding production workloads.
