# Comprehensive CLI Reference: Go Toolchain, golang-migrate, and Lua Interpreters

## 1. Introduction to Tech Support Operations for Go and Lua

In modern production environments, the ability to rapidly diagnose, debug, and resolve issues is paramount. Tech support operations often require deep knowledge of the underlying toolchains and interpreters used by the applications. This document serves as a comprehensive, highly detailed reference for the Go toolchain (`go build`, `go test`, `go tool pprof`, `go mod`), the `golang-migrate` CLI, and Lua interpreters. It is specifically tailored for production operations, worst-case scenarios, and advanced tech support.

The focus here is not on basic usage, but on the intricate details, undocumented flags, and advanced one-liners that can save a system during a critical outage. Whether you are dealing with memory leaks in a Go microservice, database migration failures, or needing to execute rapid Lua scripts for system diagnostics, this reference provides the necessary operational knowledge.

## 2. Go Toolchain: Advanced Production Operations

The Go toolchain is a powerful suite of utilities that goes far beyond simple compilation. For tech support and operations engineers, mastering these tools is essential for profiling, debugging, and optimizing Go applications in production.

### 2.1 `go build`: Compiling for Production and Debugging

While `go build` is typically used in CI/CD pipelines, operations engineers often need to recompile binaries with specific flags for debugging or to work around production constraints.

**Key Flags for Operations:**

*   `-x`: Prints the commands as they are executed. Crucial for debugging complex build failures in isolated environments.
*   `-gcflags`: Passes flags to the Go compiler.
    *   `-gcflags="all=-N -l"`: Disables optimizations (`-N`) and inlining (`-l`). This is absolutely critical when building a binary that will be attached to a debugger like `delve` in a production environment. Without this, variables may be optimized away, making debugging impossible.
    *   `-gcflags="-m -m"`: Prints optimization decisions, including escape analysis. Useful for diagnosing unexpected memory allocations causing garbage collection pressure.
*   `-ldflags`: Passes flags to the linker.
    *   `-ldflags="-s -w"`: Strips the symbol table (`-s`) and DWARF debugging information (`-w`). Use this to reduce binary size for constrained environments, but **never** use this if you plan to profile or debug the binary later.
    *   `-ldflags="-X main.Version=1.2.3"`: Injects build-time variables. Useful for hot-patching a binary with a specific version string during an emergency release.
*   `-tags`: Specifies build tags. Essential for compiling specific versions of a binary (e.g., enabling a mock database driver for testing in a staging environment).
*   `-race`: Enables the data race detector. While typically used in testing, compiling a canary binary with `-race` and deploying it to a small subset of production traffic can help identify elusive concurrency bugs that only manifest under real-world load. Note: This significantly impacts performance and memory usage.

**Worst-Case Scenario: Emergency Hotfix Compilation**

Imagine a critical bug in production, and the CI/CD pipeline is down. You need to compile a hotfix directly on a jump host and deploy it.

```bash
# Compile for Linux AMD64, injecting the emergency version, disabling optimizations for potential live debugging
GOOS=linux GOARCH=amd64 go build -gcflags="all=-N -l" -ldflags="-X main.Version=EMERGENCY-HOTFIX-01" -o myapp-hotfix ./cmd/myapp
```

### 2.2 `go test`: Beyond Basic Unit Testing

In operations, `go test` is often used to run integration tests against live staging environments or to execute specific benchmarks to validate performance fixes.

**Advanced Operational Usage:**

*   `-run <regexp>`: Runs only tests matching the regular expression. Crucial for isolating a specific failing test without running the entire suite.
*   `-bench <regexp>`: Runs benchmarks. Use this to validate that a performance hotfix actually improves throughput before deploying.
*   `-benchmem`: Prints memory allocation statistics for benchmarks. Essential for identifying memory leaks or excessive allocations.
*   `-cpuprofile <file>`, `-memprofile <file>`, `-mutexprofile <file>`, `-blockprofile <file>`: Generates profiles during test execution. This is often the safest way to profile a specific code path without impacting a live production system.
*   `-count N`: Runs tests multiple times. Useful for identifying flaky tests that only fail intermittently under specific timing conditions.

**Tech Support One-Liner: Isolating a Flaky Integration Test**

