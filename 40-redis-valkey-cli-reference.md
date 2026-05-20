# Comprehensive CLI Reference for redis-cli and valkey-cli

## 1. Introduction

In the fast-paced world of modern infrastructure, in-memory data stores like Redis and its highly compatible fork, Valkey, are the backbone of high-performance applications. While graphical user interfaces and monitoring dashboards provide a high-level overview of system health, the true power of administration, troubleshooting, and incident response lies in the command-line interface. The `redis-cli` and `valkey-cli` tools are indispensable utilities for database administrators, site reliability engineers, and technical support specialists.

This comprehensive reference guide delves deep into the advanced usage of `redis-cli` and `valkey-cli`. It is specifically tailored for production operations, worst-case scenario mitigation, and advanced technical support. By mastering these tools, operators can diagnose complex performance bottlenecks, manipulate data at scale, and ensure the resilience of mission-critical systems. The guide covers everything from basic connectivity and authentication to advanced one-liners for monitoring, data manipulation, and cluster management.

## 2. Core Connectivity and Authentication

Before diving into advanced operations, it is crucial to understand the myriad ways to connect to a Redis or Valkey instance. Production environments often employ complex network topologies, TLS encryption, and robust authentication mechanisms.

### 2.1 Basic Connection Parameters

The most fundamental usage involves specifying the hostname, port, and password.

```bash
# Connect to a local instance on the default port (6379)
redis-cli

# Connect to a remote instance with a specific port and password
redis-cli -h redis.production.internal -p 6380 -a 'SuperSecretPassword123!'

# Connect using a URI (Redis 6.0+)
redis-cli -u redis://username:password@redis.production.internal:6380/0
```

### 2.2 TLS/SSL Connections

In modern deployments, data in transit must be encrypted. Both `redis-cli` and `valkey-cli` support TLS natively.

```bash
# Connect using TLS with a specific certificate and key
redis-cli -h secure-redis.internal -p 6379 --tls \
  --cert /etc/ssl/certs/redis-client.crt \
  --key /etc/ssl/private/redis-client.key \
  --cacert /etc/ssl/certs/ca.crt

# Connect using TLS but skip certificate verification (useful for debugging, dangerous in production)
redis-cli -h secure-redis.internal -p 6379 --tls --insecure
```

### 2.3 Connecting to Unix Sockets

For maximum performance on a local machine, connecting via Unix domain sockets bypasses the TCP/IP stack entirely, reducing latency and CPU overhead.

```bash
# Connect via Unix socket
redis-cli -s /var/run/redis/redis-server.sock
```

## 3. Advanced Operational Modes

The CLI tools offer several specialized modes that transform them from simple query interfaces into powerful diagnostic utilities.

### 3.1 Continuous Stat Mode (`--stat`)

When diagnosing performance issues, you often need a real-time view of server metrics without overwhelming the server with `INFO` commands. The `--stat` mode provides a rolling display of key metrics.

```bash
# Display continuous statistics every second
redis-cli -h production-db --stat
```

**Output Explanation:**
- `keys`: Total number of keys across all databases.
- `mem`: Total memory allocated by Redis/Valkey.
- `clients`: Number of connected clients.
- `blocked`: Number of clients blocked on list operations (e.g., `BLPOP`).
- `requests`: Number of commands processed per second.
- `connections`: Number of new connections accepted per second.

### 3.2 Latency Monitoring Mode (`--latency`, `--latency-history`, `--latency-dist`)

Latency is the enemy of in-memory databases. The CLI provides built-in tools to measure and visualize latency from the client's perspective.

```bash
# Measure continuous latency
redis-cli -h production-db --latency

# Measure latency and record history over time (useful for spotting intermittent spikes)
redis-cli -h production-db --latency-history

# Display a latency distribution spectrum (requires Redis 2.8.13+)
redis-cli -h production-db --latency-dist
```

### 3.3 Big Keys and Hot Keys Analysis

Identifying keys that consume excessive memory or receive a disproportionate amount of traffic is critical for capacity planning and performance tuning.

