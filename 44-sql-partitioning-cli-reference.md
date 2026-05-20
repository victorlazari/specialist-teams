# Topic 3: Comprehensive CLI and SQL Reference for Table Partitioning, Index Creation on Partitions, EXPLAIN ANALYZE, and Advanced One-Liners for Operations

## 1. Introduction and Scope

Welcome to the comprehensive reference guide for database partitioning, advanced SQL operations, and command-line interface (CLI) utilities tailored specifically for tech support operations, database administrators (DBAs), and site reliability engineers (SREs). This document is designed to be your ultimate resource when dealing with massive datasets, production emergencies, query optimization, and complex database migrations.

In modern high-throughput environments, monolithic tables quickly become bottlenecks. Table partitioning is not just a performance optimization; it is a survival mechanism for databases handling terabytes of data. This guide dives deep into the mechanics of partitioning, the nuances of index creation across partitions, the art of interpreting `EXPLAIN ANALYZE` outputs, and a curated collection of advanced CLI one-liners that can save your production environment during a crisis.

### 1.1 Relationship to Other Specialist Files

This document, **Topic 3**, is a critical component of the broader specialist-teams repository. It serves as the deep-dive technical manual for database operations, complementing the other six files in the following ways:

*   **Topic 1 (Incident Response Framework):** Provides the technical commands and SQL queries needed to diagnose and mitigate database-related incidents identified in the framework. When an alert fires for high latency, Topic 1 guides the triage, while Topic 3 provides the exact `EXPLAIN ANALYZE` commands to find the root cause.
*   **Topic 2 (System Architecture & Scaling):** Acts as the practical implementation guide for the database scaling strategies discussed in the architecture overview. While Topic 2 discusses the theory of sharding and partitioning, Topic 3 provides the exact SQL syntax to execute it safely.
*   **Topic 4 (Network Diagnostics):** Works in tandem with network troubleshooting; often, what appears to be a network timeout is actually a poorly optimized query locking a massive unpartitioned table. Topic 3 helps rule out the database as the source of network-level timeouts.
*   **Topic 5 (Security & Access Control):** Ensures that the advanced operations detailed here are executed with the appropriate privileges and audit logging. Partitioning and index creation require elevated permissions, which are governed by Topic 5.
*   **Topic 6 (Disaster Recovery):** Provides the data manipulation and migration techniques necessary for restoring service during catastrophic failures. The CLI one-liners in Topic 3 are essential for verifying data integrity post-recovery.
*   **Topic 7 (Application Performance Monitoring):** Connects the database-level metrics (`EXPLAIN ANALYZE`) with application-level tracing to provide a holistic view of system performance.

---

## 2. Table Partitioning: Strategies and Implementation

Table partitioning involves dividing a large logical table into smaller, more manageable physical pieces called partitions. This section focuses on PostgreSQL, as it offers robust declarative partitioning, but the concepts apply broadly to other relational database management systems (RDBMS) like MySQL and Oracle.

### 2.1 Why Partition? The Production Reality

In a tech support or SRE role, you will encounter scenarios where a single table has grown to billions of rows. The symptoms are classic and often lead to severe production degradation:
*   **Index Bloat:** Indexes become too large to fit in RAM (shared buffers). When the working set exceeds available memory, the database is forced to perform massive disk I/O, slowing down all operations.
*   **Vacuuming Nightmares:** Autovacuum processes cannot keep up with dead tuples on a monolithic table. This leads to table bloat and, in extreme cases, transaction ID wraparound risks, which can force the database into read-only mode to prevent data corruption.
*   **Query Timeouts:** Sequential scans on the table take hours, causing application-level timeouts, connection pool exhaustion, and cascading failures across microservices.
*   **Archival Impossibility:** Deleting old data using `DELETE FROM table WHERE date < '...'` causes massive WAL (Write-Ahead Log) generation, replication lag, and heavy locking.

Partitioning solves these issues by allowing you to drop entire partitions (which is a fast metadata operation) instead of deleting rows, and by enabling partition pruning during query execution, drastically reducing the amount of data scanned.

### 2.2 Partitioning Strategies

#### 2.2.1 Range Partitioning

The most common strategy, typically based on a timestamp or date column. Ideal for time-series data, logs, audit trails, and historical records.

