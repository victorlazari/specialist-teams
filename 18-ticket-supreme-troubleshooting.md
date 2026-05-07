# Ticket Supreme: Comprehensive Troubleshooting & Diagnostics Guide

## 1. Introduction & Overview
Ticket Supreme is an enterprise-grade ticketing, incident management, and customer support platform. This guide provides an exhaustive, deep-dive troubleshooting and diagnostics reference for system administrators, site reliability engineers (SREs), and support personnel. It covers component-level health checks, error codes, performance tuning, network diagnostics, and recovery strategies to ensure high availability and rapid incident resolution.

## 2. Architecture & Component Health Checks
Ticket Supreme operates on a microservices architecture. Understanding the health of each component is critical for effective troubleshooting.

### 2.1 Core Services
- **API Gateway (Supreme-Gateway):** Handles all incoming REST and GraphQL requests.
- **Ticket Engine (Supreme-Core):** Manages the lifecycle of tickets, state transitions, and SLA calculations.
- **Notification Service (Supreme-Notify):** Dispatches emails, SMS, and webhook payloads.
- **Search Indexer (Supreme-Search):** Maintains the Elasticsearch cluster for full-text search capabilities.

### 2.2 Health Check Endpoints
Every microservice exposes a `/health` and `/metrics` endpoint.
- **Liveness Probe:** `GET /health/live` - Returns HTTP 200 if the service is running.
- **Readiness Probe:** `GET /health/ready` - Returns HTTP 200 if the service is ready to accept traffic (e.g., database connections are established).
- **Deep Health Check:** `GET /health/deep` - Performs a comprehensive check of all downstream dependencies.

Example output of a deep health check:
```json
{
  "status": "DEGRADED",
  "components": {
    "database": { "status": "UP", "latency_ms": 12 },
    "cache": { "status": "UP", "latency_ms": 2 },
    "search": { "status": "DOWN", "error": "Connection refused" }
  }
}
```

## 3. Common Error Codes & Resolutions
When Ticket Supreme encounters an issue, it generates specific error codes. Below is a comprehensive list of error codes, their meanings, and resolution steps.

### 3.1 TS-ERR-1000 Series: Authentication & Authorization
- **TS-ERR-1001: Invalid API Key**
  - *Cause:* The provided API key is expired, revoked, or malformed.
  - *Resolution:* Verify the API key in the Supreme Admin Console. Rotate the key if necessary. Ensure the `Authorization` header is correctly formatted as `Bearer <token>`.
- **TS-ERR-1005: Insufficient Permissions**
  - *Cause:* The user or service account lacks the required RBAC roles to perform the action.
  - *Resolution:* Check the user's assigned roles. For ticket deletion, the `ticket.delete` permission is required.

### 3.2 TS-ERR-2000 Series: Ticket Lifecycle
- **TS-ERR-2042: State Transition Invalid**
  - *Cause:* Attempted to move a ticket from `CLOSED` to `IN_PROGRESS` without the `reopen` flag.
  - *Resolution:* Ensure the workflow rules allow the transition. Use the `/api/v2/tickets/{id}/reopen` endpoint instead of a standard update.
- **TS-ERR-2050: SLA Calculation Timeout**
  - *Cause:* The SLA engine took too long to compute the due date based on complex business hours and holiday schedules.
  - *Resolution:* Simplify the SLA matrix or increase the `SLA_ENGINE_TIMEOUT_MS` environment variable.

### 3.3 TS-ERR-3000 Series: Database & Storage
- **TS-ERR-3010: Connection Pool Exhausted**
  - *Cause:* The maximum number of database connections has been reached.
  - *Resolution:* Increase `DB_MAX_CONNECTIONS` in `supreme.conf`. Identify long-running queries using `pg_stat_activity`.
- **TS-ERR-3025: Deadlock Detected**
  - *Cause:* Concurrent transactions attempted to update the same ticket and its associated audit logs in a conflicting order.
  - *Resolution:* The application will automatically retry. If persistent, review custom automation scripts that might be causing race conditions.

## 4. Diagnostic Tools & Commands
Ticket Supreme ships with a suite of CLI tools for diagnostics.

### 4.1 `supreme-cli diag`
The `diag` command runs a full system diagnostic suite.
```bash
supreme-cli diag --level=deep --output=json > diag_report.json
```
This command checks:
- Disk space and inode usage.
- Memory and CPU utilization.
- Database connectivity and replication lag.
- Cache hit ratios.

### 4.2 `supreme-cli net-test`
Tests connectivity between microservices.
```bash
supreme-cli net-test --source=supreme-core --target=supreme-search
```

### 4.3 `supreme-cli log-tail`
Aggregates logs from multiple services in real-time.
```bash
supreme-cli log-tail --services=core,notify --level=ERROR
```