```bash
# Scan the entire keyspace for large keys (memory intensive)
redis-cli -h production-db --bigkeys

# Scan the keyspace for frequently accessed keys (requires maxmemory-policy to be an LFU variant)
redis-cli -h production-db --hotkeys
```

**Warning:** The `--bigkeys` and `--hotkeys` commands use the `SCAN` command under the hood, which is generally safe for production. However, on extremely large datasets, they can still consume significant CPU and network bandwidth. Run them during off-peak hours if possible.

### 3.4 Monitor Mode (`MONITOR`)

The `MONITOR` command streams every command processed by the server in real-time. This is invaluable for debugging application behavior but comes with a severe performance penalty.

```bash
# Stream all commands (use with extreme caution in production)
redis-cli -h production-db MONITOR

# Filter the monitor output for specific commands or keys using grep
redis-cli -h production-db MONITOR | grep -i "user:12345"
```

**Production Rule:** Never leave a `MONITOR` session running indefinitely on a high-throughput production server. It can halve the server's throughput and consume massive amounts of memory for output buffers.

## 4. Advanced One-Liners for Data Manipulation

Technical support and operations teams frequently need to perform bulk data manipulation, cleanup, or migration tasks. The following one-liners leverage the power of Unix pipes and the CLI's non-interactive mode.

### 4.1 Bulk Deletion by Pattern

The `KEYS` command is dangerous in production because it blocks the single-threaded event loop. The correct approach is to use `SCAN` combined with `xargs` and `DEL` or `UNLINK`.

```bash
# BAD: Blocking the server
# redis-cli KEYS "session:*" | xargs redis-cli DEL

# GOOD: Non-blocking bulk deletion using SCAN and UNLINK (Redis 4.0+)
redis-cli --scan --pattern "session:*" | xargs -L 1000 redis-cli UNLINK
```

### 4.2 Exporting and Importing Data

While `BGSAVE` and copying the RDB file is the standard way to backup data, sometimes you need to export specific keys or migrate data between instances without file system access.

```bash
# Export keys matching a pattern to a local file
redis-cli --scan --pattern "config:*" | while read key; do
  echo "SET $key \"$(redis-cli GET $key)\"" >> export.txt
done

# Import data from a file using pipe mode (highly efficient)
cat data.txt | redis-cli --pipe
```

### 4.3 Mass Insertion with `--pipe`

When seeding a database or restoring a massive dataset, sending commands one by one is too slow due to network round-trip times. The `--pipe` mode is designed for maximum throughput.

```bash
# Generate a file with Redis protocol commands and pipe it
# (Assuming generate_data.py outputs valid Redis protocol)
python generate_data.py | redis-cli --pipe
```

### 4.4 Finding and Analyzing TTLs

Identifying keys that lack an expiration time (TTL) is a common task when debugging memory leaks.

```bash
# Find keys without a TTL (returns -1)
redis-cli --scan | while read key; do
  ttl=$(redis-cli TTL $key)
  if [ $ttl -eq -1 ]; then
    echo "No TTL: $key"
  fi
done > keys_without_ttl.txt
```

## 5. Cluster Management and Troubleshooting

Redis and Valkey Clusters introduce significant complexity. The CLI provides robust tools for managing nodes, slots, and rebalancing.

### 5.1 Cluster Diagnostics

When a cluster is in a degraded state, the first step is to gather diagnostic information.

```bash
# Check the overall health of the cluster
redis-cli -c -h cluster-node-1 cluster info

# List all nodes, their roles, and slot assignments
redis-cli -c -h cluster-node-1 cluster nodes

# Use the built-in cluster check utility
redis-cli --cluster check cluster-node-1:6379
```

### 5.2 Resharding and Rebalancing

Adding or removing nodes requires moving hash slots between instances.

```bash
# Interactively reshard the cluster
redis-cli --cluster reshard cluster-node-1:6379

# Automatically rebalance slots across all nodes to achieve even distribution
redis-cli --cluster rebalance cluster-node-1:6379 --cluster-use-empty-masters
```