```bash
# Run a specific integration test 100 times, failing fast on the first error, to catch a race condition
go test -v -run TestCriticalDatabaseIntegration -count 100 -failfast ./integration
```

### 2.3 `go tool pprof`: The Ultimate Diagnostic Weapon

When a Go application in production is consuming excessive CPU, leaking memory, or deadlocking, `go tool pprof` is the primary diagnostic tool. It analyzes profile data generated by the Go runtime.

**Acquiring Profiles in Production:**

Most production Go applications should expose the `net/http/pprof` endpoints.

*   **CPU Profile:** `curl -o cpu.prof "http://localhost:6060/debug/pprof/profile?seconds=30"`
*   **Heap Profile:** `curl -o heap.prof "http://localhost:6060/debug/pprof/heap"`
*   **Goroutine Profile:** `curl -o goroutine.prof "http://localhost:6060/debug/pprof/goroutine"`
*   **Block Profile:** `curl -o block.prof "http://localhost:6060/debug/pprof/block"`
*   **Mutex Profile:** `curl -o mutex.prof "http://localhost:6060/debug/pprof/mutex"`

**Analyzing Profiles with `go tool pprof`:**

Once you have the profile, you analyze it interactively or generate reports.

```bash
go tool pprof cpu.prof
```

**Key Interactive Commands:**

*   `top`: Shows the top functions consuming resources.
*   `top -cum`: Shows the top functions, including the resources consumed by functions they call (cumulative). This is often more useful than flat `top` for finding the root cause.
*   `list <regexp>`: Shows the annotated source code for functions matching the regular expression, highlighting the exact lines consuming resources.
*   `web`: Generates an SVG graph and opens it in a web browser. This is the most intuitive way to understand complex call graphs.
*   `traces`: Shows the execution traces that led to the profiled events.

**Worst-Case Scenario: Diagnosing a Memory Leak**

The application is OOM-killing every few hours.

1.  Capture a heap profile immediately after startup: `curl -o heap_base.prof ...`
2.  Capture another heap profile right before the expected OOM: `curl -o heap_current.prof ...`
3.  Compare the profiles to see what grew:
    ```bash
    go tool pprof -base heap_base.prof heap_current.prof
    ```
4.  Inside pprof, use `top` and `list` to identify the objects being allocated and not garbage collected.

**Tech Support One-Liner: Generating a Flame Graph**

Flame graphs are excellent for visualizing CPU usage.

```bash
# Requires graphviz to be installed
go tool pprof -http=:8080 cpu.prof
# Navigate to http://localhost:8080/ui/flamegraph
```

### 2.4 `go mod`: Dependency Management in Crisis

Dependency issues can halt deployments and cause unpredictable behavior. Operations engineers must know how to manipulate `go.mod` and `go.sum` effectively.

**Critical Commands:**

*   `go mod tidy`: Adds missing and removes unused modules. Always run this after manually editing `go.mod`.
*   `go mod vendor`: Creates a `vendor` directory containing all dependencies. This is crucial for air-gapped environments or when upstream repositories are unavailable (e.g., GitHub is down).
*   `go mod verify`: Verifies that the dependencies in the module cache match the cryptographic hashes in `go.sum`. Use this if you suspect a dependency has been tampered with or corrupted.
*   `go mod graph`: Prints the module requirement graph. Useful for understanding complex transitive dependencies.
*   `go mod why -m <module>`: Explains why a specific module is in the dependency graph. Essential for tracking down unwanted or vulnerable transitive dependencies.

**Worst-Case Scenario: Upstream Dependency Disappears**

A critical deployment is failing because an upstream dependency repository was deleted.

1.  If you have a previous successful build, extract the dependency from the Go module cache (`$GOPATH/pkg/mod`) on the build server.
2.  Copy it to a local directory.
3.  Use the `replace` directive in `go.mod` to point to the local copy:
    ```go
    replace github.com/deleted/repo => ../local-copy-of-repo
    ```
4.  Run `go mod vendor` to ensure it's packaged with the application.

## 3. `golang-migrate` CLI: Database Operations and Recovery

Database migrations are often the most fragile part of a deployment. The `golang-migrate` CLI is a standard tool for managing these migrations. Operations engineers must be prepared to handle failed migrations, dirty database states, and rollbacks.

