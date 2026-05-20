# PostgreSQL 15+ CLI Reference: Operations, Monitoring, and Tech Support

## Introduction

This document serves as a comprehensive, production-grade reference for PostgreSQL 15+ command-line tools and advanced SQL queries. Designed specifically for tech support operations, database administrators, and system reliability engineers, this guide focuses on worst-case scenarios, massive datasets, complex database migrations, and high-stakes production environments. 

PostgreSQL 15 introduced several critical enhancements, including the `MERGE` command, improved logical replication, structured server log output (JSON), and performance improvements for sorting and window functions. This reference leverages these features to provide actionable, robust solutions for managing enterprise-scale PostgreSQL deployments.

---

## 1. `psql`: The Interactive Terminal

The `psql` utility is the primary interface for interacting with PostgreSQL. In production environments, mastering `psql` is essential for rapid diagnostics, automated scripting, and secure data manipulation.

### 1.1 Essential Connection and Execution Flags

When connecting to production databases, security and reliability are paramount. Avoid passing passwords in the command line; instead, use `.pgpass` files or environment variables (`PGPASSWORD`).

```bash
# Connect using a specific URI (recommended for complex connection strings)
psql "postgresql://username:password@hostname:5432/dbname?sslmode=require"

# Execute a single command and exit (useful for scripting)
psql -U admin_user -h prod-db.example.com -d production_db -c "SELECT pg_reload_conf();"

# Execute a script file with transaction control
# -v ON_ERROR_STOP=1 ensures the script stops if any command fails
# -1 wraps the entire script in a single transaction
psql -U admin_user -h prod-db.example.com -d production_db -v ON_ERROR_STOP=1 -1 -f /path/to/migration.sql
```

### 1.2 Advanced `psql` Meta-Commands