```sql
-- Creating the parent table
CREATE TABLE production_logs (
    log_id BIGSERIAL,
    service_name VARCHAR(255) NOT NULL,
    log_level VARCHAR(50) NOT NULL,
    message TEXT,
    created_at TIMESTAMPTZ NOT NULL
) PARTITION BY RANGE (created_at);

-- Creating partitions (e.g., monthly)
CREATE TABLE production_logs_2023_10 PARTITION OF production_logs
    FOR VALUES FROM ('2023-10-01 00:00:00Z') TO ('2023-11-01 00:00:00Z');

CREATE TABLE production_logs_2023_11 PARTITION OF production_logs
    FOR VALUES FROM ('2023-11-01 00:00:00Z') TO ('2023-12-01 00:00:00Z');
```

**Tech Support Tip:** Always create partitions ahead of time. A common production incident is the application failing to insert data at midnight on the first of the month because the new partition was not created. Use a cron job or an extension like `pg_partman` to automate partition creation and retention policies.

#### 2.2.2 List Partitioning

Used when data can be naturally grouped by a specific, finite set of values, such as region, tenant ID, or status. This is highly effective for multi-tenant SaaS applications.

```sql
CREATE TABLE customer_transactions (
    transaction_id BIGSERIAL,
    customer_id BIGINT NOT NULL,
    region_code VARCHAR(10) NOT NULL,
    amount DECIMAL(15, 2) NOT NULL,
    transaction_date DATE NOT NULL
) PARTITION BY LIST (region_code);

CREATE TABLE transactions_na PARTITION OF customer_transactions
    FOR VALUES IN ('US', 'CA', 'MX');

CREATE TABLE transactions_eu PARTITION OF customer_transactions
    FOR VALUES IN ('UK', 'FR', 'DE', 'IT');
    
CREATE TABLE transactions_default PARTITION OF customer_transactions DEFAULT;
```
*Note on Default Partitions:* Always include a default partition to catch unexpected values. Without it, inserts with unmapped `region_code` values will fail, causing application errors.

#### 2.2.3 Hash Partitioning

Useful for distributing data evenly across partitions when there is no natural range or list, often used for load balancing massive write-heavy tables to prevent hot spots.

```sql
CREATE TABLE user_sessions (
    session_id UUID NOT NULL,
    user_id BIGINT NOT NULL,
    session_data JSONB,
    last_active TIMESTAMPTZ NOT NULL
) PARTITION BY HASH (user_id);

-- Creating 4 partitions
CREATE TABLE user_sessions_p0 PARTITION OF user_sessions FOR VALUES WITH (MODULUS 4, REMAINDER 0);
CREATE TABLE user_sessions_p1 PARTITION OF user_sessions FOR VALUES WITH (MODULUS 4, REMAINDER 1);
CREATE TABLE user_sessions_p2 PARTITION OF user_sessions FOR VALUES WITH (MODULUS 4, REMAINDER 2);
CREATE TABLE user_sessions_p3 PARTITION OF user_sessions FOR VALUES WITH (MODULUS 4, REMAINDER 3);
```

### 2.3 Managing Partitions in Production

#### 2.3.1 Attaching and Detaching Partitions

During migrations or archival processes, you need to move data without locking the parent table for extended periods.

**Detaching a partition (Archival):**
```sql
-- This requires an ACCESS EXCLUSIVE lock on the parent table, but it's very fast.
ALTER TABLE production_logs DETACH PARTITION production_logs_2022_01;

-- Now production_logs_2022_01 is a standalone table. You can back it up and drop it.
pg_dump -t production_logs_2022_01 mydb > archive_2022_01.sql
DROP TABLE production_logs_2022_01;
```

**Attaching a partition concurrently (Migration):**
When migrating a massive unpartitioned table to a partitioned structure, you can attach existing tables as partitions. This is a critical skill for zero-downtime migrations.

```sql
-- 1. Create the new table with constraints matching the partition bounds
CREATE TABLE new_partition_2023_12 (LIKE production_logs INCLUDING ALL);
ALTER TABLE new_partition_2023_12 ADD CONSTRAINT check_date 
    CHECK (created_at >= '2023-12-01 00:00:00Z' AND created_at < '2024-01-01 00:00:00Z');

-- 2. Load data into the new table (can be done slowly in the background using logical replication or batch inserts)
INSERT INTO new_partition_2023_12 SELECT * FROM old_massive_table WHERE created_at >= '2023-12-01 00:00:00Z' AND created_at < '2024-01-01 00:00:00Z';

-- 3. Attach the partition. Because the CHECK constraint exists, Postgres doesn't need to scan the table to verify data, making the attach operation nearly instantaneous.
ALTER TABLE production_logs ATTACH PARTITION new_partition_2023_12
    FOR VALUES FROM ('2023-12-01 00:00:00Z') TO ('2024-01-01 00:00:00Z');

-- 4. Drop the constraint as it's now redundant
ALTER TABLE new_partition_2023_12 DROP CONSTRAINT check_date;
```

