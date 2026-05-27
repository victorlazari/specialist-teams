# OpenTelemetry Collector CLI Reference

The OpenTelemetry Collector is a vendor-agnostic implementation on how to receive, process, and export telemetry data. It removes the need to run, operate, and maintain multiple agents/collectors. This document provides a comprehensive CLI reference for the OpenTelemetry Collector, including commands, flags, subcommands, tools, and Kubernetes deployments.

## 1. Core CLI Commands and Flags

The primary binary for the OpenTelemetry Collector is typically named `otelcol` (or `otelcol-contrib`, `otelcol-k8s`, depending on the distribution).

### 1.1 Starting the Collector

The most common command is starting the collector with a configuration file.

```bash
# Start with a local configuration file
otelcol --config /etc/otelcol/config.yaml

# Start with multiple configuration files (merged in order)
otelcol --config /etc/otelcol/config.yaml --config /etc/otelcol/overrides.yaml

# Start with an environment variable as config
otelcol --config env:MY_CONFIG_YAML

# Start with an HTTP URL as config
otelcol --config https://example.com/config.yaml
```

### 1.2 Configuration Overrides (`--set`)

You can override specific configuration values using the `--set` flag. This is useful for dynamic environments or testing.

```bash
# Override a specific property
otelcol --config config.yaml --set "exporters::otlp::endpoint=YOUR_ENDPOINT:4317"

# Override multiple properties
otelcol --config config.yaml \
  --set "receivers::otlp::protocols::grpc::endpoint=0.0.0.0:4317" \
  --set "service::telemetry::logs::level=debug"

# Override properties with arrays
otelcol --config config.yaml --set "service::pipelines::traces::exporters=[otlp, debug]"
```

### 1.3 Feature Gates (`--feature-gates`)

Feature gates allow you to enable or disable experimental features.

```bash
# Enable a specific feature gate
otelcol --config config.yaml --feature-gates=telemetry.useOtelForInternalMetrics

# Disable a specific feature gate (prefix with '-')
otelcol --config config.yaml --feature-gates=-pkg.translator.prometheus.NormalizeName

# Multiple feature gates (comma-separated)
otelcol --config config.yaml --feature-gates=telemetry.useOtelForInternalMetrics,-pkg.translator.prometheus.NormalizeName
```

### 1.4 Subcommands

#### `otelcol components`

Lists all available components (receivers, processors, exporters, extensions, connectors) built into the binary, along with their stability levels.

```bash
# List all components
otelcol components

# Example Output Snippet:
# Receivers:
#   - otlp (stable)
#   - prometheus (beta)
# Processors:
#   - batch (stable)
#   - memory_limiter (stable)
# Exporters:
#   - otlp (stable)
#   - debug (stable)
```

#### `otelcol validate`

Validates the configuration file without starting the collector. This is crucial for CI/CD pipelines.

```bash
# Validate a configuration file
otelcol validate --config config.yaml

# Example Output (Success):
# 2023-10-27T10:00:00.000Z	info	service/telemetry.go:113	Setting up internal telemetry	{"buildinfo": {"Command": "otelcol", "Version": "0.87.0", "OS": "linux", "Arch": "amd64"}}
# 2023-10-27T10:00:00.000Z	info	service/service.go:139	Validating configuration...
# 2023-10-27T10:00:00.000Z	info	service/service.go:141	Configuration is valid.

# Example Output (Error):
# Error: cannot load configuration: error reading providers: error reading file: open config.yaml: no such file or directory
```

## 2. OpenTelemetry Collector Builder (OCB)

The OpenTelemetry Collector Builder (`ocb`) is a tool used to generate a custom OpenTelemetry Collector binary based on a given configuration (manifest).

### 2.1 Manifest Format (`builder-config.yaml`)

The manifest defines the distribution details and the modules to include.

```yaml
dist:
  name: custom-collector
  description: "Custom OpenTelemetry Collector"
  output_path: ./dist
  otelcol_version: 0.87.0
  version: 1.0.0
  go: /usr/local/go/bin/go

receivers:
  - gomod: go.opentelemetry.io/collector/receiver/otlpreceiver v0.87.0
  - gomod: github.com/open-telemetry/opentelemetry-collector-contrib/receiver/hostmetricsreceiver v0.87.0

processors:
  - gomod: go.opentelemetry.io/collector/processor/batchprocessor v0.87.0
  - gomod: go.opentelemetry.io/collector/processor/memorylimiterprocessor v0.87.0

exporters:
  - gomod: go.opentelemetry.io/collector/exporter/otlpexporter v0.87.0
  - gomod: go.opentelemetry.io/collector/exporter/debugexporter v0.87.0

extensions:
  - gomod: go.opentelemetry.io/collector/extension/zpagesextension v0.87.0
```

### 2.2 OCB Commands