Meta-commands (starting with `\`) are powerful tools for database introspection and formatting.

*   `\x auto`: Toggles expanded output automatically. Crucial for reading rows with many columns.
*   `\watch [seconds]`: Executes the current query buffer repeatedly. Excellent for monitoring active queries or lock queues.
*   `\copy`: Performs client-side data transfer. Unlike the SQL `COPY` command, `\copy` runs with the permissions of the local user, not the PostgreSQL server process.
*   `\timing`: Toggles execution time display. Essential for performance tuning.
*   `\d+ [pattern]`: Shows detailed information about tables, views, or sequences, including size and description.
*   `\di+`: Lists indexes with their sizes.
*   `\df+`: Lists functions with their source code.

### 1.3 Handling Massive Datasets with `\copy`

When dealing with huge datasets, standard `INSERT` statements are too slow. `\copy` is the preferred method for bulk data loading and extraction.

```sql
-- Exporting a massive table to a CSV file with a header
\copy (SELECT * FROM massive_audit_log WHERE created_at >= '2023-01-01') TO '/tmp/audit_export.csv' WITH (FORMAT csv, HEADER true);

-- Importing data from a CSV file, handling nulls and specific delimiters
\copy target_table FROM '/tmp/data_import.csv' WITH (FORMAT csv, HEADER true, DELIMITER ',', NULL 'NULL_STRING');
```

---

## 2. `pg_dump` and `pg_restore`: Logical Backups

Logical backups are essential for migrations, selective data restoration, and archiving. For databases larger than a few hundred gigabytes, logical backups can be slow, but they offer unparalleled flexibility.

### 2.1 `pg_dump`: Exporting Data

`pg_dump` extracts a PostgreSQL database into a script file or an archive file. For production, the custom format (`-F c`) or directory format (`-F d`) is highly recommended as they support parallel restoration.

```bash
# Dump a database in custom format with high compression
pg_dump -U backup_user -h prod-db.example.com -d production_db -F c -Z 9 -f /backups/prod_db_$(date +%F).dump

# Dump using directory format with parallel jobs (requires -j)
# This is significantly faster for large databases
pg_dump -U backup_user -h prod-db.example.com -d production_db -F d -j 4 -f /backups/prod_db_dir_$(date +%F)

# Dump only the schema (no data)
pg_dump -U backup_user -h prod-db.example.com -d production_db -s -F p -f /backups/prod_schema.sql

# Dump specific tables matching a pattern
pg_dump -U backup_user -h prod-db.example.com -d production_db -t 'public.user_*' -F c -f /backups/user_tables.dump
```

### 2.2 `pg_restore`: Importing Data

`pg_restore` is used to restore databases from custom or directory format archives created by `pg_dump`.

```bash
# Restore a custom format dump using parallel jobs
# -C creates the database before restoring
# -e exits on error
pg_restore -U admin_user -h target-db.example.com -d postgres -C -e -j 8 /backups/prod_db_2023-10-27.dump

# Restore only a specific table from a dump
pg_restore -U admin_user -h target-db.example.com -d target_db -t specific_table -j 4 /backups/prod_db_2023-10-27.dump

# Generate a SQL script from a custom dump (useful for inspection)
pg_restore -f inspection_script.sql /backups/prod_db_2023-10-27.dump
```

### 2.3 Worst-Case Scenario: Corrupted Indexes During Restore

If a restore fails due to corrupted indexes or constraints, you can restore the data first, and then build the indexes manually.

```bash
# 1. Restore only the data (no schema, no indexes)
pg_restore -U admin_user -d target_db --data-only -j 8 /backups/prod_db.dump

# 2. Extract the index creation statements
pg_restore -l /backups/prod_db.dump | grep -i index > index_list.txt
pg_restore -U admin_user -d target_db -L index_list.txt /backups/prod_db.dump
```

---

## 3. `pg_basebackup`: Physical Backups and Replication

`pg_basebackup` takes a physical, binary copy of the database cluster files. It is the foundation for Point-In-Time Recovery (PITR) and setting up streaming replication standbys.

### 3.1 Creating a Base Backup

Physical backups are generally faster than logical backups for massive databases because they copy files directly.

```bash
# Create a base backup in a specific directory
# -X stream includes necessary WAL files during the backup
# -P shows progress
# -c fast forces a checkpoint immediately
pg_basebackup -U replication_user -h primary-db.example.com -D /var/lib/postgresql/15/backups/base_$(date +%F) -X stream -P -c fast

# Create a compressed tar format backup
pg_basebackup -U replication_user -h primary-db.example.com -D - -F t -z -X fetch -c fast > /backups/base_backup.tar.gz
```

### 3.2 Setting Up a Streaming Replica

To set up a read replica, `pg_basebackup` is used to clone the primary node.

```bash
# On the replica server, ensure the data directory is empty
rm -rf /var/lib/postgresql/15/main/*

# Run pg_basebackup with the -R flag to automatically create standby.signal
# and configure connection settings in postgresql.auto.conf
pg_basebackup -U replication_user -h primary-db.example.com -D /var/lib/postgresql/15/main -X stream -P -R -c fast

# Start the replica
systemctl start postgresql
```

---

## 4. Advanced SQL Queries for Operations and Monitoring

In tech support and operations, identifying bottlenecks, deadlocks, and bloat is a daily task. The following queries are indispensable for diagnosing production issues.

### 4.1 Identifying Long-Running and Blocking Queries

When the database grinds to a halt, the first step is to identify what is running and what is blocking.

```sql
-- View active queries running for more than 1 minute
SELECT pid, 
       usename, 
       application_name, 
       client_addr, 
       state, 
       now() - query_start AS duration, 
       query 
FROM pg_stat_activity 
WHERE state = 'active' 
  AND now() - query_start > interval '1 minute' 
ORDER BY duration DESC;

-- Identify blocking and blocked queries (The "Lock Tree")
SELECT blocked_locks.pid     AS blocked_pid,
       blocked_activity.usename  AS blocked_user,
       blocking_locks.pid     AS blocking_pid,
       blocking_activity.usename AS blocking_user,
       blocked_activity.query    AS blocked_statement,
       blocking_activity.query   AS current_statement_in_blocking_process
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
    AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
    AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
    AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
    AND blocking_locks.pid != blocked_locks.pid
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;
```

### 4.2 Terminating Problematic Connections

Once a blocking query is identified, it may need to be terminated to restore service.

```sql
-- Cancel a specific query gracefully (sends SIGINT)
SELECT pg_cancel_backend(pid_of_query);

-- Terminate a connection forcefully (sends SIGTERM)
-- Use with caution, as it severs the connection entirely
SELECT pg_terminate_backend(pid_of_connection);

-- Terminate all connections to a specific database (useful for dropping a DB)
SELECT pg_terminate_backend(pid) 
FROM pg_stat_activity 
WHERE datname = 'target_database' AND pid <> pg_backend_pid();
```

### 4.3 Monitoring Table and Index Bloat

PostgreSQL uses Multi-Version Concurrency Control (MVCC), which leaves "dead tuples" behind after updates and deletes. If autovacuum cannot keep up, tables and indexes become bloated, degrading performance.

```sql
-- Estimate table bloat (Requires the pgstattuple extension for exact numbers, 
-- but this heuristic query provides a quick estimate)
WITH constants AS (
    SELECT current_setting('block_size')::numeric AS bs, 23 AS hdr, 4 AS ma
), bloat_info AS (
    SELECT
        ma,bs,schemaname,tablename,
        (datawidth+(hdr+ma-(case when hdr%ma=0 THEN ma ELSE hdr%ma END)))::numeric AS datahdr,
        (maxfracsum*(nullhdr+ma-(case when nullhdr%ma=0 THEN ma ELSE nullhdr%ma END))) AS nullhdr2
    FROM (
        SELECT
            schemaname, tablename, hdr, ma, bs,
            SUM((1-null_frac)*avg_width) AS datawidth,
            MAX(null_frac) AS maxfracsum,
            hdr+(
                SELECT 1+count(*)/8
                FROM pg_stats s2
                WHERE null_frac<>0 AND s2.schemaname = s.schemaname AND s2.tablename = s.tablename
            ) AS nullhdr
        FROM pg_stats s, constants
        GROUP BY 1,2,3,4,5
    ) AS foo
), table_bloat AS (
    SELECT
        schemaname, tablename, cc.reltuples, cc.relpages, bs,
        CEIL((cc.reltuples*((datahdr+ma-
            (CASE WHEN datahdr%ma=0 THEN ma ELSE datahdr%ma END))+nullhdr2+4))/(bs-20::float)) AS otta
    FROM bloat_info
    JOIN pg_class cc ON cc.relname = bloat_info.tablename
    JOIN pg_namespace nn ON cc.relnamespace = nn.oid AND nn.nspname = bloat_info.schemaname AND nn.nspname <> 'information_schema'
)
SELECT
    schemaname || '.' || tablename AS relation,
    pg_size_pretty((relpages::bigint*bs)::bigint) AS total_size,
    pg_size_pretty(((relpages-otta)*bs)::bigint) AS bloat_size,
    ROUND(((relpages-otta)::numeric/relpages::numeric)*100, 2) AS bloat_ratio
FROM table_bloat
WHERE relpages > otta AND relpages > 1000
ORDER BY bloat_ratio DESC;
```

### 4.4 Index Usage and Optimization

Unused indexes consume disk space and slow down `INSERT`, `UPDATE`, and `DELETE` operations. Identifying and removing them is crucial for write-heavy workloads.

```sql
-- Identify unused indexes
SELECT
    schemaname || '.' || relname AS table_name,
    indexrelname AS index_name,
    pg_size_pretty(pg_relation_size(i.indexrelid)) AS index_size,
    idx_scan AS index_scans
FROM pg_stat_user_indexes ui
JOIN pg_index i ON ui.indexrelid = i.indexrelid
WHERE NOT i.indisunique   -- Do not drop unique indexes
  AND idx_scan < 50       -- Threshold for "unused"
  AND pg_relation_size(i.indexrelid) > 1024 * 1024 * 10 -- Larger than 10MB
ORDER BY pg_relation_size(i.indexrelid) DESC;

-- Identify duplicate indexes
SELECT pg_size_pretty(SUM(pg_relation_size(idx))::BIGINT) AS size,
       (array_agg(idx))[1] AS idx1, (array_agg(idx))[2] AS idx2,
       (array_agg(idx))[3] AS idx3, (array_agg(idx))[4] AS idx4
FROM (
    SELECT indexrelid::regclass AS idx, (indrelid::text ||E'\n'|| indclass::text ||E'\n'|| indkey::text ||E'\n'||
                                         COALESCE(indexprs::text,'')||E'\n' || COALESCE(indpred::text,'')) AS key
    FROM pg_index) sub
GROUP BY key HAVING COUNT(*)>1
ORDER BY SUM(pg_relation_size(idx)) DESC;
```

### 4.5 Monitoring Replication Lag

For high-availability setups, monitoring replication lag is critical to ensure read replicas are up-to-date and failovers will not result in significant data loss.

```sql
-- Run on the PRIMARY node to see the status of all replicas
SELECT client_addr, 
       state, 
       sync_state, 
       pg_wal_lsn_diff(pg_current_wal_lsn(), sent_lsn) AS pending_bytes,
       pg_wal_lsn_diff(sent_lsn, write_lsn) AS write_lag_bytes,
       pg_wal_lsn_diff(write_lsn, flush_lsn) AS flush_lag_bytes,
       pg_wal_lsn_diff(flush_lsn, replay_lsn) AS replay_lag_bytes,
       pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn) AS total_lag_bytes
FROM pg_stat_replication;

-- Run on the REPLICA node to see how far behind it is
SELECT now() - pg_last_xact_replay_timestamp() AS replication_delay;
```

---

## 5. Worst-Case Scenarios and Disaster Recovery

Tech support operations must be prepared for catastrophic failures. The following scenarios outline recovery strategies.

### 5.1 Transaction ID (XID) Wraparound

PostgreSQL uses 32-bit transaction IDs. If a database processes 2 billion transactions without vacuuming, it will reach XID wraparound, causing the database to shut down to prevent data loss.

**Symptoms:**
*   Warnings in logs: `WARNING: database "mydb" must be vacuumed within 1000000 transactions`
*   Database refuses connections with: `ERROR: database is not accepting commands to avoid wraparound data loss in database "mydb"`

**Resolution:**
1.  Stop the application to prevent new transactions.
2.  Start PostgreSQL in single-user mode.
    ```bash
    sudo -u postgres postgres --single -D /var/lib/postgresql/15/main mydb
    ```
3.  Run a standalone `VACUUM FREEZE` on the affected tables or the entire database.
    ```sql
    VACUUM FREEZE;
    ```
4.  Restart PostgreSQL normally.

### 5.2 Corrupted Write-Ahead Logs (WAL)

If a WAL file is corrupted or missing, the database may fail to start or recover.

**Resolution (Data Loss Possible):**
If you cannot restore the WAL file from an archive, you may need to reset the WAL. **This is a destructive operation and should only be used as a last resort.**

```bash
# Ensure PostgreSQL is stopped
systemctl stop postgresql

# Reset the WAL using pg_resetwal
# -f forces the reset even if the server seems to be running or data is inconsistent
sudo -u postgres pg_resetwal -f /var/lib/postgresql/15/main

# Start PostgreSQL and immediately take a logical backup (pg_dump), 
# as the database may be in an inconsistent state.
```

### 5.3 Accidental `DROP TABLE` or `TRUNCATE`

If a table is accidentally dropped and you have Point-In-Time Recovery (PITR) configured:

1.  Identify the exact time the drop occurred.
2.  Restore a base backup to a temporary directory or server.
3.  Configure `recovery.conf` (or `postgresql.conf` in PG 12+) to recover up to the moment just before the drop.
    ```ini
    restore_command = 'cp /mnt/wal_archive/%f %p'
    recovery_target_time = '2023-10-27 14:30:00 UTC'
    recovery_target_action = 'promote'
    ```
4.  Start the temporary PostgreSQL instance.
5.  Use `pg_dump` to extract the dropped table from the temporary instance.
6.  Use `pg_restore` or `psql` to import the table back into the production instance.

---

## 6. PostgreSQL 15 Specific Features for Operations

PostgreSQL 15 introduced several features that directly impact operations and tech support.

### 6.1 Structured Server Log Output (JSON)

Parsing traditional PostgreSQL logs can be difficult. PG 15 allows logging in JSON format, making it trivial to ingest logs into systems like Elasticsearch, Splunk, or Datadog.

**Configuration (`postgresql.conf`):**
```ini
log_destination = 'jsonlog'
logging_collector = on
log_directory = 'log'
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.json'
```

### 6.2 The `MERGE` Command

The `MERGE` command simplifies complex "upsert" logic, reducing the need for custom PL/pgSQL functions and improving performance for bulk data synchronization.

```sql
MERGE INTO target_inventory t
USING source_updates s
ON t.item_id = s.item_id
WHEN MATCHED AND s.quantity = 0 THEN
    DELETE
WHEN MATCHED THEN
    UPDATE SET quantity = s.quantity, last_updated = now()
WHEN NOT MATCHED THEN
    INSERT (item_id, quantity, last_updated) VALUES (s.item_id, s.quantity, now());
```

### 6.3 Logical Replication Enhancements

PostgreSQL 15 allows logical replication to publish all tables in a schema, and supports row filtering and column lists, significantly reducing network traffic and storage requirements on the subscriber.

```sql
-- Publish only specific columns
CREATE PUBLICATION user_pub FOR TABLE users (id, username, email);

-- Publish with a row filter (e.g., only active users)
CREATE PUBLICATION active_user_pub FOR TABLE users WHERE (is_active = true);

-- Publish all tables in a specific schema
CREATE PUBLICATION schema_pub FOR TABLES IN SCHEMA tenant_a;
```

---

## Conclusion

Mastering these CLI tools and advanced SQL queries is non-negotiable for anyone responsible for PostgreSQL operations. By understanding how to efficiently move data, diagnose performance bottlenecks, and recover from catastrophic failures, tech support teams can ensure high availability and data integrity in the most demanding production environments. Always test backup and recovery procedures regularly—a backup is only as good as its last successful restore.

## 7. Performance Tuning and Configuration Management

Tech support operations often involve diagnosing performance issues that stem from suboptimal configuration. PostgreSQL's default settings are highly conservative to ensure compatibility across a wide range of hardware. Tuning these parameters is essential for production workloads.

### 7.1 Memory Configuration

PostgreSQL relies heavily on memory for caching and sorting. The two most critical parameters are `shared_buffers` and `work_mem`.

*   **`shared_buffers`**: Determines how much memory PostgreSQL uses for caching data. A common rule of thumb is to set this to 25% of total system RAM. However, on systems with massive amounts of RAM (e.g., 256GB+), setting it higher than 64GB may yield diminishing returns due to kernel overhead.
*   **`work_mem`**: Specifies the amount of memory to be used by internal sort operations and hash tables before writing to temporary disk files. This is a per-operation setting, meaning a complex query with multiple sorts could use several times this amount. Setting it too high can lead to Out-Of-Memory (OOM) errors.

```sql
-- Check current memory settings
SHOW shared_buffers;
SHOW work_mem;

-- Dynamically change work_mem for a specific session (useful for heavy batch jobs)
SET work_mem = '256MB';
```

### 7.2 Autovacuum Tuning

Autovacuum is critical for maintaining database health. If it runs too infrequently, tables bloat and performance degrades. If it runs too aggressively, it can consume excessive I/O.

*   **`autovacuum_vacuum_scale_factor`**: The fraction of the table size that must be updated or deleted before a vacuum is triggered. The default is 0.2 (20%). For massive tables, 20% is too large.
*   **`autovacuum_analyze_scale_factor`**: Similar to the vacuum scale factor, but triggers an `ANALYZE` to update statistics.

```sql
-- Tune autovacuum for a specific massive table
ALTER TABLE massive_events_table SET (
    autovacuum_vacuum_scale_factor = 0.01, -- Trigger vacuum at 1% changes
    autovacuum_analyze_scale_factor = 0.005 -- Trigger analyze at 0.5% changes
);
```

### 7.3 Connection Pooling with PgBouncer

PostgreSQL uses a process-per-connection model, which consumes significant memory and CPU overhead for each connection. In high-concurrency environments, a connection pooler like PgBouncer is mandatory.

PgBouncer operates in three modes:
1.  **Session pooling**: A server connection is assigned to a client for the duration of the session.
2.  **Transaction pooling**: A server connection is assigned to a client only for the duration of a transaction. This is the most common and efficient mode for web applications.
3.  **Statement pooling**: A server connection is assigned for a single statement. Rarely used.

**Tech Support Tip:** When diagnosing connection issues, always check if PgBouncer is the bottleneck. Use the PgBouncer administrative console to monitor pool usage.

```bash
# Connect to the PgBouncer admin console
psql -p 6432 -U pgbouncer pgbouncer

# View pool statistics
SHOW POOLS;
SHOW CLIENTS;
SHOW SERVERS;
```

## 8. Security and Auditing

Security is a continuous operational concern. Tech support must be able to audit access, manage roles, and ensure data encryption.

### 8.1 Role Management and Least Privilege

Avoid using the `postgres` superuser for application connections. Implement the principle of least privilege by creating specific roles for specific tasks.

```sql
-- Create a read-only role
CREATE ROLE readonly_user WITH LOGIN PASSWORD 'secure_password';
GRANT CONNECT ON DATABASE production_db TO readonly_user;
GRANT USAGE ON SCHEMA public TO readonly_user;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly_user;

-- Ensure future tables are also readable by this role
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO readonly_user;
```

### 8.2 Auditing with `pgaudit`

For compliance (e.g., SOC2, HIPAA), auditing database activity is required. The `pgaudit` extension provides detailed session and object audit logging.

**Configuration (`postgresql.conf`):**
```ini
shared_preload_libraries = 'pgaudit'
pgaudit.log = 'write, ddl'
pgaudit.log_catalog = off
```

With `pgaudit` enabled, all `INSERT`, `UPDATE`, `DELETE`, and DDL statements will be logged to the standard PostgreSQL log, which can then be ingested by a SIEM system.

## 9. Handling Extension Upgrades

PostgreSQL's extensibility is one of its greatest strengths, but upgrading extensions during major version upgrades or regular maintenance requires careful handling.

### 9.1 Upgrading PostGIS

PostGIS is a complex extension that frequently requires upgrades.

```sql
-- Check current PostGIS version
SELECT postgis_full_version();

-- Upgrade the extension (must be run in each database where it is installed)
ALTER EXTENSION postgis UPDATE;
ALTER EXTENSION postgis_topology UPDATE;
```

### 9.2 Managing `pg_stat_statements`

`pg_stat_statements` is essential for performance monitoring, as it records execution statistics of all SQL statements executed.

```sql
-- Reset statistics (useful after a major application deployment or index creation)
SELECT pg_stat_statements_reset();

-- Find the top 5 queries consuming the most total time
SELECT query, calls, total_exec_time, rows, 100.0 * shared_blks_hit / nullif(shared_blks_hit + shared_blks_read, 0) AS hit_percent
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 5;
```

## 10. Summary of Daily Operations Checklist

For tech support and operations teams, maintaining a daily checklist ensures proactive management rather than reactive firefighting.

1.  **Check Backups**: Verify that `pg_dump` or `pg_basebackup` completed successfully and test a restore in a staging environment at least weekly.
2.  **Monitor Replication Lag**: Ensure standby servers are within acceptable lag thresholds (typically < 10 seconds).
3.  **Review Error Logs**: Scan PostgreSQL logs for `FATAL`, `PANIC`, or frequent `ERROR` messages.
4.  **Check for Long-Running Queries**: Use the queries provided in Section 4.1 to identify and resolve stuck transactions.
5.  **Monitor Bloat**: Run bloat estimation queries to ensure autovacuum is keeping up with the workload.
6.  **Review Connection Counts**: Ensure the number of active connections is well below `max_connections` and that PgBouncer pools are not exhausted.

By adhering to these practices and utilizing the advanced features of PostgreSQL 15, operations teams can maintain robust, high-performance database systems capable of handling enterprise-scale demands.
