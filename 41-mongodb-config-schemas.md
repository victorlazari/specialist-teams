# MongoDB Configuration Schemas and Tuning Recommendations

## 1. Introduction to MongoDB Configuration

MongoDB's configuration file (`mongod.conf`) is the central control mechanism for the database server. It dictates how the database interacts with the operating system, how it manages memory, how it handles network connections, and how it ensures data durability. For tech support operations and database administrators, understanding and tuning these configurations is critical for maintaining high performance, ensuring stability under load, and recovering from worst-case scenarios.

This document provides a comprehensive guide to MongoDB configuration schemas, focusing on production operations, tuning recommendations, and troubleshooting common issues related to configuration settings.

## 2. The `mongod.conf` Schema Overview

The `mongod.conf` file uses YAML format. It is divided into several main sections:

*   **`systemLog`**: Configures logging behavior.
*   **`storage`**: Configures the storage engine, database path, and journaling.
*   **`processManagement`**: Configures how the process runs (e.g., daemonization).
*   **`net`**: Configures network interfaces and ports.
*   **`security`**: Configures authentication and authorization.
*   **`operationProfiling`**: Configures database profiling.
*   **`replication`**: Configures replica set settings.
*   **`sharding`**: Configures sharded cluster settings.

### 2.1. Best Practices for Configuration Management

*   **Version Control**: Always keep your `mongod.conf` files in version control (e.g., Git). This allows you to track changes, revert to previous configurations, and maintain consistency across environments.
*   **Configuration Management Tools**: Use tools like Ansible, Chef, or Puppet to deploy and manage configurations across multiple servers.
*   **Testing**: Never apply configuration changes directly to production. Test them in a staging environment that mirrors production as closely as possible.

## 3. Storage Engine Tuning: WiredTiger

WiredTiger is the default storage engine for MongoDB. Tuning its settings is crucial for optimal performance.

### 3.1. WiredTiger Cache Size (`storage.wiredTiger.engineConfig.cacheSizeGB`)

The WiredTiger cache is where MongoDB stores uncompressed data. By default, MongoDB reserves 50% of (RAM - 1 GB), or 256 MB, whichever is larger.

**Tuning Recommendations:**

*   **Dedicated Server**: If MongoDB is the only major application running on the server, the default setting is usually appropriate.
*   **Co-located Services**: If other memory-intensive applications are running on the same server, you must reduce the `cacheSizeGB` to prevent the OS from swapping MongoDB out of memory.
*   **Working Set Size**: Ideally, your working set (the data and indexes most frequently accessed) should fit entirely within the WiredTiger cache. If page faults are high, consider increasing the cache size or adding more RAM.

**Worst-Case Scenario: OOM Killer**

If the WiredTiger cache is set too high and the OS runs out of memory, the Linux Out-Of-Memory (OOM) killer may terminate the `mongod` process. This results in an ungraceful shutdown and potential data corruption.

**Tech Support Action:**

1.  Check system logs (`/var/log/messages` or `/var/log/syslog`) for OOM killer events.
2.  Review the `mongod.conf` and reduce `cacheSizeGB`.
3.  Ensure sufficient swap space is configured as a safety net, although swapping will severely degrade performance.

### 3.2. Journaling (`storage.journal.enabled`)

Journaling ensures data durability in the event of a crash. MongoDB writes operations to the journal before applying them to the data files.

**Tuning Recommendations:**

*   **Production**: Journaling must **always** be enabled in production environments. Disabling it risks data loss.
*   **Commit Interval (`storage.journal.commitIntervalMs`)**: The default is 100ms. Decreasing this value increases durability but can impact write performance. Increasing it improves write throughput but increases the window of potential data loss during a crash.

**Worst-Case Scenario: Unclean Shutdown without Journaling**

If a server crashes and journaling is disabled, MongoDB cannot guarantee the consistency of the data files. You will likely need to perform a full resync from another replica set member or restore from a backup.

**Tech Support Action:**

1.  Verify journaling is enabled in `mongod.conf`.
2.  If recovering from a crash with journaling enabled, MongoDB will automatically replay the journal upon startup. Monitor the startup logs to ensure successful recovery.

