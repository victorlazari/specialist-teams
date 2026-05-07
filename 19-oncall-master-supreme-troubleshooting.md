# Oncall-Master-Supreme: Comprehensive Troubleshooting & Diagnostics Guide

## 1. Introduction

Welcome to the definitive troubleshooting and diagnostics guide for **Oncall-Master-Supreme**. This document is designed for Site Reliability Engineers (SREs), DevOps professionals, and on-call responders who are tasked with maintaining the health, performance, and reliability of the Oncall-Master-Supreme ecosystem. When production systems degrade or fail, this guide provides the structured methodologies, error code references, and recovery strategies necessary to restore service rapidly and safely.

Oncall-Master-Supreme is a highly distributed, fault-tolerant incident management and alerting orchestration platform. Due to its complex microservices architecture, troubleshooting requires a systematic approach to isolate faults across network layers, database clusters, message queues, and application nodes. This guide covers everything from initial triage to deep-dive diagnostics.

---

## 2. Initial Triage and Health Checks

When an alert fires or a degradation is reported, the first step is to assess the overall health of the Oncall-Master-Supreme cluster.

### 2.1. System Health Dashboard

The primary health dashboard is accessible via the `/healthz` and `/readyz` endpoints on the management nodes.

- **`/healthz`**: Indicates if the application is running. A `200 OK` means the process is up.
- **`/readyz`**: Indicates if the application is ready to serve traffic (e.g., database connections are established, caches are warmed).

**Command Line Verification:**
```bash
curl -s http://localhost:8080/healthz | jq .
curl -s http://localhost:8080/readyz | jq .
```

### 2.2. Core Component Status

Verify the status of the core dependencies:
1.  **PostgreSQL Database**: Check connection pools and replication lag.
2.  **Redis Cache**: Verify memory usage and hit/miss ratios.
3.  **Kafka Event Bus**: Check consumer group lag and partition health.

**Diagnostic Script:**
```bash
oncall-admin check-dependencies --all
```

---

## 3. Error Codes and Resolutions

Oncall-Master-Supreme uses a standardized error code format: `OMS-[Category]-[Code]`. Below is a comprehensive reference for common error codes.

### 3.1. Authentication and Authorization (OMS-AUTH-*)

| Error Code | Description | Potential Causes | Resolution Strategy |
| :--- | :--- | :--- | :--- |
| `OMS-AUTH-001` | Invalid API Key | Expired key, revoked access, or malformed header. | Verify the API key in the control plane. Rotate the key if compromised. |
| `OMS-AUTH-002` | Token Expired | JWT token has passed its `exp` claim. | Client must request a new token using the refresh token flow. |
| `OMS-AUTH-003` | Insufficient Permissions | User/Service lacks required RBAC roles. | Audit the user's roles via `oncall-admin rbac view <user_id>`. Grant necessary roles. |

### 3.2. Database and Storage (OMS-DB-*)

| Error Code | Description | Potential Causes | Resolution Strategy |
| :--- | :--- | :--- | :--- |
| `OMS-DB-101` | Connection Pool Exhausted | High traffic, slow queries holding connections, or connection leaks. | 1. Check `pg_stat_activity`. 2. Increase `max_connections` temporarily. 3. Analyze slow query logs. |
| `OMS-DB-102` | Deadlock Detected | Concurrent transactions modifying the same rows in different orders. | The application will auto-retry. If persistent, review application transaction logic and index usage. |
| `OMS-DB-103` | Disk Space Critical | WAL logs accumulating, or massive data ingestion. | Expand PVCs (Persistent Volume Claims). Run vacuuming. Archive old incident data. |

### 3.3. Messaging and Eventing (OMS-MSG-*)

| Error Code | Description | Potential Causes | Resolution Strategy |
| :--- | :--- | :--- | :--- |
| `OMS-MSG-201` | Broker Unreachable | Network partition, Kafka brokers down. | Check network connectivity to Kafka cluster. Verify broker logs. |
| `OMS-MSG-202` | Consumer Lag Critical | Consumers are processing events slower than they are produced. | Scale out consumer pods. Profile consumer CPU/Memory. Check for downstream bottlenecks. |