---

## 3. Index Creation on Partitions: The Worst-Case Scenarios

Creating indexes on massive tables is one of the most dangerous operations in a production database. A standard `CREATE INDEX` blocks all writes to the table until the index is built, which can take hours and cause a complete system outage.

### 3.1 The Golden Rule: CREATE INDEX CONCURRENTLY

Always use `CONCURRENTLY` when creating indexes on live production tables. This builds the index without locking out concurrent inserts, updates, or deletes. It takes longer and uses more resources, but it keeps the application online.

```sql
CREATE INDEX CONCURRENTLY idx_prod_logs_service ON production_logs (service_name);
```

**The Catch with Partitioned Tables:**
In PostgreSQL (prior to version 11, and with caveats in later versions), you cannot use `CREATE INDEX CONCURRENTLY` directly on the parent partitioned table. You must build the indexes on the individual partitions. Even in newer versions, building it on the parent can hold locks longer than desired.

### 3.2 The Safe Indexing Workflow for Partitioned Tables

If you need to add an index to a partitioned table with terabytes of data, follow this procedure to avoid locking the database:

1.  **Create the index on the parent table as INVALID:**
    ```sql
    -- This creates the metadata but doesn't build the index, so it's fast and doesn't lock.
    CREATE INDEX idx_logs_level ON ONLY production_logs (log_level);
    ```

2.  **Build the index concurrently on each partition:**
    ```sql
    CREATE INDEX CONCURRENTLY idx_logs_level_2023_10 ON production_logs_2023_10 (log_level);
    CREATE INDEX CONCURRENTLY idx_logs_level_2023_11 ON production_logs_2023_11 (log_level);
    -- Repeat for all partitions...
    ```
    *Automation Tip:* Write a script to iterate through `pg_class` and `pg_inherits` to find all partitions and execute these statements sequentially to avoid overwhelming the I/O subsystem.

3.  **Attach the partition indexes to the parent index:**
    ```sql
    ALTER INDEX idx_logs_level ATTACH PARTITION idx_logs_level_2023_10;
    ALTER INDEX idx_logs_level ATTACH PARTITION idx_logs_level_2023_11;
    -- Repeat for all partitions...
    ```
    Once all partition indexes are attached, the parent index automatically becomes valid and usable by the query planner.

### 3.3 Handling Failed Concurrent Indexes

If a `CREATE INDEX CONCURRENTLY` operation fails (e.g., due to a deadlock, a unique constraint violation, or a server crash), it leaves behind an `INVALID` index. This invalid index still consumes disk space and slows down `INSERT`/`UPDATE` operations because the database still tries to maintain it, even though it cannot be used for read queries.

**Tech Support Action:**
1.  Identify invalid indexes:
    ```sql
    SELECT indexrelid::regclass AS index_name, relname AS table_name
    FROM pg_index i
    JOIN pg_class c ON i.indrelid = c.oid
    WHERE i.indisvalid = false;
    ```
2.  Drop the invalid index:
    ```sql
    DROP INDEX CONCURRENTLY index_name;
    ```
3.  Investigate the cause of the failure (check PostgreSQL logs for deadlock or constraint errors), fix the underlying issue, and retry the creation.

---

## 4. Mastering EXPLAIN ANALYZE

When a query is timing out or consuming excessive CPU, `EXPLAIN ANALYZE` is your primary diagnostic tool. It executes the query and provides the actual run times and row counts, comparing them to the planner's estimates.

### 4.1 The Anatomy of an EXPLAIN ANALYZE Output

```sql
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT) 
SELECT * FROM customer_transactions WHERE customer_id = 12345 AND transaction_date > '2023-01-01';
```

**Key components to look for:**