### 5.3 Fixing a Broken Cluster

In worst-case scenarios, such as multiple node failures, the cluster may lose hash slots and enter a failed state. The `fix` command attempts to repair the cluster topology.

```bash
# Attempt to fix a broken cluster (use with caution, may result in data loss for orphaned slots)
redis-cli --cluster fix cluster-node-1:6379
```

## 6. Memory Management and Optimization

Memory is the most constrained resource in an in-memory database. Effective memory management is crucial for stability.

### 6.1 Analyzing Memory Usage

The `MEMORY` command family provides deep insights into how memory is allocated.

```bash
# Get a high-level overview of memory usage
redis-cli MEMORY STATS

# Analyze the memory usage of a specific key
redis-cli MEMORY USAGE "user:profile:12345"

# Force the allocator to release memory back to the OS (jemalloc only)
redis-cli MEMORY PURGE
```

### 6.2 Eviction and Maxmemory Policies

When the server reaches its `maxmemory` limit, it must evict keys to make room for new writes. Monitoring eviction rates is essential.

```bash
# Check the current maxmemory configuration and policy
redis-cli CONFIG GET maxmemory
redis-cli CONFIG GET maxmemory-policy

# Monitor the eviction rate in real-time
redis-cli INFO STATS | grep evicted_keys
```

## 7. Worst-Case Scenarios and Incident Response

When things go wrong, operators need fast, reliable commands to mitigate impact and restore service.

### 7.1 The "OOM (Out of Memory)" Scenario

If the server is rejecting writes due to OOM, you must quickly identify the cause and free up memory.

1.  **Identify Big Keys:** Run `redis-cli --bigkeys` (if the server is responsive enough).
2.  **Check Client Buffers:** A slow client can cause output buffers to consume massive amounts of memory.
    ```bash
    redis-cli CLIENT LIST | awk '{print $1, $2, $12}' | sort -k3 -nr | head -n 10
    ```
3.  **Kill Problematic Clients:**
    ```bash
    redis-cli CLIENT KILL id <client_id>
    ```
4.  **Emergency Flush (Extreme Caution):** If the data is entirely ephemeral (e.g., a cache) and the service is completely down, flushing the database may be the only option.
    ```bash
    redis-cli FLUSHALL ASYNC
    ```

### 7.2 The "High CPU Usage" Scenario

If the Redis/Valkey process is pegged at 100% CPU, it is likely executing expensive commands or stuck in a tight loop.

1.  **Check the Slow Log:** The slow log records commands that exceed a specified execution time.
    ```bash
    # Get the top 10 slowest commands
    redis-cli SLOWLOG GET 10
    ```
2.  **Identify Expensive Commands:** Look for `KEYS`, `SMEMBERS` on large sets, or complex Lua scripts.
3.  **Kill Long-Running Scripts:** If a Lua script is stuck in an infinite loop, you can terminate it.
    ```bash
    redis-cli SCRIPT KILL
    ```

### 7.3 The "Network Partition / Split Brain" Scenario

In a Sentinel or Cluster deployment, network partitions can lead to split-brain scenarios where multiple masters exist.

1.  **Verify Replication Status:**
    ```bash
    redis-cli INFO REPLICATION
    ```
2.  **Check Sentinel Logs:** Connect to the Sentinel instances and review their logs for failover events.
3.  **Force a Failover (if necessary):**
    ```bash
    redis-cli -p 26379 SENTINEL FAILOVER mymaster
    ```

## 8. Scripting and Automation

The CLI tools are highly scriptable, allowing operators to build custom automation workflows.

### 8.1 Executing Lua Scripts

Lua scripting allows you to execute complex logic atomically on the server.

```bash
# Execute a simple Lua script
redis-cli --eval myscript.lua key1 key2 , arg1 arg2

# Example Lua script (myscript.lua) to atomically increment and set TTL:
# local current = redis.call('INCR', KEYS[1])
# if current == 1 then
#     redis.call('EXPIRE', KEYS[1], ARGV[1])
# end
# return current
```

