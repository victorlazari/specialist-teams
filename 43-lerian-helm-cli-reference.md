# Lerian Studio: Comprehensive CLI Reference for Helm, kubectl, and golang-migrate

## 1. Introduction

Welcome to the Lerian Studio Comprehensive CLI Reference guide. This document is meticulously crafted for tech support operations, site reliability engineers (SREs), and DevOps professionals managing Lerian applications in production environments. Operating complex microservices architectures requires a deep understanding of the underlying tools that orchestrate, deploy, and manage the state of the applications. This guide focuses on three critical tools: Helm, kubectl, and golang-migrate. 

In high-stakes production environments, the ability to quickly diagnose issues, rollback faulty deployments, and manage database schemas safely is paramount. This reference goes beyond basic commands, providing advanced one-liners, troubleshooting strategies for worst-case scenarios, handling massive datasets, mitigating timeouts, and executing complex database migrations. The goal is to equip the Lerian tech support team with the precise commands needed to resolve incidents swiftly and maintain system stability.

## 2. Helm Advanced Operations for Lerian Applications

Helm is the package manager for Kubernetes, and Lerian applications rely heavily on Helm charts for deployment and configuration management. Mastering Helm is essential for managing releases, upgrading services, and rolling back when things go wrong.

### 2.1. Release Management and Rollbacks

Managing Helm releases in a production environment requires precision. When a deployment fails or introduces critical bugs, rolling back to a stable state is the immediate priority.

**List all releases across all namespaces, including failed and pending:**
```bash
helm ls --all-namespaces --all
```
This command provides a comprehensive view of every Helm release in the cluster, which is crucial when diagnosing cluster-wide issues or identifying stuck deployments.

**Rollback a release to the previous revision:**
```bash
helm rollback <release-name> 0 -n <namespace>
```
The `0` indicates a rollback to the immediately preceding revision. This is the fastest way to revert a problematic deployment.

**Rollback to a specific revision with a timeout and wait flag:**
```bash
helm rollback <release-name> <revision-number> -n <namespace> --wait --timeout 10m
```
Using `--wait` ensures that the rollback command does not exit until all resources are in a ready state. The `--timeout` flag is critical in production to prevent the command from hanging indefinitely if the rollback encounters issues.

**Force an upgrade, replacing resources if necessary:**
```bash
helm upgrade <release-name> <chart-path> -n <namespace> --force --wait --timeout 15m
```
The `--force` flag deletes and recreates resources if they cannot be updated in place. Use this with extreme caution, as it can cause downtime, but it is sometimes necessary when dealing with immutable fields or severe state mismatches.

### 2.2. Debugging and Dry Runs

Before applying changes to production, it is imperative to understand exactly what Helm will do. Dry runs and template rendering are your best tools for preventing deployment disasters.

**Render templates locally without installing:**
```bash
helm template <release-name> <chart-path> -n <namespace> --set key=value > rendered.yaml
```
This command outputs the raw Kubernetes manifests that Helm would apply. Inspecting `rendered.yaml` allows you to verify that all values are injected correctly and that the resulting manifests are valid.

**Perform a dry run of an install or upgrade:**
```bash
helm upgrade <release-name> <chart-path> -n <namespace> --dry-run --debug
```
The `--dry-run` flag simulates the deployment process against the live cluster without making any actual changes. The `--debug` flag provides verbose output, which is invaluable for identifying syntax errors or misconfigurations in the chart.

**Get the manifest of a deployed release:**
```bash
helm get manifest <release-name> -n <namespace>
```
This retrieves the exact manifests that are currently running in the cluster for a specific release. Comparing this output with the expected manifests can help identify configuration drift or unauthorized changes.

### 2.3. Dependency Management

Lerian applications often consist of multiple interconnected microservices, managed as Helm chart dependencies.