*   **Execution Time:** The total time taken. If this is high, you have a problem.
*   **Node Types:**
    *   `Seq Scan`: Sequential Scan. Reading the entire table. Bad for large tables unless you are retrieving a huge percentage of the rows.
    *   `Index Scan`: Using an index to find rows, then fetching the data from the heap (table). Good.
    *   `Index Only Scan`: The required data is entirely contained within the index. Excellent. No heap fetch required.
    *   `Bitmap Heap Scan` / `Bitmap Index Scan`: Used when an index scan would result in too many random heap fetches. It builds a bitmap of pages to visit, then visits them sequentially. Good for medium-selectivity queries.
    *   `Nested Loop`: Joins rows by iterating through the outer relation and looking up matches in the inner relation. Good for small datasets or when the inner relation is heavily indexed.
    *   `Hash Join`: Builds a hash table of the smaller relation, then probes it with the larger relation. Good for large, unsorted datasets.
    *   `Merge Join`: Requires both inputs to be sorted. Zips them together. Good for very large datasets that are already sorted.
*   **Actual Rows vs. Estimated Rows:**
    *   `rows=1000000 (actual time=... rows=10 loops=1)`
    *   If the estimated rows (1,000,000) are vastly different from the actual rows (10), the planner is making bad decisions based on outdated statistics. This often leads to choosing a Seq Scan over an Index Scan.
    *   **Fix:** Run `ANALYZE table_name;` to update statistics. If the issue persists, you may need to increase the statistics target for the specific columns involved (`ALTER TABLE table_name ALTER COLUMN col_name SET STATISTICS 1000;`).
*   **Buffers:** (Requires `BUFFERS` option)
    *   `Buffers: shared hit=50 read=10000`
    *   `hit`: Data found in the database's shared memory (RAM). Fast.
    *   `read`: Data had to be read from disk. Slow. High `read` values indicate that your working set doesn't fit in RAM, or your query is highly unoptimized.
    *   `dirtied` / `written`: Indicates the query is modifying data or causing pages to be flushed to disk.

### 4.2 Diagnosing Partition Pruning

When querying a partitioned table, you must ensure that the database is only scanning the relevant partitions. This is called partition pruning. If pruning fails, a query meant for a single day's data might scan years of history.

```sql
EXPLAIN ANALYZE SELECT * FROM production_logs WHERE created_at >= '2023-11-15' AND created_at < '2023-11-20';
```

**What to look for:**
In the `EXPLAIN` output, you should only see `Append` nodes containing the specific partitions (e.g., `production_logs_2023_11`). If you see it scanning partitions from 2022, partition pruning has failed.

**Common causes of failed partition pruning:**
1.  **Functions on the partition key:** `WHERE DATE(created_at) = '2023-11-15'`. The `DATE()` function prevents the planner from using the partition bounds. Use range queries instead: `WHERE created_at >= '2023-11-15' AND created_at < '2023-11-16'`.
2.  **Data type mismatches:** If `created_at` is `TIMESTAMPTZ` but you compare it to a `DATE` without explicit casting, pruning might fail. Always ensure data types match exactly.
3.  **Prepared Statements:** In older versions of PostgreSQL, generic plans for prepared statements might not prune partitions effectively.
4.  **Volatile Functions:** Using functions like `NOW()` in the WHERE clause can sometimes interfere with pruning depending on the planner version.

---

## 5. Advanced CLI One-Liners for Operations

In the heat of a production incident, you don't have time to write complex scripts. You need fast, reliable CLI one-liners to diagnose and mitigate issues. These commands assume a Unix-like environment and standard database client tools (`psql`, `mysql`).

### 5.1 PostgreSQL Emergency Diagnostics

**1. Find the longest-running active queries (The "What is locking the database?" query):**
```bash
psql -U postgres -d mydb -c "
SELECT pid, age(clock_timestamp(), query_start) AS duration, usename, state, query 
FROM pg_stat_activity 
WHERE state != 'idle' AND query NOT ILIKE '%pg_stat_activity%' 
ORDER BY duration DESC LIMIT 10;"
```
*Why it matters:* This is the first command to run during a latency spike. It identifies queries that are hogging resources or holding locks.

**2. Kill a specific runaway query (Graceful termination):**
```bash
psql -U postgres -d mydb -c "SELECT pg_cancel_backend(<PID>);"
```
*Why it matters:* Sends a SIGINT to the backend process, asking it to stop the current query but keeping the connection alive.

**3. Kill a specific runaway query (Forceful termination - use with caution):**
```bash
psql -U postgres -d mydb -c "SELECT pg_terminate_backend(<PID>);"
```
*Why it matters:* Sends a SIGTERM, killing the connection entirely. Use this if `pg_cancel_backend` fails.

