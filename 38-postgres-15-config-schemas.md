# PostgreSQL 15+ Configuration Schemas and Tuning Recommendations

## 1. Introduction to PostgreSQL 15+ Configuration

In the realm of enterprise-grade database management, PostgreSQL 15 and newer versions introduce significant enhancements in logical replication, sorting performance, and compression. However, out-of-the-box configurations are notoriously conservative, designed to run on minimal hardware without exhausting system resources. For production operations dealing with huge datasets, high-concurrency workloads, and mission-critical tech support scenarios, deep tuning of `postgresql.conf` and `pg_hba.conf` is absolutely mandatory.

This document serves as a comprehensive specialist guide for configuring, tuning, and troubleshooting PostgreSQL 15+ environments. It is specifically tailored for database administrators (DBAs), site reliability engineers (SREs), and tech support teams handling worst-case scenarios, massive database migrations, and performance bottlenecks.

---

## 2. Core Memory Settings

Memory allocation is the most critical aspect of PostgreSQL tuning. Misconfigured memory settings can lead to out-of-memory (OOM) kills, excessive disk I/O, and catastrophic performance degradation during peak loads.

### 2.1 `shared_buffers`

The `shared_buffers` parameter determines how much memory PostgreSQL uses for caching data. Unlike other databases that bypass the OS cache (using Direct I/O), PostgreSQL relies heavily on the operating system's page cache.

*   **Recommendation:** Set to **25% to 40%** of total system RAM.
*   **Production Context:** For a dedicated database server with 256GB of RAM, setting `shared_buffers = 64GB` is a standard starting point. Going beyond 40% rarely yields benefits because the OS needs memory for its own cache, which PostgreSQL leverages for read-ahead operations.
*   **Worst-Case Scenario:** If `shared_buffers` is set too high (e.g., 80% of RAM), the OS will lack memory for its page cache and connection handling, leading to swapping and eventual OOM crashes.
*   **Tech Support Tip:** Use the `pg_buffercache` extension to inspect the contents of shared buffers. If the cache hit ratio drops below 95% during normal operations, consider increasing this value or investigating poorly optimized queries that are churning the cache.

### 2.2 `work_mem`

The `work_mem` parameter specifies the amount of memory to be used by internal sort operations and hash tables before writing to temporary disk files.

*   **Recommendation:** Set dynamically based on workload, typically between **16MB and 64MB** globally, but increased per-session for heavy queries.
*   **Production Context:** `work_mem` is allocated *per operation*, not per query or per connection. A complex query with multiple sorts and hash joins might allocate `work_mem` several times.
*   **Worst-Case Scenario:** Setting `work_mem` globally to a massive value (e.g., 1GB) on a system with `max_connections = 1000` can theoretically demand 1TB of RAM, instantly triggering the Linux OOM killer.
*   **Tech Support Tip:** Monitor temporary file generation by setting `log_temp_files = 0`. If you see frequent temporary files being written, increase `work_mem` for specific reporting roles using `ALTER ROLE reporting_user SET work_mem = '512MB';`.

### 2.3 `maintenance_work_mem`

This parameter dictates the maximum amount of memory used for maintenance operations, such as `VACUUM`, `CREATE INDEX`, and `ALTER TABLE ADD FOREIGN KEY`.

*   **Recommendation:** Set to **10% to 15%** of total RAM, up to a maximum of **2GB to 4GB**.
*   **Production Context:** High values significantly speed up index creation and vacuuming. In PostgreSQL 15, parallel index creation can utilize multiple workers, each consuming a portion of this memory.
*   **Database Migrations:** During bulk data loads or migrations, temporarily increase this to a very high value (e.g., `8GB`) to accelerate the recreation of indexes after the data copy phase.

### 2.4 `effective_cache_size`

This is not an allocation parameter; it is a hint to the query planner about how much memory is available for disk caching by the operating system and within PostgreSQL itself.

*   **Recommendation:** Set to **50% to 75%** of total system RAM.
*   **Production Context:** If set to 75% of a 256GB machine (`192GB`), the planner is more likely to choose index scans over sequential scans, knowing that the indexes are likely to reside in memory.
*   **Tech Support Tip:** If the database is unexpectedly choosing sequential scans for queries that should use indexes, verify that `effective_cache_size` accurately reflects the available memory.

---

## 3. Write-Ahead Logging (WAL) and Checkpoints

WAL configuration dictates how PostgreSQL ensures data durability and crash recovery. Tuning WAL is essential for write-heavy workloads and huge datasets.

### 3.1 `wal_level`

Determines how much information is written to the WAL.

