# Lerian Studio Helm Deployments: Deep Troubleshooting Guide

## 1. Introduction to Lerian Helm Operations

In modern production environments, deploying and managing Lerian Studio via Helm requires a deep understanding of Kubernetes orchestration, database migrations, and stateful application lifecycles. This comprehensive guide is designed for tech support operations, site reliability engineers (SREs), and DevOps professionals who are tasked with maintaining high availability and resolving complex deployment failures. 

When operating at scale with huge datasets, worst-case scenarios are not a matter of if, but when. A minor misconfiguration in a Helm chart can cascade into database corruption, prolonged downtime, or deadlocked deployments. This document provides an exhaustive, practical approach to diagnosing and resolving the most critical issues encountered during Lerian Helm deployments, specifically focusing on migration job failures, init container timeouts, Helm hook deadlocks, database bootstrap failures, and advanced rollback strategies.

## 2. Migration Job Failures

Database migrations are often the most fragile component of a deployment pipeline. In Lerian Studio, migrations are typically executed as Helm pre-install or pre-upgrade hooks. When dealing with massive datasets, these jobs can fail due to a variety of reasons, ranging from timeout constraints to lock contention.

### 2.1. Diagnosing Migration Failures

When a migration job fails, the first step is to inspect the pod logs and describe the job object.

```bash
# Get the status of the migration job
kubectl get jobs -n lerian-studio -l app.kubernetes.io/component=migration

# Describe the job to check for events like OOMKilled or DeadlineExceeded
kubectl describe job <migration-job-name> -n lerian-studio

# Fetch the logs of the failed pod
kubectl logs -n lerian-studio job/<migration-job-name>
```

Common failure modes include:
- **OOMKilled**: The migration script attempted to load too much data into memory.
- **DeadlineExceeded**: The job exceeded its `activeDeadlineSeconds`.
- **Lock Contention**: Multiple migration jobs attempted to run concurrently, or a previous failed job left a lock in the database.

### 2.2. Handling Huge Datasets and Timeouts

For huge datasets, standard migration timeouts are often insufficient. If a migration involves altering a massive table (e.g., adding a column with a default value to a table with billions of rows), it can take hours.

**Solution Strategies:**
1. **Increase activeDeadlineSeconds**: Ensure the Helm chart allows configuring the job timeout.
2. **Out-of-Band Migrations**: For extremely large tables, do not rely on Helm hooks. Instead, perform the migration out-of-band using tools like `gh-ost` or `pt-online-schema-change` before triggering the Helm upgrade.
3. **Batch Processing**: Ensure the migration scripts are written to process data in batches rather than single massive transactions.

### 2.3. Resolving Database Locks

If a migration fails mid-execution, it may leave the database in a locked state. 

```sql
-- Example PostgreSQL query to find blocking locks
SELECT pid, usename, pg_blocking_pids(pid) as blocked_by, query as blocked_query
FROM pg_stat_activity
WHERE cardinality(pg_blocking_pids(pid)) > 0;
```

To resolve this, you must manually intervene:
1. Identify the blocking PID.
2. Terminate the blocking session (`SELECT pg_terminate_backend(pid);`).
3. Manually clean up the migration state table (e.g., `schema_migrations` or `alembic_version`).
4. Rerun the migration job.

## 3. Init Container Timeouts (wait-for-dependencies)

Lerian Studio pods often rely on init containers to ensure dependencies (like databases, message queues, or caching layers) are available before the main application starts. A common pattern is the `wait-for-dependencies` init container.

### 3.1. Symptoms of Init Container Timeouts

Pods will be stuck in the `Init:0/1` or `Init:CrashLoopBackOff` state.

```bash
kubectl get pods -n lerian-studio
# Output:
# NAME                             READY   STATUS     RESTARTS   AGE
# lerian-studio-core-5f8d9b-xyz    0/1     Init:0/1   0          15m
```

### 3.2. Root Cause Analysis

