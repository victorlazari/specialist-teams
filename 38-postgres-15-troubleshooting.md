# PostgreSQL 15+ Deep Troubleshooting Guide: Production Operations & Tech Support

## 1. Introduction and Context

Welcome to the definitive PostgreSQL 15+ Deep Troubleshooting Guide, designed specifically for Tier 3/4 tech support operations, database reliability engineers (DBREs), and production specialists. In modern enterprise environments, PostgreSQL is often the backbone of mission-critical applications, handling massive datasets, high transaction throughput, and complex analytical workloads. When things go wrong at scale, the consequences are immediate and severe. This guide provides actionable, deep-dive strategies for diagnosing and resolving the most critical PostgreSQL emergencies.

### How This Relates to the Specialist-Teams Repository
For the main `specialist.md` file, it is crucial to understand how this troubleshooting guide integrates with the broader PostgreSQL specialist documentation suite. This file (File 38) serves as the **reactive and diagnostic core** of the repository. While the other 6 files focus on proactive architecture, high availability setup, security hardening, performance tuning baselines, backup/disaster recovery, and routine maintenance, this file is your "break-glass-in-case-of-emergency" manual. It assumes that the proactive measures have either failed or encountered an edge case, requiring immediate, surgical intervention to restore service or prevent catastrophic failure.

---

## 2. Transaction ID (TXID) Wraparound

Transaction ID (TXID) wraparound is arguably the most terrifying scenario for a PostgreSQL administrator. Because PostgreSQL uses Multi-Version Concurrency Control (MVCC), every transaction is assigned a 32-bit integer ID. This allows for approximately 4.2 billion transactions. To prevent the system from running out of IDs and considering old rows as "in the future" (and thus invisible), PostgreSQL must periodically freeze old rows using the `VACUUM` process.

### Symptoms and Warning Signs
The first signs of impending doom are usually found in the PostgreSQL logs:
```text
WARNING: database "production_db" must be vacuumed within 10000000 transactions
HINT: To avoid a database shutdown, execute a database-wide VACUUM in that database.
```
If ignored, the database will eventually reach a hard stop to protect data integrity:
```text
FATAL: database is not accepting commands to avoid wraparound data loss in database "production_db"
```

### Emergency Response: When the Database Stops Accepting Writes
If you hit the `FATAL` error, the database will only accept connections from superusers, and only for the purpose of running `VACUUM`.
1. **Connect as postgres user:** You must connect directly to the affected database.
2. **Run VACUUM FREEZE:** Execute `VACUUM (FREEZE, VERBOSE);` on the specific tables causing the issue, or database-wide if unsure.
3. **Monitor Progress:** In another session, monitor `pg_stat_progress_vacuum` to ensure it is actually making progress and not blocked by locks.

### Preventive Measures for Massive Datasets
Relying on default autovacuum settings for tables in the terabyte range is a recipe for disaster.
- **Aggressive Autovacuuming:** Decrease `autovacuum_vacuum_scale_factor` (e.g., to `0.01` or `0.02`) and increase `autovacuum_vacuum_cost_limit` (e.g., to `2000` or `5000`) to allow autovacuum workers to run faster and more frequently.
- **Targeted Table Settings:** For append-only or highly updated massive tables, set table-level parameters: `ALTER TABLE massive_table SET (autovacuum_freeze_max_age = 1000000000);`
- **Monitoring:** Continuously monitor the age of the oldest database using:
  ```sql
  SELECT datname, age(datfrozenxid) FROM pg_database ORDER BY 2 DESC;
  ```

---

## 3. High CPU and Memory Usage

Resource exhaustion is a common symptom of underlying inefficiencies. In PostgreSQL 15+, diagnosing whether the bottleneck is CPU or Memory requires a systematic approach.

### Diagnosing High CPU
When CPU usage spikes to 100%, the first step is identifying the offending processes.
1. **OS-Level Check:** Use `top` or `htop` to identify the specific PostgreSQL backend PIDs consuming CPU.
2. **Database-Level Correlation:** Map the PID to a query using `pg_stat_activity`:
   ```sql
   SELECT pid, usename, state, query, wait_event_type, wait_event
   FROM pg_stat_activity
   WHERE state = 'active' AND pid = <offending_pid>;
   ```
