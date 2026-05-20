# Deep Troubleshooting Guide for Go and Lua: Production Operations & Tech Support

## 1. Introduction to Go and Lua Production Troubleshooting

In modern distributed systems, Go (Golang) and Lua (often embedded in Redis or API gateways) form a powerful combination for high-performance, concurrent, and low-latency operations. However, when these systems fail in production, they fail in spectacular and complex ways. Tech support and site reliability engineering (SRE) teams must be equipped to handle worst-case scenarios, such as cascading failures caused by goroutine leaks, deadlocks that freeze entire microservices, silent memory leaks that trigger Out-Of-Memory (OOM) kills, and Redis instances locked in a `BUSY` state due to runaway Lua scripts.

This comprehensive guide is designed for Level 3 (L3) tech support, SREs, and backend engineers. It provides deep, practical, and battle-tested troubleshooting methodologies for the most critical Go and Lua issues encountered in production environments. We will explore the root causes, detection mechanisms, and remediation strategies for goroutine leaks, deadlocks, race conditions, memory leaks, Redis Lua script timeouts, and complex migration failures.

### Relation to Other Specialist Files
*Note: This section explains how this troubleshooting guide integrates with the broader specialist-teams repository.*
- **01-architecture-overview.md**: Provides the high-level system design where Go microservices and Redis/Lua data stores interact.
- **15-incident-response-playbook.md**: Outlines the communication and escalation protocols when the issues described in this guide cause a Sev-1 incident.
- **23-redis-cluster-management.md**: Details the infrastructure side of Redis, complementing the Lua script troubleshooting covered here.
- **34-observability-and-metrics.md**: Defines the Prometheus and Grafana dashboards used to detect the anomalies (e.g., goroutine spikes, memory growth) discussed in this guide.
- **55-post-mortem-templates.md**: Used to document the root cause and remediation after resolving the deep technical issues outlined in this document.
- **67-database-migration-strategies.md**: Expands on the migration failure scenarios, focusing on relational databases, whereas this guide focuses on Go/Lua state migrations.

---

## 2. Goroutine Leaks: Detection and Remediation

Goroutines are lightweight, but they are not free. A goroutine leak occurs when a goroutine is started but never terminates, usually because it is blocked on a channel operation, waiting for a context that never cancels, or caught in an infinite loop. In production, a goroutine leak will eventually consume all available memory or exhaust system resources, leading to an OOM kill or severe performance degradation.

### 2.1. Root Causes of Goroutine Leaks

1.  **Abandoned Channels**: A goroutine is blocked sending to or receiving from an unbuffered channel that no other goroutine will ever read from or write to.
2.  **Missing Context Cancellation**: A goroutine initiates a long-running or blocking operation (like an HTTP request or database query) without a timeout or a cancellation context. If the external service hangs, the goroutine hangs forever.
3.  **Infinite Loops without Exit Conditions**: A background worker loop `for { ... }` lacks a `select` statement to listen for a shutdown signal or context cancellation.
4.  **Unclosed Response Bodies**: Failing to close `http.Response.Body` can keep the underlying connection and associated goroutines alive.

### 2.2. Detecting Goroutine Leaks in Production

When troubleshooting a suspected goroutine leak, the first indicator is usually a steady, linear increase in memory usage and the number of active goroutines on your Grafana dashboards.

**Step 1: Check Goroutine Count Metrics**
Query your Prometheus metrics for `go_goroutines`. If the graph shows a "staircase" pattern or a continuous upward trend without dropping during off-peak hours, you have a leak.

**Step 2: Capture a Goroutine Profile via pprof**
Go's `net/http/pprof` package is essential for production debugging. If pprof is exposed (e.g., on an internal admin port), capture the goroutine profile:

```bash
curl -s http://localhost:6060/debug/pprof/goroutine?debug=1 > goroutines_summary.txt
curl -s http://localhost:6060/debug/pprof/goroutine?debug=2 > goroutines_full.txt
```

