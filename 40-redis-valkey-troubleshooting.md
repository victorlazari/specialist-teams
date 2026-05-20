# Deep Troubleshooting Guide for Redis and Valkey

## Introduction

In modern distributed architectures, Redis and Valkey serve as critical components for caching, session management, message brokering, and real-time analytics. As in-memory data stores, their performance and reliability directly impact the overall system's responsiveness and stability. However, operating Redis and Valkey at scale introduces complex challenges that require deep technical expertise to diagnose and resolve. This comprehensive troubleshooting guide is designed for tech support operations, site reliability engineers (SREs), and database administrators who manage production environments. It focuses on worst-case scenarios, practical diagnostics, and actionable remediation strategies for the most critical issues: Out of Memory (OOM) events, high latency, fork blocking time, replication lag, split-brain scenarios in Sentinel and Cluster deployments, and connection limit exhaustion.

This document serves as a definitive reference for resolving severe production incidents. It goes beyond basic configuration advice to explore the internal mechanics of Redis and Valkey, providing advanced techniques for identifying root causes and implementing robust solutions. By mastering these troubleshooting methodologies, operations teams can minimize downtime, optimize resource utilization, and ensure the high availability of their data infrastructure.

## 1. Out of Memory (OOM) Issues

Out of Memory (OOM) events are among the most catastrophic failures in Redis and Valkey environments. Because these systems store data primarily in RAM, exhausting available memory leads to immediate service degradation, data eviction, or process termination by the operating system's OOM killer. Understanding the nuances of memory management is essential for preventing and resolving these incidents.

### 1.1 Diagnosing OOM Events

When an OOM event occurs, the first step is to determine whether the issue stems from the data store's internal memory limits or the operating system's resource constraints. The `INFO memory` command provides critical insights into memory utilization.

Key metrics to analyze include:
- `used_memory`: The total amount of memory allocated by Redis/Valkey using its allocator (e.g., jemalloc).
- `used_memory_rss`: The amount of memory the operating system has allocated to the process (Resident Set Size).
- `mem_fragmentation_ratio`: The ratio of `used_memory_rss` to `used_memory`. A high ratio indicates significant memory fragmentation.
- `maxmemory`: The configured memory limit for the instance.

If `used_memory` approaches or exceeds `maxmemory`, the system will begin applying its configured eviction policy. If the eviction policy is set to `noeviction`, write operations will fail with an OOM error. Conversely, if `used_memory_rss` approaches the system's physical memory limit, the OS OOM killer may terminate the process.

### 1.2 Investigating the Root Cause

Several factors can contribute to OOM conditions. Identifying the specific cause requires a systematic approach:

1. **Unbounded Data Growth**: The most common cause is simply storing more data than the instance can handle. This often occurs when keys lack expiration times (TTL) or when application logic fails to clean up obsolete data. Use the `MEMORY STATS` command to identify memory consumption patterns and the `redis-cli --bigkeys` utility to locate unusually large keys or data structures.
2. **Client Output Buffer Leaks**: Redis and Valkey maintain output buffers for each connected client. If a client consumes data slower than the server produces it, these buffers can grow indefinitely, consuming vast amounts of memory. Monitor the `client_longest_output_list` metric in `INFO clients` and inspect individual client connections using `CLIENT LIST`. Pay close attention to the `omem` (output memory) field.
3. **Replication Backlog**: The replication backlog buffer (`repl_backlog_size`) stores recent write commands to facilitate partial resynchronizations with replicas. While its size is configurable, an excessively large backlog or a prolonged disconnection from a replica can lead to memory pressure.
4. **Memory Fragmentation**: High memory fragmentation occurs when the allocator cannot efficiently reuse freed memory blocks. This is common in workloads with frequent updates or deletions of varying-sized values. A `mem_fragmentation_ratio` significantly above 1.5 warrants investigation.

### 1.3 Remediation and Prevention

Resolving OOM issues requires immediate action to restore service, followed by long-term strategies to prevent recurrence.

**Immediate Actions:**
- **Scale Up**: If possible, dynamically increase the `maxmemory` limit or provision a larger instance.
- **Manual Eviction**: Identify and delete non-critical data or large keys using `UNLINK` (which is non-blocking) rather than `DEL`.
- **Terminate Slow Clients**: Use `CLIENT KILL` to disconnect clients with excessively large output buffers.

