# OpenTelemetry Collector Troubleshooting Guide

## Introduction

The OpenTelemetry Collector is a vendor-agnostic telemetry pipeline designed to receive, process, and export traces, metrics, logs, and profiles. While its architecture—comprising receivers, processors, exporters, connectors, and extensions—provides immense flexibility, it also introduces complexity. When operating the Collector at scale, technical support and operations teams frequently encounter issues related to data flow, resource management, configuration, and network connectivity.

This comprehensive guide is designed for operations teams managing OpenTelemetry Collector deployments in production environments. It covers the full spectrum of troubleshooting scenarios, from data dropping and memory exhaustion to Kubernetes-specific deployment challenges. By leveraging the built-in diagnostic tools and understanding the internal mechanics of the Collector, operators can effectively diagnose and resolve issues to ensure reliable telemetry delivery.

## Diagnostic Tools and Internal Telemetry

Before diving into specific failure scenarios, it is crucial to understand the diagnostic tools available within the OpenTelemetry Collector ecosystem. These tools provide visibility into the internal state of the Collector and are essential for identifying the root cause of issues.

### Internal Metrics

The Collector exposes internal metrics that provide insights into its performance and health. These metrics are typically scraped by Prometheus or another metrics backend. Key metrics to monitor include:

*   `otelcol_receiver_accepted_spans` / `metrics` / `log_records`: The number of telemetry items successfully received.
*   `otelcol_receiver_refused_spans` / `metrics` / `log_records`: The number of telemetry items rejected by the receiver (e.g., due to invalid format or rate limiting).
*   `otelcol_processor_accepted_spans` / `metrics` / `log_records`: The number of telemetry items successfully processed.
*   `otelcol_processor_refused_spans` / `metrics` / `log_records`: The number of telemetry items dropped during processing (e.g., by the `memory_limiter` or `filter` processor).
*   `otelcol_processor_dropped_spans` / `metrics` / `log_records`: The number of telemetry items dropped by processors.
*   `otelcol_exporter_enqueue_failed_spans` / `metrics` / `log_records`: The number of telemetry items that failed to enter the sending queue.
*   `otelcol_exporter_queue_size`: The current number of batches in the sending queue.
*   `otelcol_exporter_queue_capacity`: The maximum capacity of the sending queue.
*   `otelcol_exporter_send_failed_spans` / `metrics` / `log_records`: The number of telemetry items that failed to be exported to the destination.
*   `otelcol_exporter_sent_spans` / `metrics` / `log_records`: The number of telemetry items successfully exported.

### The Debug Exporter

The `debug` exporter is an invaluable tool for verifying that data is flowing through the pipeline correctly. It prints the telemetry data to the Collector's standard output or standard error.

To use the debug exporter effectively, configure it with `verbosity: detailed` to see the full payload of the telemetry data.

```yaml
exporters:
  debug:
    verbosity: detailed

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [debug, otlp]
```

**Warning:** Do not leave the debug exporter enabled with `verbosity: detailed` in high-volume production environments, as it can cause significant CPU and I/O overhead.

### zPages Extension

The `zpages` extension provides an in-process web server that serves diagnostic pages. These pages offer real-time insights into the Collector's internal state, including RPC stats, trace data, and component configurations.

To enable zPages, add it to the `extensions` section and the `service.extensions` list:

```yaml
extensions:
  zpages:
    endpoint: 0.0.0.0:55679

service:
  extensions: [zpages]
```

Key endpoints:
*   `/debug/tracez`: Provides insights into active, completed, and errored spans. This is particularly useful for identifying latency bottlenecks or deadlocks within the pipeline.
*   `/debug/rpcz`: Displays gRPC statistics for receivers and exporters.
*   `/debug/pipelinez`: Shows the configuration and status of the active pipelines.

### pprof Extension

The `pprof` extension exposes Go's standard profiling endpoints, allowing operators to analyze CPU usage, memory allocation, and goroutine blocking. This is essential for diagnosing performance issues and memory leaks.

```yaml
extensions:
  pprof:
    endpoint: 0.0.0.0:1777

service:
  extensions: [pprof]
```