### 3.1 Core Migration Operations

*   `migrate -path ./migrations -database "$DB_URL" up`: Applies all pending up migrations.
*   `migrate -path ./migrations -database "$DB_URL" down`: Reverts all migrations. **Use with extreme caution in production.**
*   `migrate -path ./migrations -database "$DB_URL" up N`: Applies the next `N` up migrations.
*   `migrate -path ./migrations -database "$DB_URL" down N`: Reverts the last `N` down migrations. This is the standard way to rollback a specific failed deployment.

### 3.2 Handling the "Dirty" State

When a migration fails midway (e.g., due to a syntax error or a timeout), `golang-migrate` marks the database as "dirty". It will refuse to run further migrations until the state is resolved. This is the most common tech support scenario for migrations.

**The Recovery Process:**

1.  **Identify the Failure:** Check the application logs or the migration output to determine exactly which migration failed and why.
2.  **Manual Cleanup:** Connect to the database manually and revert any partial changes made by the failed migration. If the migration was creating a table, drop it. If it was adding a column, remove it. **This step requires deep SQL knowledge and extreme care.**
3.  **Force the Version:** Once the database schema is manually restored to the state *before* the failed migration, use the `force` command to tell `golang-migrate` the correct current version.
    ```bash
    # If migration 004 failed, force the version back to 003
    migrate -path ./migrations -database "$DB_URL" force 3
    ```
4.  **Fix and Retry:** Fix the SQL in the failed migration file, and then run `migrate up` again.

### 3.3 Advanced `golang-migrate` Techniques

*   **Creating Migrations:** `migrate create -ext sql -dir ./migrations -seq add_users_table`. Always use sequential naming (`-seq`) to avoid conflicts.
*   **Using TLS:** For production databases, always use TLS. The connection string must include the appropriate parameters (e.g., `sslmode=verify-full`).
*   **Timeouts:** Migrations on large tables can take a long time. Use the `statement-timeout` parameter in the connection string (if supported by the driver) to prevent migrations from hanging indefinitely and locking tables.

**Tech Support One-Liner: Checking Migration Status**

```bash
# Quickly check the current migration version and dirty state
migrate -path ./migrations -database "$DB_URL" version
```

## 4. Lua Interpreters: Advanced One-Liners for Operations

Lua is a lightweight, embeddable scripting language frequently used in infrastructure tools (like Redis, Nginx/OpenResty, and HAProxy) and for rapid system scripting. Operations engineers can leverage Lua one-liners for quick data manipulation, system checks, and interacting with these embedded environments.

### 4.1 Standard Lua Interpreter (`lua`)

The standard `lua` executable is excellent for quick text processing and system interactions when tools like `awk` or `sed` are insufficient or too complex.

**Advanced One-Liners:**

*   **Parsing JSON (requires `lua-cjson`):**
    ```bash
    cat data.json | lua -l cjson -e 'local d = cjson.decode(io.read("*a")); for k,v in pairs(d) do print(k,v) end'
    ```
*   **Calculating MD5 Hashes of Files in a Directory:**
    ```bash
    ls *.txt | lua -e 'for f in io.lines() do local h = io.popen("md5sum " .. f):read("*a"); print(h) end'
    ```
*   **Generating Random Passwords:**
    ```bash
    lua -e 'math.randomseed(os.time()); local chars="abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789!@#$%^&*()"; local pw=""; for i=1,16 do local r=math.random(1,#chars); pw=pw..chars:sub(r,r) end; print(pw)'
    ```
*   **Monitoring System Load (Linux):**
    ```bash
    lua -e 'while true do local f=io.open("/proc/loadavg","r"); print(f:read("*a")); f:close(); os.execute("sleep 1") end'
    ```

### 4.2 LuaJIT: High-Performance Scripting

LuaJIT is a Just-In-Time compiler for Lua, offering significantly higher performance. It is the engine behind OpenResty. It includes the FFI (Foreign Function Interface) library, allowing Lua to call C functions directly without writing C bindings.

**Advanced FFI One-Liners (Linux):**