*   **Recommendation:** Set to `replica` for standard high-availability setups, or `logical` if logical replication is required.
*   **Production Context:** PostgreSQL 15 defaults to `replica`. Changing to `logical` increases WAL volume by 20-30% but is mandatory for Change Data Capture (CDC) tools like Debezium or native logical replication.

### 3.2 `max_wal_size` and `min_wal_size`

These parameters control the size of the WAL on disk before a checkpoint is forced.

*   **Recommendation:** Set `max_wal_size` to **16GB to 64GB** for write-heavy systems. Set `min_wal_size` to **4GB to 8GB**.
*   **Production Context:** The default `max_wal_size` of 1GB is vastly insufficient for modern applications. A small `max_wal_size` forces frequent checkpoints, causing massive I/O spikes.
*   **Worst-Case Scenario:** During a massive data migration, if `max_wal_size` is too small, the system will spend all its I/O capacity writing checkpoints, slowing the migration to a crawl.

### 3.3 `checkpoint_timeout` and `checkpoint_completion_target`

*   **Recommendation:** Set `checkpoint_timeout` to **15min to 30min**. Set `checkpoint_completion_target` to **0.9**.
*   **Production Context:** The goal is to spread the I/O load of a checkpoint over a longer period. A target of 0.9 means PostgreSQL will aim to complete the checkpoint when 90% of the time or WAL volume has elapsed.
*   **Tech Support Tip:** If you see "checkpoints are occurring too frequently" in the PostgreSQL logs, you must increase `max_wal_size` and `checkpoint_timeout`.

### 3.4 `wal_compression`

*   **Recommendation:** Set to `lz4` or `zstd` (introduced in PG 15).
*   **Production Context:** Enabling WAL compression reduces disk I/O and network bandwidth for replication at the cost of a slight CPU overhead. `zstd` offers an excellent balance of compression ratio and speed.

---

## 4. Connection Management

Handling thousands of concurrent connections requires architectural decisions beyond simple parameter tweaks.

### 4.1 `max_connections`

*   **Recommendation:** Keep relatively low, typically **200 to 500**.
*   **Production Context:** PostgreSQL uses a process-per-connection model. Each connection consumes a significant amount of memory (typically 5-10MB just for the process overhead). Setting `max_connections = 5000` will destroy performance due to context switching and memory exhaustion.
*   **Tech Support Tip:** For high-concurrency environments, **always use a connection pooler** like PgBouncer or Odyssey. Configure PgBouncer to handle 10,000 client connections while maintaining only 200 actual database connections.

### 4.2 `superuser_reserved_connections`

*   **Recommendation:** Set to **5 to 10**.
*   **Worst-Case Scenario:** When the database is completely overwhelmed and `max_connections` is reached, standard users cannot connect. This parameter ensures that DBAs and tech support can still log in as a superuser to diagnose the issue, kill rogue queries, or restart services.

---

## 5. pg_hba.conf: Client Authentication and Security

The `pg_hba.conf` (Host-Based Authentication) file controls which hosts are allowed to connect, how clients are authenticated, and which PostgreSQL user names they can use.

### 5.1 Configuration Schema and Best Practices

A typical production `pg_hba.conf` should follow the principle of least privilege.

```text
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# Local administrative access
local   all             postgres                                peer

# Application subnets (using SCRAM-SHA-256 for secure password hashing)
host    app_db          app_user        10.0.1.0/24             scram-sha-256
host    app_db          app_user        10.0.2.0/24             scram-sha-256

# Read-only replicas (Replication connections)
host    replication     repl_user       10.0.3.50/32            scram-sha-256
host    replication     repl_user       10.0.3.51/32            scram-sha-256

# Tech Support / DBA VPN subnet (Require MFA or strict password)
hostssl all             dba_team        192.168.100.0/24        scram-sha-256

# Reject all other connections explicitly (Optional, as default is reject, but good for auditing)
host    all             all             0.0.0.0/0               reject
```

### 5.2 Tech Support Operations for pg_hba.conf

*   **Reloading Configuration:** Never restart the database to apply `pg_hba.conf` changes. Use `SELECT pg_reload_conf();` or send a SIGHUP signal to the postmaster process.
*   **Troubleshooting Connection Failures:** If an application cannot connect, check the PostgreSQL logs. Errors like `no pg_hba.conf entry for host...` explicitly indicate a missing or incorrect rule. Ensure that the `ADDRESS` CIDR block accurately covers the client's IP.
*   **Migration to SCRAM-SHA-256:** PostgreSQL 14+ defaults to `scram-sha-256`. If migrating from older versions using `md5`, ensure all client drivers support SCRAM before enforcing it in `pg_hba.conf`.

---

## 6. Autovacuum Tuning for Huge Datasets