### 8.2 Using `redis-cli` in Shell Scripts

When using `redis-cli` in shell scripts, it is important to handle errors and format the output correctly.

```bash
# Check if the server is alive before executing commands
if redis-cli PING | grep -q "PONG"; then
  echo "Server is up."
  # Execute commands
else
  echo "Server is down!"
  exit 1
fi

# Extract a specific value from INFO output
used_memory=$(redis-cli INFO MEMORY | grep "used_memory:" | cut -d: -f2 | tr -d '\r')
echo "Used Memory: $used_memory bytes"
```

## 9. Security and Auditing

Securing the database is paramount. The CLI provides tools to audit configurations and manage access control.

### 9.1 Auditing ACLs (Access Control Lists)

Redis 6.0 introduced ACLs, allowing fine-grained control over which users can execute specific commands.

```bash
# List all ACL users
redis-cli ACL LIST

# Check the effective permissions of a specific user
redis-cli ACL GETUSER default

# Generate a new secure password
redis-cli ACL GENPASS
```

### 9.2 Auditing Configuration

Regularly auditing the server configuration ensures that security best practices are enforced.

```bash
# Check if dangerous commands are renamed or disabled
redis-cli CONFIG GET rename-command

# Verify that requirepass is set
redis-cli CONFIG GET requirepass

# Ensure that the server is bound to the correct interfaces
redis-cli CONFIG GET bind
```

## 10. Conclusion

The `redis-cli` and `valkey-cli` tools are far more than simple query interfaces; they are comprehensive diagnostic and operational suites. By mastering the advanced modes, one-liners, and troubleshooting techniques outlined in this reference, technical support specialists and system administrators can confidently manage large-scale deployments, rapidly mitigate incidents, and ensure the optimal performance of their in-memory data stores. Continuous practice and deep understanding of these tools are essential for any professional responsible for the reliability of Redis or Valkey infrastructure.

---
*This document is part of the specialist-teams repository, providing deep technical guidance for operations and support teams.*

## 11. Deep Dive: Advanced Data Structures and CLI Manipulation

Beyond simple strings and hashes, Redis and Valkey offer complex data structures like HyperLogLogs, Geospatial indexes, and Streams. Managing these via the CLI requires specific techniques.

### 11.1 Managing Streams for Event Sourcing

Redis Streams are powerful append-only logs. Operations teams often need to inspect streams, manage consumer groups, and trim old data to prevent unbounded memory growth.

```bash
# Inspect the latest entries in a stream
redis-cli XREVRANGE mystream + - COUNT 10

# Check the status of consumer groups for a stream
redis-cli XINFO GROUPS mystream

# Identify pending messages that haven't been acknowledged (dead-letter queue analysis)
redis-cli XPENDING mystream mygroup

# Trim a stream to a specific length to free memory (approximate trimming is more efficient)
redis-cli XTRIM mystream MAXLEN ~ 100000
```

### 11.2 Geospatial Data Operations

For applications relying on location data, the CLI can be used to query and manipulate geospatial indexes.

```bash
# Add coordinates to a geospatial index
redis-cli GEOADD fleet:vehicles -122.4194 37.7749 "truck-001" -122.4089 37.7833 "truck-002"

# Find all vehicles within a 5km radius of a specific point
redis-cli GEORADIUS fleet:vehicles -122.41 37.78 5 km WITHDIST WITHCOORD
```

### 11.3 HyperLogLog for Cardinality Estimation

HyperLogLogs provide highly efficient cardinality estimation (e.g., counting unique visitors) with a tiny memory footprint.

```bash
# Add elements to a HyperLogLog
redis-cli PFADD unique:visitors:2023-10-27 "user1" "user2" "user3"

# Get the estimated count
redis-cli PFCOUNT unique:visitors:2023-10-27

# Merge multiple HyperLogLogs (e.g., to get weekly unique visitors from daily logs)
redis-cli PFMERGE unique:visitors:week43 unique:visitors:2023-10-23 unique:visitors:2023-10-24
```