You can then use the `go tool pprof` command to analyze the profiles:

```bash
# Analyze CPU profile
go tool pprof http://YOUR_COLLECTOR_IP:1777/debug/pprof/profile

# Analyze heap profile
go tool pprof http://YOUR_COLLECTOR_IP:1777/debug/pprof/heap
```

### The `components` Command

The OpenTelemetry Collector binary includes a `components` command that lists all available receivers, processors, exporters, extensions, and connectors compiled into the distribution, along with their stability levels. This is useful for verifying that a specific component is available in your custom build or distribution.

```bash
./otelcol components
```

## Data Flow Issues

Data flow issues are the most common problems encountered when operating the Collector. These issues typically manifest as data being dropped, not received, not processed, or not exported.

### 1. Not Receiving Data

When the Collector is not receiving data, the issue usually lies outside the Collector itself or in the receiver configuration.

**Symptoms:**
*   No data appears in the backend.
*   The `otelcol_receiver_accepted_*` metrics are zero.
*   Client applications report connection refused or timeout errors.

**Troubleshooting Steps:**

1.  **Verify Network Connectivity:** Ensure that the client application can reach the Collector's receiver port. Use tools like `curl`, `telnet`, or `nc` to test connectivity.
    ```bash
    # Test OTLP gRPC port
    nc -zv YOUR_COLLECTOR_IP 4317

    # Test OTLP HTTP port
    curl -v http://YOUR_COLLECTOR_IP:4318/v1/traces
    ```
2.  **Check Receiver Configuration:** Verify that the receiver is configured correctly and listening on the expected interface and port. Ensure that the receiver is included in the `service.pipelines` section.
    ```yaml
    receivers:
      otlp:
        protocols:
          grpc:
            endpoint: 0.0.0.0:4317
          http:
            endpoint: 0.0.0.0:4318
    ```
3.  **Inspect Client Configuration:** Verify that the client application is configured to send data to the correct endpoint and using the correct protocol (e.g., OTLP/gRPC vs. OTLP/HTTP).
4.  **Check Firewall Rules and Network Policies:** Ensure that there are no firewalls, security groups, or Kubernetes NetworkPolicies blocking traffic to the receiver ports.
5.  **Review Collector Logs:** Look for errors during startup related to binding to the specified ports.
    ```text
    Error: cannot start receiver "otlp": listen tcp 0.0.0.0:4317: bind: address already in use
    ```

### 2. Not Processing Data

If data is received but not processed as expected, the issue is likely within the processor configuration or the understanding of how processors operate.

**Symptoms:**
*   Data reaches the backend, but transformations (e.g., attribute redaction, metric renaming) are not applied.
*   The `otelcol_processor_dropped_*` metrics are increasing.

**Troubleshooting Steps:**

1.  **Verify Processor Inclusion:** Ensure that the processor is included in the `service.pipelines` section and in the correct order. Processors are executed sequentially.
2.  **Understand Processor Scope:** Processors operate on individual telemetry items (spans, metrics, log records). For example, the `attributes` processor modifies attributes on a span, not the span name itself. To modify the span name, use the `transform` processor with OTTL.
3.  **Check OTTL Syntax:** If using the `transform` or `filter` processors, verify the OpenTelemetry Transformation Language (OTTL) syntax. Enable debug logging for the telemetry service to see OTTL execution details.
    ```yaml
    service:
      telemetry:
        logs:
          level: debug
    ```
4.  **Use the Debug Exporter:** Place the `debug` exporter before and after the processor in question to inspect the telemetry data and verify that the transformation is applied correctly.

### 3. Not Exporting Data

When data is processed but not exported, the issue is typically related to network connectivity to the backend, authentication, or exporter configuration.

**Symptoms:**
*   Data is received and processed, but does not appear in the backend.
*   The `otelcol_exporter_send_failed_*` metrics are increasing.
*   Collector logs show connection errors or authentication failures.

**Troubleshooting Steps:**