## 5. Database & Storage Troubleshooting
Ticket Supreme relies heavily on PostgreSQL for relational data and Redis for caching and message brokering.

### 5.1 PostgreSQL High CPU Usage
If the PostgreSQL instance is consuming excessive CPU:
1. **Identify Slow Queries:**
   Run the following SQL to find queries taking longer than 1 second:
   ```sql
   SELECT pid, now() - pg_stat_activity.query_start AS duration, query
   FROM pg_stat_activity
   WHERE state = 'active' AND now() - pg_stat_activity.query_start > interval '1 second';
   ```
2. **Analyze Execution Plans:**
   Use `EXPLAIN ANALYZE` on the slow queries. Look for sequential scans (`Seq Scan`) on large tables like `ticket_audit_logs`.
3. **Vacuum and Analyze:**
   Ensure autovacuum is running. Manually run `VACUUM ANALYZE tickets;` if statistics are stale.

### 5.2 Redis Memory Exhaustion
If Redis hits its `maxmemory` limit:
1. **Check Eviction Policy:** Ensure `maxmemory-policy` is set to `volatile-lru` or `allkeys-lru`.
2. **Analyze Memory Usage:**
   Use `redis-cli --bigkeys` to identify large keys. Often, the session cache or rate-limiting counters consume the most memory.
3. **Clear Stale Cache:**
   Run `supreme-cli cache clear --namespace=sessions` to free up memory safely.

## 6. Network & Connectivity Issues
Network partitions or misconfigurations can cause cascading failures.

### 6.1 DNS Resolution Failures
If services cannot resolve each other:
- Check CoreDNS logs in the Kubernetes cluster.
- Verify the `/etc/resolv.conf` inside the pods.
- Run `nslookup supreme-database.default.svc.cluster.local` from a debug pod.

### 6.2 TLS Certificate Expiration
Ticket Supreme enforces mutual TLS (mTLS) between internal services.
- **Symptom:** Services report `x509: certificate has expired or is not yet valid`.
- **Resolution:** Rotate the internal certificates using the cert-manager.
  ```bash
  kubectl get certificates -n ticket-supreme
  kubectl delete secret supreme-mtls-certs -n ticket-supreme
  ```
  Cert-manager will automatically regenerate the secret.

## 7. Performance Degradation & Bottlenecks
When the system is slow but not completely down, systematic profiling is required.

### 7.1 API Latency Spikes
If the 99th percentile (p99) latency of the API Gateway exceeds 500ms:
1. **Check Distributed Tracing:** Open Jaeger or Zipkin and trace a slow request. Identify which microservice is the bottleneck.
2. **Review Rate Limits:** Ensure the rate limiter is not queuing requests excessively.
3. **Database Locks:** Check for row-level locks on heavily updated tickets.

### 7.2 Search Indexing Delays
If new tickets or comments are not appearing in search results immediately:
1. **Check Kafka Lag:** The `Supreme-Search` service consumes from a Kafka topic. Check the consumer group lag.
   ```bash
   kafka-consumer-groups.sh --bootstrap-server kafka:9092 --describe --group supreme-search-group
   ```
2. **Elasticsearch Bulk Queue:** Check if the Elasticsearch bulk thread pool is rejecting requests.
   ```bash
   curl -X GET "elasticsearch:9200/_cat/thread_pool/bulk?v&s=queue:desc"
   ```

## 8. Authentication & Authorization Failures
Issues with logging in or accessing resources.

### 8.1 SSO / SAML Integration Issues
If users cannot log in via their Identity Provider (IdP):
- **Symptom:** "Invalid SAML Response" error.
- **Diagnostics:**
  1. Capture the SAML response using a browser extension (e.g., SAML Tracer).
  2. Verify the `AssertionConsumerServiceURL` matches the Ticket Supreme configuration.
  3. Ensure the clock on the Ticket Supreme server is synchronized with NTP. A time skew of more than 5 minutes will invalidate SAML assertions.
  4. Check the signing certificate. If the IdP rotated their certificate, update it in the Ticket Supreme SSO settings.

### 8.2 LDAP Sync Failures
If user roles are not updating:
- Check the `supreme-ldap-sync` cronjob logs.
- Verify the bind credentials and search base DN.
- Test connectivity: `ldapsearch -x -H ldaps://ldap.company.com -D "cn=admin,dc=company,dc=com" -W -b "ou=users,dc=company,dc=com"`

## 9. Integration & API Troubleshooting
Ticket Supreme integrates with Jira, Slack, and PagerDuty.

