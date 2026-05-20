# MongoDB Deep Troubleshooting Guide: Production Operations & Tech Support

## 1. Introduction

Welcome to the definitive MongoDB Deep Troubleshooting Guide. This document is engineered specifically for tech support operations, database administrators, and site reliability engineers who are tasked with maintaining, diagnosing, and recovering MongoDB clusters in high-stakes production environments. When a critical database cluster experiences degradation or failure, the cost is measured in lost revenue, degraded user experience, and compromised data integrity. This guide bypasses basic introductory concepts and dives straight into the deep end of production operations, focusing on worst-case scenarios, complex failure modes, and advanced diagnostic techniques.

In modern distributed systems, MongoDB is often the backbone of data persistence. However, its distributed nature—relying on replica sets, sharded clusters, and complex consensus algorithms—introduces unique failure domains. This guide covers the most critical and frequently encountered production issues: replication lag, primary elections and split-brain scenarios, Out-Of-Memory (OOM) events, slow queries, lock contention, connection storms, and disk space exhaustion. Each section provides a deep architectural understanding of the problem, precise diagnostic commands, and step-by-step mitigation and recovery procedures.

---

## 2. Replication Lag: Diagnosis and Mitigation

Replication lag is one of the most insidious issues in a MongoDB replica set. It occurs when secondary nodes fall behind the primary node in applying operations from the oplog (operations log). While a small amount of lag is normal in asynchronous replication, significant lag can lead to stale reads, compromised failover safety, and in severe cases, the secondary falling off the oplog entirely, requiring an expensive initial sync.

### 2.1. Architectural Context

MongoDB replication relies on the primary node recording all data modifications in a capped collection called the `oplog.rs`. Secondary nodes continuously tail this oplog, fetch the operations, and apply them locally. The replication process is multi-threaded, but certain operations (like building indexes in the foreground or massive document updates) can bottleneck the application process on secondaries.

### 2.2. Symptoms and Impact

- **Stale Reads:** Applications using `secondary` or `secondaryPreferred` read preferences may serve outdated data to users.
- **Failover Risk:** If the primary fails, a lagging secondary might be elected, resulting in data rollback if the old primary comes back online.
- **Oplog Window Exhaustion:** If a secondary lags beyond the time window covered by the primary's oplog, it can no longer catch up and transitions to the `RECOVERING` state.

### 2.3. Diagnostic Procedures

To diagnose replication lag, you must first quantify it and then identify the bottleneck.

1.  **Check Replica Set Status:**
    Run `rs.status()` on the primary. Look at the `optime` and `lastHeartbeat` fields for each member.
    ```javascript
    rs.status();
    rs.printSlaveReplicationInfo(); // Deprecated in newer versions, use rs.printSecondaryReplicationInfo()
    ```
    The output will show you the time difference between the primary's last operation and the secondary's last applied operation.

2.  **Analyze the Oplog Window:**
    Determine how much time your oplog covers. If the window is too short, secondaries are at high risk of falling off.
    ```javascript
    db.getReplicationInfo();
    ```
    This command reveals the configured oplog size, the used size, and the time difference between the first and last event in the oplog.

3.  **Identify the Bottleneck:**
    - **Network:** Check for high latency or packet loss between the primary and secondary nodes using standard OS tools (`ping`, `mtr`, `iperf`).
    - **Disk I/O:** Secondaries might be struggling to write data to disk. Use `iostat -x 1` to check for high `%util` or `await` times on the secondary's storage volumes.
    - **CPU:** Check if the replication threads on the secondary are CPU-bound.
    - **Long-Running Operations:** A massive `updateMany` or `deleteMany` operation on the primary can cause a single massive oplog entry that takes a long time to apply on the secondary.

### 2.4. Mitigation and Recovery

- **Immediate Action:** If a secondary is dangerously close to falling off the oplog, you may need to temporarily stop heavy write workloads on the primary to allow the secondary to catch up.
- **Increase Oplog Size:** If the oplog window is consistently too short for your workload, dynamically resize the oplog. In MongoDB 3.6+, this can be done without restarting the node:
  ```javascript
  db.adminCommand({replSetResizeOplog: 1, size: <new_size_in_MB>});
  ```