3. **Common Causes:**
   - **Missing Indexes:** Sequential scans on massive tables will peg the CPU.
   - **Bad Query Plans:** A sudden shift from an index scan to a nested loop join due to outdated statistics.
   - **Spinlock Contention:** High concurrency on specific internal structures (e.g., buffer mapping locks). Use `perf top` on Linux to identify kernel-level spinlocks.

### Memory Issues and the OOM Killer
PostgreSQL relies heavily on the OS page cache. Misconfiguring memory parameters can lead to the Linux Out-Of-Memory (OOM) killer terminating the PostgreSQL postmaster process, resulting in a crash recovery cycle.
- **`shared_buffers`:** Typically set to 25% of total RAM. Setting it too high starves the OS page cache.
- **`work_mem`:** This is allocated *per sort/hash operation*, not per query or per connection. A complex query with multiple sorts can allocate `work_mem` multiple times. If `max_connections` is 1000 and `work_mem` is 100MB, you risk allocating 100GB of RAM.
- **Diagnosing OOM:** Check `/var/log/messages` or `dmesg` for `Out of memory: Killed process <pid> (postgres)`.
- **Huge Pages:** For databases with large `shared_buffers` (e.g., > 8GB), enabling Linux Huge Pages is mandatory in production to reduce Page Table overhead and CPU usage. Configure `vm.nr_hugepages` in `sysctl.conf` and set `huge_pages = on` in `postgresql.conf`.

---

## 4. Slow Queries and Query Plan Degradation

A query that ran in milliseconds yesterday might take minutes today. This "plan degradation" is a classic tech support nightmare.

### Identifying Slow Queries
Enable and utilize `pg_stat_statements`. It is the most critical extension for performance troubleshooting.
```sql
SELECT query, calls, total_exec_time, rows, 100.0 * shared_blks_hit / nullif(shared_blks_hit + shared_blks_read, 0) AS hit_percent
FROM pg_stat_statements
ORDER BY total_exec_time DESC LIMIT 10;
```
For deep, automated logging of slow queries, configure `auto_explain` in `shared_preload_libraries` to capture execution plans of queries exceeding a specific threshold without requiring manual `EXPLAIN`.

### Analyzing Execution Plans
Always use `EXPLAIN (ANALYZE, BUFFERS)` to see actual execution times and memory/disk block usage.
- Look for `Seq Scan` on large tables.
- Look for `External merge Disk` in sort operations, indicating `work_mem` is too small.
- Look for massive discrepancies between `estimated rows` and `actual rows`. This indicates stale statistics.

### Statistics, ANALYZE, and JIT
- **Stale Statistics:** If estimates are wildly off, run `ANALYZE verbose table_name;`. For complex correlations between columns, create Extended Statistics (`CREATE STATISTICS`).
- **JIT Compilation:** PostgreSQL 15+ enables Just-In-Time (JIT) compilation by default. While great for long-running analytical queries, JIT overhead can severely degrade the performance of short, transactional queries. If you see `JIT:` taking up significant time in `EXPLAIN ANALYZE`, consider disabling it globally (`jit = off`) or per-session.
- **Parameter Sniffing:** Prepared statements cache execution plans. If the first execution uses an atypical parameter (e.g., querying a tenant with 1 row vs. a tenant with 1 million rows), the cached plan might be disastrous for subsequent queries. PostgreSQL 12+ improved this, but if issues persist, use `plan_cache_mode = force_custom_plan`.

---

## 5. Lock Contention and Deadlocks

In high-concurrency environments, queries often block each other, leading to a cascading failure where connection pools exhaust and the application grinds to a halt.

### Diagnosing Lock Contention
When the system feels "hung," check for blocking locks immediately:
```sql
SELECT blocked_locks.pid     AS blocked_pid,
       blocked_activity.usename  AS blocked_user,
       blocking_locks.pid     AS blocking_pid,
       blocking_activity.usename AS blocking_user,
       blocked_activity.query    AS blocked_query,
       blocking_activity.query   AS blocking_query
FROM  pg_catalog.pg_locks         blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity  ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks         blocking_locks 
    ON blocking_locks.locktype = blocked_locks.locktype
    AND blocking_locks.database IS NOT DISTINCT FROM blocked_locks.database
    AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
    AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
    AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
    AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
    AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
    AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
    AND blocking_locks.pid != blocked_locks.pid
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;
```