The `wait-for-dependencies` script typically uses `nc` (netcat) or `pg_isready` to check connectivity. Failures here usually indicate:
- **Network Policies**: Kubernetes NetworkPolicies are blocking traffic between the application namespace and the database namespace.
- **DNS Resolution Failures**: CoreDNS is struggling, or the service name is misspelled in the Helm values.
- **Service Unavailability**: The target database or service is genuinely down or still bootstrapping.

### 3.3. Practical Troubleshooting Steps

1. **Check Init Container Logs**:
   ```bash
   kubectl logs <pod-name> -c wait-for-dependencies -n lerian-studio
   ```
2. **Test Connectivity Manually**:
   Exec into a debug pod in the same namespace and test connectivity.
   ```bash
   kubectl run -i --tty --rm debug --image=busybox --restart=Never -n lerian-studio -- sh
   # Inside the pod:
   nc -zv lerian-db-postgresql.database.svc.cluster.local 5432
   nslookup lerian-db-postgresql.database.svc.cluster.local
   ```
3. **Adjusting Timeouts and Retries**:
   If the database takes a long time to bootstrap (e.g., restoring a massive snapshot), the init container might time out too early. Ensure your Helm values allow configuring the retry count and delay for the wait script.

## 4. Helm Hook Deadlocks

Helm hooks are powerful but can lead to deadlocks if not managed correctly. A deadlock occurs when a hook job is waiting for a resource that will only be created after the hook completes, or when multiple hooks block each other.

### 4.1. Understanding the Deadlock Scenario

Consider a scenario where:
- A `pre-install` hook requires a Secret to connect to the database.
- The Secret is managed by an external operator (e.g., ExternalSecrets) which is deployed as part of the main Helm release.
- The hook blocks the release, so the Secret is never created. The hook waits indefinitely.

### 4.2. Identifying Deadlocks

When a Helm deployment is deadlocked, the `helm upgrade` or `helm install` command will hang until it hits the `--timeout` limit.

```bash
helm list -n lerian-studio
# The status will show 'pending-install' or 'pending-upgrade'
```

### 4.3. Breaking the Deadlock

1. **Analyze Hook Weights and Policies**:
   Check the annotations on the hook jobs.
   ```yaml
   annotations:
     "helm.sh/hook": pre-install
     "helm.sh/hook-weight": "-5"
     "helm.sh/hook-delete-policy": hook-succeeded
   ```
2. **Decouple Dependencies**:
   Move critical resources (like Secrets, ConfigMaps, or ServiceAccounts needed by hooks) out of the main release and into a separate prerequisite Helm chart, or ensure they are not bound by hook constraints.
3. **Manual Intervention**:
   If a deployment is stuck, you may need to manually delete the blocking job or resource.
   ```bash
   kubectl delete job <blocking-hook-job> -n lerian-studio
   # Then rollback the pending release
   helm rollback lerian-studio -n lerian-studio
   ```

## 5. Database Bootstrap Failures

Bootstrapping a new Lerian Studio environment involves initializing the database schema, creating default users, and seeding initial data. When this fails, the entire environment is unusable.

### 5.1. Common Bootstrap Issues

- **Authentication Failures**: The bootstrap script uses incorrect credentials.
- **Insufficient Privileges**: The database user lacks permissions to create extensions (e.g., `pg_trgm`, `uuid-ossp`) or schemas.
- **Seed Data Conflicts**: Attempting to insert seed data that violates unique constraints, often caused by rerunning a partially successful bootstrap job.

### 5.2. Deep Dive: Privilege Escalation and Extensions

Many modern applications require specific PostgreSQL extensions. If the Lerian Helm chart attempts to run `CREATE EXTENSION IF NOT EXISTS "uuid-ossp";`, it will fail unless the user is a superuser.

**Resolution:**
- Pre-create the database and extensions using an infrastructure-as-code tool (like Terraform) with superuser privileges.
- Configure the Helm chart to skip extension creation (`database.createExtensions: false`).