- **Optimize Writes:** Break down massive bulk operations into smaller batches. Instead of deleting 10 million documents in one query, delete them in batches of 10,000.
- **Hardware Upgrades:** If the secondary is consistently I/O bound, consider upgrading to faster storage (e.g., NVMe SSDs) or increasing provisioned IOPS.

---

## 3. Primary Elections and Split-Brain Scenarios

MongoDB's high availability relies on its automatic failover mechanism. When a primary node becomes unreachable, the remaining nodes hold an election to choose a new primary. However, network partitions can lead to complex election scenarios, including the dreaded "split-brain" where multiple nodes believe they should be primary, or situations where no primary can be elected.

### 3.1. The Election Process

MongoDB uses the Raft consensus algorithm for elections. A node can only be elected primary if it receives votes from a strict majority of the configured members in the replica set. This majority requirement is crucial for preventing split-brain scenarios.

### 3.2. Diagnosing Election Issues

When a cluster loses its primary and fails to elect a new one, the cluster becomes read-only (or completely unavailable depending on read preferences).

1.  **Check the Logs:**
    The MongoDB logs (`mongod.log`) are the primary source of truth for election issues. Look for log entries containing `replSet`, `election`, and `heartbeat`.
    ```bash
    grep -i "election" /var/log/mongodb/mongod.log
    ```
    You will see messages indicating why a node called an election, who voted for whom, and why an election failed (e.g., "not enough votes").

2.  **Verify Network Connectivity:**
    Elections fail most often due to network partitions. Ensure that all nodes can communicate with each other on the MongoDB port (default 27017). Check firewall rules, security groups, and routing tables.

3.  **Check Priority Settings:**
    Nodes with `priority: 0` cannot be elected primary. Ensure that you have enough electable nodes.
    ```javascript
    rs.conf();
    ```

### 3.3. The "Split-Brain" Myth and Reality

In a properly configured MongoDB replica set with an odd number of voting members, a true split-brain (two primaries accepting writes simultaneously) is theoretically impossible due to the strict majority requirement. If a network partition isolates the primary from the majority of the nodes, the primary will step down and become a secondary. The majority partition will elect a new primary.

However, a "pseudo split-brain" can occur from the application's perspective if the application is not handling topology changes correctly. If an application maintains a connection to the old primary and ignores the replica set topology updates, it might attempt to write to the demoted primary, resulting in `NotMaster` errors.

### 3.4. Recovery Strategies

- **Resolve Network Partitions:** The most common fix is to restore network connectivity between the nodes. Once communication is restored, the nodes will automatically resolve their state.
- **Force a Reconfiguration:** In catastrophic scenarios where a majority of nodes are permanently lost (e.g., an entire data center goes offline), you may need to force a reconfiguration of the replica set to allow the surviving minority to elect a primary. **WARNING: This can lead to data loss (rollbacks) if the lost nodes had acknowledged writes that hadn't replicated.**
  ```javascript
  // Connect to a surviving secondary
  cfg = rs.conf();
  // Remove the dead nodes from the cfg.members array
  cfg.members = [cfg.members[0]]; // Example: keep only the first member
  rs.reconfig(cfg, {force: true});
  ```
- **Arbiter Management:** If you are using arbiters to maintain an odd number of votes, ensure the arbiter is placed in a neutral failure domain. If the arbiter goes down along with a data node, you might lose the majority.

---

## 4. Out-Of-Memory (OOM) Events

The Linux Out-Of-Memory (OOM) killer is a kernel mechanism that terminates processes to free up memory when the system is critically low on RAM. MongoDB, being a memory-intensive application, is a frequent target of the OOM killer if not configured correctly.

### 4.1. WiredTiger Memory Management

MongoDB's default storage engine, WiredTiger, uses an internal cache to store uncompressed data and indexes. By default, WiredTiger will use up to 50% of (RAM - 1 GB), or 256 MB, whichever is larger. In addition to the WiredTiger cache, MongoDB uses memory for connection overhead, aggregation pipelines, sorting, and the filesystem cache (which caches compressed data blocks).