**4. Identify tables with the most bloat (Dead tuples):**
```bash
psql -U postgres -d mydb -c "
SELECT relname AS table_name, n_dead_tup AS dead_tuples, n_live_tup AS live_tuples, 
       ROUND((n_dead_tup::numeric / NULLIF(n_live_tup + n_dead_tup, 0)) * 100, 2) AS bloat_ratio 
FROM pg_stat_user_tables 
ORDER BY n_dead_tup DESC LIMIT 10;"
```
*Action:* If bloat is high (e.g., > 20%), you may need to tune autovacuum settings or run a manual `VACUUM`. If bloat is extreme, `pg_repack` might be necessary.

**5. Check replication lag (Crucial for read-replica consistency):**
```bash
# Run on the primary node
psql -U postgres -c "
SELECT client_addr, state, sync_state, 
       pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn) AS lag_bytes 
FROM pg_stat_replication;"
```
*Why it matters:* High replication lag means read replicas are serving stale data, and in synchronous replication setups, it can block writes on the primary.

**6. Find blocking locks (Who is blocking whom?):**
```bash
psql -U postgres -d mydb -c "
SELECT blocked_locks.pid AS blocked_pid,
       blocked_activity.usename AS blocked_user,
       blocking_locks.pid AS blocking_pid,
       blocking_activity.usename AS blocking_user,
       blocked_activity.query AS blocked_statement,
       blocking_activity.query AS current_statement_in_blocking_process
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks 
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
WHERE NOT blocked_locks.granted;"
```

### 5.2 MySQL/InnoDB Emergency Diagnostics

**1. Show full process list (Identify stuck queries):**
```bash
mysql -u root -p -e "SHOW FULL PROCESSLIST;" | grep -v "Sleep" | sort -nr -k 6 | head -n 20
```

**2. Check InnoDB Engine Status (Deep dive into deadlocks and locks):**
```bash
mysql -u root -p -e "SHOW ENGINE INNODB STATUS\G" | grep -A 20 "LATEST DETECTED DEADLOCK"
```

**3. Find tables lacking primary keys (A major performance killer in InnoDB):**
```bash
mysql -u root -p -e "
SELECT tables.table_schema, tables.table_name 
FROM information_schema.tables 
LEFT JOIN (
    SELECT table_schema, table_name 
    FROM information_schema.statistics 
    GROUP BY table_schema, table_name, index_name 
    HAVING SUM(CASE WHEN non_unique = 0 AND column_name = 'id' THEN 1 ELSE 0 END) = 1
) AS pks ON tables.table_schema = pks.table_schema AND tables.table_name = pks.table_name 
WHERE pks.table_name IS NULL AND tables.table_schema NOT IN ('information_schema', 'mysql', 'performance_schema', 'sys');"
```
*Why it matters:* InnoDB uses clustered indexes. Without a primary key, it creates a hidden 6-byte row ID, which can become a massive bottleneck for concurrent inserts.

### 5.3 OS-Level Database Diagnostics

Sometimes the issue isn't the database software, but the underlying operating system resources.

**1. Monitor disk I/O specifically for the database process:**
```bash
# Assuming PostgreSQL is running
pidstat -d -p $(pgrep -d, -x postgres) 1
```
*Why it matters:* Helps determine if the database is I/O bound. Look for high `kB_rd/s` or `kB_wr/s`.

**2. Check for Out-Of-Memory (OOM) kills:**
If the database suddenly restarted, the Linux OOM killer might have terminated it due to excessive memory usage (often caused by high `work_mem` settings combined with many connections).
```bash
dmesg -T | grep -i "out of memory"
# OR
grep -i "killed process" /var/log/syslog
```

**3. Track network connections to the database port (e.g., 5432):**
```bash
# Count connections by IP address to identify connection spikes or rogue applications
ss -tn src :5432 | awk '{print $4}' | cut -d: -f1 | sort | uniq -c | sort -nr
```

---

## 6. Handling Huge Datasets and Migrations

Migrating or altering tables with billions of rows requires extreme caution. A simple `ALTER TABLE ADD COLUMN` with a default value can rewrite the entire table, causing hours of downtime.

### 6.1 Adding Columns Safely

**Bad:**
```sql
ALTER TABLE massive_table ADD COLUMN new_status VARCHAR(50) DEFAULT 'pending';
```
*Why it's bad:* In older PostgreSQL versions (pre-11), this rewrites the entire table to physically add the default value to every row, holding an exclusive lock the entire time.

