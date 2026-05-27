# OpenTelemetry Collector Deep Dive: Architecture, Operations, and Troubleshooting

## 1. Introduction to OpenTelemetry Collector

The OpenTelemetry Collector is a vendor-agnostic proxy that can receive, process, and export telemetry data. It supports receiving telemetry data in multiple formats (for example, OTLP, Jaeger, Prometheus, as well as many commercial/proprietary formats) and sending data to one or more backends. It also supports processing and filtering telemetry data before it is exported.

This document provides a comprehensive deep dive into the OpenTelemetry Collector, focusing on architecture internals, configuration, scaling, deployment patterns, and extensive troubleshooting guidelines for technical support and operations teams.

## 2. Architecture Internals

The OpenTelemetry Collector is written in Go and is designed to be highly extensible and performant. It operates as a pipeline for telemetry data, handling traces, metrics, logs, and profiles.

### 2.1 Service Startup Sequence and Pipeline Graph Construction

When the OpenTelemetry Collector starts, it goes through a specific sequence to initialize its components and construct the telemetry pipeline.

1.  **Configuration Resolution:** The Collector reads its configuration from the specified sources (file, environment variables, HTTP, etc.). The configuration resolution pipeline involves providers, converters, and resolvers.
2.  **Component Initialization:** Based on the resolved configuration, the Collector initializes the configured extensions, receivers, processors, exporters, and connectors.
3.  **Pipeline Graph Construction:** The Collector constructs a Directed Acyclic Graph (DAG) representing the telemetry pipelines. Nodes in the graph are the components (receivers, processors, exporters), and edges represent the flow of data.
4.  **Component Lifecycle (Start):** The Collector starts the components in a specific order to ensure dependencies are met. Extensions are started first, followed by exporters, processors, and finally receivers. This ensures that the downstream components are ready to receive data before the upstream components start sending it.

### 2.2 Confmap Resolution Pipeline

The configuration resolution process in the OpenTelemetry Collector is highly flexible and extensible, handled by the `confmap` package. It consists of three main stages:

1.  **Providers:** Providers are responsible for retrieving configuration data from various sources. Built-in providers include `file`, `env`, `yaml`, `http`, and `https`. Providers are identified by a scheme (e.g., `file:`, `env:`).
2.  **Converters:** Converters transform the configuration data retrieved by providers. They can be used to migrate older configuration formats to newer ones or apply specific transformations.
3.  **Resolver:** The resolver takes the configuration data from providers, applies converters, and merges multiple configuration sources into a single, unified configuration map. It also handles variable substitution (e.g., `${env:MY_VAR}`).

### 2.3 Data Flow Through Pipeline (pdata types)

Telemetry data flows through the Collector's pipelines using internal data structures defined in the `pdata` package. These structures provide a unified representation of telemetry data, regardless of the original format.

*   **`pcommon`:** Contains common data structures used across different signal types, such as attributes, resources, and instrumentation scopes.
*   **`ptrace`:** Represents trace data (spans, span events, span links).
*   **`pmetric`:** Represents metric data (data points, metric types like gauge, sum, histogram).
*   **`plog`:** Represents log data (log records).

When a receiver ingests data, it translates it into the appropriate `pdata` format. Processors manipulate the `pdata` structures, and exporters translate the `pdata` back into the required format for the destination backend.

### 2.4 Fanout and Backpressure Propagation

The OpenTelemetry Collector supports fanout, where a single receiver can send data to multiple pipelines, or a single pipeline can send data to multiple exporters.

Backpressure is a critical mechanism for preventing the Collector from being overwhelmed by incoming data. When an exporter or processor cannot keep up with the incoming data rate, it signals backpressure upstream. This backpressure propagates back to the receivers, which can then slow down or reject incoming requests.

### 2.5 Memory Management

Proper memory management is crucial for the stability and performance of the OpenTelemetry Collector.

*   **`GOMEMLIMIT`:** It is highly recommended to set the `GOMEMLIMIT` environment variable to approximately 80% of the container's hard memory limit. This helps the Go garbage collector manage memory more aggressively and prevents Out-Of-Memory (OOM) kills.
*   **`memory_limiter` Processor:** The `memory_limiter` processor should be the first processor in every pipeline. It monitors memory usage and drops data or forces garbage collection when memory limits are approached.
    *   `limit_mib` or `limit_percentage`: The hard limit at which data will be dropped.
    *   `spike_limit_mib` or `spike_limit_percentage`: A buffer to handle sudden spikes in memory usage. Recommended to be 20% of the hard limit.
    *   `check_interval`: How often to check memory usage (e.g., `1s`).