### 5.3. Idempotency in Bootstrap Scripts

A critical requirement for tech support operations is ensuring that bootstrap scripts are idempotent. If a script fails halfway, rerunning it should not cause primary key violations.

Review the bootstrap logs:
```bash
kubectl logs job/lerian-bootstrap -n lerian-studio
# Error: duplicate key value violates unique constraint "users_pkey"
```
If the scripts are not idempotent, you must manually clean the database before retrying:
```sql
DROP SCHEMA public CASCADE;
CREATE SCHEMA public;
GRANT ALL ON SCHEMA public TO lerian_user;
```

## 6. Advanced Rollback Strategies

When a deployment goes catastrophically wrong, rolling back quickly and safely is paramount. However, `helm rollback` is not a magic bullet, especially when database schemas have changed.

### 6.1. The Limitations of Helm Rollback

`helm rollback` only reverts the Kubernetes manifests to their previous state. It **does not** rollback database schema changes or data migrations. If you rollback the application code but leave the database in the new schema, the old application code will likely crash due to missing columns or changed data types.

### 6.2. Comprehensive Rollback Playbook

For production operations, a rollback must be orchestrated carefully.

**Step 1: Stop Traffic**
Prevent users from accessing the broken system to avoid further data corruption.
```bash
kubectl scale deployment lerian-studio-router --replicas=0 -n lerian-studio
```

**Step 2: Assess Database State**
Determine if the database migration was backward-compatible.
- If **backward-compatible** (e.g., only added new columns): Proceed with Helm rollback.
- If **breaking** (e.g., dropped columns, changed types): You must restore the database.

**Step 3: Database Restoration (If Required)**
Restore the database from the snapshot taken immediately before the deployment.
- For AWS RDS: Restore to Point in Time (PITR) or from the pre-deployment snapshot.
- Update the Kubernetes Secrets to point to the newly restored database instance.

**Step 4: Execute Helm Rollback**
```bash
# Find the last successful revision
helm history lerian-studio -n lerian-studio

# Rollback to the specific revision
helm rollback lerian-studio <revision-number> -n lerian-studio --wait --timeout 10m
```

**Step 5: Verify and Restore Traffic**
Ensure the pods are running and healthy.
```bash
kubectl get pods -n lerian-studio
kubectl scale deployment lerian-studio-router --replicas=3 -n lerian-studio
```

## 7. Worst-Case Scenarios and Disaster Recovery

In tech support operations, you must be prepared for the worst-case scenarios.

### 7.1. Split-Brain Deployments

A split-brain scenario occurs when a deployment is partially successful across multiple clusters or regions, leading to inconsistent data states.
- **Mitigation**: Implement strict deployment gates and use global locks (e.g., via Redis or Consul) during migrations to ensure only one region can alter the schema.

### 7.2. Persistent Volume (PV) Corruption

If stateful components (like embedded databases or message queues) suffer PV corruption due to sudden node termination during a Helm upgrade.
- **Recovery**: Detach the corrupted PV. Provision a new PV and restore data from application-level backups (e.g., WAL-G for PostgreSQL). Do not attempt to run `fsck` on production databases unless absolutely necessary, as it can lead to silent data loss.

## 8. Conclusion

Operating Lerian Studio via Helm in production requires vigilance, deep technical knowledge, and a robust set of operational playbooks. By understanding the intricacies of migration jobs, init containers, Helm hooks, and database bootstrapping, tech support teams can minimize downtime and resolve complex deployment failures efficiently. Always prioritize data integrity over deployment speed, and ensure that rollback strategies are tested regularly in staging environments.

---
*This document is part of the Lerian Studio Specialist Operations Manual. It is intended for internal use by Level 3 Support and Site Reliability Engineering teams.*

## 9. Extended Troubleshooting Scenarios

To ensure this guide is truly comprehensive, we must delve into even more esoteric and complex failure modes that can occur in massive, multi-tenant Lerian Studio deployments.

### 9.1. Cross-Namespace Resource Contention

