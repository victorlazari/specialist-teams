# Specialist Guide: Go Runtime, Golang-Migrate, and Redis Lua Configuration Schemas

## 1. Introduction and Scope

This document serves as the definitive tech support and operations guide for configuring, tuning, and troubleshooting the Go runtime environment, database migration pipelines using `golang-migrate`, and Redis Lua scripting limits. Designed for Site Reliability Engineers (SREs), DevOps professionals, and Tier 3 Tech Support, this guide dives deep into production-grade configurations, worst-case scenarios, and practical recovery strategies.

As part of the broader specialist-teams repository, this guide intersects with database connection pooling, memory management, and distributed caching strategies. It provides the necessary schemas and tuning recommendations to ensure high availability, optimal performance, and resilience under extreme load.

## 2. Go Runtime Tuning: GOGC and GOMAXPROCS

The Go runtime is highly optimized out of the box, but at scale, default settings often lead to suboptimal performance, excessive memory consumption, or CPU throttling. The two most critical environment variables for tuning the Go runtime are `GOGC` and `GOMAXPROCS`.

### 2.1. GOGC: Garbage Collection Tuning

The `GOGC` variable controls the aggressiveness of the Go garbage collector (GC). It sets the target percentage of heap growth before the next GC cycle is triggered. The default value is `100`, meaning the GC runs when the heap size doubles.

#### 2.1.1. Configuration Schema and Recommendations

| Setting | Value | Use Case | Pros | Cons |
|---------|-------|----------|------|------|
| Default | `100` | General purpose applications. | Balanced CPU and memory usage. | May cause latency spikes in high-throughput systems. |
| Aggressive | `50` | Memory-constrained environments (e.g., small containers). | Keeps memory footprint low. | High CPU overhead due to frequent GC cycles. |
| Relaxed | `200` - `1000` | High-throughput, latency-sensitive APIs with ample memory. | Reduces GC frequency, lowering CPU usage and latency. | Higher memory consumption; risk of OOM kills if unbounded. |
| Disabled | `off` | Short-lived batch jobs or highly specialized systems with manual memory management. | Zero GC overhead. | Guaranteed OOM if memory is not managed manually. |

#### 2.1.2. Production Operations and Worst-Case Scenarios

**Scenario: CPU Starvation due to GC Thrashing**
In high-throughput microservices, a default `GOGC=100` can lead to the GC running continuously, consuming up to 25% of available CPU resources. This is known as GC thrashing.
*   **Symptoms:** High CPU utilization, increased p99 latency, and frequent GC pauses visible in profiling tools (`pprof`).
*   **Resolution:** Increase `GOGC` to `200` or `400` to reduce GC frequency. Ensure the container has sufficient memory limits to accommodate the larger heap. Alternatively, implement the `GOMEMLIMIT` variable (introduced in Go 1.19) to set a soft memory limit, allowing you to safely increase `GOGC` without risking OOM kills.

**Scenario: OOM Kills in Kubernetes**
A service with `GOGC=200` experiences sudden spikes in traffic, causing the heap to grow rapidly. The container exceeds its Kubernetes memory limit and is OOMKilled before the GC can reclaim memory.
*   **Symptoms:** Pod restarts with `OOMKilled` status.
*   **Resolution:** Set `GOMEMLIMIT` to approximately 80-90% of the container's memory limit. This forces the GC to run aggressively as the memory approaches the limit, preventing the OOM kill while maintaining the benefits of a higher `GOGC` during normal operation.

### 2.2. GOMAXPROCS: Concurrency and CPU Allocation

The `GOMAXPROCS` variable determines the maximum number of operating system threads that can execute user-level Go code simultaneously. By default, it is set to the number of logical CPUs available to the process.

#### 2.2.1. Configuration Schema and Recommendations

| Environment | Recommended Setting | Rationale |
|-------------|---------------------|-----------|
| Bare Metal / VM | Default (Number of logical CPUs) | Maximizes hardware utilization. |
| Kubernetes / Docker (CPU Limits) | Use `automaxprocs` library or set manually to the CPU quota. | Prevents CPU throttling. The Go runtime is unaware of cgroup limits by default and may spawn too many threads, leading to severe context switching and throttling. |
| I/O Bound Services | Default or slightly higher | Go's scheduler handles I/O efficiently, but in extreme cases, increasing threads can help if many goroutines are blocked on CGO or syscalls. |