## 12. Advanced Replication Troubleshooting

Replication is the foundation of high availability. When replication lags or breaks, the CLI is the primary tool for diagnosis.

### 12.1 Analyzing Replication Lag

Replication lag can lead to stale reads on replicas or data loss during a failover.

```bash
# On the master, check the offset of all connected replicas
redis-cli INFO REPLICATION | grep "slave"

# On the replica, check the master link status and the replication offset
redis-cli INFO REPLICATION | grep -E "master_link_status|master_repl_offset|slave_repl_offset"
```
*The difference between `master_repl_offset` and `slave_repl_offset` indicates the replication lag in bytes.*

### 12.2 Forcing Full Resynchronization

Sometimes, partial resynchronization (PSYNC) fails, and a full resynchronization is required. This can be network-intensive.

```bash
# Monitor the master's logs during a resync
redis-cli MONITOR | grep "SYNC"

# Adjust the replication backlog size to prevent frequent full resyncs
redis-cli CONFIG SET repl-backlog-size 512mb
```

## 13. Persistence: RDB and AOF Deep Dive

Understanding how data is persisted to disk is critical for disaster recovery.

### 13.1 Managing RDB Snapshots

RDB snapshots provide point-in-time backups.

```bash
# Trigger a background save manually
redis-cli BGSAVE

# Check the status of the last save operation
redis-cli INFO PERSISTENCE | grep "rdb_last_bgsave_status"

# Monitor the progress of an ongoing BGSAVE
redis-cli INFO PERSISTENCE | grep "rdb_bgsave_in_progress"
```

### 13.2 Managing Append-Only Files (AOF)

AOF provides higher durability by logging every write operation.

```bash
# Trigger an AOF rewrite to compact the file
redis-cli BGREWRITEAOF

# Check the status of the AOF rewrite
redis-cli INFO PERSISTENCE | grep "aof_rewrite_in_progress"

# Temporarily disable AOF (e.g., during a massive data import)
redis-cli CONFIG SET appendonly no
```

## 14. Customizing the CLI Experience

The `redis-cli` can be customized to improve operator efficiency.

### 14.1 Using `redis-cli` Hints

By default, `redis-cli` provides command hints. This can be toggled.

```bash
# Start CLI without hints (useful for clean copy-pasting)
redis-cli --no-auth-warning
```

### 14.2 Output Formatting

The CLI supports different output formats, which is crucial when integrating with other tools.

```bash
# Raw output (no formatting, useful for piping to other commands)
redis-cli --raw GET mykey

# CSV output (useful for exporting data to spreadsheets)
redis-cli --csv LRANGE mylist 0 -1
```

## 15. The Role of this Document in the Specialist Teams Repository

This document, `40-redis-valkey-cli-reference.md`, serves as the definitive operational manual for in-memory data store management within the `specialist-teams` repository. It is designed to be used in conjunction with the other specialized guides:

1.  **Architecture and Topology:** While other documents define *how* the clusters are built, this document defines *how to operate* them.
2.  **Incident Response Playbooks:** This reference provides the specific command-line invocations required to execute the mitigation strategies outlined in the high-level playbooks.
3.  **Monitoring and Alerting:** The metrics discussed here (e.g., latency, memory usage, replication lag) form the basis of the automated alerting rules defined elsewhere in the repository.
4.  **Security Policies:** The ACL and configuration auditing commands detailed in Section 9 are the practical implementation of the organization's security standards.
5.  **Disaster Recovery:** The persistence and replication troubleshooting sections are critical components of the DR strategy.
6.  **Performance Tuning:** The advanced profiling tools (`--latency`, `--stat`, `SLOWLOG`) are essential for the continuous performance optimization efforts described in the engineering guidelines.

By centralizing these advanced CLI techniques, we ensure that all technical support specialists and SREs have immediate access to the tools they need to maintain the highest levels of reliability and performance.
