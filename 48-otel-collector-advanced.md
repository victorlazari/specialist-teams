# OpenTelemetry Collector Advanced Patterns: The Complete Guide

The OpenTelemetry (OTel) Collector is the vendor-agnostic telemetry pipeline that receives, processes, and exports traces, metrics, and logs. While basic configurations are straightforward, production environments require advanced patterns for transformation, scaling, routing, and custom builds. This guide provides a comprehensive, production-ready reference for advanced OpenTelemetry Collector operations, focusing on the OpenTelemetry Transformation Language (OTTL), scaling architectures, connector patterns, and custom collector building.

## 1. OpenTelemetry Transformation Language (OTTL)

The OpenTelemetry Transformation Language (OTTL) is a domain-specific language (DSL) designed specifically for transforming telemetry data within the Collector. It is the engine behind the `transform` processor, `filter` processor, `tailsampling` processor, and `routing` connector.

### 1.1 OTTL Contexts

OTTL operates within specific contexts, which determine what data is available for transformation. Understanding contexts is crucial for writing effective OTTL statements.

| Context | Description | Available Fields |
| :--- | :--- | :--- |
| `resource` | Operates on Resource attributes (e.g., host, cloud provider). | `attributes`, `dropped_attributes_count` |
| `scope` | Operates on Instrumentation Scope (e.g., library name/version). | `name`, `version`, `attributes`, `dropped_attributes_count` |
| `span` | Operates on individual Spans (traces). | `trace_id`, `span_id`, `trace_state`, `parent_span_id`, `name`, `kind`, `start_time_unix_nano`, `end_time_unix_nano`, `attributes`, `dropped_attributes_count`, `events`, `dropped_events_count`, `links`, `dropped_links_count`, `status` |
| `spanevent` | Operates on Span Events (logs within traces). | `time_unix_nano`, `name`, `attributes`, `dropped_attributes_count` |
| `metric` | Operates on Metrics (metadata). | `name`, `description`, `unit`, `type`, `is_monotonic`, `aggregation_temporality` |
| `datapoint` | Operates on individual Metric Data Points. | `attributes`, `start_time_unix_nano`, `time_unix_nano`, `value_double`, `value_int`, `exemplars`, `flags` |
| `log` | Operates on Log Records. | `time_unix_nano`, `observed_time_unix_nano`, `severity_number`, `severity_text`, `body`, `attributes`, `dropped_attributes_count`, `flags`, `trace_id`, `span_id` |

### 1.2 OTTL Functions Reference

OTTL provides a rich set of functions for manipulating data. Below is a complete reference of all available functions.

#### Modification Functions

*   **`set(target, value)`**: Sets the `target` to the specified `value`.
    *   *Example*: `set(attributes["environment"], "production")`
*   **`delete_key(target, key)`**: Deletes a specific `key` from a map (like `attributes`).
    *   *Example*: `delete_key(attributes, "password")`
*   **`keep_keys(target, keys[])`**: Keeps only the specified `keys` in a map, deleting all others.
    *   *Example*: `keep_keys(attributes, ["http.method", "http.status_code"])`
*   **`truncate_all(target, limit)`**: Truncates all string values in a map to the specified `limit`.
    *   *Example*: `truncate_all(attributes, 256)`
*   **`replace_match(target, pattern, replacement)`**: Replaces the first match of `pattern` in `target` with `replacement`.
    *   *Example*: `replace_match(attributes["url"], "http://", "https://")`
*   **`replace_pattern(target, regex, replacement)`**: Replaces all matches of `regex` in `target` with `replacement`.
    *   *Example*: `replace_pattern(attributes["message"], "\\d{4}-\\d{2}-\\d{2}", "[REDACTED_DATE]")`
*   **`merge_maps(target, source, strategy)`**: Merges `source` map into `target` map. Strategies: `insert` (keep target), `update` (overwrite target), `upsert` (insert or update).
    *   *Example*: `merge_maps(attributes, resource.attributes, "insert")`