**Good (The Safe Way for all versions):**
```sql
-- 1. Add the column without a default (metadata only, very fast)
ALTER TABLE massive_table ADD COLUMN new_status VARCHAR(50);

-- 2. Set the default for future inserts
ALTER TABLE massive_table ALTER COLUMN new_status SET DEFAULT 'pending';

-- 3. Backfill existing rows in small batches to avoid locking and replication lag
-- Run this in a script, looping until no rows are updated.
UPDATE massive_table 
SET new_status = 'pending' 
WHERE id IN (
    SELECT id FROM massive_table WHERE new_status IS NULL LIMIT 10000
);
```

### 6.2 Data Archival and Deletion

Deleting millions of rows using a single `DELETE` statement will bloat the WAL, cause massive replication lag, and lock rows.

**The Batch Deletion Strategy:**
Always delete in small chunks.

```bash
#!/bin/bash
# A simple bash script to delete old data in batches

DB_NAME="mydb"
BATCH_SIZE=5000
SLEEP_TIME=1 # Sleep to allow vacuuming and replication to catch up

while true; do
    ROWS_DELETED=$(psql -d $DB_NAME -t -c "
        WITH deleted AS (
            DELETE FROM audit_logs 
            WHERE created_at < '2022-01-01' 
            AND id IN (SELECT id FROM audit_logs WHERE created_at < '2022-01-01' LIMIT $BATCH_SIZE)
            RETURNING *
        ) SELECT count(*) FROM deleted;
    ")
    
    # Trim whitespace
    ROWS_DELETED=$(echo $ROWS_DELETED | xargs)
    
    echo "Deleted $ROWS_DELETED rows."
    
    if [ "$ROWS_DELETED" -eq "0" ]; then
        echo "Archival complete."
        break
    fi
    
    sleep $SLEEP_TIME
done
```

### 6.3 Using pg_repack for Zero-Downtime Table Rebuilds

When a table is heavily bloated and `VACUUM FULL` is not an option due to its exclusive lock, `pg_repack` is the industry standard tool. It creates a new table, copies the data, tracks changes using triggers, and then swaps the tables with only a brief lock.

```bash
# Repack a specific table without blocking reads or writes
pg_repack -d mydb -t public.massive_bloated_table -j 4
```
*Note:* `-j 4` uses 4 parallel jobs for the index build phase, speeding up the process significantly. Ensure you have enough disk space (at least equal to the size of the table and its indexes) before running `pg_repack`.

### 6.4 Changing Data Types Safely

Changing a column's data type (e.g., `INT` to `BIGINT`) requires a full table rewrite.

**The Zero-Downtime Approach:**
1.  Add a new column with the new type: `ALTER TABLE t ADD COLUMN new_id BIGINT;`
2.  Create a trigger to keep the new column in sync with the old column for new writes.
3.  Backfill the new column in batches (similar to the backfill script above).
4.  Rename the columns within a single transaction:
    ```sql
    BEGIN;
    ALTER TABLE t RENAME COLUMN id TO old_id;
    ALTER TABLE t RENAME COLUMN new_id TO id;
    COMMIT;
    ```
5.  Drop the old column later.

### 6.5 Dealing with Long-Running Transactions

Long-running transactions prevent autovacuum from cleaning up dead tuples, leading to severe bloat.

**Identify and terminate idle in transaction sessions:**
```sql
SELECT pid, usename, state, age(clock_timestamp(), xact_start) AS duration
FROM pg_stat_activity
WHERE state = 'idle in transaction' AND age(clock_timestamp(), xact_start) > interval '10 minutes';

-- Terminate them
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE state = 'idle in transaction' AND age(clock_timestamp(), xact_start) > interval '10 minutes';
```
*Best Practice:* Set `idle_in_transaction_session_timeout` in `postgresql.conf` to automatically terminate these sessions.

---

## 8. Conclusion

Operating databases at scale requires a fundamental shift in mindset. Operations that are trivial on a 100MB table become catastrophic on a 10TB table. By mastering table partitioning, understanding the intricacies of concurrent index creation, leveraging `EXPLAIN ANALYZE` for deep query insights, and utilizing advanced CLI tools, tech support and SRE teams can maintain high availability and performance even under the most extreme conditions. 

Always remember the golden rules of production database operations:
1.  Never run a DDL statement without understanding its locking implications.
2.  Always batch massive updates or deletes.
3.  Test complex migrations in a staging environment that mirrors production data volumes.
4.  When in doubt, use `EXPLAIN ANALYZE` before executing a query.

This guide serves as your foundation. Keep it accessible during incidents, and continuously update it as your infrastructure evolves.