**Long-Term Strategies:**
- **Implement Eviction Policies**: Configure an appropriate `maxmemory-policy` (e.g., `volatile-lru`, `allkeys-lru`) to automatically evict less important data when memory limits are reached.
- **Enforce TTLs**: Ensure that all transient data has an appropriate Time-To-Live (TTL) set.
- **Optimize Data Structures**: Use memory-efficient data structures, such as hashes for small objects, and leverage features like ziplists.
- **Configure Client Output Buffer Limits**: Set strict limits on client output buffers using the `client-output-buffer-limit` directive in the configuration file to prevent slow clients from exhausting memory.
- **Active Defragmentation**: Enable active defragmentation (`activedefrag yes`) to allow the system to continuously compact memory in the background, reducing fragmentation over time.

## 2. High Latency Diagnostics

Redis and Valkey are renowned for their sub-millisecond response times. When latency spikes occur, they can severely impact application performance. Diagnosing high latency requires distinguishing between network issues, host-level constraints, and internal processing delays.

### 2.1 Identifying Latency Sources

The first step is to isolate the source of the latency. Is it occurring on the client side, the network, or the server?

- **Client-Side Latency**: Use the `redis-cli --latency` and `redis-cli --latency-history` tools from the client machine to measure the round-trip time (RTT) to the server. If the RTT is high but server-side metrics are normal, the issue likely lies in the network or the client application itself.
- **Server-Side Latency**: The `INFO commandstats` command provides execution times for individual commands. The `SLOWLOG` feature is invaluable for identifying specific queries that exceed a configured execution time threshold.

### 2.2 Common Causes of Server-Side Latency

If the latency is isolated to the server, investigate the following common culprits:

1. **Blocking Commands**: Commands like `KEYS`, `SMEMBERS` on large sets, or `HGETALL` on large hashes can block the single-threaded event loop, delaying the execution of all subsequent commands. Always prefer iterative commands like `SCAN`, `SSCAN`, and `HSCAN` in production environments.
2. **Algorithmic Complexity**: Be aware of the time complexity (Big O notation) of the commands being executed. Operations with O(N) complexity, where N is large, will inherently take longer to process.
3. **Transparent Huge Pages (THP)**: The Linux kernel feature THP can cause significant latency spikes during memory allocation and fork operations. It is strongly recommended to disable THP on systems running Redis or Valkey.
4. **Disk I/O Interference**: While primarily in-memory, Redis and Valkey interact with the disk for persistence (RDB snapshots and AOF logs). If the underlying storage is slow or heavily utilized by other processes, disk I/O can block the main thread, especially during AOF `fsync` operations. Monitor disk performance using tools like `iostat`.
5. **Swapping**: If the system runs out of physical memory and begins swapping to disk, performance will degrade catastrophically. Ensure that swapping is disabled or that the `vm.swappiness` kernel parameter is set to a very low value (e.g., 1).

### 2.3 Utilizing the Latency Monitor

Redis and Valkey include a built-in Latency Monitor feature that tracks operations exceeding a specified threshold (`latency-monitor-threshold`).

Enable the monitor:
```redis
CONFIG SET latency-monitor-threshold 100
```

Analyze the collected data using the `LATENCY` command suite:
- `LATENCY LATEST`: Displays the latest latency events for various subsystems (e.g., command execution, fast-command, fork).
- `LATENCY HISTORY`: Provides a historical view of latency events for a specific subsystem.
- `LATENCY DOCTOR`: Generates a human-readable report analyzing the latency data and suggesting potential causes and solutions.

By systematically analyzing slow logs, command statistics, and latency monitor data, operations teams can pinpoint the exact operations or system conditions causing performance degradation.

## 3. Fork Blocking Time

Redis and Valkey rely on the `fork()` system call to create background processes for tasks such as RDB snapshotting and AOF rewriting. While the background process handles the disk I/O, the `fork()` operation itself blocks the main event loop. In environments with large datasets, this blocking time can become a significant source of latency.

### 3.1 Understanding the Fork Operation

When `fork()` is called, the operating system creates a copy of the parent process's page tables. The time required for this operation is directly proportional to the amount of memory allocated to the process. For massive datasets (e.g., tens or hundreds of gigabytes), copying the page tables can take hundreds of milliseconds or even seconds, during which the server cannot process any client commands.

### 3.2 Measuring Fork Latency

The `INFO stats` command provides metrics related to fork operations:
- `latest_fork_usec`: The duration of the most recent fork operation in microseconds.
- `total_forks`: The total number of fork operations performed since the server started.

Additionally, the Latency Monitor (discussed in Section 2) tracks fork events under the `fork` subsystem.

### 3.3 Mitigating Fork Blocking

Reducing fork blocking time involves optimizing both the operating system configuration and the data store's deployment architecture.