**Update chart dependencies:**
```bash
helm dependency update <chart-path>
```
This command resolves and downloads the dependencies specified in the `Chart.yaml` file. It is a necessary step before packaging or installing a chart with dependencies.

**List dependencies and their status:**
```bash
helm dependency list <chart-path>
```
This provides a quick overview of the required dependencies, their versions, and whether they are currently present in the `charts/` directory.

### 2.4. Advanced Helm One-Liners

For rapid operations during incidents, these one-liners combine multiple commands to achieve complex tasks efficiently.

**Uninstall all failed releases in a specific namespace:**
```bash
helm ls -n <namespace> --failed -q | xargs -I {} helm uninstall {} -n <namespace>
```
This command identifies all releases in a `failed` state and systematically uninstalls them. This is useful for cleaning up a namespace after a series of botched deployments.

**Extract the values used for a specific release and save to a file:**
```bash
helm get values <release-name> -n <namespace> -o yaml > current-values.yaml
```
When you need to replicate a production environment or understand the exact configuration of a running service, extracting the deployed values is the first step.

**Find all releases using a specific chart version:**
```bash
helm ls --all-namespaces -o json | jq '.[] | select(.chart | test("lerian-app-1.2.3")) | .name'
```
This utilizes `jq` to parse the JSON output of `helm ls` and filter for releases running a specific version of a chart. This is critical when auditing the cluster for vulnerable or outdated chart versions.

## 3. kubectl Mastery for Tech Support Operations

While Helm manages the deployment lifecycle, `kubectl` is the primary tool for interacting with the Kubernetes API, inspecting resources, and troubleshooting running applications.

### 3.1. Resource Troubleshooting and Inspection

When a Lerian application is failing, the first step is to inspect the underlying Kubernetes resources.

**Get all resources in a namespace, sorted by creation timestamp:**
```bash
kubectl get all -n <namespace> --sort-by=.metadata.creationTimestamp
```
This provides a chronological view of resource creation, which can help identify the sequence of events leading up to a failure.

**Describe a pod and extract the events section:**
```bash
kubectl describe pod <pod-name> -n <namespace> | grep -A 20 "Events:"
```
The events section of a pod description contains critical information about scheduling failures, image pull errors, and container crashes. Filtering for this section saves time during high-pressure incidents.

**Check the resource usage (CPU/Memory) of pods:**
```bash
kubectl top pods -n <namespace> --containers
```
This command requires the Metrics Server to be installed. It is essential for identifying memory leaks, CPU throttling, and resource starvation issues.

**Find pods that are not in a Running state:**
```bash
kubectl get pods -n <namespace> --field-selector=status.phase!=Running
```
This quickly isolates problematic pods, filtering out the noise of healthy instances.

### 3.2. Log Extraction and Analysis

Logs are the lifeblood of troubleshooting. Extracting and analyzing logs efficiently is a core competency for tech support.

**Tail logs for a specific container in a multi-container pod:**
```bash
kubectl logs -f <pod-name> -c <container-name> -n <namespace>
```
The `-f` flag streams the logs in real-time. Specifying the container name is necessary when dealing with sidecar patterns or complex pod architectures.

**Get logs from a previously crashed container:**
```bash
kubectl logs <pod-name> -c <container-name> -n <namespace> --previous
```
When a container crashes and restarts, the current logs will only show the new instance. The `--previous` flag retrieves the logs from the dead container, which usually contain the stack trace or error that caused the crash.

**Stream logs from all pods with a specific label:**
```bash
kubectl logs -f -l app=lerian-backend -n <namespace> --all-containers=true --max-log-requests=10
```
This is incredibly powerful for monitoring a distributed service. It aggregates logs from all replicas matching the label selector.

### 3.3. Network Troubleshooting

Network issues in Kubernetes can be notoriously difficult to diagnose. These commands help isolate connectivity problems.