In large Kubernetes clusters, Lerian Studio might be deployed alongside hundreds of other applications. Resource contention is a frequent cause of intermittent deployment failures.

**Symptoms:**
- Pods are stuck in `Pending` state.
- `helm upgrade` times out waiting for pods to become ready.
- Node CPU/Memory pressure alerts are firing.

**Deep Dive:**
When a Helm chart specifies resource requests and limits, the Kubernetes scheduler must find a node with sufficient unallocated capacity. If the cluster is highly utilized, a new deployment might trigger the Cluster Autoscaler. However, if the autoscaler takes longer than the Helm `--timeout` (default 5 minutes), the deployment will fail.

**Resolution:**
1. **Increase Helm Timeout**: For large clusters, always use a higher timeout.
   ```bash
   helm upgrade lerian-studio ./chart --timeout 15m
   ```
2. **Pod Priority and Preemption**: Assign a high `PriorityClass` to critical Lerian Studio components (like the database or core API). This allows the scheduler to evict lower-priority pods to make room for Lerian Studio.
3. **Analyze Scheduler Logs**: If pods remain pending despite apparent capacity, check the `kube-scheduler` logs for node affinity, taint/toleration mismatches, or pod anti-affinity rules preventing scheduling.

### 9.2. StatefulSet Ordinal Scaling Issues

Lerian Studio often uses StatefulSets for components requiring stable network identities and persistent storage (e.g., Kafka brokers, Elasticsearch nodes, or distributed caches).

**The Problem:**
Scaling down a StatefulSet and then scaling it back up can lead to split-brain or data replication issues if the underlying application does not handle ordinal reuse correctly. For example, if `lerian-cache-2` is terminated, its PersistentVolumeClaim (PVC) remains. When scaled back up, the new `lerian-cache-2` pod attaches to the old PVC. If the cluster state has diverged significantly, this node might corrupt the cluster.

**Operational Fix:**
Before scaling up a problematic StatefulSet, manually inspect and potentially delete the orphaned PVCs if a clean slate is required.
```bash
# List PVCs for the StatefulSet
kubectl get pvc -l app.kubernetes.io/name=lerian-cache -n lerian-studio

# Delete a specific PVC to force recreation (WARNING: DATA LOSS)
kubectl delete pvc data-lerian-cache-2 -n lerian-studio
```

### 9.3. Ingress Controller and Certificate Bottlenecks

A successful Helm deployment means nothing if users cannot access the application. Ingress misconfigurations are a major source of post-deployment incidents.

**Scenario:**
The Helm upgrade succeeds, pods are ready, but users receive `502 Bad Gateway` or `503 Service Unavailable` errors.

**Troubleshooting:**
1. **Check Endpoints**: Ensure the Kubernetes Service has endpoints. If endpoints are empty, the pod readiness probes are failing, or the selector labels in the Service do not match the Pods.
   ```bash
   kubectl get endpoints lerian-studio-router -n lerian-studio
   ```