1. **Disable Transparent Huge Pages (THP)**: As mentioned earlier, THP significantly exacerbates fork latency. When THP is enabled, the OS manages memory in 2MB pages instead of the standard 4KB pages. During a fork, copying these larger page tables takes considerably longer. Disabling THP is a critical optimization.
2. **Hardware Considerations**: The speed of the CPU and memory architecture directly impacts fork performance. Modern processors with faster memory bandwidth will execute the `fork()` system call more quickly.
3. **Control Dataset Size**: The most effective way to reduce fork time is to limit the size of the dataset managed by a single instance. Instead of running a single massive instance, consider partitioning the data across multiple smaller instances using Redis Cluster or application-level sharding.
4. **Optimize Persistence Strategies**: Evaluate whether frequent RDB snapshots or AOF rewrites are strictly necessary. Adjust the `save` directives for RDB and the `auto-aof-rewrite-percentage` for AOF to reduce the frequency of background operations. In some high-performance caching scenarios, persistence can be disabled entirely.
5. **Replica-Only Persistence**: A common architectural pattern is to disable persistence on the master node to ensure maximum performance and offload the snapshotting and AOF rewriting tasks to a dedicated replica node.

## 4. Replication Lag and Disconnects

Replication is fundamental to high availability and read scaling in Redis and Valkey. However, maintaining synchronization between the master and replicas can be challenging under heavy load or suboptimal network conditions. Replication lag occurs when a replica falls behind the master's dataset, potentially leading to stale reads or full resynchronizations.

### 4.1 Monitoring Replication Health

The `INFO replication` command is the primary tool for monitoring replication status.

Key metrics include:
- `master_repl_offset`: The current replication offset of the master.
- `slave_repl_offset` (on the replica): The replication offset currently processed by the replica.
- `master_last_io_seconds_ago` (on the replica): The time elapsed since the last communication with the master.

The replication lag can be calculated as the difference between the `master_repl_offset` and the `slave_repl_offset`.

### 4.2 Causes of Replication Lag

Several factors can contribute to replication lag:

1. **Network Congestion**: Insufficient network bandwidth or high latency between the master and replica can delay the transmission of replication data.
2. **Slow Replicas**: If the replica is running on underpowered hardware or is overwhelmed by read queries, it may not be able to process the replication stream fast enough to keep up with the master.
3. **Blocking Operations on Replicas**: Executing long-running commands (e.g., `KEYS`, complex Lua scripts) on a replica will block its event loop, preventing it from processing incoming replication data.
4. **Disk I/O Bottlenecks**: If the replica is configured to persist data to disk, slow disk I/O can delay the application of replication commands.

### 4.3 Handling Disconnects and Resynchronization

When a replica disconnects from the master, it attempts to reconnect and perform a partial resynchronization (PSYNC). The master maintains a replication backlog buffer (`repl_backlog_size`) that stores recent write commands. If the replica's offset is still within the backlog, the master sends only the missing commands.

However, if the disconnection is prolonged or the backlog is too small, the replica's offset will fall out of the backlog. In this case, a full resynchronization is required. A full resync involves the master generating a complete RDB snapshot, transmitting it to the replica, and then sending the buffered write commands. This process is highly resource-intensive, causing significant CPU, memory, and network overhead on both nodes.

**Mitigation Strategies:**
- **Increase Replication Backlog**: Configure a sufficiently large `repl_backlog_size` to accommodate temporary network partitions or replica restarts without triggering a full resync. The ideal size depends on the write volume and the expected duration of disconnects.
- **Optimize Network Infrastructure**: Ensure robust and high-bandwidth network connectivity between master and replica nodes, especially across different availability zones or data centers.
- **Monitor Replica Load**: Distribute read queries evenly across multiple replicas to prevent any single node from becoming a bottleneck.

## 5. Split-Brain in Sentinel and Cluster

High availability deployments using Redis Sentinel or Redis Cluster rely on consensus mechanisms to detect failures and perform automatic failovers. A split-brain scenario occurs when a network partition divides the deployment into isolated segments, and multiple nodes assume the role of the master for the same dataset. This leads to conflicting writes and severe data inconsistency.

### 5.1 The Mechanics of Split-Brain

In a typical split-brain scenario, a network partition isolates the current master from the majority of Sentinels or Cluster nodes. The isolated master continues to accept writes from clients connected to its side of the partition. Meanwhile, the majority segment detects the master's absence, elects a new master, and begins accepting writes. When the network partition resolves, the old master is demoted to a replica, and its divergent dataset is overwritten by the new master, resulting in permanent data loss.

### 5.2 Preventing Split-Brain in Sentinel

Redis Sentinel uses a quorum-based approach for failover. However, Sentinel itself does not prevent clients from writing to an isolated master.

To mitigate split-brain in Sentinel deployments, utilize the `min-replicas-to-write` and `min-replicas-max-lag` configuration directives on the master node.