*   **`convert_case(target, case)`**: Converts string to `lower`, `upper`, `snake`, or `camel` case.
    *   *Example*: `convert_case(attributes["status"], "upper")`
*   **`Append(target, value)`**: Appends a value to an array.
    *   *Example*: `Append(attributes["tags"], "processed")`
*   **`Flatten(target)`**: Flattens a nested map into a single-level map with dot-separated keys.
    *   *Example*: `Flatten(attributes["kubernetes"])`

#### Hashing and Cryptography

*   **`fnv(target)`**: Computes the FNV-1a hash of the target.
    *   *Example*: `set(attributes["user_id_hash"], fnv(attributes["user_id"]))`
*   **`sha256(target)`**: Computes the SHA-256 hash of the target.
    *   *Example*: `set(attributes["email_hash"], sha256(attributes["email"]))`

#### Evaluation and Type Conversion

*   **`IsMatch(target, regex)`**: Returns true if `target` matches the `regex`.
    *   *Example*: `IsMatch(attributes["http.route"], "^/api/v1/.*")`
*   **`Int(target)`**: Converts target to an integer.
    *   *Example*: `set(attributes["status_code"], Int(attributes["status_string"]))`
*   **`Double(target)`**: Converts target to a double (float).
    *   *Example*: `set(attributes["response_time"], Double(attributes["time_ms"]))`
*   **`String(target)`**: Converts target to a string.
    *   *Example*: `set(attributes["id_str"], String(attributes["id_int"]))`

#### String Manipulation

*   **`Concat(values[], separator)`**: Concatenates an array of strings with a separator.
    *   *Example*: `set(attributes["full_name"], Concat([attributes["first"], attributes["last"]], " "))`
*   **`Split(target, separator)`**: Splits a string into an array using a separator.
    *   *Example*: `set(attributes["path_parts"], Split(attributes["url.path"], "/"))`
*   **`Substring(target, start, length)`**: Extracts a substring.
    *   *Example*: `set(attributes["short_id"], Substring(attributes["uuid"], 0, 8))`
*   **`Len(target)`**: Returns the length of a string, array, or map.
    *   *Example*: `set(attributes["tag_count"], Len(attributes["tags"]))`

#### Generation and Parsing

*   **`UUID()`**: Generates a new UUIDv4.
    *   *Example*: `set(attributes["request_id"], UUID()) where attributes["request_id"] == nil`
*   **`Now()`**: Returns the current time.
    *   *Example*: `set(attributes["processed_at"], Now())`
*   **`Duration(start, end)`**: Calculates the duration between two times.
    *   *Example*: `set(attributes["processing_time"], Duration(start_time_unix_nano, end_time_unix_nano))`
*   **`ParseJSON(target)`**: Parses a JSON string into a map.
    *   *Example*: `merge_maps(attributes, ParseJSON(body), "upsert")`
*   **`ParseCSV(target, headers[])`**: Parses a CSV string into a map using provided headers.
    *   *Example*: `merge_maps(attributes, ParseCSV(body, ["timestamp", "level", "message"]), "upsert")`
*   **`ExtractPatterns(target, regex)`**: Extracts named capture groups from a regex into a map.
    *   *Example*: `merge_maps(attributes, ExtractPatterns(body, "^(?P<time>\\S+) (?P<level>\\S+) (?P<msg>.*)$"), "upsert")`

### 1.3 OTTL Usage in Processors

#### Transform Processor

The `transform` processor is the primary vehicle for OTTL. It allows you to modify telemetry data in flight.

```yaml
processors:
  transform:
    error_mode: ignore
    trace_statements:
      - context: span
        statements:
          # Redact sensitive information from span names
          - replace_pattern(name, "password=[^&]+", "password=REDACTED")
          # Add a custom attribute based on an existing one
          - set(attributes["is_error"], true) where status.code == 2
          # Extract JSON from a log body (if this was a log context)
    metric_statements:
      - context: datapoint
        statements:
          # Convert bytes to megabytes
          - set(value_double, value_int / 1048576.0) where metric.name == "system.memory.usage"
```

