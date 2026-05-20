# 43-Lerian-Helm-Specialist.md

# Lerian Studio Helm Deployments Architecture and Tech Support Operations

---

## Table of Contents

1. [Introduction](#introduction)  
2. [Overview of Lerian Studio Helm Deployments](#overview-of-lerian-studio-helm-deployments)  
   - 2.1 [Core Components: Midaz, Matcher, Fetcher, Plugins](#core-components-midaz-matcher-fetcher-plugins)  
   - 2.2 [Migration Jobs](#migration-jobs)  
3. [Deep Dive into Helm Architecture and Deployment](#deep-dive-into-helm-architecture-and-deployment)  
   - 3.1 [Helm Charts Structure and Best Practices](#helm-charts-structure-and-best-practices)  
   - 3.2 [Helm Hooks: Lifecycle and Usage](#helm-hooks-lifecycle-and-usage)  
   - 3.3 [Init Containers: Initialization and Readiness Probes](#init-containers-initialization-and-readiness-probes)  
4. [Handling Large Scale Data and Worst-case Scenarios](#handling-large-scale-data-and-worst-case-scenarios)  
   - 4.1 [Timeout Management](#timeout-management)  
   - 4.2 [Database Migration Strategies for Massive Datasets](#database-migration-strategies-for-massive-datasets)  
   - 4.3 [Scaling Midaz, Matcher, and Fetcher Efficiently](#scaling-midaz-matcher-and-fetcher-efficiently)  
5. [Client-Facing Upgrade Procedures](#client-facing-upgrade-procedures)  
   - 5.1 [Pre-Upgrade Checks and Validation](#pre-upgrade-checks-and-validation)  
   - 5.2 [Step-by-step Helm Upgrade Process](#step-by-step-helm-upgrade-process)  
   - 5.3 [Rollback and Disaster Recovery](#rollback-and-disaster-recovery)  
6. [Tech Support Best Practices and Troubleshooting](#tech-support-best-practices-and-troubleshooting)  
   - 6.1 [Monitoring and Logging](#monitoring-and-logging)  
   - 6.2 [Common Issues and Resolution Patterns](#common-issues-and-resolution-patterns)  
   - 6.3 [Communication Protocols for Incident Management](#communication-protocols-for-incident-management)  
7. [Relation to Other Specialist Files](#relation-to-other-specialist-files)  
8. [Appendices](#appendices)  
   - 8.1 [Glossary of Terms](#glossary-of-terms)  
   - 8.2 [Helm Hooks Reference Table](#helm-hooks-reference-table)  
   - 8.3 [Useful kubectl and Helm CLI Commands](#useful-kubectl-and-helm-cli-commands)  

---

## Introduction

This document provides a **super comprehensive** insight into **Lerian Studio's Helm deployments architecture**, focusing on the core services—Midaz, Matcher, Fetcher, and their supporting plugins. It serves as an in-depth guide for specialists operating tech support and production DevOps teams, ensuring full mastery over deployments, handling massive datasets, managing migrations, troubleshooting, and managing client upgrades.

Capacity planning, worst-case scenario handling, and detailed operational procedures are emphasized. This document is intended for senior engineers, architects, and specialist operators tasked with maintaining uptime, handling live incidents, and supporting smooth client upgrade experiences.

---

## Overview of Lerian Studio Helm Deployments

### Core Components: Midaz, Matcher, Fetcher, Plugins

Lerian's operational backbone consists of microservices deployed via sophisticated Helm charts designed for Kubernetes. The architecture leverages these core components:

- **Midaz:** The central processing unit performing data aggregation and orchestration.
- **Matcher:** Responsible for high-performance matching algorithms, pairing datasets, and external data sources.
- **Fetcher:** Manages data retrieval from third-party APIs and orchestrates cache refreshing.
- **Plugins:** Modular components enhancing functionality such as logging, authentication, metric collection, and custom business logic extensions.

Each component is deployed as a separate Helm chart or a subchart within a parent chart that coordinates inter-service dependencies.

### Migration Jobs

Migrations are performed via special Kubernetes Jobs triggered as Helm hooks during deployment upgrades or standalone runs during maintenance windows. Migration jobs handle:

- Schema alterations for the persistent storage backing Midaz and Matcher.
- Data transformation scripts needed for compatibility between App versions.
- Backfill operations for large datasets lacking critical indices or columns.
  
Migration jobs are designed to be idempotent, resume safely after interruptions, and self-monitor resource consumption to avoid cluster instability.

---

## Deep Dive into Helm Architecture and Deployment

### Helm Charts Structure and Best Practices

The Helm chart structure follows strict conventions to maintain clarity and maintainability:

```
lerian-helm/
├── Chart.yaml
├── values.yaml
├── charts/
│   ├── midaz/
│   ├── matcher/
│   ├── fetcher/
│   └── plugins/
├── templates/
│   ├── deployments.yaml
│   ├── services.yaml
│   ├── configmaps.yaml
│   ├── secrets.yaml
│   ├── _helpers.tpl
│   ├── migration-jobs.yaml
│   └── hooks.yaml
└── README.md
```

**Best practices embraced:**

- Values.yaml is split by component with comments specifying tuning parameters for production workloads.
- Templates use named partials for reuse and readability.
- Resource requests and limits are explicitly declared to avoid Pod eviction and node instability.
- Liveness and readiness probes are configured to prevent service downtime during rolling upgrades.
- Chart versions track backwards-compatible and breaking changes following SemVer.

### Helm Hooks: Lifecycle and Usage

Helm hooks enable controlled execution of special Kubernetes resources during deployment lifecycles. Proper use is critical for migrations and plugin initialization.

| Hook Type      | Description                                  | Usage                                              | Failure handling                          |
|----------------|----------------------------------------------|----------------------------------------------------|-------------------------------------------|
| `pre-install`  | Executed before resource installation        | Setup initial DB schemas                            | Abort install on failure                   |
| `post-install` | Executed after resource installation         | Warm up caches, trigger downstream jobs            | Log failure; allow continued operation    |
| `pre-upgrade`  | Before upgrade starts                         | Check current schema; lock tables                   | Block upgrade on validation failure       |
| `post-upgrade` | After upgrade completes                       | Run data migrations, clear cache                    | Alert via monitoring; flag manual intervention |
| `pre-delete`   | Before resources deleted                      | Safely drain traffic, persist state externally     | Delay deletion if job fails                |
| `post-delete`  | After resource deletion                       | Cleanup resources and configs                        | Ensure no orphaned resources remain       |

**Example of a migration job hook in `migration-jobs.yaml`:**

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: lerian-migrate-{{ .Release.Name }}
  annotations:
    "helm.sh/hook": pre-upgrade
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    spec:
      containers:
      - name: migration
        image: lerian/migration:{{ .Values.migration.version }}
        command: ["sh", "-c", "/scripts/run_migration.sh"]
      restartPolicy: OnFailure
```

**Key points:**

- Use `hook-delete-policy` to auto-clean successful jobs.
- Migration jobs must have retries and fail fast if critical errors occur.
- Use of init containers (see next section) to validate the environment before performing migrations.

### Init Containers: Initialization and Readiness Probes

Init containers serve as gatekeepers ensuring pods start only after prerequisites are met.

**Common init container responsibilities in Lerian's architecture:**

- Validate database connectivity before launching main app containers.
- Check for the presence and integrity of critical config files or secrets.
- Run lightweight data transformations or schema validations that don't require main app runtime.
- Acquire distributed locks to prevent concurrent migration runs across replicas.
- Prepare volume mounts and filesystems especially when plugins introduce shared state.

**Example init container YAML snippet for Midaz pod:**

```yaml
initContainers:
- name: db-connection-check
  image: busybox:1.33
  command:
  - sh
  - -c
  - |
    until nc -z -v -w30 {{ .Values.database.host }} {{ .Values.database.port }}; do
      echo "Waiting for database connection..."
      sleep 5
    done
  resources:
    requests:
      cpu: 10m
      memory: 32Mi
```

**Readiness and Liveness Probes practical tips:**

- Use HTTP-based readiness probes that confirm the pod is ready to serve traffic.
- Liveness probes should detect deadlocks or internal errors causing the pod to become unresponsive.
- For the Matcher service, readiness probes include validating data indexes loaded into memory.
- Midaz pods use health endpoints that confirm job queues are empty or processing correctly.

---

## Handling Large Scale Data and Worst-case Scenarios

### Timeout Management

Timeouts are critical in preventing cascading failures and degraded cluster performance.

**Areas requiring tight timeout control:**

- API calls from Fetcher to third-party services - exceeding typical 5 seconds triggers retry policy.
- Database query timeout thresholds are set at 30 seconds default, adjustable based on query complexity.
- Helm upgrade or migration job timeouts are configured globally at 10 minutes to prevent blocking cluster operations.
- Client-facing services establish circuit breakers (via Envoy or Istio) to fail fast and fall back gracefully.

**Example YAML configuring timeout for Fetcher service HTTP client:**

```yaml
fetcher:
  httpTimeout: 5s
  maxRetries: 3
```

### Database Migration Strategies for Massive Datasets

Migrating databases with tens to hundreds of millions of rows across multiple tables requires precise strategies:

- **Online migrations using feature toggles:** Deploy schema changes backward-compatible with current services, then enable new features gradually.
- **Chunked data migrations:** Migrate large tables in batches, using pagination and limits to avoid locks and excessive I/O.
- **Down-time windows:** For breaking migrations, schedule 1-2 hour maintenance with clear client announcements.
- **Audit logging:** Capture migration steps, affected row counts, and errors for rollback triggers.
- **Resource throttling:** Limit CPU and I/O consumption of migration jobs to prevent impact on production workloads.

**Example approach for a column type change migration:**

1. Add a new column with the target type (e.g., `new_timestamp TIMESTAMP NULL`).
2. Backfill `new_timestamp` from existing column `old_timestamp` in batches.
3. Modify application code to read from `new_timestamp`.
4. Remove `old_timestamp` column in final migration step.

### Scaling Midaz, Matcher, and Fetcher Efficiently

To handle massive workloads, scale components based on metrics like queue depth, request latency, and CPU load:

| Component | Scaling Metric          | Scaling Strategy                                   |
|-----------|------------------------|---------------------------------------------------|
| Midaz     | Job queue length       | Horizontal pod autoscaling (HPA) on queue length |
| Matcher   | Request latency        | HPA on average latency and memory consumption     |
| Fetcher   | External API rate limits | Rate-limit aware scaling and queue backpressure   |
| Plugins   | CPU and memory usage   | Monitor and autoscale depending on plugin complexity |

**Pro tip:** Use custom metrics server integration with Prometheus to feed Kubernetes HPA for more precise scaling.

---

## Client-Facing Upgrade Procedures

### Pre-Upgrade Checks and Validation

Before upgrading helm charts on client clusters:

- Verify cluster health: check node readiness, API server availability, and resource quotas.
- Validate current application state, ensure no pending migration jobs.
- Confirm backups exist and integrity is verified.
- Run `helm lint` and dry-run upgrade commands to detect manifest or validation errors.
- Communicate expected downtime or service interruptions clearly to clients.
- Review the versions of Kubernetes, Helm, and CLI compatibility.

### Step-by-step Helm Upgrade Process

1. **Preparation:**

   ```bash
   helm repo update
   helm fetch lerian/lerian-studio --version <target-version>
   helm dependency update lerian-helm-chart/
   ```

2. **Dry Run Upgrade:**

   ```bash
   helm upgrade lerian-studio lerian/lerian-studio \
     --values prod-values.yaml \
     --dry-run --debug
   ```

   Review output for warnings, resource conflicts, and hooks execution order.

3. **Perform Upgrade with Logs:**

   ```bash
   helm upgrade lerian-studio lerian/lerian-studio \
     --values prod-values.yaml \
     --timeout 10m0s \
     --wait
   ```

4. **Monitor Migration Job Status:**

   ```bash
   kubectl get jobs -n lerian-studio
   kubectl logs job/lerian-migrate-<release-name>
   ```

5. **Validate service readiness:**

   ```bash
   kubectl rollout status deployment/midaz
   kubectl rollout status deployment/matcher
   kubectl rollout status deployment/fetcher
   ```

6. **Post-upgrade verification:**

   - Run integration tests on staging if available.
   - Notify clients of upgrade completion.
   - Monitor logs and metrics intensively for 1-2 hours post-upgrade.

### Rollback and Disaster Recovery

If a failure is detected:

1. Abort upgrade and run:

   ```bash
   helm rollback lerian-studio <previous-revision>
   ```

2. If migrations have irreversible side-effects, follow manual rollback documentation.

3. Restore from backups if necessary.

4. Escalate to specialist support teams immediately for critical production incidents.

---

## Tech Support Best Practices and Troubleshooting

### Monitoring and Logging

Robust monitoring infrastructure is essential to catch incidents early.

- **Prometheus metrics** scraped from pods expose CPU, memory, queue depths, and custom business metrics.
- **Grafana dashboards** provide visualizations of service health trends.
- **Centralized logging** via EFK (Elasticsearch, Fluentd, Kibana) stack aggregates logs with request tracing.
- Deploy alertmanager rules to dispatch notifications on anomalies like frequent restarts, migration job failures, or high error rates.

### Common Issues and Resolution Patterns

| Symptom                              | Possible Cause                  | Resolution Steps                                                      |
|------------------------------------|-------------------------------|---------------------------------------------------------------------|
| Migration Job stuck/running too long | Deadlock on DB tables          | Check migration logs, kill stuck jobs, prioritize lightweight migrations, investigate DB locks |
| Pod fails to start (CrashLoopBackOff) | Init container failure or expired secrets | Check init container logs, rotate secrets, validate configmaps      |
| Upgrade hangs on Helm hook          | Migration job hung or failed    | Identify hook jobs, check logs, manually delete failed jobs, retry helm upgrade |
| High memory usage on Matcher        | Large dataset in memory load    | Review cache sizes, scale replicas, perform data pruning             |
| API latency spikes on Fetcher       | External API rate limiting or network issues | Check external endpoint status, apply backoff strategies, scale out pods |

### Communication Protocols for Incident Management

- Use Slack and PagerDuty for real-time alerting.
- Incident Commander assigns roles: primary engineer, communication lead, database specialist.
- Maintain incident log with root cause, immediate resolution, and long-term fix plans.
- Post-mortem reports shared with clients and internal teams.

---

## Relation to Other Specialist Files

This file, **43-lerian-helm-specialist.md**, ties tightly into the broader specialist-teams repository by serving as the definitive guide for operations and tech support related to Lerian Studio's Helm-driven deployments.

- It complements **01-kubernetes-infrastructure.md**, which covers cluster setup and node management foundational for application deployments.
- Works alongside **12-database-admin-specialist.md**, since Midaz and Matcher migrations often require database operations expertise.
- Relates to **27-logging-and-monitoring-specialist.md**, providing the monitoring and alerting backbone referenced herein.
- Interfaces with **35-release-engineering-specialist.md**, governing CI/CD pipeline integration with Helm upgrades.
- Connects to **40-performance-tuning-specialist.md**, as scaling and tuning Midaz and Matcher are critical for dealing with huge datasets.
- Supports **44-plugin-development-specialist.md**, detailing plugin lifecycle and extension points within Helm charts.

Together, these files create a cohesive knowledge base ensuring end-to-end coverage for deploying, maintaining, and troubleshooting Lerian Studio systems in production.

---

## Appendices

### Glossary of Terms

| Term           | Definition                                                        |
|----------------|------------------------------------------------------------------|
| **Helm**       | Kubernetes package manager for defining, installing, and upgrading applications. |
| **Helm Hook**  | Special annotations on Kubernetes resources for lifecycle events. |
| **Init Container** | Containers that run before app containers to perform initialization. |
| **Migration Job** | Kubernetes batch job that performs schema or data migrations.   |
| **HPA**        | Horizontal Pod Autoscaler; scales pods based on metrics.         |
| **Liveness Probe** | Checks if a pod is alive; triggers restart if failed.          |
| **Readiness Probe** | Checks if a pod is ready to accept traffic.                    |
| **CrashLoopBackOff** | Pod restart failure pattern indicating repeated crashes.    |
| **Idempotent** | Operations that can be applied multiple times without changing the result beyond the initial application. |
| **Backfill**   | Operation to populate missing or old data retroactively.         |

### Helm Hooks Reference Table

| Hook Name       | Event               | Typical Use Case                              |
|-----------------|---------------------|-----------------------------------------------|
| `pre-install`   | Before install      | Initial DB setup, pre-flight checks          |
| `post-install`  | After install       | Cache warming, plugin activation              |
| `pre-upgrade`   | Before upgrade      | Locking DB, validating schemas                 |
| `post-upgrade`  | After upgrade       | Running migrations, cleanup                    |
| `pre-delete`    | Before deletion     | Draining traffic, persisting state             |
| `post-delete`   | After deletion      | Cleanup leftovers, de-registering plugins      |

### Useful kubectl and Helm CLI Commands

#### Helm

```bash
# Upgrade with wait and timeout
helm upgrade <release> <chart> --wait --timeout 10m0s -f values.yaml

# Rollback to previous revision
helm rollback <release> [revision]

# List release history
helm history <release>

# Dry-run upgrade to validate manifest
helm upgrade <release> <chart> --dry-run --debug

# Run helm lint to check chart validity
helm lint <path-to-chart>
```

#### kubectl

```bash
# Get all pods with labels in namespace
kubectl get pods -n lerian-studio -l app=midaz

# Check logs of a specific job
kubectl logs job/<job-name> -n lerian-studio

# Wait until deployment rollout completes
kubectl rollout status deployment/<deployment-name> -n lerian-studio

# Describe pod for troubleshooting events
kubectl describe pod <pod-name> -n lerian-studio

# Check for configmaps and secrets
kubectl get configmaps -n lerian-studio
kubectl get secrets -n lerian-studio
```

---

# End of 43-lerian-helm-specialist.md

This file must be continuously updated to reflect evolving production scenarios, upgrades in Kubernetes, Helm versions, and operational lessons learned through support tickets and incident reviews. This living document ensures Lerian Studio specialists remain equipped to handle the full spectrum of challenges in a production environment.

### Deep-Dive Troubleshooting Scenarios

In complex Helm deployments such as Lerian Studio's, incidents frequently involve multi-component failures or subtle misconfigurations that do not immediately present clear error messages. Specialists must develop a methodical troubleshooting approach rooted in layered investigation and correlation of logs, metrics, and event histories.

#### Scenario 1: Midaz Pod CrashLoopBackOff Immediately After Upgrade

**Symptom:**  
After running `helm upgrade` to a new chart version with Midaz updates, the Midaz pod repeatedly crashes within 5 seconds of start, showing CrashLoopBackOff status.

**Investigation steps:**

- Run `kubectl describe pod <midaz-pod>` to check for pod-level events, OOMKilled conditions, or probe failures.
- Review Midaz container logs: `kubectl logs <midaz-pod>`
- Confirm resource requests and limits are adequate; monitor node memory usage.
- Check recent Helm `pre-upgrade` hooks for DB schema locks that could cause Midaz service deadlock.
- Validate ConfigMaps and Secrets mounted on Midaz pod have correct values for environment variables.
- Confirm any new environment variables introduced in upgrade align with upstream service expectations.
- Check if Init Containers related to Midaz completed successfully, particularly those setting up connectivity or downloading plugins.

**Root causes found in past incidents:**

- Schema validation failure causing Midaz to reject DB connection.
- Plugin activation hooks overriding environment variables unexpectedly.
- Resource limits causing pod eviction or SIGKILL during initialization.
- Sidecar container crashing and triggering pod restart.

**Remediation:**

- Rollback Helm release (`helm rollback`) to last known good version.
- Adjust resource limits and restart deployment.
- Correct ConfigMap or Secret values after comparing with previous working release.
- Fix or disable failing plugin temporarily to isolate problem.

---

#### Scenario 2: Fetcher Job Hangs on Massive Dataset Import

**Symptom:**  
Fetcher batch jobs intermittently hang or run far beyond anticipated durations when ingesting large datasets from external sources, leading to delays in downstream data availability.

**Investigation steps:**

- Inspect Fetcher pod logs for any indication of retries, connection timeouts, or unhandled exceptions.
- Confirm Fetcher job resource allocation is sufficient, including CPU throttling or memory swapping issues.
- Check network policies or firewall rules affecting Fetcher’s connectivity to data sources.
- Analyze Kubernetes events for pod restarts or node pressure signals.
- Review any recent upgrades to Fetcher image or changes in job concurrency settings.
- Examine Helm values overrides to ensure batch size and parallelism parameters are tuned for large datasets.

**Root causes found:**

- Inefficient parallelization causing bottlenecks in processing streams.
- Memory leaks or unclosed database connections leading to resource exhaustion.
- Network interruptions causing retries that prolong job lifetime.
- Misconfigured batch sizes beyond backend DB capacity.

**Remediation:**

- Tune Helm `values.yaml` settings such as `fetcher.batchSize` and `fetcher.parallelWorkers`.
- Implement horizontal pod autoscaling (HPA) or use Kubernetes jobs with controlled parallelism.
- Coordinate with network team to ensure reliable connectivity.
- Analyze and patch Fetcher application code for resource misuse.
- Use Kubernetes `initContainers` to perform environment pre-checks for connectivity and resource quotas before fetching starts.

---

### Advanced Database Migration Recovery

Database migrations are critical in Lerian Studio’s Helm upgrade lifecycle and common culprits of failed upgrades or inconsistent states. Recovery from failed or partially applied migrations requires cautious orchestration to avoid data loss or corruption.

#### Common Migration Failure Modes

- Schema changes locking tables excessively, blocking application writes.
- Migration jobs timing out or crashing midway.
- Missing migration scripts or version skew between application and DB schema.
- Rollback failures leaving DB in inconsistent states.

#### Best Practices for Migration Recovery

1. **Backup Before Migration**  
   Always take a consistent backup of the database before triggering any migration job. Automated snapshots should be triggered as part of the `pre-upgrade` Helm hook.

2. **Validate Migration Scripts Locally**  
   Run migrations against staging or local dev DBs to ensure no errors or locking issues.

3. **Incremental Migrations**  
   Break complex migrations into smaller atomic steps to isolate failures easily.

4. **Use Transactional Migrations Where Possible**  
   Ensure DB migration scripts wrap schema changes inside transactions to allow rollback on errors (Postgres, MySQL support this natively).

5. **Implement Detailed Migration Logging**  
   Enhance migration jobs with verbose logging and capture exit codes for auditability.

#### Recovery Procedure for Failed Migration Job

1. Identify the migration phase that failed by checking Helm `post-upgrade` hook job logs and release history.
2. Confirm if the migration partially applied any schema changes (check version tables or migration meta tables).
3. Restore database from backup if inconsistency or corruption is detected.
4. Manually run pending migrations step-by-step in a controlled environment, correcting any issues detected.
5. Update Helm Chart version and embedded migration scripts to reflect manual fixes.
6. Re-deploy Helm chart with `--force` to reapply migrations safely.
7. Monitor application connectivity to DB post migration to confirm stability.

**Incident example:**  
A schema migration added a new column while locking a frequently accessed table, causing the Midaz pod to timeout on DB queries. Recovery involved restoring from backups and applying migration during low-traffic hours with increased DB timeout settings.

---

### Handling Massive Datasets in Fetcher

Fetcher is at the heart of ingestion workflows in Lerian Studio that process massive volumes of data requiring specialised tuning and architecture considerations.

#### Architectural Considerations

- **Batch Processing**  
  Fetcher processes data in pre-configured batch sizes to balance memory usage and throughput.

- **Parallelization and Sharding**  
  Using multiple parallel workers, possibly distributed across pods or nodes, to split datasets and accelerate processing.

- **Backpressure and Rate Limiting**  
  Implement backpressure mechanisms to avoid overloading downstream storage or processing pipelines.

- **Failure Handling and Retry Logic**  
  Robust retry strategies on network failures or partial write errors.

#### Performance Optimization Techniques

- Enable streaming processing inside Fetcher where possible instead of bulk loading.
- Use Helm `values.yaml` tunables such as `fetcher.parallelism`, `fetcher.batchSize`, and `fetcher.retryLimit`.
- Monitor resource usage extensively; employ pod resource limits to prevent node pressure.
- Leverage Kubernetes `affinity` and `nodeSelector` to schedule Fetcher pods on nodes with optimal CPU and memory.
- Use persistent volume claims (PVCs) for intermediate data storage with optimized IOPS.

#### Monitoring and Alerting

- Enable Prometheus exporters for Fetcher to track job success/failure, processing latency, throughput.
- Set alerts for prolonged job execution beyond thresholds.
- Use Kubernetes pod restart counts and error rates as early-warning signals.

---

### Detailed Tech Support Runbooks for Common Tasks

To empower specialists with quick and reliable resolutions, the following runbooks address frequent operational tasks with clear step-by-step instructions.

#### Runbook: Performing a Safe Helm Upgrade Including Migration Jobs

1. Prepare and review change logs and Helm chart version updates.
2. Backup databases involved including Midaz DB.
3. Run `helm lint` to validate chart integrity.
4. Execute a dry-run upgrade to validate templates:
   ```
   helm upgrade <release> <chart> --dry-run --debug -f values.yaml
   ```
5. Apply upgrade with wait and timeout:
   ```
   helm upgrade <release> <chart> --wait --timeout 15m -f values.yaml
   ```
6. Monitor rollout status and job completions:
   ```
   kubectl rollout status deployment/midaz -n lerian-studio
   kubectl get jobs -n lerian-studio
   ```
7. Inspect logs of migration jobs and post-upgrade hooks.
8. Verify application readiness via health endpoints or synthetic transactions.
9. If failure detected, rollback and open incident ticket with logs.

---

#### Runbook: Recovering from Plugin Activation Failures

1. Identify the plugin causing failure from Helm hook logs (`post-install` or `post-upgrade`).
2. Check ConfigMap/Secret values passed to plugins.
3. Disable problematic plugin in Helm values and rerun upgrade.
4. Analyze plugin logs and adjust configuration or update plugin version.
5. Re-enable plugin on successful fixes.
6. Document findings and update plugin activation procedures.

---

#### Runbook: Investigating Init Container Failures

1. Get pod details and look for init container statuses:
   ```
   kubectl describe pod <pod-name> -n lerian-studio
   ```
2. Check init container logs:
   ```
   kubectl logs <pod-name> -c <init-container-name> -n lerian-studio
   ```
3. Common causes include missing downloads, permission issues, or config errors.
4. Correct ConfigMaps/Secrets or Helm values accordingly.
5. Redeploy pod or Helm release.

---

#### Runbook: Handling Helm Release Rollbacks

1. Identify stable previous release revision:
   ```
   helm history <release>
   ```
2. Initiate rollback command:
   ```
   helm rollback <release> <revision>
   ```
3. Monitor rollout and watch for any migration reversions or data issues.
4. Communicate rollback actions to stakeholders and update documentation.

---

### Conclusion

Mastering the complexity of Lerian Studio Helm deployments demands a deep understanding of Helm lifecycle hooks, Kubernetes resource management, and application-specific components like Midaz, Matcher, Fetcher, and plugins. Through detailed troubleshooting steps, migration safeguards, and performance tuning strategies, specialists ensure smooth deployment workflows and rapid incident recovery.

This evolving knowledge base must continually integrate operational insights, tooling advances, and application behavior patterns to maintain resilient production environments capable of handling scale, change, and unexpected failures gracefully. Tech support engineers stand at the intersection of infrastructure, application logic, and end-user impact, equipped through documentation such as this to minimize downtime and maximize Lerian Studio’s service reliability.