The `debug=1` output provides a summary of goroutines grouped by their current execution point. Look for unusually high counts:

```text
goroutine profile: total 10543
10000 @ 0x435f0e 0x436433 0x46b895 0x4d5e21 0x4d69a5 0x4d698d
#	0x4d5e20	github.com/company/service/worker.processTask+0x120	/app/worker/worker.go:45
#	0x4d69a4	github.com/company/service/worker.Start.func1+0x44	/app/worker/worker.go:22
```

In this example, 10,000 goroutines are stuck at `worker.go:45`.

**Step 3: Analyze the Full Dump**
The `debug=2` output provides the exact stack trace and state of every single goroutine. Search for the function identified in Step 2. You will likely see states like `semacquire` (waiting on a mutex/channel) or `IO wait`.

### 2.3. Remediation and Worst-Case Scenarios

**Immediate Mitigation:**
If the service is critical and failing, the only immediate mitigation is a rolling restart of the affected pods/instances to clear the leaked goroutines. However, you must capture the pprof dump *before* restarting.

**Code-Level Fixes:**
-   **Implement Contexts:** Ensure every blocking operation uses `context.WithTimeout` or `context.WithCancel`.
-   **Audit Channel Operations:** Verify that every channel send/receive has a corresponding receiver/sender, or use `select` with a `default` or timeout case.

```go
// BAD: Can block forever if the channel is unbuffered and no one is reading
func sendResult(ch chan<- Result, res Result) {
    ch <- res
}

// GOOD: Uses a select with a context to prevent leaking
func sendResultSafe(ctx context.Context, ch chan<- Result, res Result) {
    select {
    case ch <- res:
        // Successfully sent
    case <-ctx.Done():
        // Context cancelled, exit to prevent leak
        log.Printf("Failed to send result: %v", ctx.Err())
    }
}
```

---

## 3. Deadlocks and Race Conditions in Go

Concurrency is Go's strongest feature, but improper synchronization leads to deadlocks (where the application freezes) and race conditions (where data becomes corrupted unpredictably).

### 3.1. Diagnosing Deadlocks

A deadlock occurs when two or more goroutines are waiting for each other to release resources, resulting in a complete standstill. In Go, a global deadlock (where *all* goroutines are asleep) will cause the runtime to panic and crash the program. However, a *partial deadlock* (where only some goroutines are stuck) will not crash the program but will degrade functionality.

**Symptoms of a Partial Deadlock:**
-   API endpoints suddenly stop responding and time out.
-   CPU usage drops to near zero, but memory remains allocated.
-   The number of active goroutines spikes and stays flat.

**Troubleshooting Steps:**
1.  **Trigger a Core Dump or Stack Trace:** If the application is unresponsive, send a `SIGQUIT` signal to the Go process. This forces the Go runtime to print the stack traces of all currently running goroutines to standard error and then exit.
    ```bash
    kill -SIGQUIT <pid>
    ```
2.  **Analyze the Stack Trace:** Look for goroutines in the `sync.Mutex.Lock` state.
    ```text
    goroutine 45 [semacquire]:
    sync.runtime_SemacquireMutex(0xc0000b4014, 0x0, 0x1)
        /usr/local/go/src/runtime/sema.go:71 +0x47
    sync.(*Mutex).lockSlow(0xc0000b4010)
        /usr/local/go/src/sync/mutex.go:138 +0x105
    sync.(*Mutex).Lock(...)
        /usr/local/go/src/sync/mutex.go:81
    main.updateCache()
        /app/main.go:120 +0x45
    ```
3.  **Identify the Lock Ordering:** Deadlocks almost always occur due to inconsistent lock ordering (e.g., Goroutine A locks Mutex 1 then Mutex 2; Goroutine B locks Mutex 2 then Mutex 1). Trace the code to ensure locks are always acquired in the exact same order globally.

### 3.2. Hunting Down Race Conditions