*   **Ballast (Deprecated):** The use of memory ballast extensions is deprecated in favor of `GOMEMLIMIT`.

### 2.6 Internal Telemetry

The OpenTelemetry Collector exposes internal telemetry (metrics) that are essential for monitoring its health and performance. Key metrics include:

*   **Receivers:**
    *   `otelcol_receiver_accepted_spans` / `_metric_points` / `_log_records`: Number of items successfully accepted.
    *   `otelcol_receiver_refused_spans` / `_metric_points` / `_log_records`: Number of items rejected (e.g., due to backpressure or invalid format).
*   **Processors:**
    *   `otelcol_processor_accepted_spans` / `_metric_points` / `_log_records`: Number of items successfully processed.
    *   `otelcol_processor_refused_spans` / `_metric_points` / `_log_records`: Number of items dropped during processing.
    *   `otelcol_processor_dropped_spans` / `_metric_points` / `_log_records`: Number of items dropped (e.g., by a filter processor).
*   **Exporters:**
    *   `otelcol_exporter_queue_size`: Current number of batches in the sending queue.
    *   `otelcol_exporter_queue_capacity`: Maximum capacity of the sending queue.
    *   `otelcol_exporter_enqueue_failed_spans` / `_metric_points` / `_log_records`: Number of items that failed to enter the queue (queue full).
    *   `otelcol_exporter_send_failed_spans` / `_metric_points` / `_log_records`: Number of items that failed to be sent to the destination.

### 2.7 Persistent Queue (filestorage extension)

To prevent data loss during Collector restarts or transient backend outages, the Collector supports persistent queuing using the `filestorage` extension. This extension allows exporters to buffer data to disk instead of memory.

### 2.8 Retry Mechanisms

Exporters typically implement retry mechanisms to handle transient errors when sending data to backends. This is configured using the `retry_on_failure` setting, which uses exponential backoff.

*   `enabled`: Whether retries are enabled (default: true).
*   `initial_interval`: The initial wait time between retries.
*   `max_interval`: The maximum wait time between retries.
*   `max_elapsed_time`: The maximum total time spent retrying before dropping the data.

### 2.9 Batch Processor Internals

The `batch` processor groups telemetry data into batches before sending it to the next component in the pipeline. This significantly improves export efficiency and reduces network overhead.

Batching is triggered by two main conditions:

1.  **Size Trigger:** When the batch reaches a specific size (`send_batch_size`).
2.  **Timeout Trigger:** When a specific time interval has elapsed since the batch was created (`timeout`).

The processor also supports a `send_batch_max_size` to ensure batches do not exceed a hard limit, which is important for backends with payload size restrictions.

### 2.10 Connector Internals

Connectors are special components that act as both an exporter and a receiver, bridging two pipelines. They receive data from one pipeline (acting as an exporter) and send it to another pipeline (acting as a receiver).

Connectors are useful for tasks like routing data based on specific criteria, generating metrics from spans (spanmetrics connector), or counting telemetry items. They handle signal type routing, ensuring data is passed to the correct pipeline type.

### 2.11 Extension Capabilities

Extensions provide auxiliary capabilities to the Collector that are not directly involved in processing telemetry data.

*   **`health_check`:** Provides an HTTP endpoint for health monitoring (e.g., for Kubernetes liveness/readiness probes).
*   **`pprof`:** Exposes Go pprof endpoints for performance profiling and debugging.
*   **`zpages`:** Provides web pages with internal debugging information (e.g., tracez for latency and error analysis).
*   **Authentication:** Extensions like `basicauth`, `bearertokenauth`, and `oidcauth` provide authentication mechanisms for receivers and exporters.

### 2.12 Operator Internals

The OpenTelemetry Operator is a Kubernetes operator that manages the lifecycle of OpenTelemetry Collectors and auto-instruments workloads.

*   **Webhook:** The Operator uses a mutating admission webhook (typically on port 9443) to intercept pod creation requests and inject the OpenTelemetry Collector sidecar or auto-instrumentation libraries based on annotations (e.g., `sidecar.opentelemetry.io/inject: "true"`).
*   **Reconciliation Loop:** The Operator continuously monitors `OpenTelemetryCollector` Custom Resources (CRs) and ensures the actual state of the cluster matches the desired state defined in the CRs.
*   **Target Allocator:** The Target Allocator is a component of the Operator that discovers Prometheus scrape targets and distributes them across a StatefulSet of OpenTelemetry Collectors. It supports allocation strategies like `least-weighted` and `consistent-hashing` to ensure even load distribution.