### 9.1 Webhook Delivery Failures
If webhooks are failing to deliver to external systems:
1. **Check Webhook Logs:** Navigate to Admin -> Integrations -> Webhook Logs.
2. **Inspect HTTP Status Codes:**
   - `429 Too Many Requests`: The destination system is rate-limiting Ticket Supreme. Implement exponential backoff.
   - `500 Internal Server Error`: The destination system is failing to process the payload.
   - `Timeout`: The destination system took longer than 10 seconds to respond.
3. **Retry Mechanism:** Ticket Supreme automatically retries failed webhooks up to 5 times. You can manually trigger a retry via the API:
   ```bash
   curl -X POST https://api.ticketsupreme.com/v1/webhooks/events/{event_id}/retry -H "Authorization: Bearer $TOKEN"
   ```

### 9.2 Jira Sync Conflicts
If a ticket in Ticket Supreme is out of sync with a Jira issue:
- **Symptom:** "Version Conflict" or "Stale Data" error.
- **Resolution:** Force a full resync for the specific ticket.
  ```bash
  supreme-cli integration sync --provider=jira --ticket=TS-12345 --force
  ```

## 10. Recovery Strategies & Runbooks
Standard operating procedures for critical failures.

### 10.1 Complete Database Failure (Split-Brain or Corruption)
1. **Halt Traffic:** Scale down the API Gateway to prevent further data corruption.
   ```bash
   kubectl scale deployment supreme-gateway --replicas=0
   ```
2. **Assess Damage:** Check PostgreSQL logs for corruption errors.
3. **Restore from Backup:** If corruption is severe, initiate a Point-in-Time Recovery (PITR) using pgBackRest.
   ```bash
   pgbackrest --stanza=supreme --type=time --target="2023-10-27 14:00:00" restore
   ```
4. **Verify Data Integrity:** Run the `supreme-cli db verify` command.
5. **Restore Traffic:** Scale the API Gateway back up.

### 10.2 Elasticsearch Cluster Red State
If the Elasticsearch cluster goes red (missing primary shards):
1. **Identify Unassigned Shards:**
   ```bash
   curl -X GET "elasticsearch:9200/_cluster/allocation/explain?pretty"
   ```
2. **Attempt Reroute:** If a node temporarily dropped, force a reroute.
   ```bash
   curl -X POST "elasticsearch:9200/_cluster/reroute?retry_failed=true"
   ```
3. **Rebuild Index:** If data is permanently lost, rebuild the search index from the primary PostgreSQL database.
   ```bash
   supreme-cli search rebuild --index=tickets --batch-size=1000
   ```

## 11. Logging & Telemetry
Effective troubleshooting relies on comprehensive observability.

### 11.1 Log Formats and Levels
Ticket Supreme uses structured JSON logging.
- **DEBUG:** Detailed information for developers (e.g., raw SQL queries).
- **INFO:** Standard operational events (e.g., ticket created, user logged in).
- **WARN:** Non-critical issues that require attention (e.g., API rate limit approaching).
- **ERROR:** Critical failures that impact user experience (e.g., database connection lost).

### 11.2 Correlating Logs
Every request is assigned a `X-Request-ID`. This ID is injected into all log entries across all microservices.
To trace a specific request:
```bash
grep "req_id=550e8400-e29b-41d4-a716-446655440000" /var/log/supreme/*.log
```

### 11.3 Prometheus Metrics
Key metrics to monitor:
- `supreme_http_requests_total`: Total number of HTTP requests.
- `supreme_http_request_duration_seconds`: Histogram of request latencies.
- `supreme_db_connection_pool_active`: Number of active database connections.
- `supreme_ticket_creation_rate`: Number of tickets created per minute.

## 12. Support Escalation Procedures
If internal troubleshooting fails to resolve the issue, escalate to Ticket Supreme Vendor Support.

### 12.1 Gathering Diagnostic Data
Before opening a support ticket, generate a support bundle:
```bash
supreme-cli support bundle --include-logs=true --include-metrics=true --days=3
```
This will create a `supreme-support-bundle-<date>.tar.gz` file containing sanitized logs, configuration files, and system metrics.

### 12.2 Contacting Support
- **Severity 1 (Critical Production Outage):** Call the 24/7 emergency hotline at +1-800-555-0199.
- **Severity 2 (Major Functionality Degraded):** Open a ticket via the Support Portal at `support.ticketsupreme.com`.
- **Severity 3 (Minor Issue / Question):** Email `support@ticketsupreme.com`.

### 12.3 Required Information for Escalation
When escalating, provide:
1. The support bundle generated in step 12.1.
2. A detailed description of the symptoms and business impact.
3. Steps already taken to troubleshoot the issue.
4. Recent changes to the environment (e.g., upgrades, network changes).

---
*Document Version: 2.4.0*
*Last Updated: October 2023*
*Author: Ticket Supreme Engineering Team*