Race conditions are insidious because they do not always cause immediate crashes. They cause silent data corruption, inconsistent API responses, and bizarre logic failures.

**The Go Race Detector:**
The absolute best tool for finding race conditions is the built-in Go race detector. However, it introduces significant overhead (up to 10x CPU and memory usage), so it should **never** be run in production.

**Troubleshooting Workflow:**
1.  **Replicate in Staging:** Deploy a build of the application compiled with the `-race` flag to a staging environment that mirrors production traffic.
    ```bash
    go build -race -o myapp main.go
    ```
2.  **Monitor Logs:** When a race condition occurs, the runtime will print a detailed warning to stderr, showing the exact goroutines and memory addresses involved.
    ```text
    WARNING: DATA RACE
    Write at 0x00c0000b2040 by goroutine 7:
      main.incrementCounter()
          /app/main.go:45 +0x3a

    Previous read at 0x00c0000b2040 by goroutine 6:
      main.readCounter()
          /app/main.go:50 +0x2b
    ```
3.  **Fixing the Race:** Protect the shared resource using `sync.Mutex`, `sync.RWMutex`, or atomic operations (`sync/atomic`). Alternatively, refactor the code to use channels to pass data ownership rather than sharing memory.

---

## 4. Go Memory Leaks and Profiling

Unlike C/C++, Go is garbage-collected, meaning memory leaks are rarely caused by forgetting to free memory. Instead, Go memory leaks are usually "logical leaks"—holding onto references of objects that are no longer needed, preventing the garbage collector (GC) from reclaiming them.

### 4.1. Common Causes of Logical Memory Leaks

1.  **Global Maps and Slices:** Appending data to a global slice or map without ever deleting old entries. This is common in poorly implemented in-memory caches.
2.  **Substring and Subslice References:** Slicing a large array or string keeps the *entire* underlying array in memory. If you read a 10MB file into a string and keep only a 10-byte substring, the full 10MB remains in memory.
3.  **Time Tickers:** Creating a `time.Ticker` in a loop or function and failing to call `ticker.Stop()`. The runtime keeps the ticker alive.

### 4.2. Deep Profiling with pprof

When an OOM kill occurs, you must analyze the heap profile.

**Step 1: Capture the Heap Profile**
```bash
curl -s http://localhost:6060/debug/pprof/heap > heap.out
```

**Step 2: Analyze with `go tool pprof`**
```bash
go tool pprof heap.out
```

Inside the interactive pprof shell, use the following commands:
-   `top`: Shows the functions consuming the most memory.
-   `top -cum`: Shows the functions allocating the most memory (including their children).
-   `list <function_name>`: Shows the exact lines of code in a function where allocations occur.
-   `web`: Generates an SVG call graph and opens it in a browser (highly recommended for visual tracing).

**Step 3: Differentiate `inuse_space` vs `alloc_space`**
-   `inuse_space` (default): Shows memory currently allocated and not yet garbage collected. High `inuse_space` indicates a leak.
-   `alloc_space`: Shows all memory allocated since the program started, even if it was garbage collected. High `alloc_space` indicates high allocation churn, which causes high CPU usage due to GC pressure, but not necessarily a leak.

### 4.3. Fixing Subslice Leaks

If pprof shows high memory usage on a line that simply slices a byte array, you have a subslice leak.

```go
// BAD: Keeps the entire 10MB payload in memory
func extractHeader(payload []byte) []byte {
    return payload[0:100] 
}

// GOOD: Allocates a new, small slice and copies the data, allowing the large payload to be GC'd
func extractHeaderSafe(payload []byte) []byte {
    header := make([]byte, 100)
    copy(header, payload[0:100])
    return header
}
```

---

## 5. Redis Lua Script Timeouts (The BUSY State)

Redis is single-threaded. When a Lua script executes via `EVAL` or `EVALSHA`, it blocks all other Redis commands until it completes. This guarantees atomicity, but it also means a poorly written Lua script can take down your entire Redis cluster.