## 3. Configuration Examples

### 3.1 Basic OTLP Pipeline with Memory Limiter and Batching

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  memory_limiter:
    check_interval: 1s
    limit_percentage: 80
    spike_limit_percentage: 20
  batch:
    send_batch_size: 8192
    timeout: 1s

exporters:
  otlp:
    endpoint: YOUR_ENDPOINT:4317
    tls:
      insecure: false
    headers:
      "api-key": "YOUR_API_KEY"

extensions:
  health_check:
  pprof:
    endpoint: 0.0.0.0:1777
  zpages:
    endpoint: 0.0.0.0:55679

service:
  extensions: [health_check, pprof, zpages]
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [otlp]
    metrics:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [otlp]
    logs:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [otlp]
```

### 3.2 Persistent Queue Configuration

```yaml
extensions:
  filestorage:
    directory: /var/lib/otelcol/file_storage
    timeout: 1s
    compaction:
      on_start: true
      directory: /var/lib/otelcol/file_storage_compaction
      max_transaction_size: 65536

exporters:
  otlp:
    endpoint: YOUR_ENDPOINT:4317
    sending_queue:
      enabled: true
      num_consumers: 10
      queue_size: 10000
      storage: filestorage
    retry_on_failure:
      enabled: true
      initial_interval: 5s
      max_interval: 30s
      max_elapsed_time: 300s
```

### 3.3 OTTL Transformation Example

```yaml
processors:
  transform:
    error_mode: ignore
    trace_statements:
      - context: span
        statements:
          - set(status.code, 1) where attributes["http.status_code"] >= 400
          - replace_match(name, "GET /api/v1/users/.*", "GET /api/v1/users/{id}")