#### Filter Processor

The `filter` processor uses OTTL conditions to drop telemetry data.

```yaml
processors:
  filter/healthchecks:
    error_mode: ignore
    traces:
      span:
        # Drop all spans related to health checks
        - 'attributes["http.route"] == "/healthz"'
        - 'attributes["http.route"] == "/ready"'
    logs:
      log:
        # Drop debug logs
        - 'severity_text == "DEBUG"'
```

#### Tail Sampling Processor

Tail sampling uses OTTL to define complex sampling policies based on the entire trace.

```yaml
processors:
  tailsampling:
    decision_wait: 10s
    num_traces: 100000
    expected_new_traces_per_sec: 1000
    policies:
      - name: errors-policy
        type: string_attribute
        string_attribute:
          key: error
          values:
            - "true"
      - name: ottl-policy
        type: ottl_condition
        ottl_condition:
          error_mode: ignore
          span_properties:
            # Sample traces where any span takes longer than 5 seconds
            - 'Duration(start_time_unix_nano, end_time_unix_nano) > 5000000000'
```

#### Routing Connector

The `routing` connector uses OTTL to route telemetry to different pipelines based on conditions.

```yaml
connectors:
  routing:
    default_pipelines: [traces/default]
    error_mode: ignore
    table:
      - statement: route() where attributes["tenant_id"] == "premium"
        pipelines: [traces/premium]
      - statement: route() where attributes["environment"] == "dev"
        pipelines: [traces/dev]
```

## 2. Scaling Patterns

Scaling the OpenTelemetry Collector requires understanding the nature of the telemetry data and the components in use. There are three primary scaling patterns: Stateless, Scrapers, and Stateful.

### 2.1 Stateless Scaling

Stateless scaling is the simplest pattern. It applies when the Collector is only performing stateless operations (e.g., basic filtering, transformation, batching) and exporting to a backend that handles aggregation.

*   **Architecture**: Multiple Collector replicas behind a load balancer.
*   **Load Balancing**:
    *   For HTTP (OTLP/HTTP, Zipkin), a standard L4/L7 load balancer works.
    *   For gRPC (OTLP/gRPC), an L7 load balancer (like Envoy or HAProxy) is **required** to balance individual gRPC streams, not just connections. Otherwise, one Collector might receive all traffic from a long-lived connection.
*   **Metrics to Monitor**:
    *   `otelcol_processor_refused_spans`
    *   `otelcol_exporter_queue_size` / `otelcol_exporter_queue_capacity`
    *   `otelcol_exporter_enqueue_failed_spans`
    *   `otelcol_exporter_send_failed_spans`
*   **Scaling Trigger**: Scale up when `otelcol_exporter_queue_size` consistently reaches 60-70% of capacity.
*   **Warning**: Do not scale the Collector if the backend (e.g., Jaeger, Prometheus) is the bottleneck. If `otelcol_exporter_send_failed_spans` is high but CPU/Memory on the Collector is low, the backend is likely rejecting data.

### 2.2 Scrapers (Target Allocator)

When using the Collector to scrape Prometheus metrics, a single Collector can quickly become overwhelmed by a large number of targets.

*   **Architecture**: OpenTelemetry Operator with the Target Allocator component.
*   **How it works**:
    1.  The Target Allocator discovers Prometheus scrape targets (via ServiceMonitors, PodMonitors, or static configs).
    2.  It distributes (shards) these targets evenly across a StatefulSet of OpenTelemetry Collectors.
    3.  Each Collector replica only scrapes its assigned subset of targets.
*   **Configuration (Operator CRD)**:

```yaml
apiVersion: opentelemetry.io/v1alpha1
kind: OpenTelemetryCollector
metadata:
  name: prometheus-scraper
spec:
  mode: statefulset
  targetAllocator:
    enabled: true
    prometheusCR:
      enabled: true
  config: |
    receivers:
      prometheus:
        config:
          scrape_configs:
            - job_name: 'otel-collector'
              scrape_interval: 10s
              static_configs:
                - targets: ['0.0.0.0:8888']
    # ... rest of config
```

### 2.3 Stateful Scaling (Consistent Hashing)

Stateful scaling is required when the Collector performs operations that require a complete view of a trace or metric stream. Examples include:
*   Tail Sampling (needs all spans of a trace to make a decision).
*   Span-to-Metrics generation (needs all spans to calculate accurate rates/durations).
*   GroupByTrace processor.

*   **Architecture**: A two-tier architecture.
    1.  **Tier 1 (Stateless/Receiving)**: Receives traffic, performs initial stateless processing, and uses the `loadbalancing` exporter to route data to Tier 2 based on a consistent hash (usually `trace_id`).
    2.  **Tier 2 (Stateful/Processing)**: Receives all data for a specific `trace_id`, performs stateful processing (e.g., tail sampling), and exports to the final backend.

*   **Tier 1 Configuration (loadbalancing exporter)**:

```yaml
exporters:
  loadbalancing:
    protocol:
      otlp:
        tls:
          insecure: true
    resolver:
      dns:
        hostname: otelcol-tier2-headless.default.svc.cluster.local
        port: 4317
    routing_key: "traceID" # Ensure all spans for a trace go to the same Tier 2 pod
```

## 3. Connector Patterns

Connectors bridge two pipelines, acting as both an exporter (at the end of one pipeline) and a receiver (at the beginning of another). This enables complex, multi-layer architectures within a single Collector instance.

### 3.1 Routing Connector

Routes telemetry to different pipelines based on OTTL conditions (see section 1.3). Useful for tenant isolation, environment separation, or sending high-value data to a premium backend and low-value data to cold storage.

### 3.2 Spanmetrics Connector

Generates Request, Error, and Duration (RED) metrics from incoming spans. This is a stateful operation and requires consistent hashing if scaled.

```yaml
connectors:
  spanmetrics:
    histogram:
      explicit:
        buckets: [2ms, 5ms, 10ms, 20ms, 50ms, 100ms, 200ms, 500ms, 1s, 2s, 5s]
    dimensions:
      - name: http.method
      - name: http.status_code
    metrics_flush_interval: 15s

service:
  pipelines:
    traces:
      receivers: [otlp]
      exporters: [spanmetrics, otlp/backend] # Send to connector AND backend
    metrics/spanmetrics:
      receivers: [spanmetrics] # Receive from connector
      exporters: [prometheus]
```

### 3.3 Servicegraph Connector

Generates a service graph (nodes and edges representing service dependencies) from spans. Like `spanmetrics`, this is stateful.

```yaml
connectors:
  servicegraph:
    metrics_flush_interval: 15s
    store:
      ttl: 2s
      max_items: 1000

service:
  pipelines:
    traces:
      receivers: [otlp]
      exporters: [servicegraph, otlp/backend]
    metrics/servicegraph:
      receivers: [servicegraph]
      exporters: [prometheus]
```

### 3.4 Count Connector

A simple connector that counts the number of telemetry items (spans, metric data points, log records) passing through it and emits a metric. Useful for internal pipeline monitoring.

```yaml
connectors:
  count:
    spans.count:
      description: "Number of spans processed"

service:
  pipelines:
    traces:
      receivers: [otlp]
      exporters: [count, otlp/backend]
    metrics/internal:
      receivers: [count]
      exporters: [prometheus]
```

### 3.5 Failover Connector

Provides high availability by attempting to send data to a primary pipeline, and falling back to secondary pipelines if the primary fails.