*   **Calling `gettimeofday` directly via C for microsecond precision:**
    ```bash
    luajit -e 'local ffi = require("ffi"); ffi.cdef[[struct timeval { long tv_sec; long tv_usec; }; int gettimeofday(struct timeval *tv, void *tz);]]; local tv = ffi.new("struct timeval"); ffi.C.gettimeofday(tv, nil); print(tv.tv_sec .. "." .. string.format("%06d", tv.tv_usec))'
    ```
*   **Reading a file directly using C `open`/`read` (bypassing Lua's IO for specific low-level needs):**
    ```bash
    luajit -e 'local ffi = require("ffi"); ffi.cdef[[int open(const char *pathname, int flags); int read(int fd, void *buf, size_t count); int close(int fd);]]; local fd = ffi.C.open("/etc/hostname", 0); local buf = ffi.new("char[256]"); local n = ffi.C.read(fd, buf, 256); print(ffi.string(buf, n)); ffi.C.close(fd)'
    ```

### 4.3 Redis Lua Scripting (`EVAL`)

Redis uses Lua for atomic server-side scripting. This is crucial for complex operations that must be executed without race conditions.

**Tech Support Scenarios in Redis:**

*   **Atomic Rate Limiting (Token Bucket):**
    ```lua
    -- KEYS[1]: rate limit key, ARGV[1]: capacity, ARGV[2]: refill rate, ARGV[3]: current time
    local key = KEYS[1]
    local capacity = tonumber(ARGV[1])
    local refill_rate = tonumber(ARGV[2])
    local now = tonumber(ARGV[3])

    local last_tokens = tonumber(redis.call('hget', key, 'tokens') or capacity)
    local last_refreshed = tonumber(redis.call('hget', key, 'last_refreshed') or now)

    local delta = math.max(0, now - last_refreshed)
    local tokens = math.min(capacity, last_tokens + (delta * refill_rate))

    if tokens >= 1 then
        tokens = tokens - 1
        redis.call('hset', key, 'tokens', tokens)
        redis.call('hset', key, 'last_refreshed', now)
        return 1 -- Allowed
    else
        return 0 -- Rate limited
    end
    ```
    *Execution:* `redis-cli --eval ratelimit.lua mykey , 10 1 1678886400`

*   **Bulk Deletion by Pattern (Safer than `KEYS` in production):**
    Using `SCAN` inside a Lua script to delete keys matching a pattern without blocking the Redis server for a long time.
    ```lua
    local cursor = "0"
    local pattern = ARGV[1]
    local count = 0
    repeat
        local result = redis.call('SCAN', cursor, 'MATCH', pattern, 'COUNT', 1000)
        cursor = result[1]
        local keys = result[2]
        if #keys > 0 then
            redis.call('DEL', unpack(keys))
            count = count + #keys
        end
    until cursor == "0"
    return count
    ```
    *Execution:* `redis-cli --eval bulk_delete.lua , "session:*"`

## 5. Worst-Case Scenarios and Tech Support Operations

This section outlines comprehensive strategies for handling severe production incidents involving the technologies discussed.

### 5.1 Scenario: The "Zombie" Go Process

**Symptoms:** A Go application is running, consuming 100% CPU on multiple cores, but is completely unresponsive to network requests. It cannot be gracefully shut down.

**Diagnosis and Resolution:**

1.  **Attempt pprof:** Try to grab a CPU profile or goroutine dump via `net/http/pprof`. If the process is completely deadlocked, this might time out.
2.  **Send SIGQUIT:** If pprof fails, send a `SIGQUIT` signal to the process.
    ```bash
    kill -QUIT <pid>
    ```
    Unlike `SIGKILL` or `SIGTERM`, `SIGQUIT` forces the Go runtime to dump the stack traces of *all* currently running goroutines to standard error before exiting.
3.  **Analyze the Dump:** Redirect the standard error to a file and analyze the stack traces. Look for goroutines stuck in `runtime.gopark` (waiting on channels or mutexes) or in infinite loops. This is the definitive way to identify deadlocks.
4.  **Restart:** Once the dump is captured, the process will have exited. Restart the service.

### 5.2 Scenario: Corrupted Database Migrations During a Major Outage

**Symptoms:** A deployment failed, the database is in a dirty state, and the application is down. The original migration author is unavailable.

**Diagnosis and Resolution:**

1.  **Stop the Bleeding:** Ensure no automated systems are trying to retry the deployment or run migrations.
2.  **Assess the Damage:** Use `migrate version` to confirm the dirty state. Connect to the database and inspect the schema. Compare it to the expected schema of the failed migration.
3.  **Manual Intervention:** This is the critical step. You must manually execute SQL commands to revert the partial changes.
    *   *Example:* If the migration was `ALTER TABLE users ADD COLUMN phone VARCHAR(20); CREATE INDEX idx_phone ON users(phone);` and it failed on the index creation, you must manually run `ALTER TABLE users DROP COLUMN phone;`.
4.  **Force State:** Run `migrate force <previous_version>`.
5.  **Verify:** Run the application locally or in a staging environment against a snapshot of the production database to ensure the rollback was successful.
6.  **Deploy Previous Version:** Deploy the last known good version of the application.

### 5.3 Scenario: Redis Overload due to Bad Lua Script

**Symptoms:** Redis CPU is at 100%, and all commands are timing out. The `SLOWLOG` indicates a specific `EVAL` command is taking seconds to execute.

**Diagnosis and Resolution:**

1.  **Identify the Script:** Use `redis-cli SLOWLOG GET 10` to identify the problematic Lua script.
2.  **Kill the Script:** Redis is single-threaded. A long-running Lua script blocks everything. You cannot use normal commands. You must use `SCRIPT KILL`.
    ```bash
    redis-cli SCRIPT KILL
    ```
    *Note:* `SCRIPT KILL` only works if the script has not yet performed any write operations. If it has written data, Redis will refuse to kill it to maintain data consistency.
3.  **The Nuclear Option:** If `SCRIPT KILL` fails because writes have occurred, the only way to recover the Redis server is to shut it down forcefully.
    ```bash
    redis-cli SHUTDOWN NOSAVE
    ```
    **Warning:** This will result in data loss for any data not yet persisted to disk. This is a true worst-case scenario action.
4.  **Fix the Script:** Analyze the script. Common issues include infinite loops, iterating over massive datasets without yielding, or using inefficient algorithms. Rewrite the script to be O(1) or O(N) with a small N.

## 6. Relation to Other Specialist Files

This document (`42-go-lua-cli-reference.md`) serves as the deep-dive technical reference for specific execution environments (Go and Lua) and database state management (`golang-migrate`). It is a critical component of the broader specialist-teams repository.

*   **Relation to Incident Response (e.g., `01-incident-response-playbook.md`):** When the incident response playbook dictates that a deep technical investigation is required for a Go service or a database migration failure, the responder will consult this CLI reference for the exact commands and diagnostic techniques.
*   **Relation to Infrastructure as Code (e.g., `15-terraform-kubernetes-ops.md`):** While Terraform and Kubernetes manage the deployment of the applications, this document covers the internal operations of the applications themselves. If a Kubernetes pod running a Go application is crash-looping, this reference provides the tools (`go build -gcflags`, `pprof`) to figure out *why*.
*   **Relation to Monitoring and Observability (e.g., `22-prometheus-grafana-alerts.md`):** Observability tools will trigger alerts (e.g., "High Memory Usage"). This CLI reference provides the next step: how to use `go tool pprof` to capture a heap profile and identify the exact line of code causing the memory leak detected by Prometheus.
*   **Relation to Security Operations (e.g., `30-security-auditing-tools.md`):** The `go mod` commands detailed here are essential for security audits, specifically for verifying dependency integrity (`go mod verify`) and tracking down vulnerable transitive dependencies (`go mod why`).
*   **Relation to Database Administration (e.g., `18-postgresql-dba-guide.md`):** The `golang-migrate` section directly complements general DBA guides. While the DBA guide covers performance tuning and backups, this document covers the specific lifecycle of schema changes and how to recover when those changes fail during application deployments.
*   **Relation to General Scripting (e.g., `05-bash-python-automation.md`):** The Lua section provides an alternative to Bash and Python for specific high-performance or embedded scripting scenarios, particularly when interacting with systems like Redis or OpenResty where Lua is the native scripting language.

This comprehensive reference ensures that when operations engineers face complex, low-level issues in Go or Lua environments, they have the exact commands, flags, and strategies needed to diagnose and resolve the problem efficiently.