```bash
# Download OCB (example for Linux amd64)
curl --proto '=https' --tlsv1.2 -fL -o ocb \
  https://github.com/open-telemetry/opentelemetry-collector/releases/download/cmd%2Fbuilder%2Fv0.87.0/ocb_0.87.0_linux_amd64
chmod +x ocb

# Build the custom collector
./ocb --config builder-config.yaml

# Example Output:
# 2023-10-27T10:00:00.000Z	INFO	internal/builder/main.go:60	OpenTelemetry Collector Builder	{"version": "0.87.0", "date": "2023-10-27T10:00:00Z"}
# 2023-10-27T10:00:00.000Z	INFO	internal/builder/main.go:93	Parsing configuration	{"config": "builder-config.yaml"}
# 2023-10-27T10:00:00.000Z	INFO	internal/builder/main.go:120	Generating source code
# 2023-10-27T10:00:00.000Z	INFO	internal/builder/main.go:150	Compiling
# 2023-10-27T10:00:00.000Z	INFO	internal/builder/main.go:180	Compiled successfully	{"binary": "./dist/custom-collector"}
```

## 3. Telemetrygen Tool

`telemetrygen` is a utility for generating telemetry data (traces, metrics, logs) to test the collector.

```bash
# Install telemetrygen
go install github.com/open-telemetry/opentelemetry-collector-contrib/cmd/telemetrygen@latest

# Generate traces (OTLP gRPC)
telemetrygen traces --otlp-endpoint localhost:4317 --otlp-insecure --traces 100

# Generate metrics (OTLP HTTP)
telemetrygen metrics --otlp-endpoint localhost:4318 --otlp-http --otlp-insecure --metrics 50

# Generate logs
telemetrygen logs --otlp-endpoint localhost:4317 --otlp-insecure --logs 200
```

## 4. Kubernetes Deployment (Helm & Operator)

### 4.1 Helm Chart Commands

The official Helm charts are the recommended way to deploy the collector in Kubernetes.

```bash
# Add the Helm repository
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm repo update

# Install the Collector (DaemonSet by default)
helm install my-otel-collector open-telemetry/opentelemetry-collector \
  --namespace observability --create-namespace \
  -f values.yaml

# Upgrade the Collector
helm upgrade my-otel-collector open-telemetry/opentelemetry-collector \
  --namespace observability \
  -f values.yaml

# Install the Operator
helm install my-otel-operator open-telemetry/opentelemetry-operator \
  --namespace observability --create-namespace \
  --set "admissionWebhooks.certManager.enabled=true"
```

#### `values.yaml` Structure (Collector Chart)

```yaml
mode: daemonset # Options: daemonset, deployment, statefulset

config:
  receivers:
    otlp:
      protocols:
        grpc:
          endpoint: 0.0.0.0:4317
        http:
          endpoint: 0.0.0.0:4318
  processors:
    batch: {}
    memory_limiter:
      check_interval: 1s
      limit_percentage: 80
      spike_limit_percentage: 20
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
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

### 4.2 OpenTelemetry Operator (CRDs)

The Operator manages `OpenTelemetryCollector` and `Instrumentation` Custom Resource Definitions (CRDs).

#### `OpenTelemetryCollector` CRD

```yaml
apiVersion: opentelemetry.io/v1alpha1
kind: OpenTelemetryCollector
metadata:
  name: my-collector
  namespace: observability
spec:
  mode: deployment # Options: deployment, daemonset, statefulset, sidecar
  replicas: 3
  config: |
    receivers:
      otlp:
        protocols:
          grpc:
            endpoint: 0.0.0.0:4317
    processors:
      batch: {}
    exporters:
      debug: {}
    service:
      pipelines:
        traces:
          receivers: [otlp]
          processors: [batch]
          exporters: [debug]
```

```bash
# Apply the CRD
kubectl apply -f otelcol-crd.yaml

# Check the status
kubectl get opentelemetrycollector -n observability
```

#### `Instrumentation` CRD (Auto-instrumentation)

```yaml
apiVersion: opentelemetry.io/v1alpha1
kind: Instrumentation
metadata:
  name: my-instrumentation
  namespace: default
spec:
  exporter:
    endpoint: http://my-collector-collector.observability.svc.cluster.local:4317
  propagators:
    - tracecontext
    - baggage
  sampler:
    type: parentbased_traceidratio
    argument: "0.25"
```

```bash
# Apply the CRD
kubectl apply -f instrumentation-crd.yaml

# Inject sidecar via annotation on a Pod/Deployment
kubectl patch deployment my-app -p '{"spec": {"template": {"metadata": {"annotations": {"sidecar.opentelemetry.io/inject": "true"}}}}}'
```

## 5. Docker Commands

Running the collector via Docker is common for local testing and development.

```bash
# Run the core collector
docker run -d --name otelcol \
  -p 4317:4317 -p 4318:4318 \
  -v $(pwd)/config.yaml:/etc/otelcol/config.yaml \
  otel/opentelemetry-collector:latest

# Run the contrib collector
docker run -d --name otelcol-contrib \
  -p 4317:4317 -p 4318:4318 \
  -v $(pwd)/config.yaml:/etc/otelcol-contrib/config.yaml \
  otel/opentelemetry-collector-contrib:latest
