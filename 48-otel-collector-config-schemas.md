# OpenTelemetry Collector Configuration Schemas

The OpenTelemetry (OTel) Collector is a vendor-agnostic implementation on how to receive, process, and export telemetry data. It removes the need to run, operate, and maintain multiple agents/collectors. This document provides a comprehensive, production-ready guide to configuring the OpenTelemetry Collector, focusing on configuration schemas, deployment patterns, and troubleshooting from a tech support operations perspective.

---

## 1. Top-Level Configuration Structure

The OpenTelemetry Collector configuration is defined in a YAML file (typically `/etc/otelcol/config.yaml`). The configuration is divided into several top-level sections, each responsible for a specific part of the telemetry pipeline.

```yaml
# Receivers: How data gets into the Collector
receivers:
  # ... receiver configurations ...

# Processors: What happens to the data once it's inside
processors:
  # ... processor configurations ...

# Exporters: Where the data goes
exporters:
  # ... exporter configurations ...

# Connectors: Bridge two pipelines (act as both exporter and receiver)
connectors:
  # ... connector configurations ...

# Extensions: Auxiliary capabilities (health checks, pprof, zpages)
extensions:
  # ... extension configurations ...

# Service: Ties everything together into pipelines
service:
  # ... service configuration ...
```

### Component ID Format
Every component in the configuration is identified by a unique ID in the format `type[/name]`.
- `type`: The type of the component (e.g., `otlp`, `prometheus`, `batch`).
- `name`: An optional identifier to distinguish multiple instances of the same type (e.g., `otlp/metrics`, `otlp/traces`).

---

## 2. Service Section

The `service` section is the heart of the configuration. It defines which extensions are enabled, configures the internal telemetry of the Collector itself, and defines the data pipelines.

```yaml
service:
  # Extensions to enable
  extensions: [health_check, pprof, zpages]

  # Internal telemetry configuration
  telemetry:
    logs:
      level: info # debug, info, warn, error
      encoding: json # console, json
      output_paths: ["stdout", "/var/log/otelcol/collector.log"]
    metrics:
      level: detailed # none, basic, normal, detailed
      address: 0.0.0.0:8888
    traces:
      propagators: [tracecontext, b3]
      sampling:
        default_sampler:
          type: always_on

  # Data pipelines
  pipelines:
    traces:
      receivers: [otlp, jaeger]
      processors: [memory_limiter, batch]
      exporters: [otlp/honeycomb]
    metrics:
      receivers: [otlp, prometheus]
      processors: [memory_limiter, batch]
      exporters: [prometheusremotewrite]
    logs:
      receivers: [otlp, filelog]
      processors: [memory_limiter, batch]
      exporters: [elasticsearch]
```

### Troubleshooting the Service Section
- **Issue:** Collector starts but no data is flowing.
  - **Check:** Ensure the components are actually listed in the `pipelines` section. A component defined in the top-level section but not included in a pipeline will not be instantiated.
- **Issue:** Need to debug Collector internals.
  - **Action:** Change `service.telemetry.logs.level` to `debug` and restart the Collector.

---

## 3. Receivers

Receivers define how data gets into the Collector. They can be push-based (e.g., OTLP) or pull-based (e.g., Prometheus).

### OTLP Receiver
The OTLP receiver is the standard way to receive telemetry data. It supports both gRPC and HTTP.

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
        tls:
          cert_file: /etc/certs/cert.pem
          key_file: /etc/certs/key.pem
      http:
        endpoint: 0.0.0.0:4318
        cors:
          allowed_origins:
            - https://foo.bar.com
            - https://*.test.com
```

### Prometheus Receiver
The Prometheus receiver acts as a drop-in replacement for a Prometheus server, scraping metrics from targets.

```yaml
receivers:
  prometheus:
    config:
      scrape_configs:
        - job_name: 'kubernetes-apiservers'
          kubernetes_sd_configs:
            - role: endpoints
          scheme: https
          tls_config:
            ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
            insecure_skip_verify: true
          bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