## 4. Replication Settings

Replication provides high availability and data redundancy.

### 4.1. Oplog Size (`replication.oplogSizeMB`)

The oplog (operations log) is a capped collection that records all operations that modify the data. Replica set members use the oplog to replicate changes.

**Tuning Recommendations:**

*   **Default Size**: By default, MongoDB allocates 5% of free disk space for the oplog.
*   **Write-Heavy Workloads**: For applications with high write volumes, the default oplog size may be insufficient. A small oplog can lead to members falling too far behind to catch up, requiring a full resync.
*   **Oplog Window**: The oplog window is the time difference between the oldest and newest entries in the oplog. Aim for an oplog window that covers your longest expected downtime or maintenance window (e.g., 24-48 hours).

**Worst-Case Scenario: Oplog Rollover**

If a secondary member goes offline for an extended period and the primary's oplog rolls over (overwrites the oldest entries needed by the secondary), the secondary becomes "stale" and cannot catch up.

**Tech Support Action:**

1.  Monitor the oplog window using `rs.printReplicationInfo()`.
2.  If a member is stale, you must perform an initial sync.
3.  To prevent future occurrences, increase the oplog size dynamically using the `replSetResizeOplog` command, or update `mongod.conf` and restart the node.

## 5. Network and Connection Pool Limits

Managing connections is vital for preventing resource exhaustion.

### 5.1. Max Incoming Connections (`net.maxIncomingConnections`)

This setting limits the maximum number of simultaneous connections the `mongod` process will accept. The default is 65536.

**Tuning Recommendations:**

*   **Connection Spikes**: If your application experiences sudden spikes in connections, ensure this limit is high enough to accommodate them.
*   **Resource Limits**: Each connection consumes memory and file descriptors. Setting this limit too high can lead to resource exhaustion.
*   **Connection Pooling**: Ensure your application drivers are properly configured to use connection pooling. This reduces the overhead of establishing new connections.

**Worst-Case Scenario: Connection Exhaustion**

If the number of incoming connections reaches the limit, MongoDB will refuse new connections. Applications will experience connection timeouts and failures.

**Tech Support Action:**

1.  Monitor the number of active connections using `db.serverStatus().connections`.
2.  Investigate the application to identify connection leaks or inefficient connection management.
3.  If necessary, increase `net.maxIncomingConnections` in `mongod.conf`, but ensure the OS file descriptor limits (`ulimit -n`) are also increased accordingly.

## 6. System and OS Level Tuning

MongoDB performance is heavily dependent on the underlying operating system.

### 6.1. Transparent Huge Pages (THP)

THP is a Linux memory management feature that attempts to allocate memory in large blocks. However, it often causes performance degradation and memory bloat with database workloads like MongoDB.

**Tuning Recommendations:**

*   **Disable THP**: Always disable THP on servers running MongoDB.

**Tech Support Action:**

1.  Check THP status: `cat /sys/kernel/mm/transparent_hugepage/enabled`.
2.  If enabled, disable it temporarily using `echo never > /sys/kernel/mm/transparent_hugepage/enabled` and `echo never > /sys/kernel/mm/transparent_hugepage/defrag`.
3.  Configure the OS to disable THP permanently on boot (e.g., via GRUB or a systemd service).

### 6.2. File Descriptor Limits (`ulimit`)

MongoDB requires a large number of file descriptors for connections and data files.

**Tuning Recommendations:**

*   **Increase Limits**: The default OS limits are usually too low. Set the soft and hard limits for open files (`nofile`) to at least 64000.

**Tech Support Action:**

1.  Check current limits: `ulimit -n`.
2.  Update `/etc/security/limits.conf` to increase the limits for the user running the `mongod` process.

## 7. Security Configurations

Securing the database is paramount.

### 7.1. Authentication and Authorization (`security.authorization`)

**Tuning Recommendations:**

*   **Enable Authorization**: Always enable Role-Based Access Control (RBAC) by setting `security.authorization: enabled`.
*   **Principle of Least Privilege**: Grant users only the permissions they need to perform their tasks.