### 4.2. Diagnosing OOM Kills

When MongoDB suddenly stops without logging a clean shutdown sequence, an OOM kill is the prime suspect.

1.  **Check System Logs:**
    The MongoDB logs will likely be abruptly truncated. You must check the system logs (e.g., `/var/log/messages`, `/var/log/syslog`, or `dmesg`).
    ```bash
    dmesg -T | grep -i oom
    grep -i "out of memory" /var/log/syslog
    ```
    You will see an entry indicating that the kernel invoked the `oom-killer` and sacrificed the `mongod` process.

2.  **Analyze Memory Usage Trends:**
    Use monitoring tools (Datadog, Prometheus, MongoDB Cloud Manager) to review memory usage leading up to the crash. Look for spikes in resident memory or a steady depletion of available system memory.

### 4.3. Root Causes and Mitigation

- **Oversized WiredTiger Cache:** If you are running other memory-intensive processes on the same server, the default WiredTiger cache size might be too large. Reduce it in the `mongod.conf`:
  ```yaml
  storage:
    wiredTiger:
      engineConfig:
        cacheSizeGB: <smaller_value>
  ```
- **Connection Storms:** Each connection to MongoDB consumes a small amount of memory (typically around 1MB). A sudden spike in connections (a connection storm) can rapidly exhaust available RAM. Implement connection pooling in your application and set a `maxIncomingConnections` limit in MongoDB.
- **Memory-Intensive Queries:** Aggregation pipelines with large `$sort` or `$group` stages that exceed the 100MB memory limit (unless `allowDiskUse` is true) can cause memory spikes. Optimize these queries or ensure `allowDiskUse` is utilized for massive aggregations.
- **Lack of Swap Space:** While running databases on swap is generally bad for performance, having a small amount of swap space can provide a buffer that prevents the OOM killer from immediately terminating the process during transient memory spikes.
- **NUMA Configuration:** On systems with Non-Uniform Memory Access (NUMA), MongoDB can experience memory allocation issues. Ensure MongoDB is started with `numactl --interleave=all`.

---

## 5. Slow Queries and Performance Degradation

Slow queries are the most common cause of application performance degradation. They consume CPU, monopolize disk I/O, and can lead to lock contention, affecting the entire cluster.

### 5.1. The Diagnostic Arsenal

MongoDB provides several tools to identify and analyze slow queries.

1.  **The Database Profiler:**
    The profiler records detailed information about database operations. You can configure it to log all operations, or only those taking longer than a specified threshold.
    ```javascript
    // Enable profiling for operations taking longer than 100ms
    db.setProfilingLevel(1, { slowms: 100 });
    ```
    Query the `system.profile` collection to find the culprits:
    ```javascript
    db.system.profile.find({ millis: { $gt: 500 } }).sort({ millis: -1 }).limit(10);
    ```

2.  **The MongoDB Logs:**
    Even without the profiler enabled, MongoDB logs slow queries (based on the `slowms` threshold) to the `mongod.log` file. Look for lines containing `COMMAND` or `QUERY` with a high execution time appended at the end.

3.  **CurrentOp:**
    To see what is currently running and potentially blocking other operations, use `db.currentOp()`.
    ```javascript
    // Find operations running longer than 3 seconds
    db.currentOp({ "secs_running": { $gt: 3 } });
    ```

### 5.2. Analyzing Query Execution Plans

Once you identify a slow query, you must understand *why* it is slow. The `explain()` method is your primary tool.

```javascript
db.collection.find({ status: "active", category: "electronics" }).explain("executionStats");
```

Pay close attention to the following metrics in the `executionStats` output:
- **`executionTimeMillis`:** Total time taken.
- **`totalDocsExamined`:** The number of documents MongoDB had to read from disk or cache.
- **`totalKeysExamined`:** The number of index entries scanned.
- **`nReturned`:** The number of documents actually returned to the application.