Autovacuum is essential for reclaiming storage occupied by dead tuples and updating statistics. In massive databases, the default autovacuum settings are far too passive.

### 6.1 Aggressive Autovacuum Settings

*   `autovacuum_max_workers`: Increase from 3 to **5 or 8** depending on CPU cores.
*   `autovacuum_naptime`: Decrease from 1min to **15s**.
*   `autovacuum_vacuum_scale_factor`: Decrease from 0.2 (20%) to **0.05 (5%)** or even **0.01 (1%)** for tables with billions of rows.
*   `autovacuum_analyze_scale_factor`: Decrease from 0.1 (10%) to **0.02 (2%)**.
*   `autovacuum_vacuum_cost_limit`: Increase from 200 to **2000 or 5000** to allow autovacuum to run faster without throttling itself too aggressively.

### 6.2 Worst-Case Scenario: Transaction ID (TXID) Wraparound

If autovacuum fails to keep up with a high-throughput write workload, the database may approach Transaction ID Wraparound.

*   **Symptoms:** Warnings in the log: `WARNING: database "mydb" must be vacuumed within 1000000 transactions`.
*   **Catastrophic Failure:** If ignored, PostgreSQL will shut down and refuse to start to prevent data corruption.
*   **Tech Support Action:** If the database shuts down due to wraparound, you must start it in single-user mode and run a manual `VACUUM FREEZE`. This requires significant downtime. Proactive tuning of autovacuum is the only way to prevent this.

---

## 7. Database Migrations and Bulk Loading

When migrating huge datasets into PostgreSQL 15+, standard configurations will cause the migration to take days instead of hours.

### 7.1 Pre-Migration Tuning (Temporary Settings)

Before starting a massive `pg_restore` or `COPY` operation, apply these temporary settings:

1.  `shared_buffers`: Increase slightly if memory allows.
2.  `maintenance_work_mem`: Maximize (e.g., 4GB to 16GB).
3.  `max_wal_size`: Increase to 64GB or 128GB to prevent checkpoint throttling.
4.  `checkpoint_timeout`: Increase to 60min.
5.  `wal_level`: Set to `minimal` (ONLY if you do not need replication or point-in-time recovery during the load).
6.  `archive_mode`: Set to `off` (temporarily disable WAL archiving).
7.  `autovacuum`: Set to `off` (temporarily disable autovacuum to prevent it from interfering with the load).

### 7.2 Post-Migration Actions

**CRITICAL:** Once the data load is complete, you MUST:
1.  Revert all temporary settings to their production values.
2.  Restart PostgreSQL (if `wal_level` or `archive_mode` were changed).
3.  Run a manual `VACUUM ANALYZE` on the entire database to generate statistics for the query planner. Without this, queries will perform abysmally.

---

## 8. Tech Support Operations and Troubleshooting Workflows

For SREs and tech support teams, diagnosing performance issues requires a systematic approach.

### 8.1 Identifying Slow Queries

1.  **Enable `pg_stat_statements`:** This extension is mandatory for production. It records execution statistics of all SQL statements.
    *   Add to `shared_preload_libraries = 'pg_stat_statements'`.
    *   Set `pg_stat_statements.track = all`.
2.  **Log Slow Queries:** Set `log_min_duration_statement = 1000` (logs queries taking longer than 1 second).

### 8.2 Handling CPU Spikes

*   **Diagnosis:** High CPU is rarely a CPU problem; it is almost always a missing index causing sequential scans, or a sudden surge in connections.
*   **Action:** Query `pg_stat_activity` to find active queries:
    ```sql
    SELECT pid, user, state, query, wait_event_type, wait_event
    FROM pg_stat_activity
    WHERE state = 'active' AND pid <> pg_backend_pid();
    ```
*   **Resolution:** Use `pg_cancel_backend(pid)` to gracefully stop a rogue query, or `pg_terminate_backend(pid)` to forcefully kill the connection if it refuses to cancel.

### 8.3 Handling I/O Bottlenecks

*   **Diagnosis:** High `iowait` on the system, and queries in `pg_stat_activity` showing `wait_event_type = 'IO'`.
*   **Action:** Check if checkpoints are running constantly. Review `shared_buffers` hit ratio. Ensure that the underlying storage (e.g., AWS EBS io2/gp3 or NVMe SSDs) provides sufficient IOPS.

## 9. Conclusion

Tuning PostgreSQL 15+ is not a set-and-forget operation. It requires continuous monitoring, baseline establishment, and iterative adjustments. By understanding the intricate balance between memory allocation, WAL management, and autovacuum behavior, tech support teams can ensure that the database remains resilient, performant, and capable of handling the most demanding enterprise workloads.