```yaml
connectors:
  failover:
    pipeline_priority:
      - [traces/primary]
      - [traces/secondary]
    retry_interval: 5m
    retry_gap: 10s
    max_retries: 3

service:
  pipelines:
    traces/in:
      receivers: [otlp]
      exporters: [failover]
    traces/primary:
      receivers: [failover]
      exporters: [otlp/primary_backend]
    traces/secondary:
      receivers: [failover]
      exporters: [otlp/backup_backend]
```

## 4. Custom Collector Building with OCB

The OpenTelemetry Collector Builder (OCB) is a CLI tool used to generate a custom Collector binary containing only the specific components (receivers, processors, exporters, etc.) you need. This reduces the attack surface, binary size, and memory footprint.

### 4.1 The Manifest Format (`builder-config.yaml`)

The manifest defines the distribution details and the Go modules to include.

```yaml
dist:
  name: custom-otelcol
  description: "Custom OTel Collector for Acme Corp"
  output_path: ./dist
  otelcol_version: 0.95.0
  version: 1.0.0
  go: /usr/local/go/bin/go
  debug_compilation: false

receivers:
  - gomod: go.opentelemetry.io/collector/receiver/otlpreceiver v0.95.0
  - gomod: github.com/open-telemetry/opentelemetry-collector-contrib/receiver/hostmetricsreceiver v0.95.0

processors:
  - gomod: go.opentelemetry.io/collector/processor/batchprocessor v0.95.0
  - gomod: go.opentelemetry.io/collector/processor/memorylimiterprocessor v0.95.0
  - gomod: github.com/open-telemetry/opentelemetry-collector-contrib/processor/transformprocessor v0.95.0

exporters:
  - gomod: go.opentelemetry.io/collector/exporter/otlpexporter v0.95.0
  - gomod: go.opentelemetry.io/collector/exporter/debugexporter v0.95.0

connectors:
  - gomod: github.com/open-telemetry/opentelemetry-collector-contrib/connector/routingconnector v0.95.0

extensions:
  - gomod: go.opentelemetry.io/collector/extension/zpagesextension v0.95.0
  - gomod: github.com/open-telemetry/opentelemetry-collector-contrib/extension/healthcheckextension v0.95.0

providers:
  - gomod: go.opentelemetry.io/collector/confmap/provider/envprovider v0.95.0
  - gomod: go.opentelemetry.io/collector/confmap/provider/fileprovider v0.95.0
  - gomod: go.opentelemetry.io/collector/confmap/provider/yamlprovider v0.95.0
```

### 4.2 Building and Containerization

1.  **Download OCB**: Download the `ocb` binary for your platform from the OpenTelemetry releases page.
2.  **Run OCB**:
    ```bash
    ./ocb --config builder-config.yaml
    ```
    This generates Go source code in the `output_path` and compiles the binary.

3.  **Multi-stage Dockerfile**:

```dockerfile
# Stage 1: Build
FROM golang:1.21 AS builder
WORKDIR /build
# Copy OCB binary and manifest
COPY ocb builder-config.yaml ./
# Run OCB
RUN ./ocb --config builder-config.yaml

# Stage 2: Run
FROM cgr.dev/chainguard/glibc-dynamic:latest
# Copy the compiled binary from the builder stage
COPY --from=builder /build/dist/custom-otelcol /custom-otelcol
# Copy default config
COPY config.yaml /etc/otelcol/config.yaml
USER nonroot
ENTRYPOINT ["/custom-otelcol"]
CMD ["--config", "/etc/otelcol/config.yaml"]
```

## 5. Troubleshooting and Operations

### 5.1 Memory Management

Out of Memory (OOM) kills are the most common issue with the Collector. Proper memory management is critical.

1.  **GOMEMLIMIT**: Set the `GOMEMLIMIT` environment variable to 80% of the container's hard memory limit. This tells the Go garbage collector to work harder as memory approaches this limit.
    ```yaml
    env:
      - name: GOMEMLIMIT
        value: "800MiB" # Assuming a 1GiB container limit
    ```