1.  **Verify Network Connectivity to Backend:** Ensure that the Collector can reach the backend endpoint. Check for DNS resolution issues, firewall rules, or egress proxies.
2.  **Check Authentication Credentials:** Verify that the API keys, tokens, or certificates used for authentication are correct and have not expired. Ensure they are correctly referenced in the configuration (e.g., using environment variables).
    ```yaml
    exporters:
      otlp:
        endpoint: YOUR_BACKEND_ENDPOINT:443
        headers:
          "x-api-key": "${env:YOUR_API_KEY}"
    ```
3.  **Verify TLS Configuration:** If the backend requires TLS (which is the default for the `otlp` gRPC exporter), ensure that the certificates are valid and trusted. If connecting to an insecure endpoint, explicitly set `insecure: true`.
    ```yaml
    exporters:
      otlp:
        endpoint: YOUR_INSECURE_ENDPOINT:4317
        tls:
          insecure: true
    ```
4.  **Check Exporter Inclusion:** Ensure that the exporter is included in the `service.pipelines` section.
5.  **Review Backend Limits:** Check if the backend is rate-limiting or rejecting the data due to payload size or invalid format.

### 4. Dropping Data

Data dropping occurs when the Collector is overwhelmed and cannot process or export data fast enough. This is a critical issue that requires careful tuning of the pipeline.

**Symptoms:**
*   The `otelcol_processor_refused_*` or `otelcol_exporter_enqueue_failed_*` metrics are increasing.
*   The `otelcol_exporter_queue_size` metric is consistently at or near `otelcol_exporter_queue_capacity`.
*   Collector logs show warnings about dropped data or full queues.

**Troubleshooting Steps:**

1.  **Analyze Queue Metrics:** The sending queue is the primary buffer between processors and exporters. If `otelcol_exporter_queue_size` is consistently high, the exporter cannot send data fast enough.
2.  **Tune Batch Processor:** Ensure the `batch` processor is configured correctly. It groups telemetry items into batches, reducing the number of outgoing requests and improving compression.
    ```yaml
    processors:
      batch:
        send_batch_size: 8192
        timeout: 1s
        send_batch_max_size: 10000
    ```
3.  **Configure Retry and Queue Settings:** Most exporters support `retry_on_failure` and `sending_queue` configurations. Increase the queue size to handle temporary network blips or backend slowdowns, but be aware that a larger queue consumes more memory.
    ```yaml
    exporters:
      otlp:
        endpoint: YOUR_BACKEND_ENDPOINT:443
        retry_on_failure:
          enabled: true
          initial_interval: 5s
          max_interval: 30s
          max_elapsed_time: 300s
        sending_queue:
          enabled: true
          num_consumers: 10
          queue_size: 10000
    ```
4.  **Scale the Collector:** If the backend can handle the load but the Collector is CPU-bound, scale horizontally by adding more Collector replicas. See the "Scaling Issues" section for more details.
5.  **Investigate Backend Performance:** If the Collector is dropping data because the backend is slow or unresponsive, scaling the Collector will not solve the problem. You must address the performance issues on the backend.

## Memory Issues

Memory management is a critical aspect of operating the OpenTelemetry Collector. Improper configuration can lead to Out-Of-Memory (OOM) kills, resulting in data loss and service disruption.

### 1. Out-Of-Memory (OOM) Kills

OOM kills occur when the Collector consumes more memory than the operating system or container runtime allows.

**Symptoms:**
*   The Collector process exits unexpectedly.
*   Container orchestration systems (e.g., Kubernetes) report `OOMKilled` events.
*   System logs (e.g., `dmesg`) show the OOM killer terminating the `otelcol` process.

**Troubleshooting Steps:**

1.  **Configure the `memory_limiter` Processor:** The `memory_limiter` processor is essential for preventing OOM kills. It monitors memory usage and forces garbage collection or drops data when memory limits are approached. **It MUST be the first processor in the pipeline.**
    ```yaml
    processors:
      memory_limiter:
        check_interval: 1s
        limit_mib: 4000
        spike_limit_mib: 800
    ```
    *   `limit_mib`: The hard limit. Set this to about 80-90% of the container's memory limit.
    *   `spike_limit_mib`: The buffer for memory spikes. Set this to about 20% of the `limit_mib`.
    *   In containerized environments, use `limit_percentage` and `spike_limit_percentage` instead of absolute values.