### 7.2. Network Binding (`net.bindIp`)

**Tuning Recommendations:**

*   **Restrict Binding**: Never bind MongoDB to `0.0.0.0` (all interfaces) unless absolutely necessary and protected by a strict firewall. Bind only to the specific IP addresses needed for application and replica set communication.

## 8. Example Production `mongod.conf`

```yaml
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log
  logRotate: reopen

storage:
  dbPath: /var/lib/mongo
  journal:
    enabled: true
    commitIntervalMs: 100
  wiredTiger:
    engineConfig:
      cacheSizeGB: 16 # Adjust based on available RAM
      directoryForIndexes: true
    collectionConfig:
      blockCompressor: snappy

processManagement:
  fork: true
  pidFilePath: /var/run/mongodb/mongod.pid
  timeZoneInfo: /usr/share/zoneinfo

net:
  port: 27017
  bindIp: 10.0.0.5,127.0.0.1 # Bind to specific interfaces
  maxIncomingConnections: 20000

security:
  authorization: enabled
  keyFile: /etc/mongodb/pki/keyfile

replication:
  replSetName: rs0
  oplogSizeMB: 50000 # 50GB oplog

operationProfiling:
  mode: slowOp
  slowOpThresholdMs: 100
```

## 9. Troubleshooting Configuration Issues

When configuration issues arise, follow these steps:

1.  **Check the Logs**: The MongoDB logs (`/var/log/mongodb/mongod.log`) are the first place to look. They will often indicate syntax errors in the configuration file or issues starting up due to invalid settings.
2.  **Validate YAML**: Ensure the `mongod.conf` file is valid YAML. Incorrect indentation is a common cause of startup failures.
3.  **Verify Permissions**: Ensure the `mongod` user has read access to the configuration file and read/write access to the database path and log files.
4.  **Test Configuration**: Use the `--config` and `--repair` flags to test the configuration without fully starting the server.

## 10. Conclusion

Properly configuring MongoDB is an ongoing process. As your application grows and workloads change, you must continuously monitor performance and adjust settings accordingly. By understanding the schemas and tuning recommendations outlined in this document, tech support operations can proactively prevent issues, optimize performance, and ensure the stability and reliability of the MongoDB infrastructure.

## 11. Deep Dive: Advanced WiredTiger Tuning

While `cacheSizeGB` is the most commonly adjusted WiredTiger setting, several other parameters can significantly impact performance under specific workloads.

### 11.1. Eviction Tuning

WiredTiger uses background threads to evict pages from the cache to make room for new data. If eviction cannot keep up with the rate of incoming data, performance will degrade severely.

*   **`eviction_target`**: The percentage of cache utilization at which background eviction begins. The default is 80%.
*   **`eviction_trigger`**: The percentage of cache utilization at which application threads are forced to participate in eviction. The default is 95%.

**Tech Support Scenario: High Cache Pressure**

If you observe high cache utilization and application threads are frequently blocking to perform eviction, you may need to adjust these settings. Lowering the `eviction_target` can start eviction earlier, preventing the cache from reaching the `eviction_trigger`. However, this must be done cautiously, as aggressive eviction can also impact performance.

### 11.2. Checkpoint Tuning

WiredTiger periodically creates checkpoints, which are consistent snapshots of the data on disk. Checkpoints are essential for recovery but can cause I/O spikes.

*   **Checkpoint Interval**: By default, MongoDB creates a checkpoint every 60 seconds or when 2GB of journal data has been written.

**Tech Support Scenario: I/O Spikes During Checkpoints**

If you notice significant performance drops or I/O latency spikes every 60 seconds, it may be due to checkpointing. While you cannot disable checkpoints, you can mitigate the impact by ensuring your storage subsystem (e.g., SSDs, NVMe) can handle the burst of write activity.

## 12. Deep Dive: Advanced Replication Tuning

Replication is complex, and tuning it requires a deep understanding of network latency and write patterns.

### 12.1. Write Concern and Read Preference

While not strictly configured in `mongod.conf`, write concern and read preference are critical concepts for tech support to understand when troubleshooting replication issues.