### Resolving Deadlocks and Heavy DDL
- **Deadlocks:** PostgreSQL automatically detects deadlocks (circular dependencies) after `deadlock_timeout` (default 1s) and aborts one of the transactions. Frequent deadlocks indicate an application design flaw (e.g., updating rows in inconsistent orders).
- **DDL Operations:** Commands like `ALTER TABLE`, `CREATE INDEX` (without `CONCURRENTLY`), and `TRUNCATE` require an `AccessExclusiveLock`, blocking all reads and writes.
- **Best Practice:** Never run DDL on a busy production system without setting a `lock_timeout`.
  ```sql
  SET lock_timeout = '2s';
  ALTER TABLE massive_table ADD COLUMN new_col INT;
  ```
  If the lock cannot be acquired in 2 seconds, the statement fails, preventing a massive queue of blocked queries.

---

## 6. Replication Lag and Streaming Replication Issues

High availability relies on streaming replication. When the replica falls behind, you risk data loss during a failover and serve stale data to read-only queries.

### Monitoring Lag
Do not rely solely on `pg_stat_replication.replay_lag`, as it only updates when transactions are committed. Use LSN (Log Sequence Number) math:
```sql
-- On Primary
SELECT client_addr, state, sync_state,
       pg_wal_lsn_diff(pg_current_wal_lsn(), write_lsn) AS write_lag_bytes,
       pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn) AS replay_lag_bytes
FROM pg_stat_replication;
```

### Causes of Replication Lag
1. **Network Bottlenecks:** The primary is generating WAL faster than the network can transmit it to the replica.
2. **Disk I/O on Replica:** The replica must replay WAL sequentially. If the replica has slower disks than the primary, or if it is serving heavy read queries, replay will fall behind.
3. **Long-Running Transactions on Primary:** These can hold back the global xmin, preventing vacuuming and causing bloat, which in turn generates massive WAL.

### Replication Slots and WAL Accumulation
Replication slots ensure the primary does not delete WAL files before the replica has consumed them.
- **The Danger:** If a replica goes offline permanently and the slot is not dropped, the primary will accumulate WAL files indefinitely until the disk fills up, causing a total database crash.
- **The Fix:** Monitor `pg_replication_slots`. If a slot is inactive, drop it immediately: `SELECT pg_drop_replication_slot('slot_name');`.
- **PG 13+ Feature:** Use `max_slot_wal_keep_size` to put a hard limit on how much WAL a slot can retain, sacrificing the replica to save the primary.

### Logical Replication in PG 15+
PostgreSQL 15 introduced row filters and column lists for logical replication. However, conflicts still occur (e.g., inserting a row on the subscriber that already exists).
- **Conflict Resolution:** When logical replication halts due to a conflict, check the subscriber logs. You may need to manually advance the replication origin using `pg_replication_origin_advance()` to skip the offending transaction.

---

## 7. Connection Limits and Connection Pooling

PostgreSQL uses a process-per-connection model. Each connection consumes significant OS resources (typically 5-10MB of RAM just for the process).

### The `max_connections` Problem
Setting `max_connections` to 5000 is a severe anti-pattern. It leads to CPU context-switching nightmares and memory exhaustion. If you see `FATAL: sorry, too many clients already`, increasing `max_connections` is rarely the correct long-term fix.

### Implementing PgBouncer / Odyssey
You must use an external connection pooler.
- **Transaction Pooling:** This is the recommended mode. The pooler assigns a server connection to a client only for the duration of a single transaction. This allows 5000 client connections to be multiplexed over just 100 actual PostgreSQL connections.
- **Session Pooling:** Assigns a server connection for the entire client session. Less efficient but required if the application uses session-level features (like prepared statements, though PgBouncer 1.21+ supports prepared statements in transaction mode).