2.  **Tune GOMEMLIMIT:** The Go runtime's garbage collector can be tuned using the `GOMEMLIMIT` environment variable. Set `GOMEMLIMIT` to approximately 80% of the container's hard memory limit. This encourages the Go GC to run more aggressively before the OS OOM killer intervenes.
    ```yaml
    env:
      - name: GOMEMLIMIT
        value: "3200MiB" # Assuming a 4000MiB container limit
    ```
3.  **Analyze Memory Profiles:** If OOM kills persist despite proper configuration, use the `pprof` extension to capture a heap profile and identify memory leaks or inefficient memory usage by specific components.
4.  **Review Queue Sizes:** Large `sending_queue` sizes in exporters consume significant memory. Reduce the queue size if memory is constrained.

## Performance Issues

Performance issues typically manifest as high CPU usage, slow exports, or queue saturation.

### 1. High CPU Usage

High CPU usage can be caused by inefficient processing, high telemetry volume, or excessive garbage collection.

**Symptoms:**
*   The Collector consumes a large amount of CPU resources.
*   Telemetry processing latency increases.

**Troubleshooting Steps:**

1.  **Analyze CPU Profiles:** Use the `pprof` extension to capture a CPU profile and identify the functions consuming the most CPU time.
2.  **Optimize OTTL Statements:** Complex OTTL statements in `transform` or `filter` processors can be CPU-intensive. Optimize the statements or use more specific processors (e.g., `attributes` processor) if possible.
3.  **Disable Debug Logging:** Ensure that the `debug` exporter with `verbosity: detailed` is disabled and that the telemetry log level is not set to `debug`.
4.  **Scale Horizontally:** If the CPU usage is justified by the telemetry volume, scale the Collector horizontally by adding more replicas.

### 2. Slow Exports and Queue Saturation

Slow exports occur when the Collector cannot send data to the backend fast enough, leading to queue saturation and eventually data dropping.

**Symptoms:**
*   The `otelcol_exporter_queue_size` metric is consistently high.
*   The `otelcol_exporter_send_failed_*` metrics are increasing due to timeouts.

**Troubleshooting Steps:**

1.  **Increase `num_consumers`:** In the exporter's `sending_queue` configuration, increase the `num_consumers` to send more concurrent requests to the backend.
    ```yaml
    exporters:
      otlp:
        sending_queue:
          num_consumers: 20 # Increase from the default of 10
    ```
2.  **Tune Batch Size:** Adjust the `send_batch_size` in the `batch` processor. A larger batch size reduces the number of requests but increases the payload size. Find the optimal balance for your backend.
3.  **Investigate Network Latency:** High network latency between the Collector and the backend can slow down exports. Use tools like `ping` or `traceroute` to measure latency.
4.  **Check Backend Performance:** Ensure that the backend is not rate-limiting or struggling to ingest the data.

## Configuration Issues

Configuration errors are a common source of frustration. The Collector's YAML configuration is strict, and minor errors can prevent the Collector from starting or functioning correctly.

### 1. Invalid YAML and Null Maps

The Collector uses a strict YAML parser. A common gotcha is the "null map" error, which occurs when a configuration block is declared but left empty.

**Symptoms:**
*   The Collector fails to start with an error message like:
    ```text
    Error: cannot unmarshal the configuration: error reading config: yaml: unmarshal errors:
      line 10: cannot unmarshal !!null into map[string]interface {}
    ```

**Troubleshooting Steps:**

1.  **Check for Empty Blocks:** Ensure that no configuration blocks are left empty. If a component requires no configuration, either omit it entirely or provide an empty map `{}`.
    ```yaml
    # Incorrect (Null Map)
    processors:
      batch:

    # Correct
    processors:
      batch: {}
    ```