### 5.1. The `BUSY` State Explained

By default, Redis has a `lua-time-limit` configuration (usually 5000 milliseconds). If a Lua script runs longer than this limit, Redis does *not* automatically kill it. Instead, Redis enters a `BUSY` state.

In the `BUSY` state:
-   Redis stops processing normal commands.
-   Clients receive the error: `BUSY Redis is busy running a script. You can only call SCRIPT KILL or SHUTDOWN NOSAVE.`
-   Your Go microservices will start throwing connection timeouts, leading to cascading failures across the system.

### 5.2. Root Causes of Slow Lua Scripts

1.  **Infinite Loops:** A `while` or `repeat` loop in Lua that fails to meet its exit condition.
2.  **Massive Iterations:** Iterating over a massive Redis Set or Hash (e.g., using `SMEMBERS` or `HGETALL` on a key with millions of elements) inside the script.
3.  **Complex Logic:** Performing heavy computational tasks (like complex JSON parsing or cryptography) inside Lua instead of in the Go application layer.

### 5.3. Incident Response: Recovering from a BUSY State

When PagerDuty alerts you that Redis is `BUSY`, you must act immediately.

**Step 1: Attempt `SCRIPT KILL`**
Connect to the Redis instance via `redis-cli` and execute:
```bash
redis-cli SCRIPT KILL
```
*Note:* `SCRIPT KILL` only works if the Lua script has *not* yet performed any write operations (e.g., `SET`, `DEL`). If it has only performed reads, Redis will kill the script and resume normal operations.

**Step 2: The Worst-Case Scenario (`SHUTDOWN NOSAVE`)**
If the script has already performed a write operation, Redis refuses to kill it to prevent data inconsistency. `SCRIPT KILL` will return an error.

In this catastrophic scenario, your only option is to forcefully terminate the Redis instance:
```bash
redis-cli SHUTDOWN NOSAVE
```
This will kill the Redis process without saving the dataset to disk. You will lose any data written since the last RDB snapshot or AOF rewrite. The Redis instance will then be restarted by your orchestrator (e.g., Kubernetes or systemd), and it will load the last valid state from disk.

### 5.4. Prevention and Best Practices

To prevent Lua script timeouts:
-   **Never use `KEYS *`, `SMEMBERS`, or `HGETALL` on large datasets inside Lua.** Use `SSCAN` or `HSCAN` instead, or process the data in Go.
-   **Keep scripts small and fast.** Lua in Redis is for atomicity, not heavy computation.
-   **Test with production-sized data.** A script that runs in 1ms on a developer's machine might take 10 seconds on a production dataset.
-   **Use `EVALSHA` instead of `EVAL`.** Loading the script once and calling it by its SHA1 hash saves bandwidth and parsing time.

---

## 6. Migration Failures and Lua/Go Integration

Migrating data structures or logic that relies heavily on the Go/Lua boundary is fraught with peril. A common scenario is migrating from a legacy Go-based locking mechanism to a Redis Lua-based distributed lock, or changing the schema of data manipulated by Lua scripts.

### 6.1. The "Split-Brain" Migration Failure

When migrating logic, you often have a period where both the old Go logic and the new Lua logic are running simultaneously (e.g., during a canary deployment). If the two systems do not perfectly agree on the state of the data, you get a split-brain scenario.

**Example Scenario:**
You are migrating a rate-limiter. The old version uses Go's in-memory counters. The new version uses a Redis Lua script. During the rollout, some pods use the old logic, some use the new. A user's requests are routed to both, effectively doubling their allowed rate limit because the state is split between Go memory and Redis.

**Troubleshooting and Resolution:**
1.  **Halt the Rollout:** Immediately pause the deployment. Do not roll back yet, as rolling back might cause further state corruption depending on the migration design.
2.  **Analyze the State:** Inspect the Redis keys created by the Lua script and compare them to the logs/metrics of the Go in-memory state.
3.  **Force a Single Source of Truth:** The remediation requires forcing all traffic to use a single source of truth. If the Redis state is corrupted, you may need to flush the specific rate-limit keys and fail back to the Go implementation entirely until the bug is fixed.