```

## 6. Systemd Service Management

When deployed on VMs or bare metal, the collector is often managed via systemd.

```ini
# /etc/systemd/system/otelcol.service
[Unit]
Description=OpenTelemetry Collector
After=network.target

[Service]
ExecStart=/usr/bin/otelcol --config=/etc/otelcol/config.yaml
KillMode=process
Restart=on-failure
RestartSec=5s
User=otelcol
Group=otelcol
EnvironmentFile=-/etc/default/otelcol

[Install]
WantedBy=multi-user.target
```

```bash
# Reload systemd
sudo systemctl daemon-reload

# Start the service
sudo systemctl start otelcol

# Enable on boot
sudo systemctl enable otelcol

# Check status
sudo systemctl status otelcol

# View logs
journalctl -u otelcol -f
```

## 7. Health Checks and Endpoints

The collector provides several endpoints for health monitoring and debugging.

### 7.1 Health Check Extension

Requires the `health_check` extension configured.

```yaml
extensions:
  health_check:
    endpoint: 0.0.0.0:13133
```

```bash
# Check health
curl http://localhost:13133/

# Expected Output:
# {"status":"Server available"}
```

### 7.2 zPages Extension

Requires the `zpages` extension configured. Useful for debugging pipelines.

```yaml
extensions:
  zpages:
    endpoint: 0.0.0.0:55679
```

```bash
# Access zPages (typically via browser, but curl works)
curl http://localhost:55679/debug/tracez
curl http://localhost:55679/debug/pipelinez
```

### 7.3 pprof Extension

Requires the `pprof` extension configured. Used for Go profiling.

```yaml
extensions:
  pprof:
    endpoint: 0.0.0.0:1777
```

```bash
# Collect a CPU profile
go tool pprof http://localhost:1777/debug/pprof/profile?seconds=30

# Collect a heap profile
go tool pprof http://localhost:1777/debug/pprof/heap
```

## 8. Environment Variables

The collector relies heavily on environment variables for configuration and memory management.

### 8.1 Memory Management (`GOMEMLIMIT`)

Go 1.19+ introduced `GOMEMLIMIT` to help prevent Out-Of-Memory (OOM) kills.

```bash
# Set GOMEMLIMIT to 80% of the container/system memory limit
export GOMEMLIMIT=400MiB
```

*Best Practice:* Always set `GOMEMLIMIT` in containerized environments. If your container limit is 512Mi, set `GOMEMLIMIT=400MiB` (approx 80%).

### 8.2 Configuration Variables (`OTEL_*`)

You can use environment variables within your `config.yaml` using the `${ENV_VAR}` syntax.

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: ${OTEL_RECEIVER_ENDPOINT}
```

```bash
export OTEL_RECEIVER_ENDPOINT="0.0.0.0:4317"
otelcol --config config.yaml
```

## 9. Troubleshooting Guide

### 9.1 Collector Fails to Start

*   **Symptom:** Process exits immediately.
*   **Action:** Run `otelcol validate --config config.yaml`.
*   **Common Causes:**
    *   Invalid YAML syntax (e.g., tabs instead of spaces).
    *   Missing required fields in a component.
    *   Component not defined in the `service.pipelines` section.
    *   Port conflicts (e.g., 4317 already in use).

### 9.2 Data is Dropped (OOM Kills)

*   **Symptom:** Collector restarts frequently; `dmesg` shows OOM killer.
*   **Action:**
    1.  Ensure `GOMEMLIMIT` is set to ~80% of the hard limit.
    2.  Configure the `memory_limiter` processor as the **first** processor in every pipeline.
    3.  Configure the `batch` processor after the `memory_limiter`.

```yaml
processors:
  memory_limiter:
    check_interval: 1s
    limit_percentage: 80
    spike_limit_percentage: 20
  batch:
    send_batch_size: 8192
    timeout: 1s
```

### 9.3 Data is Not Exported

*   **Symptom:** No data appears in the backend (e.g., Jaeger, Prometheus).
*   **Action:**
    1.  Enable the `debug` exporter with `verbosity: detailed`.
    2.  Check the collector logs for connection errors (e.g., `connection refused`, `context deadline exceeded`).
    3.  Verify network connectivity (firewalls, DNS).
    4.  Ensure the exporter is added to the pipeline in the `service` section.

### 9.4 High CPU Usage

*   **Symptom:** Collector consumes excessive CPU.
*   **Action:**
    1.  Enable the `pprof` extension.
    2.  Capture a CPU profile: `go tool pprof http://localhost:1777/debug/pprof/profile?seconds=30`.
    3.  Analyze the profile to identify the bottleneck (often complex regex in processors or high throughput without batching).

### 9.5 Kubernetes Specific Issues

*   **Symptom:** Sidecar not injected.
*   **Action:**
    1.  Verify the `OpenTelemetryCollector` CRD is deployed and healthy.
    2.  Check the Operator logs: `kubectl logs -n observability deployment/opentelemetry-operator`.
    3.  Ensure the Pod/Deployment has the correct annotation: `sidecar.opentelemetry.io/inject: "true"`.
    4.  Check for mutating webhook configuration issues.