```redis
min-replicas-to-write 1
min-replicas-max-lag 10
```

With this configuration, the master will stop accepting write commands if it cannot communicate with at least one replica within the specified lag time. During a network partition, the isolated master will quickly lose contact with its replicas and become read-only, preventing divergent writes.

### 5.3 Preventing Split-Brain in Redis Cluster

Redis Cluster employs a different architecture, where data is partitioned across multiple master nodes, each with its own replicas. Cluster nodes use a gossip protocol to monitor health and reach consensus on failovers.

Redis Cluster is designed to be resilient against split-brain scenarios through its `cluster-require-full-coverage` and node timeout mechanisms. If a master node becomes isolated from the majority of the cluster, it will eventually detect the partition (based on the `cluster-node-timeout`) and stop accepting writes.

However, a brief window exists between the onset of the partition and the node detecting it, during which divergent writes can occur. To minimize this window, carefully tune the `cluster-node-timeout` parameter. A lower timeout ensures faster detection but increases the risk of false positives during transient network blips.

**Client-Side Mitigation:**
Robust client libraries play a crucial role in handling split-brain scenarios. Clients should be configured to verify the master's status and handle `READONLY` or `CLUSTERDOWN` errors gracefully. Implementing retry logic with exponential backoff can help clients navigate transient state changes during failovers.

## 6. Connection Limits and Exhaustion

Redis and Valkey are designed to handle thousands of concurrent client connections. However, connection resources are finite, and exhausting them can render the service inaccessible to new clients, causing cascading failures across the application stack.

### 6.1 Monitoring Connection Usage

The `INFO clients` command provides essential metrics for monitoring connection health:
- `connected_clients`: The total number of active client connections.
- `rejected_connections`: The number of connections rejected because the `maxclients` limit was reached.
- `blocked_clients`: The number of clients blocked pending an event (e.g., `BLPOP`).

The `maxclients` configuration directive defines the maximum number of concurrent connections the server will accept. By default, this is typically set to 10,000.

### 6.2 Causes of Connection Exhaustion

Connection exhaustion usually stems from application-level issues or misconfigurations:

1. **Connection Leaks**: The most common cause is application code failing to close connections properly after use. Over time, these orphaned connections accumulate until the `maxclients` limit is reached.
2. **Lack of Connection Pooling**: Applications that create a new connection for every request will rapidly exhaust available connections under high load. Implementing connection pooling is mandatory for high-throughput applications.
3. **Slow Operations**: If the server is experiencing high latency (as discussed in Section 2), clients may hold connections open longer while waiting for responses, leading to a buildup of concurrent connections.
4. **DDoS Attacks or Traffic Spikes**: Sudden surges in legitimate traffic or malicious denial-of-service attacks can overwhelm the server's connection capacity.

### 6.3 Resolving Connection Issues

When the `maxclients` limit is reached, the server will reject new connections with an error. Resolving this requires identifying and addressing the root cause.

**Immediate Actions:**
- **Increase `maxclients`**: If the server has sufficient memory and file descriptors, dynamically increase the `maxclients` limit using `CONFIG SET maxclients <new_limit>`. Note that this requires the operating system's open file limit (`ulimit -n`) to be configured appropriately.
- **Identify and Kill Idle Connections**: Use the `CLIENT LIST` command to inspect active connections. Look for connections with high `idle` times. You can manually terminate these using `CLIENT KILL`.

**Long-Term Strategies:**
- **Implement Connection Pooling**: Ensure all client applications utilize robust connection pooling libraries. Configure the pool size appropriately based on the application's concurrency requirements and the server's capacity.
- **Configure Client Timeouts**: Set the `timeout` directive in the Redis/Valkey configuration. This instructs the server to automatically close client connections that have been idle for the specified number of seconds, mitigating the impact of connection leaks.
- **Monitor and Alert**: Establish comprehensive monitoring and alerting for the `connected_clients` and `rejected_connections` metrics to detect connection exhaustion trends before they impact service availability.
- **Review Application Logic**: Conduct code reviews to ensure that connections are properly acquired, utilized, and released in all execution paths, including error handling routines.

## Conclusion

Operating Redis and Valkey in mission-critical environments demands a deep understanding of their internal architecture and failure modes. By mastering the diagnostic techniques and remediation strategies outlined in this guide, operations teams can effectively manage OOM events, mitigate latency spikes, optimize fork operations, ensure robust replication, prevent split-brain scenarios, and maintain healthy connection pools. Proactive monitoring, rigorous configuration management, and a systematic approach to troubleshooting are the cornerstones of a resilient and high-performing data infrastructure.