2.  **Validate YAML Syntax:** Use a YAML linter to verify the syntax of your configuration file.
3.  **Verify Component IDs:** Ensure that component IDs in the `service.pipelines` section match the IDs defined in the `receivers`, `processors`, and `exporters` sections.

### 2. Environment Variable Substitution

The Collector supports environment variable substitution in the configuration file using the `${env:VAR_NAME}` syntax.

**Symptoms:**
*   Authentication fails because API keys are not populated.
*   The Collector fails to start with errors about missing configuration values.

**Troubleshooting Steps:**

1.  **Verify Environment Variables:** Ensure that the environment variables are actually set in the environment where the Collector is running.
2.  **Check Syntax:** Verify that the substitution syntax is correct (`${env:VAR_NAME}`).
3.  **Use `configopaque.String`:** For sensitive fields like API keys or tokens, the Collector uses `configopaque.String` internally to prevent them from being logged. Ensure that these fields are populated via environment variables rather than hardcoded in the configuration file.

## Kubernetes-Specific Issues

Deploying the OpenTelemetry Collector in Kubernetes introduces additional complexity related to RBAC, service discovery, and the OpenTelemetry Operator.

### 1. RBAC and Service Discovery Failures

Components like the `k8sattributes` processor and the `k8scluster` receiver require access to the Kubernetes API to discover resources and enrich telemetry data.

**Symptoms:**
*   The `k8sattributes` processor fails to add pod or node metadata.
*   The `k8scluster` receiver fails to collect cluster-level metrics.
*   Collector logs show RBAC errors (e.g., `User "system:serviceaccount:default:otelcol" cannot list resource "pods" in API group "" at the cluster scope`).

**Troubleshooting Steps:**