2.  **Memory Limiter Processor**: This MUST be the **first** processor in every pipeline. It drops data when memory usage gets too high to prevent OOMs.
    ```yaml
    processors:
      memory_limiter:
        check_interval: 1s
        limit_percentage: 80
        spike_limit_percentage: 20 # 20% of the limit_percentage
    ```

### 5.2 Diagnostic Tools

*   **zPages**: Enable the `zpages` extension to view internal pipeline metrics, trace latency, and errors in a web browser.
    ```yaml
    extensions:
      zpages:
        endpoint: 0.0.0.0:55679
    ```
    Access `http://<collector-ip>:55679/debug/tracez` to see spans currently being processed, queued, or errored.
*   **pprof**: Enable the `pprof` extension for Go profiling (CPU, memory, goroutines).
    ```yaml
    extensions:
      pprof:
        endpoint: 0.0.0.0:1777
    ```
    Use `go tool pprof http://<collector-ip>:1777/debug/pprof/heap` to analyze memory usage.
*   **Debug Exporter**: Use the `debug` exporter with `verbosity: detailed` to print the exact telemetry payload to stdout. This is invaluable for verifying OTTL transformations.
    ```yaml
    exporters:
      debug:
        verbosity: detailed
    ```

### 5.3 Common Scenarios and Fixes

| Symptom | Root Cause | Resolution |
| :--- | :--- | :--- |
| **Data Dropped (High `otelcol_exporter_enqueue_failed_spans`)** | Exporter queue is full because the backend is slow or unreachable. | 1. Check backend health. 2. Increase `sending_queue.queue_size` in the exporter (requires more memory). 3. Scale out the Collector (Stateless pattern). |
| **OOM Kills** | Memory spike exceeded limits before GC could run, or `memory_limiter` is misconfigured. | 1. Ensure `GOMEMLIMIT` is set. 2. Ensure `memory_limiter` is the *first* processor. 3. Decrease `spike_limit_percentage`. |
| **Transformations Not Applied** | OTTL condition is incorrect, or processor is not in the pipeline. | 1. Verify the processor is listed in `service.pipelines.<type>.processors`. 2. Use the `debug` exporter to inspect the payload before and after the transform processor. 3. Check for YAML null map issues (ensure proper indentation). |
| **"Failed to start" / Config Error** | Invalid YAML, missing component, or circular pipeline dependency. | 1. Check the Collector startup logs. 2. Ensure all components used in `service.pipelines` are defined in their respective sections. 3. If using a custom build, ensure the component is in `builder-config.yaml`. |
| **Missing Kubernetes Attributes** | `k8sattributes` processor lacks RBAC permissions to query the API server. | Ensure the Collector's ServiceAccount has a ClusterRole with `get`, `watch`, and `list` permissions for `pods`, `namespaces`, and `nodes`. |

## 6. Security Best Practices

*   **Never run as root**: Always use a non-root user in your Dockerfile (e.g., `USER nonroot`).
*   **Encrypted Connections**: Default to TLS for all receivers and exporters. Set `insecure: false` where applicable.
*   **Config Opaque Strings**: Use `configopaque.String` in custom component development for sensitive fields (passwords, tokens) so they are masked in logs and zPages.
*   **Network Policies**: Restrict access to the Collector's receiver ports (e.g., 4317, 4318) and extension ports (e.g., 55679, 1777) using Kubernetes NetworkPolicies. Do not expose health or telemetry endpoints externally.
*   **Authentication**: Use extensions like `basicauth`, `bearertokenauth`, or `oidcauth` to secure receivers.

```yaml
extensions:
  bearertokenauth:
    scheme: "Bearer"
    token: "YOUR_SECURE_TOKEN" # In practice, use env vars: ${env:AUTH_TOKEN}

receivers:
  otlp:
    protocols:
      grpc:
        auth:
          authenticator: bearertokenauth
```

---
*Author: Manus AI*