#### 2.2.2. Production Operations and Worst-Case Scenarios

**Scenario: Severe CPU Throttling in Kubernetes**
A Go application deployed in Kubernetes with a CPU limit of `2.0` (2 cores) is running on a node with 64 cores. By default, `GOMAXPROCS` is set to 64. The application spawns 64 threads, rapidly consuming its CPU quota and getting heavily throttled by the Linux CFS (Completely Fair Scheduler).
*   **Symptoms:** Extremely high latency, poor throughput, and high CPU throttling metrics in Prometheus (`container_cpu_cfs_throttled_seconds_total`).
*   **Resolution:** Integrate the `go.uber.org/automaxprocs` package, which automatically reads the cgroup CPU limits and sets `GOMAXPROCS` accordingly (in this case, to 2). This aligns the Go scheduler with the actual available resources, eliminating unnecessary context switching and throttling.

## 3. Golang-Migrate Configurations

`golang-migrate` is a popular tool for managing database schema migrations in Go applications. While powerful, improper configuration can lead to locked databases, failed deployments, and data corruption.

### 3.1. Configuration Schema and Best Practices

When configuring `golang-migrate`, several parameters are critical for production stability:

*   **`x-migrations-table`**: Customizes the name of the migrations tracking table. Useful for multi-tenant databases.
*   **`statement-timeout`**: Sets a timeout for individual migration statements. Crucial for preventing long-running migrations from locking tables indefinitely.
*   **`lock-timeout`**: Defines how long the tool should wait to acquire the migration lock.

#### 3.1.1. Connection String Examples

**PostgreSQL:**
```text
postgres://user:password@host:5432/dbname?sslmode=verify-full&x-migrations-table=schema_migrations&statement_timeout=60000
```

**MySQL:**
```text
mysql://user:password@tcp(host:3306)/dbname?multiStatements=true&x-migrations-table=schema_migrations&readTimeout=1m
```

### 3.2. Production Operations and Worst-Case Scenarios

**Scenario: The "Dirty" Database State**
A migration fails halfway through execution due to a syntax error or a timeout. `golang-migrate` marks the database as "dirty," preventing any further migrations or application startups.
*   **Symptoms:** Application fails to start, logging errors like `Dirty database version X. Fix and force version.`.
*   **Resolution:**
    1.  Investigate the cause of the failure (e.g., check database logs for timeouts or syntax errors).
    2.  Manually revert the partial changes made by the failed migration in the database.
    3.  Use the `golang-migrate` CLI to force the version back to the last successful state: `migrate -path ./migrations -database $DB_URL force <previous_version>`.
    4.  Fix the migration script and redeploy.

**Scenario: Distributed Lock Contention**
In a Kubernetes environment, multiple pods of a new application version start simultaneously and attempt to run migrations. One pod acquires the lock, but crashes before releasing it. Other pods are stuck waiting for the lock.
*   **Symptoms:** Pods are stuck in `CrashLoopBackOff` or readiness probes fail because migrations cannot proceed. Database shows an active lock in the `schema_migrations` table.
*   **Resolution:**
    1.  Identify and terminate the crashed pod or process holding the lock.
    2.  Manually clear the lock in the database. For PostgreSQL, this involves updating the `schema_migrations` table to set `is_locked = false`.
    3.  To prevent this, decouple migrations from application startup. Run migrations as a Kubernetes `Job` or an init container that executes before the main application pods are rolled out.

## 4. Redis Lua Script Limits and Tuning

Redis supports executing Lua scripts server-side, providing atomicity and reducing network round trips. However, because Redis is single-threaded, a poorly written or long-running Lua script can block the entire server, causing widespread outages.

### 4.1. Configuration Schema and Limits

Redis imposes several limits and configurations to manage Lua script execution:

*   **`lua-time-limit`**: The maximum execution time for a Lua script in milliseconds. The default is `5000` (5 seconds). If a script exceeds this limit, Redis starts logging warnings and accepting `SCRIPT KILL` commands.
*   **Memory Limits**: Lua scripts consume memory. Redis tracks this, and excessive memory usage within a script can lead to OOM errors.
*   **Deterministic Execution**: Prior to Redis 5.0, scripts had to be purely deterministic (e.g., no random numbers or time-based logic) to ensure safe replication. Redis 5.0 introduced script effects replication, relaxing this constraint.

### 4.2. Production Operations and Worst-Case Scenarios

**Scenario: The Blocking Script Outage**
A developer deploys a Lua script that iterates over a massive Redis set (e.g., millions of elements) using `SMEMBERS` instead of `SSCAN`. The script takes 15 seconds to execute. Because Redis is single-threaded, all other commands from all other clients are blocked for 15 seconds.
*   **Symptoms:** Massive spike in application latency, connection timeouts, and Redis slowlog entries showing the Lua script execution.
*   **Resolution:**
    1.  **Immediate Mitigation:** Connect to Redis via `redis-cli` and execute the `SCRIPT KILL` command. This terminates the running script (provided it hasn't performed any write operations yet). If the script has performed writes, `SCRIPT KILL` will fail, and the only option is to restart the Redis server (`SHUTDOWN NOSAVE`), which may result in data loss.
    2.  **Long-Term Fix:** Rewrite the Lua script to use iterative commands like `SSCAN`, `HSCAN`, or `ZSCAN`. Break large operations into smaller, paginated chunks that yield control back to the Redis event loop.

**Scenario: Script Cache Exhaustion**
An application dynamically generates Lua scripts with hardcoded values instead of using `KEYS` and `ARGV` arrays. Every execution results in a unique script being loaded into the Redis script cache via `EVAL`.
*   **Symptoms:** Redis memory usage grows unbounded until it hits the `maxmemory` limit, triggering evictions or OOM errors.
*   **Resolution:**
    1.  **Immediate Mitigation:** Execute the `SCRIPT FLUSH` command to clear the script cache and free up memory.
    2.  **Long-Term Fix:** Refactor the application code to use parameterized Lua scripts. Load the script once using `SCRIPT LOAD` and execute it using `EVALSHA`, passing dynamic values via the `KEYS` and `ARGV` arrays. This ensures only one copy of the script resides in the cache.

## 5. Integration with the Specialist Teams Ecosystem

This configuration schema guide is a critical component of the specialist-teams repository. It directly interacts with and supports the other operational domains:

1.  **Database Connection Pooling:** The `golang-migrate` configurations discussed here must align with the connection pooling settings to ensure migration scripts do not exhaust available connections or conflict with application traffic.
2.  **Distributed Caching:** The Redis Lua script limits are essential for maintaining the stability of the distributed caching layer. Blocking scripts directly impact cache retrieval latencies.
3.  **Incident Response:** The worst-case scenarios and recovery strategies outlined for Go runtime OOMs, dirty migrations, and blocked Redis instances form the basis of Tier 3 incident response playbooks.
4.  **Capacity Planning:** Understanding `GOGC` and `GOMAXPROCS` is fundamental for accurate capacity planning and Kubernetes resource allocation (requests and limits).
5.  **Security Operations:** Proper configuration of migration timeouts and Redis script limits prevents denial-of-service (DoS) vectors caused by resource exhaustion.
6.  **Observability:** The symptoms described in the worst-case scenarios dictate the metrics and alerts that must be configured in Prometheus and Grafana (e.g., GC pause times, CPU throttling, Redis slowlog length).

## 6. Conclusion

Mastering the configuration schemas for the Go runtime, `golang-migrate`, and Redis Lua scripting is non-negotiable for operating high-scale, resilient systems. By applying the tuning recommendations and understanding the worst-case scenarios detailed in this guide, operations teams can proactively prevent outages, optimize resource utilization, and respond effectively when complex failures occur. Continuous monitoring, profiling, and load testing are required to validate these configurations as application workloads evolve.