---

## 4. Common Issues and Recovery Strategies

### 4.1. High API Latency

**Symptoms:**
- P99 latency spikes above 500ms.
- Timeouts reported by upstream clients.
- `OMS-NET-504` (Gateway Timeout) errors.

**Diagnostic Steps:**
1.  **Isolate the Endpoint:** Use distributed tracing (e.g., Jaeger, Zipkin) to identify which specific endpoints are slow.
2.  **Check Database Performance:** Look for slow queries. Are indexes missing? Is the database CPU pegged?
3.  **Analyze Garbage Collection:** If running on the JVM or Go, check GC pause times. Long pauses can cause latency spikes.
4.  **Inspect Network I/O:** Check for packet loss or bandwidth saturation between microservices.

**Recovery Strategy:**
- Implement rate limiting to shed load.
- Scale up the affected microservice deployments.
- If a specific database query is the culprit, kill the long-running query and apply an emergency index.

### 4.2. Alert Delivery Failure (The "Silent Night" Scenario)

**Symptoms:**
- Incidents are created, but notifications (SMS, Email, PagerDuty) are not being dispatched.
- The `alert_dispatch_queue` is backing up.

**Diagnostic Steps:**
1.  **Check Third-Party Integrations:** Are the APIs for Twilio, SendGrid, or Slack reachable? Check their status pages.
2.  **Inspect the Dispatcher Logs:** Look for authentication errors or rate-limiting responses from third-party providers.
3.  **Verify Worker Nodes:** Ensure the background worker nodes responsible for dispatching are running and not stuck in a crash loop.

**Recovery Strategy:**
- If a third-party provider is down, switch to a fallback provider if configured (e.g., fallback from SMS to Voice).
- If rate-limited, implement exponential backoff in the dispatcher.
- Manually flush the queue if messages are poisoned (failing repeatedly and blocking the queue).

### 4.3. Split-Brain in the Clustering Mechanism

**Symptoms:**
- Nodes report different cluster states.
- Duplicate alerts are sent for the same incident.
- Data inconsistencies in the distributed cache.

**Diagnostic Steps:**
1.  **Check Quorum:** Verify that a majority of nodes are healthy and can communicate.
2.  **Network Partition:** Look for network drops between availability zones.
3.  **Examine Raft/Gossip Logs:** Check the logs of the consensus module for election storms or dropped heartbeats.

**Recovery Strategy:**
- Identify the isolated minority partition and forcefully restart those nodes to force them to rejoin the cluster.
- Ensure the network link between AZs is stable.
- Review the `election_timeout` configuration; it may be too aggressive for the current network latency.

---

## 5. Advanced Diagnostics

### 5.1. Profiling CPU and Memory

When standard metrics are not enough, you must profile the application.

**CPU Profiling (Go Example):**
```bash
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30
```
Look for functions consuming excessive CPU cycles. Common culprits include inefficient JSON serialization or tight loops.

**Memory Profiling:**
```bash
go tool pprof http://localhost:6060/debug/pprof/heap
```
Analyze the heap to find memory leaks. Look for objects that are allocated but never garbage collected.

### 5.2. Distributed Tracing Analysis

Oncall-Master-Supreme instruments all requests with OpenTelemetry.
- Trace ID is injected into all logs.
- Use the Trace ID to query your tracing backend (e.g., Tempo, Jaeger).
- Look for "spans" that take an unusually long time. Pay attention to the gaps between spans, which indicate network latency or scheduling delays.

### 5.3. Database Query Plan Analysis

If a query is slow, use `EXPLAIN ANALYZE` to understand how PostgreSQL is executing it.

```sql
EXPLAIN ANALYZE SELECT * FROM incidents WHERE status = 'open' AND severity = 'SEV-1';
```
- **Seq Scan:** Indicates a missing index.
- **Index Scan:** Good, but check if it's reading too many rows.
- **Loops:** High loop counts in nested loops can cause severe performance degradation.

---

## 6. Runbooks for Critical Incidents

### 6.1. Runbook: Complete Database Outage