**Run a temporary debug pod with network tools:**
```bash
kubectl run -i --tty --rm debug --image=nicolaka/netshoot --restart=Never -n <namespace> -- sh
```
The `netshoot` image contains a comprehensive suite of network troubleshooting tools (curl, ping, traceroute, tcpdump, etc.). Running this pod allows you to test connectivity from within the cluster network.

**Port-forward a service to your local machine:**
```bash
kubectl port-forward svc/<service-name> 8080:80 -n <namespace>
```
This securely tunnels traffic from your local machine to a service within the cluster, allowing you to interact with internal APIs or databases without exposing them externally.

**Check endpoints for a service:**
```bash
kubectl get endpoints <service-name> -n <namespace>
```
If a service is not routing traffic, checking its endpoints verifies whether the service has successfully discovered the underlying pods. Empty endpoints indicate a selector mismatch or failing readiness probes.

### 3.4. Advanced kubectl One-Liners

**Delete all evicted pods in a namespace:**
```bash
kubectl get pods -n <namespace> | grep Evicted | awk '{print $1}' | xargs kubectl delete pod -n <namespace>
```
Evicted pods can clutter the namespace and obscure real issues. This one-liner cleanly removes them.

**Force delete a stuck namespace:**
```bash
kubectl get namespace <namespace> -o json | tr -d "
" | sed "s/"finalizers": \[[^]]\+\]/"finalizers": []/" | kubectl replace --raw /api/v1/namespaces/<namespace>/finalize -f -
```
Namespaces can sometimes get stuck in a `Terminating` state due to lingering finalizers. This advanced command manually removes the finalizers via the Kubernetes API, forcing the deletion.

**Scale all deployments in a namespace to zero (Emergency Stop):**
```bash
kubectl get deployments -n <namespace> -o name | xargs -I {} kubectl scale {} --replicas=0 -n <namespace>
```
In the event of a catastrophic failure or security breach, this command instantly stops all application workloads in the specified namespace.

## 4. golang-migrate: Database Migrations in Production

Managing database schemas is one of the most critical and risky operations in software deployment. Lerian applications utilize `golang-migrate` for robust schema management.

### 4.1. Migration Execution and Rollbacks

Executing migrations must be done carefully, especially with large datasets where operations can take significant time.

**Apply all pending up migrations:**
```bash
migrate -path ./migrations -database "$DATABASE_URL" up
```
This command reads the migration files from the `./migrations` directory and applies them sequentially to the database specified by the connection string.

**Apply a specific number of up migrations:**
```bash
migrate -path ./migrations -database "$DATABASE_URL" up 2
```
Applying migrations in batches can be safer than applying all at once, allowing for verification between steps.

**Rollback the last applied migration:**
```bash
migrate -path ./migrations -database "$DATABASE_URL" down 1
```
If a migration introduces an issue, rolling back a single step is often the safest recovery path.

**Rollback all migrations (Use with extreme caution):**
```bash
migrate -path ./migrations -database "$DATABASE_URL" down -all
```
This command will drop all tables and data managed by the migrations. It should only be used in development or during a complete environment reset.

### 4.2. Handling Dirty States and Failures

A "dirty" state occurs when a migration fails midway, leaving the database schema in an inconsistent state. `golang-migrate` tracks this in the `schema_migrations` table.

**Force the database version to a specific state:**
```bash
migrate -path ./migrations -database "$DATABASE_URL" force <version-number>
```
When a migration fails, you must manually inspect the database, fix the issue (e.g., drop a partially created table), and then use the `force` command to tell `golang-migrate` that the database is now at a specific, clean version. This clears the dirty flag.

**Check the current migration version:**
```bash
migrate -path ./migrations -database "$DATABASE_URL" version
```
This outputs the current version and whether the database is in a dirty state.

### 4.3. Advanced golang-migrate One-Liners

**Create a new migration file pair (up and down):**
```bash
migrate create -ext sql -dir ./migrations -seq add_user_indexes
```
This generates two files (e.g., `000001_add_user_indexes.up.sql` and `000001_add_user_indexes.down.sql`) with sequential numbering, ensuring proper ordering.