```

### Filelog Receiver
Tails and parses logs from files.

```yaml
receivers:
  filelog:
    include: [ /var/log/**/*.log ]
    exclude: [ /var/log/otelcol/*.log ]
    start_at: end
    operators:
      - type: json_parser
        timestamp:
          parse_from: attributes.time
          layout: '%Y-%m-%dT%H:%M:%S.%LZ'
```

### Hostmetrics Receiver
Collects system metrics (CPU, memory, disk, network).

```yaml
receivers:
  hostmetrics:
    collection_interval: 10s
    scrapers:
      cpu: {}
      memory: {}
      disk: {}
      filesystem: {}
      network: {}
      load: {}
      paging: {}
```

### Kubernetes Cluster Receiver
Collects cluster-level metrics and events. Requires RBAC permissions.

```yaml
receivers:
  k8s_cluster:
    collection_interval: 10s
    node_conditions_to_report: [Ready, MemoryPressure]
    allocatable_types_to_report: [cpu, memory]
```

### Kafka Receiver
Consumes telemetry data from Kafka topics.

```yaml
receivers:
  kafka:
    protocol_version: 2.0.0
    brokers:
      - kafka1.example.com:9092
      - kafka2.example.com:9092
    topic: otlp_spans
    encoding: otlp_proto
    group_id: otel-collector
```

### Troubleshooting Receivers
- **Issue:** OTLP receiver failing with "transport: authentication handshake failed".
  - **Check:** Verify TLS certificates and ensure the client is configured to use TLS.
- **Issue:** Prometheus receiver not scraping targets.
  - **Check:** Verify the `scrape_configs` syntax. Use `otelcol components` to ensure the Prometheus receiver is included in your distribution. Check RBAC permissions if using `kubernetes_sd_configs`.

---

## 4. Processors

Processors modify, filter, or batch data before it is exported. Order matters!

### Memory Limiter Processor
**CRITICAL:** This should almost always be the first processor in the pipeline to prevent Out-Of-Memory (OOM) crashes.

```yaml
processors:
  memory_limiter:
    check_interval: 1s
    limit_mib: 4000
    spike_limit_mib: 800
    # Alternatively, use percentages in containerized environments:
    # limit_percentage: 80
    # spike_limit_percentage: 20
```
*Best Practice:* Set `GOMEMLIMIT` environment variable to 80% of the hard memory limit.

### Batch Processor
Groups telemetry data into batches. Highly recommended for performance.

```yaml
processors:
  batch:
    timeout: 1s
    send_batch_size: 8192
    send_batch_max_size: 0 # 0 means no upper limit
```

### Attributes Processor
Modifies attributes on spans or metrics.

```yaml
processors:
  attributes:
    actions:
      - key: environment
        value: production
        action: insert
      - key: db.statement
        action: delete
      - key: http.status_code
        action: hash
```

### Filter Processor (with OTTL)
Drops telemetry data based on OpenTelemetry Transformation Language (OTTL) conditions.

```yaml
processors:
  filter:
    error_mode: ignore
    traces:
      span:
        - 'attributes["http.route"] == "/health"'
    metrics:
      metric:
        - 'name == "http.server.duration"'
```

### Transform Processor (with OTTL)
Modifies telemetry data using OTTL.

```yaml
processors:
  transform:
    error_mode: ignore
    trace_statements:
      - context: span
        statements:
          - set(status.code, 1) where attributes["http.status_code"] >= 500
          - replace_pattern(name, "GET /api/v1/users/.*", "GET /api/v1/users/{id}")
```

### Kubernetes Attributes Processor
Enriches telemetry with Kubernetes metadata (pod name, namespace, etc.).

```yaml
processors:
  k8sattributes:
    auth_type: "serviceAccount"
    passthrough: false
    extract:
      metadata:
        - k8s.pod.name
        - k8s.pod.uid
        - k8s.deployment.name
        - k8s.namespace.name
        - k8s.node.name
        - k8s.pod.start_time
```

### Tail Sampling Processor
Samples traces based on policies evaluated after the entire trace is collected. Requires stateful deployment.

```yaml
processors:
  tail_sampling:
    decision_wait: 10s
    num_traces: 50000
    expected_new_traces_per_sec: 1000
    policies:
      - name: errors-policy
        type: status_code
        status_code:
          status_codes: [ERROR]
      - name: randomized-policy
        type: probabilistic
        probabilistic:
          sampling_percentage: 10
```

### Troubleshooting Processors
- **Issue:** Collector OOM kills.
  - **Check:** Ensure `memory_limiter` is the first processor. Verify `limit_mib` is lower than the container memory limit.
- **Issue:** Attributes not being modified.
  - **Check:** Ensure you are using the correct context (e.g., `resource` vs `attributes` processor). Processors work on individual spans/metrics, not the entire payload at once.

---

## 5. Exporters

Exporters define where the data is sent.

### OTLP Exporter
Sends data via gRPC.

```yaml
exporters:
  otlp:
    endpoint: YOUR_ENDPOINT:4317
    tls:
      insecure: false
      cert_file: /etc/certs/cert.pem
      key_file: /etc/certs/key.pem
    compression: gzip
    retry_on_failure:
      enabled: true
      initial_interval: 5s
      max_interval: 30s
      max_elapsed_time: 300s
    sending_queue:
      enabled: true
      num_consumers: 10
      queue_size: 5000
    timeout: 10s
```

### OTLP HTTP Exporter
Sends data via HTTP.

```yaml
exporters:
  otlphttp:
    endpoint: https://YOUR_ENDPOINT
    tls:
      insecure: false
    headers:
      "x-api-key": "YOUR_API_KEY"
```

### Prometheus Exporter
Exposes a Prometheus metrics endpoint.

```yaml
exporters:
  prometheus:
    endpoint: "0.0.0.0:8889"
    namespace: "my_app"
    const_labels:
      environment: "production"
```

### Load Balancing Exporter
Routes spans/metrics to multiple backends based on a routing key (e.g., trace ID). Essential for stateful components like tail sampling.

```yaml
exporters:
  loadbalancing:
    protocol:
      otlp:
        tls:
          insecure: true
    resolver:
      dns:
        hostname: otelcol-backend.default.svc.cluster.local
        port: 4317
    routing_key: traceID
```

### Troubleshooting Exporters
- **Issue:** Data dropping, `otelcol_exporter_queue_size` is high.
  - **Check:** The backend might be slow or unavailable. Increase `sending_queue.queue_size` or scale up the Collector. Do NOT scale up if the backend is the bottleneck.
- **Issue:** "context deadline exceeded" errors.
  - **Check:** Increase the `timeout` setting. Verify network connectivity to the endpoint.

---

## 6. Extensions

Extensions provide auxiliary capabilities.

### Health Check Extension
Exposes an HTTP endpoint for health probes.

```yaml
extensions:
  health_check:
    endpoint: 0.0.0.0:13133
    path: /
```

### Pprof Extension
Exposes Go pprof endpoints for profiling.

```yaml
extensions:
  pprof:
    endpoint: 0.0.0.0:1777
```

### ZPages Extension
Exposes debugging pages.

```yaml
extensions:
  zpages:
    endpoint: 0.0.0.0:55679
```

### Basic Auth Extension
Provides basic authentication for receivers.

```yaml
extensions:
  basicauth/server:
    htpasswd:
      file: /etc/otelcol/.htpasswd
```

### File Storage Extension
Provides persistent storage for queues.

```yaml
extensions:
  file_storage:
    directory: /var/lib/otelcol/storage
    timeout: 1s
```

---

## 7. Kubernetes Deployment (Helm & Operator)

### Helm values.yaml Schema
When deploying via the `opentelemetry-collector` Helm chart, the configuration is passed via the `config` block.

```yaml
mode: deployment # deployment, daemonset, statefulset

image:
  repository: otel/opentelemetry-collector-contrib
  tag: "0.90.0"

config:
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
    batch: {}
  exporters:
    debug:
      verbosity: detailed
  service:
    pipelines:
      traces:
        receivers: [otlp]
        processors: [memory_limiter, batch]
        exporters: [debug]

resources:
  limits:
    cpu: 1000m
    memory: 2Gi
  requests:
    cpu: 200m
    memory: 400Mi
```

### Operator CRD Spec
The OpenTelemetry Operator manages `OpenTelemetryCollector` Custom Resources.

```yaml
apiVersion: opentelemetry.io/v1alpha1
kind: OpenTelemetryCollector
metadata:
  name: my-collector
  namespace: observability
spec:
  mode: deployment # deployment, daemonset, statefulset, sidecar
  replicas: 3
  upgradeStrategy: automatic
  config: |
    receivers:
      otlp:
        protocols:
          grpc:
          http:
    processors:
      batch:
    exporters:
      debug:
    service:
      pipelines:
        traces:
          receivers: [otlp]
          processors: [batch]
          exporters: [debug]
```

#### Sidecar Injection
To inject a Collector sidecar into a pod, use the following annotation on the Pod template:
```yaml
annotations:
  sidecar.opentelemetry.io/inject: "true" # Or specify the name of the OpenTelemetryCollector CR
```

---

## 8. Environment Variables Reference

The Collector supports environment variable substitution in the configuration file using the `${ENV_VAR}` syntax.

Standard OpenTelemetry Environment Variables:
- `OTEL_RESOURCE_ATTRIBUTES`: Key-value pairs to be used as resource attributes (e.g., `key1=value1,key2=value2`).
- `OTEL_SERVICE_NAME`: Logical name of the service.
- `OTEL_EXPORTER_OTLP_ENDPOINT`: Target URL to which the exporter is going to send spans or metrics.
- `OTEL_EXPORTER_OTLP_HEADERS`: Key-value pairs to be used as headers.
- `GOMEMLIMIT`: Go runtime memory limit (e.g., `1500MiB`). Crucial for preventing OOMs.

Example usage in config:
```yaml
exporters:
  otlp:
    endpoint: ${OTEL_EXPORTER_OTLP_ENDPOINT}
    headers:
      "x-api-key": ${API_KEY}
```

---

## 9. Advanced Troubleshooting & Operations

### Debugging Tools
1.  **Debug Exporter:** Add the `debug` exporter to your pipeline with `verbosity: detailed` to see the exact payload the Collector is processing.
2.  **ZPages:** Access `http://<collector-ip>:55679/debug/tracez` to view latency, deadlocks, and errors in real-time.
3.  **Pprof:** Access `http://<collector-ip>:1777/debug/pprof/` for Go profiling (CPU, heap, goroutines).
4.  **`otelcol components`:** Run this command on the binary to list all compiled-in components and their stability levels.

### Common Operational Scenarios

#### Scenario 1: High Memory Usage / OOM Kills
- **Symptom:** Pod restarts with `OOMKilled`.
- **Resolution:**
  1. Ensure `memory_limiter` is the *first* processor.
  2. Set `limit_mib` to ~80% of the container's memory limit.
  3. Set `spike_limit_mib` to ~20% of the container's memory limit.
  4. Set the `GOMEMLIMIT` environment variable to match `limit_mib`.

#### Scenario 2: Data Dropping (Queue Full)
- **Symptom:** `otelcol_exporter_queue_size` metric is at capacity; `otelcol_exporter_enqueue_failed_spans` is increasing.
- **Resolution:**
  1. Check the destination backend. If it's slow or returning 429 (Too Many Requests), scaling the Collector will *not* help and might make it worse.
  2. If the backend is healthy, increase `sending_queue.queue_size` in the exporter config.
  3. Scale horizontally (add more Collector replicas).

#### Scenario 3: YAML Configuration Errors
- **Symptom:** Collector fails to start with parsing errors.
- **Resolution:**
  - Watch out for null maps. In YAML, `batch:` is a null map, whereas `batch: {}` is an empty map. Some components require an empty map to initialize with default settings.
  - Ensure environment variables used in `${VAR}` syntax are actually set in the environment.

#### Scenario 4: Tail Sampling Not Working
- **Symptom:** Tail sampling policies seem to be ignored or inconsistent.
- **Resolution:**
  - Tail sampling requires seeing the *entire* trace. If you have multiple Collector replicas, spans for the same trace might be sent to different replicas.
  - **Fix:** Use a two-tier architecture. Tier 1 (DaemonSet/Deployment) uses the `loadbalancing` exporter to route spans based on `traceID` to Tier 2. Tier 2 (StatefulSet) runs the `tail_sampling` processor.