### 6.2. Lua Script Versioning and Cache Poisoning

When you update a Lua script in your Go code, you change its SHA1 hash.

**The Failure Mode:**
1.  Go Service v1 uses Lua Script A (Hash A).
2.  You deploy Go Service v2, which uses Lua Script B (Hash B).
3.  During the rolling update, both v1 and v2 are running.
4.  If Script B modifies a Redis key in a way that Script A cannot understand (e.g., changing a value from an integer to a JSON string), Service v1 will start throwing errors when it tries to read that key. This is a form of cache poisoning.

**Troubleshooting:**
The logs will show Go services failing to unmarshal or parse Redis responses.
```text
ERR Error running script (call to f_8a7b...): @user_script:1: WRONGTYPE Operation against a key holding the wrong kind of value
```

**Safe Migration Strategy:**
To prevent this, Lua script migrations must be backward compatible.
-   **Phase 1:** Deploy Go Service v2. Script B writes to the *new* format but can read *both* the old and new formats. Script A continues to write and read the old format.
-   **Phase 2:** Once v1 is fully deprecated, deploy Go Service v3. Script C only reads and writes the new format.
-   **Phase 3:** Run a background Go worker to clean up any lingering old-format data.

### 6.3. Handling Network Partitions During Migrations

If a network partition occurs between your Go microservices and Redis during a critical Lua script execution, the Go context will time out, but the Lua script might still execute successfully on the Redis server.

**The Problem:**
The Go application thinks the operation failed and might retry it. If the Lua script is not idempotent, the retry will cause data duplication or corruption (e.g., charging a customer twice).

**The Solution:**
Every Lua script executed from Go **must be idempotent**.
Pass a unique transaction ID (UUID) from Go to the Lua script. The Lua script should check if this transaction ID has already been processed (e.g., by storing it in a Redis Set with an expiration).

```lua
-- Idempotent Lua Script Example
local tx_id = ARGV[1]
local key = KEYS[1]

-- Check if already processed
if redis.call("SISMEMBER", "processed_txs", tx_id) == 1 then
    return "ALREADY_PROCESSED"
end

-- Perform the actual operation
redis.call("INCR", key)

-- Mark as processed
redis.call("SADD", "processed_txs", tx_id)
redis.call("EXPIRE", "processed_txs", 3600)

return "SUCCESS"
```

---

## 7. Advanced Debugging Techniques

### 7.1. Using `strace` on Go Processes

When pprof isn't enough (e.g., the process is completely locked up and the HTTP server serving pprof is dead), you can use `strace` to see what the Go runtime is doing at the OS level.

```bash
strace -p <pid> -c
```
This provides a summary of system calls. If you see a massive number of `futex` calls, the application is heavily contending on locks (mutexes). If you see `epoll_wait` dominating, the application is mostly idle, waiting for network I/O.

### 7.2. Redis `MONITOR` and `SLOWLOG`

To debug Lua scripts interacting with Redis, use `SLOWLOG`.

```bash
redis-cli SLOWLOG GET 10
```
This shows the 10 slowest queries. It will reveal exactly which `EVALSHA` commands are taking too long, along with their arguments.

**Warning:** Never use the `MONITOR` command in a high-throughput production environment. It streams every single command processed by Redis to the client, which can reduce Redis throughput by over 50% and cause the very outages you are trying to prevent.

## 8. Conclusion

Troubleshooting Go and Lua in production requires a deep understanding of concurrency, memory management, and distributed state. By mastering pprof for goroutine and memory analysis, understanding the catastrophic implications of the Redis `BUSY` state, and designing idempotent, backward-compatible Lua scripts, SREs and tech support teams can rapidly mitigate Sev-1 incidents and build highly resilient systems. Always prioritize observability—you cannot fix what you cannot see.