### Idle in Transaction
A client opens a transaction, runs a query, and then goes to sleep without committing or rolling back. This holds locks and prevents `VACUUM` from cleaning up dead tuples.
- **The Fix:** Set `idle_in_transaction_session_timeout = '5min'` in `postgresql.conf`. PostgreSQL will automatically terminate any session that remains idle in a transaction for longer than 5 minutes.

---

## 8. Disk Space Issues and Bloat

Running out of disk space is a hard crash. Managing bloat and temporary files is a daily operational requirement.

### Identifying Bloat
MVCC means `UPDATE` and `DELETE` operations leave behind "dead tuples." If `autovacuum` cannot keep up, tables and indexes become bloated, consuming excess disk space and slowing down sequential scans.
Use the `pgstattuple` extension for precise bloat analysis:
```sql
CREATE EXTENSION pgstattuple;
SELECT * FROM pgstattuple('massive_table');
```
Look at the `dead_tuple_percent` and `free_percent`.

### Resolving Bloat
- **VACUUM FULL:** Reclaims space to the OS but requires an `AccessExclusiveLock`, blocking all access. Unusable for large production tables.
- **pg_repack / pg_squeeze:** These external tools rebuild the table in the background using triggers, requiring only a brief lock at the end to swap the files. This is the industry standard for zero-downtime bloat removal.
- **Index Bloat:** PostgreSQL 12+ allows `REINDEX INDEX CONCURRENTLY`, which rebuilds the index without blocking writes. Schedule this regularly for heavily updated indexes.

### WAL Directory (`pg_wal`) Filling Up
If `/var/lib/postgresql/15/main/pg_wal` reaches 100% disk usage, the database stops.
- **Causes:** Abandoned replication slots (discussed above) or a failing `archive_command`.
- **Troubleshooting Archiving:** Check `pg_stat_archiver`. If `failed_count` is increasing, your archive script (e.g., pushing to S3 via WAL-G or pgBackRest) is failing. Fix the script or temporarily change `archive_command = '/bin/true'` (WARNING: this breaks point-in-time recovery) to allow the database to clear the WAL backlog.

### Temporary Files
Complex queries with large sorts or hashes that exceed `work_mem` will spill to disk, creating temporary files in `base/pgsql_tmp`.
- **Monitoring:** Set `log_temp_files = 0` to log all temp file creations.
- **Protection:** Set `temp_file_limit` to prevent a single runaway query from consuming all available disk space.

---

## 9. Database Migrations and Huge Datasets

Migrating massive datasets or performing major version upgrades (e.g., PG 11 to PG 15) requires meticulous planning to avoid extended downtime.

### Strategies for Zero-Downtime Migrations
- **pg_upgrade:** The standard tool for in-place upgrades. Use the `--link` option. It uses hard links instead of copying data files, reducing a multi-terabyte upgrade from hours to seconds. However, you cannot easily roll back if the new version fails.
- **Logical Replication:** For near-zero downtime cross-version upgrades. Set up the PG 15 instance as a logical replica of the older version. Let it sync, then perform a quick cutover. This allows for easy rollback (the old primary is still intact) but requires careful handling of sequences and DDL.

### Bulk Loading Data
When loading billions of rows, standard `INSERT` statements are too slow.
1. **Use COPY:** `COPY table_name FROM '/path/to/file.csv' WITH (FORMAT csv);` is orders of magnitude faster.
2. **Drop Indexes and Constraints:** Before a massive bulk load, drop all non-primary key indexes and foreign key constraints. Recreate them afterward. Building an index from scratch on a populated table is much faster than updating the index incrementally for every inserted row.
3. **Increase `maintenance_work_mem`:** Set this to a high value (e.g., 2GB or 4GB) before recreating indexes to speed up the sorting phase of index creation.
4. **Disable WAL (Temporarily):** For unlogged tables or initial data loads where you can afford to restart the load on failure, use `UNLOGGED` tables to bypass WAL writing entirely, then `ALTER TABLE ... SET LOGGED` when finished.

