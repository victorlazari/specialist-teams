# Lerian Studio Helm Configuration Schemas and Tuning Recommendations

## 1. Introduction

Welcome to the comprehensive guide on configuration schemas and tuning recommendations for Lerian applications deployed via Helm. This document is designed specifically for tech support operations, Site Reliability Engineers (SREs), and DevOps professionals who are responsible for maintaining Lerian Studio applications in high-stakes, production environments. The configurations detailed herein focus on ensuring maximum reliability, performance under heavy load, and graceful degradation during worst-case scenarios.

Lerian applications are designed to be highly scalable and resilient, but achieving these characteristics in a real-world Kubernetes environment requires meticulous tuning of Helm chart values. This guide covers the critical aspects of resource management, autoscaling, high availability, application-specific environment variables, database migrations, and timeout configurations. By adhering to these recommendations, support teams can proactively prevent outages, optimize resource utilization, and rapidly troubleshoot complex issues when they arise.

## 2. Resource Requests and Limits Tuning

Properly configuring resource requests and limits is the foundation of a stable Kubernetes deployment. Incorrect values can lead to CPU throttling, Out-Of-Memory (OOM) kills, and unpredictable application behavior, especially when dealing with huge datasets or sudden spikes in traffic.

### 2.1 CPU Allocation

CPU requests determine the guaranteed amount of compute power allocated to a pod, while limits define the maximum allowed. For Lerian applications, CPU throttling is a common cause of latency spikes.

*   **Requests:** Set CPU requests based on the baseline usage observed during normal operations. For a typical Lerian API service, a baseline of `500m` (half a core) is recommended.
*   **Limits:** CPU limits should be set carefully. In many modern Kubernetes environments, removing CPU limits entirely (or setting them very high, e.g., `2000m` to `4000m`) is preferred to avoid aggressive throttling by the Completely Fair Scheduler (CFS). If limits must be enforced, ensure they are at least 2x to 3x the request to accommodate bursty workloads.

### 2.2 Memory Allocation

Memory is an incompressible resource. If a pod exceeds its memory limit, it will be immediately terminated (OOMKilled) by the kernel.

*   **Requests:** Set memory requests to the average memory consumption plus a 20% buffer. For a standard Lerian worker node processing background jobs, `1Gi` is a safe starting point.
*   **Limits:** Memory limits must be strictly enforced to prevent a single rogue pod from starving the entire node. Set the limit to 1.5x to 2x the request. For example, if the request is `1Gi`, the limit should be `2Gi`.

### 2.3 Tech Support Operations: Diagnosing Resource Issues

When troubleshooting performance degradation, tech support should first inspect the pod's resource usage:

```bash
kubectl top pods -n lerian-production
kubectl describe pod <pod-name> -n lerian-production | grep -i "OOMKilled"
```

If OOMKills are frequent, investigate the application for memory leaks or increase the memory limits in the Helm `values.yaml`:

```yaml
resources:
  requests:
    cpu: 500m
    memory: 1Gi
  limits:
    cpu: 2000m
    memory: 2Gi
```

## 3. Horizontal Pod Autoscaling (HPA) Configurations

The Horizontal Pod Autoscaler (HPA) automatically scales the number of pods in a deployment based on observed CPU utilization or custom metrics. For Lerian applications, configuring HPA correctly is vital for handling traffic spikes without over-provisioning resources.

### 3.1 Target Metrics

While CPU utilization is the most common metric, it is often insufficient for complex applications. Lerian applications benefit significantly from custom metrics, such as HTTP request rate or queue length.

*   **CPU Target:** A target CPU utilization of 70-80% is generally recommended. Setting it too low causes unnecessary scaling, while setting it too high risks performance degradation before new pods are ready.
*   **Custom Metrics:** For background workers, scale based on the length of the message queue (e.g., RabbitMQ or Redis queue depth). If the queue exceeds 1000 messages, trigger a scale-up event.

### 3.2 Scaling Policies

Kubernetes allows fine-tuning of scaling behavior to prevent thrashing (rapid scaling up and down).

*   **Scale-Up:** Configure a rapid scale-up policy to respond quickly to sudden traffic surges.
*   **Scale-Down:** Implement a stabilization window (e.g., 5 minutes) for scale-down events to ensure that the traffic drop is sustained before removing pods.