**The Golden Rule:** If `totalDocsExamined` is significantly larger than `nReturned`, the query is inefficient. It is scanning too many documents to find the matching ones.

### 5.3. Optimization Strategies

- **Index Creation:** The most common fix for slow queries is creating appropriate indexes. Ensure your queries are supported by indexes that cover the query predicates, sort criteria, and ideally, the projected fields (a covered query).
- **Compound Indexes:** For queries with multiple predicates, use compound indexes. Remember the ESR (Equality, Sort, Range) rule when ordering fields in a compound index.
- **Query Rewrite:** Sometimes the query itself is poorly structured. Avoid using regular expressions with leading wildcards (e.g., `/.*term/`), as they cannot use indexes efficiently.
- **Data Modeling:** If queries require complex joins (`$lookup`) across massive collections, consider denormalizing your data model to embed related data within a single document.

---

## 6. Lock Contention and Concurrency Issues

MongoDB uses a sophisticated locking mechanism to ensure data consistency during concurrent read and write operations. While WiredTiger uses document-level concurrency control (meaning multiple writes can happen simultaneously on different documents in the same collection), lock contention can still occur at higher levels (global, database, or collection) under specific circumstances.

### 6.1. Understanding Intent Locks

MongoDB uses intent locks (Intent Shared - IS, Intent Exclusive - IX) at the global, database, and collection levels before acquiring finer-grained locks at the document level. This hierarchical locking allows MongoDB to quickly determine if a lower-level lock can be granted.

### 6.2. Diagnosing Lock Contention

High lock contention manifests as increased latency, a pile-up of active connections, and low CPU utilization despite poor performance (because threads are waiting for locks, not doing actual work).

1.  **Server Status:**
    Examine the `globalLock` and `locks` sections of `db.serverStatus()`.
    ```javascript
    db.serverStatus().globalLock;
    db.serverStatus().locks;
    ```
    Look for high numbers in the `activeClients` and `currentQueue` fields. A consistently high `currentQueue` indicates that operations are waiting for locks.

2.  **CurrentOp Analysis:**
    Use `db.currentOp()` to identify operations that are waiting for locks (`waitingForLock: true`) and the operations that are currently holding those locks.

### 6.3. Common Causes of Lock Contention

- **Metadata Operations:** Operations that modify collection metadata, such as creating or dropping indexes, dropping collections, or renaming collections, require exclusive locks at the database or collection level. These operations will block all other reads and writes to that namespace.
- **Massive Bulk Writes:** While WiredTiger handles concurrent writes well, a massive, unoptimized bulk write operation can still cause contention, especially if it triggers frequent page evictions or checkpointing.
- **Long-Running Transactions:** In MongoDB 4.0+, multi-document transactions hold locks on the modified documents until the transaction is committed or aborted. Long-running transactions can severely impact concurrency.

### 6.4. Mitigation Strategies

- **Background Index Builds:** Always build indexes in the background (`{ background: true }` in older versions, or the default behavior in 4.2+) to avoid taking an exclusive lock on the collection.
- **Schedule Maintenance:** Perform DDL operations (dropping collections, renaming) during low-traffic maintenance windows.
- **Optimize Transactions:** Keep multi-document transactions as short as possible. Avoid performing slow operations (like external API calls) within a transaction block.
- **Sharding:** If a single replica set is experiencing insurmountable lock contention due to massive write volume, it may be time to shard the database to distribute the write load across multiple replica sets.

---

## 7. Connection Storms and Resource Exhaustion

A connection storm occurs when an application rapidly opens a massive number of connections to the database. This often happens during application restarts, auto-scaling events, or when a network blip causes existing connections to drop, prompting the application to aggressively reconnect.

### 7.1. The Cost of Connections

Connections are not free. Each connection to a `mongod` instance consumes memory (for thread stacks and buffers) and file descriptors. A severe connection storm can lead to OOM kills, file descriptor exhaustion, and CPU starvation as the database spends all its time managing connection handshakes rather than executing queries.

### 7.2. Diagnosing Connection Issues