### Partitioning Strategies in PG 15+
For tables exceeding 500GB, declarative partitioning is essential for maintenance.
- **Time-Series Data:** Partition by range (e.g., monthly). This allows you to drop old data instantly using `DROP TABLE partition_name` instead of a massive `DELETE` that causes bloat and WAL explosion.
- **PG 15 Improvements:** PostgreSQL 15 significantly improved partition pruning and the performance of queries accessing multiple partitions. Ensure `enable_partition_pruning = on`.

---

## 10. Advanced Diagnostics: The OS and Hardware Layer

PostgreSQL does not operate in a vacuum. Often, the root cause of a database issue lies in the underlying operating system, storage subsystem, or network configuration. Tech support specialists must be adept at cross-layer troubleshooting.

### Storage I/O Bottlenecks
Storage latency is the silent killer of database performance.
- **iostat:** Use `iostat -x 1` to monitor disk utilization. Pay close attention to `await` (average wait time for I/O requests) and `%util` (percentage of CPU time during which I/O requests were issued). If `%util` is consistently near 100% or `await` is high (e.g., > 10ms for SSDs), your storage is the bottleneck.
- **Filesystem Choice:** XFS or ext4 are recommended. Ensure `noatime` is set in `/etc/fstab` to prevent unnecessary disk writes every time a file is read.
- **EBS Volumes (AWS):** If running on AWS, monitor your EBS burst balance and IOPS limits. A sudden drop in performance often correlates with exhausting your IOPS burst credits.

### Network Latency and Packet Loss
A database might be executing queries in 1ms, but if the network introduces 50ms of latency, the application experiences poor performance.
- **ping and mtr:** Use these tools to check for packet loss and routing issues between the application servers and the database.
- **TCP Retransmissions:** High TCP retransmissions indicate network congestion or faulty hardware. Check with `netstat -s | grep retransmitted`.

### Kernel Parameters (sysctl)
Tuning the Linux kernel is critical for high-performance PostgreSQL deployments.
- **vm.swappiness:** Set to `1` or `10` (never `0` as it can invoke the OOM killer prematurely). You want the OS to prefer dropping page cache over swapping PostgreSQL memory to disk.
- **vm.overcommit_memory:** Set to `2` (strict overcommit) to prevent the OS from promising more memory than it has, which is a primary cause of OOM kills.
- **vm.dirty_background_ratio / vm.dirty_ratio:** Tune these to control how aggressively the OS flushes dirty pages to disk. Lower values provide smoother, more consistent I/O, preventing massive I/O spikes that stall the database.

---

## 11. Security Incidents and Audit Trails

Tech support operations often involve investigating security breaches, unauthorized access, or accidental data deletion.

### Auditing with pgaudit
Standard PostgreSQL logging is insufficient for strict compliance requirements (e.g., HIPAA, PCI-DSS). The `pgaudit` extension provides detailed session and object audit logging.
- **Configuration:** Add `pgaudit` to `shared_preload_libraries`.
- **Granular Logging:** Configure `pgaudit.log = 'write, ddl'` to log all data modifications and schema changes without the overwhelming noise of logging every `SELECT` statement.

### Investigating "Who Dropped the Table?"
If a table is accidentally dropped or truncated, time is of the essence.
1. **Check the Logs:** If `log_statement = 'ddl'` or `pgaudit` is enabled, grep the PostgreSQL logs for the table name.
2. **Identify the User and IP:** The logs will reveal the database user, the client IP address, and the exact timestamp of the destructive command.
3. **Point-in-Time Recovery (PITR):** If the data must be restored, you will need to perform a PITR using your backups (e.g., pgBackRest). You restore the database to the exact second *before* the destructive command was executed.

### Connection Security
- **pg_hba.conf:** Ensure strict IP whitelisting. Never use `trust` authentication in production. Use `scram-sha-256` for password encryption.
- **SSL/TLS:** Enforce encrypted connections by setting `ssl = on` and configuring `hostssl` entries in `pg_hba.conf`.

---

## 12. Conclusion and Best Practices Summary

Troubleshooting PostgreSQL 15+ in a massive production environment requires a combination of deep internal knowledge, systematic diagnostic processes, and a calm demeanor under pressure. 