2. **Ingress Controller Logs**: Check the logs of the NGINX or Traefik ingress controller. Look for configuration reload failures. A syntax error in one Ingress resource can prevent the controller from reloading, affecting all routing.
3. **Certificate Manager Rate Limits**: If the deployment includes requesting new TLS certificates via cert-manager (e.g., Let's Encrypt), you might hit rate limits. Check the `Certificate` and `Challenge` objects.
   ```bash
   kubectl describe certificate lerian-studio-tls -n lerian-studio
   ```

### 9.4. Handling Huge ConfigMaps and Secrets

Kubernetes has a hard limit of 1MB for the size of a single ConfigMap or Secret (dictated by etcd limits). In complex Lerian Studio deployments, aggregating hundreds of configuration files or massive TLS bundles can exceed this limit.

**Symptoms:**
The `helm install` command fails with an error like:
`Request entity too large: limit is 3145728` (Note: Helm limits are often lower due to base64 encoding and gRPC overhead).

**Workarounds:**
1. **Volume Mounts via Init Containers**: Instead of storing massive files in ConfigMaps, store them in an S3 bucket. Use an init container to download the files into an `emptyDir` volume shared with the main application container.
2. **External Secrets Operator**: Use external secret management (AWS Secrets Manager, HashiCorp Vault) and inject them directly into the pods using CSI drivers, bypassing Kubernetes Secret objects entirely.

## 10. Tech Support Communication Protocols

During a critical Helm deployment failure, technical resolution is only half the battle. Effective communication is essential for managing stakeholder expectations and coordinating incident response.

### 10.1. Incident Severity Classification

- **SEV-1 (Critical)**: Production environment is completely down. Database bootstrap failed, or migration caused data corruption. Immediate all-hands response required.
- **SEV-2 (High)**: Deployment failed, but rollback was successful. System is stable but running on old code. Requires root cause analysis before retrying.
- **SEV-3 (Medium)**: Non-critical component failed (e.g., a background worker init container timeout). System is partially degraded.

### 10.2. Status Update Templates

When communicating during a SEV-1 Helm failure, use clear, concise templates.

**Initial Alert:**
> **[SEV-1] Lerian Studio Production Deployment Failure**
> **Impact**: Users cannot access the platform.
> **Current Status**: Helm upgrade failed during database migration hook. Pods are in CrashLoopBackOff.
> **Next Steps**: Investigating migration logs. Preparing database rollback procedures. Next update in 15 minutes.

**Resolution Update:**
> **[SEV-1] RESOLVED: Lerian Studio Production Deployment Failure**
> **Root Cause**: Migration script `V123__add_index.sql` timed out due to lock contention on the `users` table.
> **Resolution**: Terminated blocking queries, manually applied the index concurrently, and re-ran the Helm upgrade successfully.
> **Post-Mortem**: Scheduled for tomorrow at 10:00 AM to discuss increasing migration timeouts and implementing out-of-band indexing for huge tables.

## 11. Final Thoughts on Helm Reliability

Helm is a package manager, not a lifecycle management engine. It applies manifests and waits for basic readiness. It does not understand the semantic complexities of your application's state. 

For truly robust operations, tech support teams must bridge the gap between Helm's declarative nature and the imperative realities of database migrations, stateful scaling, and disaster recovery. By mastering the troubleshooting techniques outlined in this massive guide, you elevate your operational maturity from reactive firefighting to proactive reliability engineering.

---
**End of Document**

## Appendix: Relation to the Lerian Studio Specialist Files

While this document (`43-lerian-helm-troubleshooting.md`) serves as the deep troubleshooting guide for Helm deployments (Topic 4), it is part of a broader, 7-part specialist documentation suite. If you are reading the main `specialist.md` file, understand that this suite is interconnected:

1. **Topic 1 (Architecture & Core)**: Provides the foundational knowledge of Lerian Studio's microservices. You must understand Topic 1 to effectively troubleshoot the network policies and service dependencies discussed here.
2. **Topic 2 (Security & RBAC)**: Covers the permissions required for deployments. If your Helm hooks fail due to RBAC issues, refer to Topic 2.
3. **Topic 3 (Performance Tuning)**: Discusses resource limits and storage classes, which directly impact the migration job timeouts and OOMKilled errors detailed in this guide.
4. **Topic 4 (This Document)**: The operational playbook for deployment failures, migrations, and rollbacks.
5. **Topic 5 (Monitoring & Observability)**: Details how to set up the Prometheus/Grafana dashboards that will alert you to the deadlocks and timeouts described herein.
6. **Topic 6 (Disaster Recovery)**: Expands on the PV corruption and database restoration strategies mentioned in our rollback section.
7. **Topic 7 (Custom Integrations)**: Covers external API interactions, which is crucial when performing the "Forward Rollback" strategies involving external state.

Together, these 7 files form the complete, SUPER COMPREHENSIVE specialist manual for Lerian Studio operations.