1.  **Acknowledge:** Acknowledge the PagerDuty alert.
2.  **Communicate:** Post in the `#incident-response` Slack channel: "Investigating complete DB outage for Oncall-Master-Supreme."
3.  **Investigate:**
    - Check AWS RDS / Cloud SQL console for hardware failures.
    - Check for accidental deletion or misconfiguration.
4.  **Mitigate:**
    - If the primary is down, trigger a manual failover to the replica.
    - If data is corrupted, initiate a Point-in-Time Recovery (PITR) to the last known good state.
5.  **Verify:** Run the health check script. Ensure the application reconnects to the new primary.
6.  **Resolve:** Update the status page and resolve the incident.

### 6.2. Runbook: Massive Traffic Spike (DDoS or Thundering Herd)

1.  **Acknowledge & Communicate.**
2.  **Investigate:**
    - Check WAF (Web Application Firewall) logs for malicious patterns.
    - Check ingress controller metrics for the source of the traffic.
3.  **Mitigate:**
    - Enable aggressive rate limiting at the WAF/Ingress layer.
    - Scale up the application pods to maximum capacity.
    - If it's a thundering herd from internal services, implement jitter in the retry logic of the offending service.
4.  **Verify:** Monitor latency and error rates until they return to baseline.

---

## 7. Preventative Maintenance

Troubleshooting is reactive; maintenance is proactive.

- **Weekly:** Review slow query logs and add necessary indexes.
- **Monthly:** Perform chaos engineering experiments (e.g., randomly kill pods) to ensure the system recovers gracefully.
- **Quarterly:** Conduct a capacity planning review. Forecast storage and compute needs for the next 6 months.
- **Annually:** Perform a full disaster recovery drill, including restoring the database from backups in an isolated environment.

## 8. Conclusion

Troubleshooting Oncall-Master-Supreme requires a deep understanding of its architecture and dependencies. By following the structured approaches outlined in this guide, utilizing the diagnostic tools, and adhering to the runbooks, on-call engineers can effectively mitigate issues and maintain the high availability expected of a critical incident management platform. Always remember to document new findings and update this guide as the system evolves.

---

## 9. Deep Dive: Network Layer Diagnostics

Network issues are often the most difficult to diagnose due to their transient nature. In a microservices architecture like Oncall-Master-Supreme, network reliability is paramount.

### 9.1. DNS Resolution Failures

**Symptoms:**
- Services cannot communicate with each other using hostnames.
- Logs show `NXDOMAIN` or `SERVFAIL` errors.
- Intermittent connection timeouts.

**Diagnostic Steps:**
1.  **Test Local Resolution:** Use `dig` or `nslookup` from within a failing pod to test DNS resolution.
    ```bash
    kubectl exec -it <pod-name> -- dig database.oncall-master-supreme.svc.cluster.local
    ```
2.  **Check CoreDNS Logs:** Inspect the logs of the CoreDNS pods in the `kube-system` namespace for errors or high latency.
3.  **Verify Node Configuration:** Ensure the `/etc/resolv.conf` on the worker nodes is correctly configured and pointing to the right upstream resolvers.

**Recovery Strategy:**
- Restart CoreDNS pods if they are stuck.
- Scale up CoreDNS deployments to handle high query volumes.
- Implement NodeLocal DNSCache to reduce the load on CoreDNS and improve resolution latency.

### 9.2. TCP Connection Drops and Resets

**Symptoms:**
- Applications report `Connection reset by peer` or `Broken pipe`.
- High rate of TCP retransmissions.

**Diagnostic Steps:**
1.  **Packet Capture:** Use `tcpdump` to capture traffic between the affected services.
    ```bash
    tcpdump -i any -nn -s0 -w capture.pcap port 8080
    ```
2.  **Analyze with Wireshark:** Look for RST packets, out-of-order packets, or long delays between SYN and SYN-ACK.
3.  **Check MTU Settings:** Ensure the Maximum Transmission Unit (MTU) is consistent across the network path. Mismatched MTUs can cause packet fragmentation and drops.

**Recovery Strategy:**
- Adjust MTU settings if necessary.
- Tune TCP keepalive settings in the application and OS to prevent idle connections from being dropped by firewalls or load balancers.

---

## 10. Deep Dive: Storage and Persistence Layer