**Key Takeaways for the Specialist:**
1. **Never Guess, Always Measure:** Rely on `pg_stat_activity`, `pg_stat_statements`, and `EXPLAIN ANALYZE`. Do not make configuration changes based on intuition.
2. **Protect the Primary:** Use connection pooling, set statement timeouts, and aggressively monitor replication slots and WAL accumulation.
3. **Automate Maintenance:** Bloat and TXID wraparound are solved by proactive, aggressive autovacuum tuning and regular index maintenance.
4. **Understand the OS:** PostgreSQL is deeply intertwined with the Linux kernel. Mastery of memory management, I/O subsystems, and network diagnostics is non-negotiable.

By mastering the techniques outlined in this guide, you will be equipped to handle the most severe PostgreSQL emergencies, ensuring high availability, data integrity, and optimal performance for your enterprise applications.

---

## 13. Real-World Case Studies: Post-Mortem Analysis

To solidify the concepts discussed, let's examine three real-world production incidents, the diagnostic steps taken, and the ultimate resolutions.

### Case Study 1: The "Monday Morning" CPU Spike
**Scenario:** Every Monday at 9:00 AM, the primary database CPU spiked to 100%, causing application timeouts. The issue resolved itself by 9:30 AM.
**Diagnosis:**
- `pg_stat_activity` revealed hundreds of connections executing a complex reporting query.
- `EXPLAIN ANALYZE` showed the query was using a sequential scan on a 50GB table.
- Checking `pg_stat_user_tables`, the `last_autoanalyze` timestamp was from Friday.
**Root Cause:** A massive batch job ran over the weekend, inserting millions of rows. The statistics were completely outdated by Monday morning, causing the query planner to choose a sequential scan instead of an index scan.
**Resolution:**
- **Immediate:** Ran a manual `ANALYZE` on the affected table, instantly restoring performance.
- **Long-term:** Adjusted `autovacuum_analyze_scale_factor` from the default `0.1` to `0.02` for that specific table, ensuring statistics were updated more frequently during the weekend batch job.

### Case Study 2: The Silent Disk Filler
**Scenario:** PagerDuty alerted that the primary database disk was at 95% capacity and climbing rapidly.
**Diagnosis:**
- `du -sh /var/lib/postgresql/15/main/*` showed the `pg_wal` directory was consuming 500GB.
- `pg_stat_archiver` showed no failures, meaning WAL files were being successfully archived to S3.
- `pg_replication_slots` revealed an inactive logical replication slot named `debezium_cdc` with a massive `restart_lsn` lag.
**Root Cause:** A downstream Kafka Connect cluster (Debezium) had crashed two days prior. The logical replication slot remained, forcing PostgreSQL to retain all WAL files generated since the crash to ensure Debezium could resume without data loss.
**Resolution:**
- **Immediate:** Executed `SELECT pg_drop_replication_slot('debezium_cdc');`. The database immediately deleted 450GB of old WAL files, averting a crash.
- **Long-term:** Configured `max_slot_wal_keep_size = '100GB'` in `postgresql.conf`. This ensures that if a subscriber goes offline, PostgreSQL will eventually drop the slot and delete the WAL, prioritizing the survival of the primary database over the replication stream.

### Case Study 3: The Unkillable Connection
**Scenario:** A developer accidentally ran a massive `UPDATE` statement without a `WHERE` clause on a critical production table. They panicked and closed their laptop. The application ground to a halt due to lock contention.
**Diagnosis:**
- `pg_stat_activity` showed the `UPDATE` query in an `active` state.
- `pg_locks` showed hundreds of other queries waiting for an `ExclusiveLock` on the table.
**Root Cause:** The developer's session was still active on the server, holding the lock. Closing the laptop severed the TCP connection ungracefully, but the PostgreSQL backend process was unaware and continued executing the massive update.
**Resolution:**
- **Immediate:** Identified the PID of the runaway query using `pg_stat_activity`. Executed `SELECT pg_terminate_backend(<pid>);`. The transaction rolled back, releasing the locks and restoring application access.
- **Long-term:** Implemented `statement_timeout = '30s'` for all interactive user roles. Enforced that all DML operations must be executed through reviewed migration scripts, revoking direct write access from developer accounts in the production environment.