### 3.3 Helm Configuration Example

```yaml
autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 20
  targetCPUUtilizationPercentage: 75
  targetMemoryUtilizationPercentage: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Pods
        value: 1
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
```

## 4. Pod Disruption Budgets (PDBs) and High Availability

Pod Disruption Budgets (PDBs) ensure that a minimum number of pods remain available during voluntary disruptions, such as node upgrades or cluster maintenance. This is critical for maintaining the high availability of Lerian applications.

### 4.1 Configuring PDBs

For critical services, you must define a PDB that guarantees either a minimum number of available pods (`minAvailable`) or a maximum number of unavailable pods (`maxUnavailable`).

*   **minAvailable:** Use this for stateful applications or quorum-based systems where a specific number of instances must be running. For example, `minAvailable: 2`.
*   **maxUnavailable:** Use this for stateless applications. A common setting is `maxUnavailable: 1`, ensuring that only one pod can be taken down at a time during rolling updates.

### 4.2 Worst-Case Scenarios

In a worst-case scenario where multiple nodes fail simultaneously, PDBs cannot prevent involuntary disruptions. However, they are essential for preventing administrative actions from causing an outage. Tech support must ensure that PDBs are not overly restrictive; for instance, setting `minAvailable` equal to the total number of replicas will block all node drains and upgrades.

### 4.3 Helm Configuration Example

```yaml
podDisruptionBudget:
  enabled: true
  maxUnavailable: 1
  # minAvailable: 2
```

## 5. Application-Specific Environment Variables

Lerian applications rely on a variety of environment variables to control their behavior. These variables must be carefully managed via Helm values to ensure consistency across environments.

### 5.1 Multi-Tenancy Configuration

The `MULTI_TENANT_ENABLED` variable is a critical toggle for Lerian Studio. When set to `true`, the application enforces strict data isolation between tenants.

*   **MULTI_TENANT_ENABLED:** `true` or `false`. In production, this must always be `true` unless deploying a dedicated, single-tenant instance.
*   **TENANT_RESOLUTION_STRATEGY:** Defines how the application identifies the tenant (e.g., `SUBDOMAIN`, `HEADER`, `PATH`).

### 5.2 Feature Flags and Toggles

Feature flags allow tech support to enable or disable specific functionalities without redeploying the application.

*   **ENABLE_ADVANCED_ANALYTICS:** Toggles the heavy analytics engine. In scenarios where the database is under extreme load, tech support can temporarily set this to `false` to shed non-critical load.
*   **DEBUG_MODE:** Must be `false` in production. Enabling it exposes sensitive stack traces and significantly degrades performance due to excessive logging.

### 5.3 Helm Configuration Example

```yaml
env:
  MULTI_TENANT_ENABLED: "true"
  TENANT_RESOLUTION_STRATEGY: "HEADER"
  ENABLE_ADVANCED_ANALYTICS: "true"
  DEBUG_MODE: "false"
  LOG_LEVEL: "INFO"
```

## 6. Database Migrations and Huge Datasets

Handling database migrations in environments with huge datasets is one of the most challenging aspects of operating Lerian applications. Improperly executed migrations can lock tables, cause massive latency, and lead to application downtime.

### 6.1 Helm Hooks for Migrations

Lerian Helm charts utilize Helm hooks to execute database migrations before the new application pods are rolled out. The `pre-install` and `pre-upgrade` hooks ensure that the schema is up-to-date.

### 6.2 Managing Huge Datasets

When dealing with tables containing hundreds of millions of rows, standard `ALTER TABLE` statements can take hours and lock the database.

*   **Online Schema Changes:** For massive tables, tech support must bypass the standard Helm migration hook and use tools like `gh-ost` or `pt-online-schema-change`. The Helm chart should be configured to skip the migration job in these specific instances.
*   **Migration Timeouts:** The migration job must have a configured `activeDeadlineSeconds` to prevent it from running indefinitely and blocking the deployment.

### 6.3 Tech Support Operations: Failed Migrations

If a migration fails, the Helm release will be stuck in a `pending-upgrade` state. Tech support must:

1.  Inspect the migration job logs: `kubectl logs job/lerian-migration-job -n lerian-production`
2.  Identify the failing SQL statement.
3.  Manually resolve the database state (e.g., dropping a partially created index).
4.  Rollback the Helm release: `helm rollback lerian-production 1`

## 7. Timeout Configurations and Worst-Case Scenarios

Timeouts are the ultimate defense mechanism against cascading failures. If a downstream service (like a database or an external API) becomes unresponsive, Lerian applications must fail fast rather than hanging indefinitely.

### 7.1 HTTP Client Timeouts

All outbound HTTP requests from the Lerian application must have strict timeouts configured.

*   **CONNECT_TIMEOUT_MS:** The maximum time allowed to establish a connection. Recommended: `2000` (2 seconds).
*   **READ_TIMEOUT_MS:** The maximum time allowed to wait for a response after the connection is established. Recommended: `5000` (5 seconds).

### 7.2 Database Connection Pool Timeouts

The database connection pool must be configured to handle worst-case scenarios where the database is slow or unreachable.

*   **DB_CONNECTION_TIMEOUT:** The time to wait for a connection from the pool. Recommended: `5000` (5 seconds).
*   **DB_IDLE_TIMEOUT:** The time a connection can remain idle before being closed. Recommended: `600000` (10 minutes).

### 7.3 Ingress and Load Balancer Timeouts

The Kubernetes Ingress controller (e.g., NGINX) must also have appropriate timeouts to prevent client connections from hanging.

```yaml
ingress:
  annotations:
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "5"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "60"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "60"
```

## 8. Tech Support Operations and Troubleshooting

Tech support teams require clear playbooks for diagnosing and resolving issues in the Lerian Helm deployment. This section outlines standard operating procedures for common incidents.

### 8.1 Incident: High API Latency

**Symptoms:** Monitoring alerts indicate that the 99th percentile (p99) API latency has exceeded 2 seconds.

**Troubleshooting Steps:**
1.  **Check HPA Status:** Ensure the HPA has scaled up the pods appropriately. `kubectl get hpa -n lerian-production`
2.  **Inspect Resource Usage:** Check for CPU throttling. If pods are hitting their CPU limits, consider increasing the limits or optimizing the application code.
3.  **Analyze Database Performance:** High latency is often caused by slow database queries. Check the database monitoring dashboard for long-running queries or deadlocks.
4.  **Review Logs:** Search the application logs for timeout errors or connection refused messages.

### 8.2 Incident: Pod CrashLoopBackOff

**Symptoms:** Pods are repeatedly crashing and restarting, entering a `CrashLoopBackOff` state.

**Troubleshooting Steps:**
1.  **Describe the Pod:** `kubectl describe pod <pod-name> -n lerian-production`. Look for the `Exit Code` and `Reason` in the container status.
2.  **Check Previous Logs:** View the logs from the previous crashed instance: `kubectl logs <pod-name> --previous -n lerian-production`.
3.  **Verify Environment Variables:** Ensure all required environment variables (e.g., database credentials, API keys) are correctly injected via Kubernetes Secrets and ConfigMaps.
4.  **Liveness/Readiness Probes:** If the application takes a long time to start, the liveness probe might be killing it prematurely. Increase the `initialDelaySeconds` in the Helm values.

### 8.3 Incident: Network Partition or DNS Failure

**Symptoms:** The application cannot communicate with internal services or external APIs.

**Troubleshooting Steps:**
1.  **Test DNS Resolution:** Exec into a pod and test DNS resolution: `kubectl exec -it <pod-name> -- nslookup kubernetes.default.svc.cluster.local`.
2.  **Check CoreDNS Logs:** Inspect the logs of the CoreDNS pods in the `kube-system` namespace for errors.
3.  **Verify Network Policies:** Ensure that Kubernetes NetworkPolicies are not inadvertently blocking traffic between namespaces or to the internet.

## 9. Conclusion

Configuring and tuning Lerian applications via Helm requires a deep understanding of both the application architecture and the underlying Kubernetes infrastructure. By meticulously managing resource requests, autoscaling policies, disruption budgets, and timeouts, tech support and SRE teams can build a highly resilient system capable of withstanding massive scale and worst-case scenarios. Continuous monitoring, regular load testing, and proactive tuning are essential to maintaining the health and performance of the Lerian Studio platform.