Data integrity and availability are critical. Oncall-Master-Supreme relies heavily on its storage layer.

### 10.1. PostgreSQL Performance Tuning

When the database becomes a bottleneck, standard troubleshooting must evolve into deep performance tuning.

**Advanced Diagnostics:**
- **`pg_stat_statements`:** This extension is crucial for identifying the most resource-intensive queries over time.
  ```sql
  SELECT query, calls, total_time, rows, 100.0 * shared_blks_hit / nullif(shared_blks_hit + shared_blks_read, 0) AS hit_percent
  FROM pg_stat_statements ORDER BY total_time DESC LIMIT 10;
  ```
- **Vacuum and Analyze:** Ensure autovacuum is running effectively. Bloat in tables and indexes can severely degrade performance.
  ```sql
  SELECT relname, n_dead_tup FROM pg_stat_user_tables WHERE n_dead_tup > 10000;
  ```

**Recovery Strategy:**
- Manually run `VACUUM ANALYZE` on heavily bloated tables during off-peak hours.
- Tune `work_mem` and `shared_buffers` based on the available RAM and workload characteristics.

### 10.2. Redis Cache Eviction and Memory Management

Redis is used for session management and caching frequently accessed data.

**Symptoms:**
- High cache miss rate.
- Redis instances running out of memory (OOM).
- Increased latency in API responses.

**Diagnostic Steps:**
1.  **Check Memory Usage:** Use the `INFO memory` command in `redis-cli`.
2.  **Analyze Eviction Policy:** Verify the `maxmemory-policy` setting (e.g., `volatile-lru`, `allkeys-lru`).
3.  **Identify Large Keys:** Use `redis-cli --bigkeys` to find keys consuming excessive memory.

**Recovery Strategy:**
- Scale up Redis instances (add more memory).
- Optimize data structures (e.g., use hashes instead of strings for object storage).
- Adjust the TTL (Time To Live) for cached items to ensure stale data is evicted promptly.

---

## 11. Security and Access Control Troubleshooting

Security incidents require immediate and precise action.

### 11.1. Unauthorized Access Attempts

**Symptoms:**
- High volume of `401 Unauthorized` or `403 Forbidden` responses.
- Alerts from the Intrusion Detection System (IDS).

**Diagnostic Steps:**
1.  **Analyze Audit Logs:** Review the application audit logs to identify the source IP, user agent, and targeted endpoints.
2.  **Check WAF Rules:** Ensure the WAF is correctly configured to block known malicious patterns (e.g., SQL injection, XSS).
3.  **Verify IAM Policies:** Review Identity and Access Management (IAM) policies to ensure the principle of least privilege is enforced.

**Recovery Strategy:**
- Block the offending IP addresses at the WAF or network firewall.
- Revoke compromised credentials immediately.
- Force a password reset for affected users.

### 11.2. Certificate Expiration

**Symptoms:**
- Clients report SSL/TLS errors (e.g., `CERT_DATE_INVALID`).
- Services fail to communicate securely.

**Diagnostic Steps:**
1.  **Check Certificate Validity:** Use `openssl` to check the expiration date of the certificate.
    ```bash
    echo | openssl s_client -servername oncall-master-supreme.com -connect oncall-master-supreme.com:443 2>/dev/null | openssl x509 -noout -dates
    ```
2.  **Verify Cert-Manager Logs:** If using Kubernetes cert-manager, check its logs for errors during the renewal process.