```

## 4. Scaling Patterns

Scaling the OpenTelemetry Collector depends on the deployment mode and the type of workload.

1.  **Stateless (Receiving Push Data):** This is the most common pattern. Deploy the Collector as a Deployment and scale horizontally by adding replicas. Use an L7 load balancer (for gRPC) to distribute traffic evenly across the replicas.
2.  **Scrapers (Pulling Data):** When scraping data (e.g., Prometheus metrics), use the Target Allocator to shard endpoints across a StatefulSet of Collectors.
3.  **Stateful (Tail Sampling, Span-to-Metrics):** Components like tail sampling require all spans for a specific trace to be processed by the same Collector instance. Use a two-tier architecture: a stateless tier of Collectors receives data and uses the `loadbalancing` exporter with consistent hashing (based on trace ID) to route data to a stateful tier of Collectors that perform the tail sampling.

**When to Scale:**
Monitor the `otelcol_exporter_queue_size` and `otelcol_exporter_queue_capacity` metrics. Scale up when the queue consistently reaches 60-70% capacity. However, **do not scale** if the backend destination is the bottleneck (indicated by high `otelcol_exporter_send_failed_spans` and backend performance metrics).

## 5. Troubleshooting Guide

### 5.1 General Troubleshooting Steps

1.  **Check Collector Logs:** Look for error messages, warnings, and startup failures.
2.  **Enable Debug Logging:** Set the telemetry logs level to debug in the configuration:
    ```yaml
    service:
      telemetry:
        logs:
          level: debug
    ```
3.  **Use the Debug Exporter:** Add the `debug` exporter to the pipeline to print telemetry data to the console. Use `verbosity: detailed` to see the full payload.
    ```yaml
    exporters:
      debug:
        verbosity: detailed
    ```
4.  **Check Internal Metrics:** Monitor the `otelcol_*` metrics mentioned in section 2.6 to identify bottlenecks, dropped data, or queue buildups.
5.  **Use zpages:** Access the zpages endpoint (default port 55679) and navigate to `/debug/tracez` to analyze latency, deadlocks, and errors in the pipeline.
6.  **Use pprof:** Access the pprof endpoint (default port 1777) to profile CPU and memory usage if the Collector is experiencing performance issues.

### 5.2 Component-Specific Troubleshooting

#### Receivers

*   **Issue: Not receiving data.**
    *   **Check:** Network connectivity, firewall rules, receiver port configuration.
    *   **Check:** Ensure the receiver is actually included in a pipeline in the `service` section.
    *   **Check:** Verify the format of the incoming data matches the receiver type.
*   **Issue: High `otelcol_receiver_refused_spans`.**
    *   **Check:** Downstream components (processors, exporters) might be applying backpressure due to being overwhelmed. Check exporter queues and backend performance.
    *   **Check:** The incoming data might be malformed or invalid.

#### Processors

*   **Issue: Data is not being transformed as expected.**
    *   **Check:** Misunderstanding of processor scope. For example, the `attributes` processor modifies span attributes, not the span name.
    *   **Check:** OTTL syntax errors or incorrect conditions in the `transform` or `filter` processors. Use the `debug` exporter before and after the processor to verify the changes.
*   **Issue: High `otelcol_processor_dropped_spans`.**
    *   **Check:** This is expected if you are using a `filter` processor or tail sampling. If unexpected, review the configuration of these processors.
    *   **Check:** The `memory_limiter` processor might be dropping data if memory limits are exceeded. Check memory usage and `GOMEMLIMIT` settings.

#### Exporters

*   **Issue: Not exporting data.**
    *   **Check:** Network connectivity to the backend destination.
    *   **Check:** Authentication credentials (API keys, tokens) and TLS configuration.
    *   **Check:** Ensure the exporter is included in a pipeline.
*   **Issue: High `otelcol_exporter_send_failed_spans`.**
    *   **Check:** The backend destination might be unavailable, rate-limiting requests, or rejecting payloads due to size limits.
    *   **Action:** Configure the `batch` processor to reduce payload size (`send_batch_max_size`). Ensure `retry_on_failure` is enabled and properly configured.
*   **Issue: High `otelcol_exporter_enqueue_failed_spans`.**
    *   **Check:** The sending queue is full. This indicates the exporter cannot send data fast enough to keep up with the incoming rate.
    *   **Action:** Increase `queue_size` (if memory permits), scale out the Collector, or investigate backend performance issues. Consider using the `filestorage` extension for persistent queuing.

### 5.3 Common Configuration Errors

*   **Null Maps in YAML:** Be careful with empty sections in YAML. Use `{}` to explicitly define an empty map if required by the component.
*   **Missing Components in Pipeline:** Defining a component in the `receivers`, `processors`, or `exporters` section does not activate it. It must be explicitly added to a pipeline in the `service.pipelines` section.
*   **Incorrect Component IDs:** Ensure component IDs are correctly formatted (e.g., `otlp/metrics`) and match the references in the pipeline configuration.

## 6. Security Best Practices

*   **Never run as root:** Run the Collector process as a non-root user.
*   **Use Encrypted Connections:** Default to TLS for all gRPC and HTTP connections (`insecure: false`).
*   **Protect Sensitive Data:** Use `configopaque.String` for sensitive fields like API keys and tokens in custom components. Use environment variables or secret management systems to inject secrets into the configuration.
*   **Minimize Privileged Access:** Only grant the necessary RBAC permissions to Kubernetes components (e.g., `k8sattributes` processor needs access to pods and namespaces).
*   **Restrict Endpoint Access:** Do not expose health, pprof, or zpages endpoints externally by default. Bind them to `localhost` or internal networks.

## 7. OpenTelemetry Collector Builder (OCB)

The OpenTelemetry Collector Builder (OCB) is a tool used to generate custom Collector distributions containing only the specific components required for your use case. This reduces the binary size, attack surface, and memory footprint.

**Example `builder-config.yaml`:**

```yaml
dist:
  name: custom-collector
  description: Custom OpenTelemetry Collector
  output_path: ./dist
  otelcol_version: 0.90.0

receivers:
  - gomod: go.opentelemetry.io/collector/receiver/otlpreceiver v0.90.0

processors:
  - gomod: go.opentelemetry.io/collector/processor/batchprocessor v0.90.0
  - gomod: go.opentelemetry.io/collector/processor/memorylimiterprocessor v0.90.0

exporters:
  - gomod: go.opentelemetry.io/collector/exporter/otlpexporter v0.90.0
  - gomod: go.opentelemetry.io/collector/exporter/debugexporter v0.90.0
```

Run OCB: `./ocb --config builder-config.yaml`

## 8. References

[1] OpenTelemetry Collector Documentation: https://opentelemetry.io/docs/collector/
[2] OpenTelemetry Operator Documentation: https://opentelemetry.io/docs/kubernetes/operator/
[3] OpenTelemetry Collector Builder (OCB): https://opentelemetry.io/docs/collector/custom-collector/