**Run migrations with a custom lock timeout:**
```bash
migrate -path ./migrations -database "$DATABASE_URL&lock_timeout=10" up
```
In high-concurrency environments, acquiring the migration lock can fail. Appending `lock_timeout` to the connection string ensures the process waits before giving up.

## 5. Worst-Case Scenarios and Disaster Recovery

Tech support operations must be prepared for the worst. This section covers strategies for handling massive failures and performance bottlenecks.

### 5.1. Handling Massive Datasets and Timeouts

When dealing with massive datasets, standard operations often fail due to timeouts.

**Helm Timeouts:**
Always use the `--timeout` flag with Helm commands in production. For massive deployments, increase the timeout significantly (e.g., `--timeout 30m`). If a deployment consistently times out, investigate the readiness probes of the application; they may be too aggressive or the application may be taking too long to initialize its caches.

**kubectl Exec Timeouts:**
When running long-lived scripts inside a pod via `kubectl exec`, the connection may drop. Use `nohup` and redirect output to a file within the pod, then retrieve the file later.
```bash
kubectl exec -it <pod-name> -n <namespace> -- sh -c "nohup ./long-running-script.sh > output.log 2>&1 &"
```

**Database Migration Timeouts:**
Large index creations or table alterations can take hours. `golang-migrate` might time out waiting for the database to respond. In PostgreSQL, you can use `statement_timeout` in the connection string to control this. For truly massive migrations, consider using tools like `gh-ost` (for MySQL) or performing the migration out-of-band, rather than relying solely on the deployment pipeline.

### 5.2. Complete Environment Reset

In the rare event that an environment is completely corrupted, a full reset may be necessary.

1.  **Scale down all workloads:** Use the emergency stop one-liner provided in section 3.4.
2.  **Uninstall all Helm releases:** Use the uninstall one-liner from section 2.4, modifying it to remove all releases, not just failed ones.
3.  **Drop the database schema:** Use `migrate down -all` (section 4.1) or manually drop the database and recreate it.
4.  **Delete lingering PVCs:** Persistent Volume Claims may retain corrupted data.
    ```bash
    kubectl delete pvc --all -n <namespace>
    ```
5.  **Re-deploy:** Begin the deployment process from scratch using the CI/CD pipeline.

## 6. Integration with Other Specialist Modules

This CLI reference is a core component of the Lerian Studio tech support documentation suite. It interacts closely with the other specialist files:

*   **File 1 (Architecture Overview):** Provides the context for *why* these commands are necessary. Understanding the microservices architecture is prerequisite to knowing which pods to inspect or which Helm charts to rollback.
*   **File 2 (Incident Response Playbook):** The playbook dictates the *process* of handling an incident, while this CLI reference provides the *tools*. When the playbook says "Investigate database connectivity," the engineer refers to section 3.3 of this document.
*   **File 4 (Monitoring and Alerting Guide):** Alerts trigger the need for troubleshooting. The metrics and logs discussed in section 3 are the direct result of the configurations detailed in the monitoring guide.
*   **File 5 (Security and Access Control):** Security policies dictate *who* can run these commands. Advanced `kubectl` operations often require elevated RBAC permissions, which are defined in the security module.
*   **File 6 (Post-Mortem Templates):** After an incident is resolved using the commands in this reference, the actions taken must be documented in the post-mortem. The specific one-liners used to mitigate the issue should be recorded for future reference.
*   **File 7 (Runbooks for Common Services):** Runbooks provide service-specific instructions, often referencing the generic commands found in this document. For example, a runbook for the "Payment Service" might specify the exact `kubectl logs` command needed to trace a transaction.

By mastering the commands and strategies outlined in this comprehensive reference, Lerian tech support engineers can confidently navigate complex production environments, minimize downtime, and ensure the reliability of the platform.