**Recovery Strategy:**
- Manually renew the certificate if the automated process failed.
- Investigate and fix the root cause of the automated renewal failure (e.g., DNS validation issues, rate limits from Let's Encrypt).

---

## 12. Log Aggregation and Analysis

Effective troubleshooting relies on comprehensive and accessible logs.

### 12.1. Missing or Delayed Logs

**Symptoms:**
- Logs are not appearing in the central logging system (e.g., ELK stack, Splunk, Datadog).
- Significant delay between an event occurring and its log appearing.

**Diagnostic Steps:**
1.  **Check Log Forwarder:** Verify the status of the log forwarding agent (e.g., Fluentd, Filebeat, Promtail) on the worker nodes.
2.  **Inspect Forwarder Logs:** Look for errors related to parsing, buffering, or connecting to the central logging system.
3.  **Verify Network Connectivity:** Ensure the forwarder can reach the logging backend.

**Recovery Strategy:**
- Restart the log forwarding agent.
- Increase the buffer size or adjust the flush interval if the forwarder is overwhelmed.
- Scale up the central logging system if it is dropping incoming logs due to high load.

### 12.2. Log Parsing Errors

**Symptoms:**
- Logs are appearing, but fields are not correctly extracted (e.g., JSON logs are treated as plain text).
- Dashboards and alerts relying on specific log fields are failing.

**Diagnostic Steps:**
1.  **Review Log Format:** Ensure the application is outputting logs in the expected format (e.g., structured JSON).
2.  **Check Parser Configuration:** Verify the configuration of the log parser (e.g., Logstash grok patterns, Fluentd parsers).

**Recovery Strategy:**
- Update the parser configuration to match the application's log format.
- Standardize log formats across all microservices to simplify parsing.

---

## 13. Chaos Engineering and Resilience Testing

To truly validate the troubleshooting procedures and the system's resilience, proactive chaos engineering is required.

### 13.1. Simulating Node Failures

**Procedure:**
1.  Randomly terminate a worker node in the cluster.
2.  Observe the system's behavior:
    - Do pods reschedule correctly?
    - Is there any downtime or data loss?
    - Do alerts fire as expected?

**Expected Outcome:**
The system should automatically recover without human intervention. If not, investigate the scheduling constraints, pod disruption budgets, and application startup times.

### 13.2. Simulating Network Latency

**Procedure:**
1.  Inject artificial network latency between two critical microservices (e.g., using a service mesh like Istio or tools like `tc`).
2.  Observe the system's behavior:
    - Do circuit breakers trip?
    - Do requests time out gracefully?
    - Is the user experience degraded but still functional?

**Expected Outcome:**
The system should degrade gracefully, providing informative error messages to the user rather than hanging indefinitely.

---

## 14. Escalation Procedures

When an incident cannot be resolved by the primary on-call responder within the defined Service Level Objective (SLO), it must be escalated.

### 14.1. Escalation Matrix

1.  **Tier 1 (L1):** Primary On-Call Engineer. Responsible for initial triage, executing runbooks, and basic troubleshooting.
2.  **Tier 2 (L2):** Subject Matter Experts (SMEs) / Senior Engineers. Engaged when L1 cannot resolve the issue within 30 minutes or if the issue requires deep domain knowledge.
3.  **Tier 3 (L3):** Engineering Leadership / Vendor Support. Engaged for critical, systemic failures or issues requiring external vendor assistance (e.g., AWS, GCP).

### 14.2. Communication Protocol

During an escalated incident, clear communication is vital.
- **Incident Commander (IC):** Appoint an IC to coordinate the response, manage communication, and make critical decisions.
- **Status Updates:** Provide regular updates (e.g., every 15-30 minutes) to stakeholders via the status page and internal communication channels.
- **Post-Mortem:** After the incident is resolved, conduct a blameless post-mortem to identify the root cause, document lessons learned, and implement preventative measures.

## 15. Final Thoughts

Troubleshooting is both an art and a science. It requires technical expertise, logical deduction, and a calm demeanor under pressure. This guide provides the foundation, but experience and continuous learning are the keys to mastering the Oncall-Master-Supreme platform. Stay curious, document your findings, and always strive to improve the system's resilience.

---

## 16. Appendix A: Command Line Interface (CLI) Diagnostic Tools

The `oncall-admin` CLI provides several built-in diagnostic commands that are invaluable during an incident.

### 16.1. `oncall-admin cluster status`

This command provides a high-level overview of the cluster's health, including node status, pod distribution, and resource utilization.

**Usage:**
```bash
oncall-admin cluster status --detailed
```

**Output Interpretation:**
- Look for nodes in a `NotReady` state.
- Check for pods in a `CrashLoopBackOff` or `Pending` state.
- Monitor CPU and memory utilization across the cluster.

### 16.2. `oncall-admin db check`

This command performs a deep inspection of the PostgreSQL database, checking for bloat, long-running queries, and replication lag.

**Usage:**
```bash
oncall-admin db check --all
```

**Output Interpretation:**
- High replication lag indicates that read replicas are falling behind the primary, which can lead to stale data being served.
- Long-running queries should be investigated and potentially terminated if they are blocking other operations.

### 16.3. `oncall-admin network trace`

This command initiates a distributed trace across the microservices to identify network bottlenecks and latency issues.

**Usage:**
```bash
oncall-admin network trace --source <service-a> --destination <service-b>
```

**Output Interpretation:**
- The output will show the path taken by the request and the latency at each hop.
- Look for unusually high latency between specific services, which may indicate network congestion or routing issues.

## 17. Appendix B: Glossary of Terms

- **SLO (Service Level Objective):** A target value or range of values for a service level that is measured by an SLI.
- **SLI (Service Level Indicator):** A carefully defined quantitative measure of some aspect of the level of service that is provided.
- **SLA (Service Level Agreement):** An explicit or implicit contract with your users that includes consequences of meeting (or missing) the SLOs they contain.
- **MTTR (Mean Time To Recovery):** The average time it takes to recover from a product or system failure.
- **MTBF (Mean Time Between Failures):** The average time between system breakdowns.
- **Toil:** The kind of work tied to running a production service that tends to be manual, repetitive, automatable, tactical, devoid of enduring value, and that scales linearly as a service grows.
- **Blameless Post-Mortem:** A culture of learning from failures without assigning blame to individuals. The focus is on identifying systemic issues and improving processes.

## 18. Appendix C: Reference Architecture

Understanding the reference architecture is crucial for effective troubleshooting. Oncall-Master-Supreme is built on a modern, cloud-native stack:

- **Compute:** Kubernetes (EKS/GKE/AKS)
- **Database:** PostgreSQL (Primary/Replica setup with Patroni for HA)
- **Cache:** Redis Cluster
- **Message Broker:** Apache Kafka
- **Ingress:** NGINX Ingress Controller / Envoy Proxy
- **Observability:** Prometheus, Grafana, Jaeger, OpenTelemetry
- **CI/CD:** ArgoCD, GitHub Actions

Each component plays a specific role, and failures can cascade across the system. When troubleshooting, always consider the interactions between these components and how a failure in one area might impact others.

## 19. Appendix D: Extended Error Code Reference

### 19.1. API Gateway Errors (OMS-API-*)

| Error Code | Description | Potential Causes | Resolution Strategy |
| :--- | :--- | :--- | :--- |
| `OMS-API-400` | Bad Request | Malformed JSON payload, missing required fields. | Validate the request payload against the OpenAPI schema. |
| `OMS-API-429` | Too Many Requests | Client exceeded rate limits. | Implement exponential backoff on the client side. Review rate limit configurations. |
| `OMS-API-502` | Bad Gateway | Upstream service is down or returning invalid responses. | Check the health of the upstream microservice. Review ingress controller logs. |
| `OMS-API-503` | Service Unavailable | Service is overloaded or undergoing maintenance. | Scale up the service. Check for resource exhaustion (CPU/Memory). |

### 19.2. User Management Errors (OMS-USR-*)

| Error Code | Description | Potential Causes | Resolution Strategy |
| :--- | :--- | :--- | :--- |
| `OMS-USR-001` | User Not Found | Invalid user ID or email address. | Verify the user exists in the database. |
| `OMS-USR-002` | Account Locked | Too many failed login attempts. | Unlock the account via the admin console or wait for the lockout period to expire. |
| `OMS-USR-003` | Invalid Password | Password does not meet complexity requirements. | Enforce password policies during registration and password resets. |

## 20. Conclusion and Continuous Improvement

This guide is a living document. As Oncall-Master-Supreme evolves, new features will be added, and new failure modes will emerge. It is the responsibility of every engineer to contribute to this guide, documenting new troubleshooting techniques, updating runbooks, and sharing knowledge with the team. By fostering a culture of continuous improvement and blameless learning, we can ensure that Oncall-Master-Supreme remains a robust, reliable, and indispensable tool for incident management.