1.  **Verify ServiceAccount:** Ensure that the Collector pod is running with a ServiceAccount that has the necessary permissions.
2.  **Check ClusterRole and ClusterRoleBinding:** Verify that a ClusterRole with the required permissions (e.g., `get`, `list`, `watch` on `pods`, `nodes`, `namespaces`) is bound to the Collector's ServiceAccount.
    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: otelcol-role
    rules:
    - apiGroups: [""]
      resources: ["pods", "namespaces", "nodes"]
      verbs: ["get", "watch", "list"]
    ```

### 2. Sidecar Injection Failures

The OpenTelemetry Operator can automatically inject the Collector as a sidecar container into application pods based on annotations.

**Symptoms:**
*   The application pod starts, but the `otc-container` sidecar is not injected.
*   The application fails to send telemetry data to the local sidecar.

**Troubleshooting Steps:**

1.  **Verify Annotation:** Ensure that the application pod or namespace has the correct annotation: `sidecar.opentelemetry.io/inject: "true"` (or the name of a specific OpenTelemetryCollector CR).
2.  **Check Operator Logs:** Inspect the logs of the OpenTelemetry Operator pod for errors related to the mutating webhook.
3.  **Verify Webhook Configuration:** Ensure that the `MutatingWebhookConfiguration` is correctly configured and that the Kubernetes API server can reach the Operator's webhook service on port 9443. Network policies or firewall rules may block this traffic.
4.  **Check OpenTelemetryCollector CR:** Ensure that a valid `OpenTelemetryCollector` Custom Resource exists in the namespace (or cluster-wide, depending on the annotation) and is in a ready state.

### 3. Target Allocator Issues

The Target Allocator is an optional component of the OpenTelemetry Operator that shards Prometheus scrape targets across a StatefulSet of Collectors.

**Symptoms:**
*   Prometheus metrics are missing or duplicated.
*   The Target Allocator pod is crashing or logging errors.

**Troubleshooting Steps:**

1.  **Verify ServiceMonitor/PodMonitor Discovery:** Ensure that the Target Allocator is correctly discovering the `ServiceMonitor` or `PodMonitor` resources. Check the Target Allocator logs for discovery errors.
2.  **Check RBAC:** The Target Allocator requires extensive RBAC permissions to discover targets and monitor the Collector StatefulSet. Verify the ClusterRole bindings.
3.  **Inspect Target Allocator UI:** The Target Allocator provides a simple UI (typically on port 8080) that displays the discovered targets and their assignment to Collector replicas. Use this to verify the sharding logic.

### 4. Ephemeral Debug Containers

When troubleshooting Collector pods in Kubernetes, ephemeral debug containers are a powerful tool. They allow you to attach a container with diagnostic tools (e.g., `curl`, `netcat`, `tcpdump`) to a running Collector pod without modifying its configuration.

```bash
kubectl debug -it YOUR_COLLECTOR_POD_NAME --image=busybox:1.28 --target=otc-container
```

Once attached, you can verify network connectivity, inspect the local filesystem, or capture network traffic.

## Scaling Issues

Scaling the OpenTelemetry Collector requires careful consideration of the deployment architecture and the statefulness of the telemetry data.

### 1. When to Scale

Scale the Collector horizontally when:
*   CPU usage is consistently high.
*   The `otelcol_exporter_queue_size` is consistently reaching 60-70% of its capacity, indicating that the Collector cannot process or export data fast enough.
*   You need high availability and fault tolerance.

### 2. When NOT to Scale

Do NOT scale the Collector horizontally when:
*   The backend destination is the bottleneck. If the backend cannot ingest data fast enough, adding more Collectors will only exacerbate the problem and lead to more dropped data.
*   You are experiencing OOM kills due to memory leaks or improper `memory_limiter` configuration. Fix the memory issues first.

### 3. Scaling Stateful Pipelines

Certain Collector components require a consistent view of the telemetry data. For example:
*   **Tail Sampling:** The `tailsampling` processor needs to see all spans for a given trace to make a sampling decision.
*   **Span-to-Metrics:** The `spanmetrics` connector needs to see all spans to generate accurate metrics.

If you scale these components horizontally behind a standard round-robin load balancer, spans for the same trace may be sent to different Collector replicas, resulting in broken traces or inaccurate metrics.

**Solution:** Use a two-tier architecture with the `loadbalancing` exporter.

1.  **Tier 1 (Stateless):** A set of Collectors that receive data from applications and use the `loadbalancing` exporter to route data to Tier 2 based on a consistent hash of the trace ID.
2.  **Tier 2 (Stateful):** A set of Collectors that perform the stateful processing (e.g., tail sampling) and export the data to the backend.

```yaml
# Tier 1 Configuration
exporters:
  loadbalancing:
    protocol:
      otlp:
        tls:
          insecure: true
    resolver:
      static:
        hostnames:
          - tier2-collector-0.tier2-service:4317
          - tier2-collector-1.tier2-service:4317
    routing_key: traceID
```

## Common Error Messages and Solutions

| Error Message | Potential Cause | Solution |
| :--- | :--- | :--- |
| `bind: address already in use` | Another process is listening on the required port. | Identify the conflicting process or change the port in the receiver configuration. |
| `cannot unmarshal !!null into map[string]interface {}` | A configuration block is declared but left empty. | Provide an empty map `{}` or remove the block. |
| `context deadline exceeded` | The exporter timed out while sending data to the backend. | Check network connectivity, backend performance, and increase the exporter timeout. |
| `rpc error: code = Unauthenticated desc = invalid API key` | The authentication credentials are incorrect or missing. | Verify the API key or token in the exporter configuration. |
| `OOMKilled` | The Collector exceeded its memory limit. | Configure the `memory_limiter` processor and tune `GOMEMLIMIT`. |
| `User "..." cannot list resource "..."` | The Collector lacks the necessary RBAC permissions in Kubernetes. | Update the ClusterRole bound to the Collector's ServiceAccount. |
| `dropping data because sending_queue is full` | The exporter cannot send data fast enough, and the queue is saturated. | Increase `num_consumers`, tune batch size, or scale the Collector. |

## Conclusion

Troubleshooting the OpenTelemetry Collector requires a systematic approach, leveraging internal metrics, diagnostic extensions, and a deep understanding of the pipeline architecture. By carefully configuring memory limits, tuning batch and queue settings, and utilizing the appropriate deployment patterns, operations teams can build robust and scalable telemetry pipelines that reliably deliver critical observability data.