*   **Write Concern (`w`)**: Determines the level of acknowledgment requested from MongoDB for write operations. `w: "majority"` ensures data is written to a majority of replica set members, providing high durability but increasing latency.
*   **Read Preference**: Determines which replica set member receives read operations. Reading from secondaries can distribute load but may result in reading stale data.

**Tech Support Scenario: Replication Lag and Stale Reads**

If applications complain about reading outdated data, check the read preference and replication lag. If replication lag is high, investigate network connectivity, secondary hardware performance, or long-running operations on the primary that are blocking replication.

### 12.2. Election Timeouts

When a primary fails, the replica set holds an election to choose a new primary.

*   **`settings.electionTimeoutMillis`**: The time a secondary waits before calling an election if it hasn't heard from the primary. The default is 10000ms (10 seconds).

**Tech Support Scenario: Frequent Unnecessary Elections**

In environments with unstable networks, a 10-second timeout might lead to frequent, unnecessary elections (flapping). Increasing this timeout can improve stability but increases the time it takes to recover from a genuine primary failure.

## 13. Comprehensive Monitoring Strategy

Configuration tuning is ineffective without robust monitoring. Tech support operations must rely on metrics to identify bottlenecks and validate configuration changes.

### 13.1. Key Metrics to Monitor

*   **Memory Usage**: Track Resident Set Size (RSS), Virtual Memory (VSZ), and WiredTiger cache utilization.
*   **Disk I/O**: Monitor IOPS, disk latency, and disk utilization. High disk latency is a common cause of performance issues.
*   **Network Traffic**: Track bytes in/out and the number of active connections.
*   **Replication Lag**: Monitor the replication window and the lag of each secondary.
*   **Page Faults**: High page faults indicate that the working set does not fit in memory.
*   **Lock Percentage**: Monitor the time spent acquiring locks. High lock contention indicates inefficient queries or schema design issues.

### 13.2. Tools for Monitoring

*   **MongoDB Cloud Manager / Ops Manager**: Provides comprehensive monitoring, alerting, and backup capabilities.
*   **`mongostat` and `mongotop`**: Command-line tools for real-time monitoring of database operations and performance.
*   **Prometheus and Grafana**: Popular open-source tools for collecting and visualizing MongoDB metrics (using the MongoDB Exporter).

## 14. Disaster Recovery and Worst-Case Scenarios

Tech support must be prepared for the worst. Configuration plays a role in both preventing and recovering from disasters.

### 14.1. Data Corruption

Data corruption can occur due to hardware failures, OS bugs, or ungraceful shutdowns (especially if journaling is disabled).

**Recovery Steps:**

1.  **Isolate the Node**: Remove the corrupted node from the replica set to prevent the corruption from spreading.
2.  **Attempt Repair**: Use the `mongod --repair` command. This is a destructive operation and should only be used as a last resort. It attempts to salvage readable data but may discard corrupted documents.
3.  **Restore from Backup**: The safest and most reliable method is to restore the data from a known good backup.
4.  **Resync**: If the node is part of a replica set, clear its data directory and allow it to perform an initial sync from a healthy member.

### 14.2. Accidental Data Deletion (Drop Database/Collection)

If a user accidentally drops a database or collection, replication will immediately propagate the drop to all secondaries.

**Recovery Steps:**

1.  **Delayed Secondary**: If you have configured a delayed secondary (e.g., `slaveDelay: 3600` for a 1-hour delay), you can recover the data from this node before the drop operation is applied.
2.  **Point-in-Time Recovery (PITR)**: If you are using Ops Manager or have continuous backups with oplog archiving, you can restore the database to the exact moment before the drop occurred.

## 15. Conclusion and Final Thoughts

Mastering MongoDB configuration is a continuous journey. The settings outlined in this document provide a solid foundation for production deployments, but every environment is unique. Tech support operations must combine theoretical knowledge of these schemas with practical experience, rigorous monitoring, and a deep understanding of the application's workload. By proactively tuning configurations and preparing for worst-case scenarios, you can ensure that your MongoDB infrastructure remains robust, performant, and resilient.