1.  **Monitor Connection Counts:**
    Use `db.serverStatus().connections` to monitor the current, available, and total created connections.
    ```javascript
    db.serverStatus().connections;
    ```
    A rapid spike in `current` connections or a high rate of `totalCreated` indicates a problem.

2.  **Check File Descriptors:**
    Ensure the `mongod` process has a sufficient file descriptor limit (ulimit). If the limit is reached, MongoDB will reject new connections.
    ```bash
    cat /proc/$(pidof mongod)/limits | grep "Max open files"
    ```

### 7.3. Mitigation and Prevention

- **Connection Pooling:** This is the most critical defense. Ensure your application drivers are configured to use connection pools. The pool maintains a set of open connections that are reused, preventing the overhead of constantly opening and closing connections.
- **Tune Pool Settings:** Configure the `maxPoolSize` appropriately. A common mistake is setting this value too high. A smaller, fully utilized connection pool is often more efficient than a massive pool that causes context-switching overhead on the database server.
- **Implement Rate Limiting/Backoff:** If the application loses connectivity, it should implement exponential backoff when attempting to reconnect, rather than hammering the database with simultaneous connection requests.
- **Set `maxIncomingConnections`:** Configure the `net.maxIncomingConnections` setting in `mongod.conf` to protect the database from being overwhelmed. If this limit is reached, MongoDB will refuse new connections, which is preferable to crashing due to OOM.

---

## 8. Disk Space Exhaustion

Running out of disk space is a fatal error for any database. When a MongoDB volume reaches 100% capacity, the `mongod` process will typically crash or freeze, leading to immediate downtime.

### 8.1. Causes of Disk Exhaustion

- **Data Growth:** Unchecked organic growth of the dataset.
- **Index Bloat:** Creating too many indexes, or failing to remove unused indexes.
- **Log Files:** Unrotated or excessively verbose MongoDB logs (`mongod.log`).
- **Orphaned Documents:** In sharded clusters, failed chunk migrations can leave behind orphaned documents that consume space.
- **WiredTiger Checkpoints:** While rare, issues with WiredTiger checkpointing can cause the data files to grow unexpectedly.

### 8.2. Diagnostic Steps

1.  **OS Level Checks:**
    Use standard Linux commands to identify disk usage.
    ```bash
    df -h
    du -sh /var/lib/mongodb/* | sort -h
    ```

2.  **Database Level Checks:**
    Use `db.stats()` and `db.collection.stats()` to analyze space usage within MongoDB.
    ```javascript
    db.stats();
    ```
    Pay attention to `dataSize` (uncompressed data), `storageSize` (compressed data on disk), and `indexSize`.

### 8.3. Recovery and Mitigation

- **Immediate Relief:** If the disk is 100% full, you cannot even start MongoDB to delete data. You must free up space at the OS level. Delete old log files, temporary files, or move non-essential files to another volume.
- **Log Rotation:** Ensure proper log rotation is configured using `logrotate` or MongoDB's internal `logRotate` command.
- **Drop Unused Indexes:** Identify and drop indexes that are not being used by queries.
- **Compact Collections:** If you have deleted a massive amount of data, the space on disk is not automatically returned to the OS; it is kept by WiredTiger for future reuse. To reclaim this space to the OS, you must run the `compact` command. **WARNING: `compact` is a blocking operation and should only be run on secondaries, one at a time, or during a maintenance window.**
  ```javascript
  db.runCommand({ compact: "collectionName" });
  ```
- **Storage Auto-Scaling:** In cloud environments, implement alerts for disk usage (e.g., at 80% and 90%) and automate volume resizing to prevent exhaustion.

---

## 9. Conclusion

Troubleshooting MongoDB in a production environment requires a deep understanding of its internal architecture, storage engine mechanics, and distributed consensus protocols. By mastering the diagnostic tools and mitigation strategies outlined in this guide, tech support engineers and database administrators can effectively navigate the most complex failure scenarios, minimize downtime, and ensure the resilience of their data infrastructure. Remember that proactive monitoring, proper capacity planning, and rigorous application design are the best defenses against these critical issues.
