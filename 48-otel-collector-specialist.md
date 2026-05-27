# Specialist #48: OpenTelemetry Collector

## 1. Introduction and Architecture

The OpenTelemetry Collector is a vendor-agnostic telemetry pipeline written in Go. It is designed to receive, process, and export telemetry data (traces, metrics, logs, and profiles) from various sources to multiple backends. The architecture is highly modular and configurable, allowing organizations to build complex telemetry pipelines tailored to their specific needs.

### 1.1 Core Components

The Collector's architecture is built around several core components:

*   **Receivers:** These components are responsible for ingesting telemetry data into the Collector. They can accept data in various formats and protocols (e.g., OTLP, Prometheus, Jaeger, Zipkin).
*   **Processors:** Once data is received, processors can modify, filter, or aggregate it before it is exported. Common processors include batching, memory limiting, attribute modification, and sampling.
*   **Exporters:** These components send the processed telemetry data to one or more backend systems (e.g., Prometheus, Elasticsearch, Datadog, Jaeger).
*   **Connectors:** Connectors act as both an exporter and a receiver, bridging two pipelines. They allow data to flow from one pipeline to another, enabling complex routing and transformation scenarios.
*   **Extensions:** Extensions provide auxiliary capabilities that do not directly process telemetry data. Examples include health checks, profiling (pprof), and debugging pages (zpages).
*   **Service:** The service section of the configuration file ties everything together. It defines the pipelines (combinations of receivers, processors, and exporters) and enables the extensions.

### 1.2 Data Model

The Collector uses an internal data model called `pdata` (Protocol Data) to represent telemetry data. This model is based on the OpenTelemetry Protocol (OTLP) and provides a unified representation for traces, metrics, logs, and profiles. This unified model allows processors to operate on data regardless of its original format.

### 1.3 Component Lifecycle

The lifecycle of a component in the Collector is managed by the host environment. Components are created, started, and stopped in a specific order to ensure proper initialization and shutdown. The typical lifecycle is:

1.  **Creation:** The component is instantiated based on its configuration.
2.  **Start:** The component begins its operation (e.g., a receiver starts listening for data).
3.  **Shutdown:** The component gracefully stops its operation and releases resources.

### 1.4 Confmap System

The Collector uses a configuration system called `confmap` to manage its settings. This system supports multiple providers for loading configuration data:

*   **file:** Loads configuration from a local file.
*   **env:** Loads configuration from environment variables.
*   **yaml:** Loads configuration from a YAML string.
*   **http / https:** Loads configuration from a remote URL.

The `confmap` system also supports merging multiple configuration sources and resolving environment variables within the configuration.

## 2. Receivers

Receivers are the entry point for telemetry data into the Collector. The `opentelemetry-collector-contrib` repository provides a vast array of receivers for various systems and protocols.

### 2.1 Core Receivers

*   **otlp:** Receives data via gRPC or HTTP using the OpenTelemetry Protocol. This is the standard and recommended receiver for OTel data.
*   **nop:** A no-op receiver that discards all incoming data. Useful for testing and benchmarking.

### 2.2 Contrib Receivers (80+)

The contrib repository includes over 80 receivers. Here is a comprehensive list with brief descriptions:

1.  **active_directory_ds:** Collects metrics from Active Directory Domain Services.
2.  **aerospike:** Collects metrics from Aerospike databases.
3.  **apache:** Collects metrics from Apache HTTP servers.
4.  **apache_spark:** Collects metrics from Apache Spark clusters.
5.  **awscloudwatch:** Receives metrics from AWS CloudWatch.
6.  **awscloudwatchmetrics:** Receives metrics from AWS CloudWatch Metrics.
7.  **awsecscontainermetrics:** Collects metrics from AWS ECS containers.
8.  **awsfirehose:** Receives data from AWS Kinesis Data Firehose.
9.  **awsxray:** Receives traces from AWS X-Ray.
10. **azureblob:** Receives logs from Azure Blob Storage.
11. **azureeventhub:** Receives logs from Azure Event Hubs.
12. **azuremonitor:** Receives metrics from Azure Monitor.
13. **bigip:** Collects metrics from F5 BIG-IP devices.
14. **carbon:** Receives metrics in the Carbon format.
15. **cassandra:** Collects metrics from Cassandra databases.
16. **ceph:** Collects metrics from Ceph storage clusters.
17. **chrony:** Collects metrics from Chrony NTP servers.
18. **cisco_telemetry_mdt:** Receives telemetry from Cisco devices via MDT.
19. **cloudflare:** Receives logs from Cloudflare Logpush.
20. **collectd:** Receives metrics from collectd.
21. **couchdb:** Collects metrics from CouchDB databases.
22. **datadog:** Receives traces and metrics from Datadog agents.
23. **docker_stats:** Collects metrics from Docker containers.
24. **dotnet_diagnostics:** Collects metrics from .NET applications.
25. **elasticsearch:** Collects metrics from Elasticsearch clusters.
26. **envoy:** Receives metrics from Envoy proxies.
27. **expvar:** Collects metrics from Go expvar endpoints.
28. **f5cloud:** Receives telemetry from F5 Cloud services.
29. **filelog:** Tails and parses log files.
30. **fluentforward:** Receives logs via the Fluentd Forward protocol.
31. **github:** Receives webhooks from GitHub.
32. **gitlab:** Receives webhooks from GitLab.
33. **googlecloudmonitoring:** Receives metrics from Google Cloud Monitoring.
34. **googlecloudpubsub:** Receives messages from Google Cloud Pub/Sub.
35. **googlecloudspanner:** Collects metrics from Google Cloud Spanner.
36. **haproxy:** Collects metrics from HAProxy load balancers.
37. **hostmetrics:** Collects system-level metrics (CPU, memory, disk, network) from the host.
38. **httpcheck:** Performs HTTP health checks and generates metrics.
39. **iis:** Collects metrics from Microsoft IIS servers.
40. **influxdb:** Receives metrics in the InfluxDB Line Protocol.
41. **jaeger:** Receives traces in Jaeger formats (Thrift, gRPC).
42. **jmx:** Collects metrics from Java applications via JMX.
43. **journald:** Reads logs from systemd journald.
44. **k8s_cluster:** Collects cluster-level metrics from Kubernetes.
45. **k8s_events:** Collects Kubernetes events.
46. **k8s_objects:** Collects Kubernetes objects.
47. **kafka:** Receives telemetry data from Kafka topics.
48. **kafkametrics:** Collects metrics from Kafka brokers.
49. **kubeletstats:** Collects metrics from Kubernetes kubelets.
50. **lighttpd:** Collects metrics from Lighttpd servers.
51. **loki:** Receives logs in the Loki format.
52. **memcached:** Collects metrics from Memcached servers.
53. **mongodb:** Collects metrics from MongoDB databases.
54. **mongodbatlas:** Collects metrics from MongoDB Atlas.
55. **mysql:** Collects metrics from MySQL databases.
56. **nginx:** Collects metrics from NGINX servers.
57. **nsxt:** Collects metrics from VMware NSX-T.
58. **opencensus:** Receives telemetry in the OpenCensus format.
59. **opensearch:** Collects metrics from OpenSearch clusters.
60. **oracledb:** Collects metrics from Oracle databases.
61. **osquery:** Collects metrics from osquery.
62. **otlpjsonfile:** Reads OTLP JSON data from files.
63. **phpfpm:** Collects metrics from PHP-FPM pools.
64. **podman_stats:** Collects metrics from Podman containers.
65. **postgresql:** Collects metrics from PostgreSQL databases.
66. **prometheus:** Scrapes metrics from Prometheus endpoints.
67. **prometheus_exec:** Executes commands and parses Prometheus metrics from output.
68. **pulsar:** Receives telemetry from Apache Pulsar.
69. **purefa:** Collects metrics from Pure Storage FlashArray.
70. **purefb:** Collects metrics from Pure Storage FlashBlade.
71. **rabbitmq:** Collects metrics from RabbitMQ servers.
72. **receiver_creator:** Dynamically creates receivers based on discovered endpoints.
73. **redis:** Collects metrics from Redis servers.
74. **riak:** Collects metrics from Riak KV clusters.
75. **sapm:** Receives traces in the SAPM format (Splunk).
76. **signalfx:** Receives metrics in the SignalFx format.
77. **skywalking:** Receives traces in the SkyWalking format.
78. **snmp:** Collects metrics via SNMP.
79. **solr:** Collects metrics from Apache Solr.
80. **splunk_hec:** Receives telemetry via Splunk HEC.
81. **sqlquery:** Executes SQL queries and generates metrics.
82. **sqlserver:** Collects metrics from Microsoft SQL Server.
83. **statsd:** Receives metrics in the StatsD format.
84. **syslog:** Receives logs via Syslog.
85. **tcplog:** Receives logs via TCP.
86. **tomcat:** Collects metrics from Apache Tomcat.
87. **udplog:** Receives logs via UDP.
88. **vcenter:** Collects metrics from VMware vCenter.
89. **wavefront:** Receives metrics in the Wavefront format.
90. **windowseventlog:** Reads logs from Windows Event Logs.
91. **windowsperfcounters:** Collects metrics from Windows Performance Counters.
92. **zookeeper:** Collects metrics from Apache ZooKeeper.
93. **zipkin:** Receives traces in the Zipkin format.

## 3. Processors

Processors modify, filter, or aggregate telemetry data before it is exported. They are crucial for shaping the data to meet the requirements of backend systems and for managing the volume of data.

### 3.1 Core Processors

*   **batch:** Groups telemetry data into batches before sending it to exporters. This improves export efficiency and reduces network overhead. Key configurations include `timeout`, `send_batch_size`, and `send_batch_max_size`.
*   **memory_limiter:** Prevents the Collector from running out of memory (OOM) by dropping data when memory usage exceeds configured thresholds. Key configurations include `check_interval`, `limit_mib` (hard limit), and `spike_limit_mib` (soft limit).

### 3.2 Contrib Processors (33+)

The contrib repository provides a wide range of processors for various transformations and filtering tasks:

1.  **attributes:** Modifies, inserts, deletes, or replaces attributes on spans and logs.
2.  **cumulativetodelta:** Converts cumulative metrics to delta metrics.
3.  **deltatorate:** Converts delta metrics to rate metrics.
4.  **filter:** Drops telemetry data based on OTTL conditions.
5.  **groupbyattrs:** Groups spans, logs, or datapoints by specific attributes.
6.  **groupbytrace:** Groups spans by trace ID, ensuring all spans for a trace are processed together.
7.  **k8sattributes:** Enriches telemetry data with Kubernetes metadata (e.g., pod name, namespace, node name). Requires RBAC permissions.
8.  **logstransform:** Modifies log records using OTTL.
9.  **metricsgeneration:** Generates new metrics based on existing metrics.
10. **metricstransform:** Renames metrics, renames labels, and applies regex transformations.
11. **probabilisticsampler:** Samples traces based on a configured probability.
12. **redaction:** Redacts sensitive information from attributes and log bodies.
13. **resource:** Modifies, inserts, deletes, or replaces resource attributes.
14. **resourcedetection:** Automatically detects and adds resource attributes based on the environment (e.g., AWS EC2, GCP, Azure, system).
15. **routing:** Routes telemetry data to different exporters based on OTTL conditions. (Note: Often implemented as a connector now).
16. **span:** Modifies span names and statuses using OTTL.
17. **spanmetrics:** Generates metrics (e.g., request count, error rate, latency) from spans. (Note: Often implemented as a connector now).
18. **tailsampling:** Samples traces based on complex policies evaluated after the entire trace is received. Requires the `groupbytrace` processor or a stateful deployment.
19. **transform:** A powerful processor that uses OTTL to modify telemetry data across all signals (traces, metrics, logs).

*(Note: The list above highlights the most commonly used processors. The contrib repository contains additional specialized processors.)*

## 4. Exporters

Exporters send processed telemetry data to backend systems. The Collector supports a vast array of backends through its contrib repository.

### 4.1 Core Exporters

*   **otlp:** Sends data via gRPC using the OpenTelemetry Protocol. Requires TLS by default. Supports queued retry.
*   **otlphttp:** Sends data via HTTP using the OpenTelemetry Protocol (`/v1/traces`, `/v1/metrics`, `/v1/logs`).
*   **debug:** Logs telemetry data to the console for debugging purposes.
*   **nop:** A no-op exporter that discards all data.

### 4.2 Contrib Exporters (48+)

The contrib repository includes over 48 exporters. Here is a comprehensive list:

1.  **alibabacloud_logservice:** Exports logs to Alibaba Cloud Log Service.
2.  **awscloudwatchlogs:** Exports logs to AWS CloudWatch Logs.
3.  **awsemf:** Exports metrics in AWS Embedded Metric Format (EMF).
4.  **awskinesis:** Exports data to AWS Kinesis Data Streams.
5.  **awss3:** Exports data to AWS S3 buckets.
6.  **awsxray:** Exports traces to AWS X-Ray.
7.  **azuredataexplorer:** Exports data to Azure Data Explorer.
8.  **azuremonitor:** Exports telemetry to Azure Monitor.
9.  **carbon:** Exports metrics in the Carbon format.
10. **cassandra:** Exports data to Cassandra databases.
11. **clickhouse:** Exports data to ClickHouse databases.
12. **coralogix:** Exports telemetry to Coralogix.
13. **datadog:** Exports telemetry to Datadog.
14. **dataset:** Exports telemetry to DataSet (formerly Scalyr).
15. **dynatrace:** Exports telemetry to Dynatrace.
16. **elasticsearch:** Exports logs and traces to Elasticsearch.
17. **f5cloud:** Exports telemetry to F5 Cloud services.
18. **file:** Exports telemetry to local files.
19. **googlecloud:** Exports telemetry to Google Cloud Operations Suite (formerly Stackdriver).
20. **googlecloudpubsub:** Exports data to Google Cloud Pub/Sub.
21. **googlemanagedprometheus:** Exports metrics to Google Managed Service for Prometheus.
22. **honeycomb:** Exports telemetry to Honeycomb.
23. **influxdb:** Exports metrics to InfluxDB.
24. **instana:** Exports telemetry to Instana.
25. **jaeger:** Exports traces to Jaeger.
26. **kafka:** Exports telemetry to Kafka topics.
27. **loadbalancing:** Routes telemetry to multiple backends based on consistent hashing (useful for stateful components like tail sampling).
28. **logdna:** Exports logs to LogDNA (Mezmo).
29. **logicmonitor:** Exports telemetry to LogicMonitor.
30. **logzio:** Exports telemetry to Logz.io.
31. **loki:** Exports logs to Grafana Loki.
32. **mezmo:** Exports logs to Mezmo.
33. **newrelic:** Exports telemetry to New Relic.
34. **opencensus:** Exports telemetry in the OpenCensus format.
35. **opensearch:** Exports logs and traces to OpenSearch.
36. **prometheus:** Exposes a Prometheus scrape endpoint.
37. **prometheusremotewrite:** Exports metrics via Prometheus Remote Write.
38. **pulsar:** Exports telemetry to Apache Pulsar.
39. **rabbitmq:** Exports telemetry to RabbitMQ.
40. **sentry:** Exports errors and traces to Sentry.
41. **signalfx:** Exports metrics and traces to SignalFx (Splunk).
42. **skywalking:** Exports traces to SkyWalking.
43. **splunk_hec:** Exports telemetry to Splunk via HTTP Event Collector (HEC).
44. **sumologic:** Exports telemetry to Sumo Logic.
45. **syslog:** Exports logs via Syslog.
46. **tanzuobservability:** Exports telemetry to Tanzu Observability (Wavefront).
47. **tencentcloud_logservice:** Exports logs to Tencent Cloud Log Service.
48. **zipkin:** Exports traces to Zipkin.

## 5. Connectors

Connectors bridge two pipelines by acting as an exporter in one pipeline and a receiver in another. They enable complex routing, transformation, and aggregation scenarios.

### 5.1 Key Connectors (14+)

1.  **count:** Counts telemetry items (spans, metrics, logs) and generates metrics representing the counts.
2.  **failover:** Routes data to a primary pipeline and falls back to a secondary pipeline if the primary fails.
3.  **forward:** Forwards data from one pipeline to another without modification.
4.  **roundrobin:** Distributes data across multiple pipelines in a round-robin fashion.
5.  **routing:** Routes data to different pipelines based on OTTL conditions.
6.  **servicegraph:** Generates service graph metrics (edges and nodes) from traces.
7.  **spanmetrics:** Generates metrics (request count, error rate, latency) from spans.

## 6. Extensions

Extensions provide auxiliary capabilities that do not directly process telemetry data. They are configured in the `extensions` section and enabled in the `service` section.

### 6.1 Key Extensions (30+)

1.  **basicauth:** Provides basic authentication for receivers and exporters.
2.  **bearertokenauth:** Provides bearer token authentication.
3.  **health_check:** Exposes an HTTP endpoint for health checks (useful for Kubernetes liveness/readiness probes).
4.  **k8s_leader_elector:** Enables leader election for Collector instances in Kubernetes (useful for stateful components).
5.  **memory_limiter:** (New) Replaces the memory_limiter processor. Works at the receiver level via middlewares to prevent OOM.
6.  **oauth2client:** Provides OAuth2 client authentication.
7.  **oidc:** Provides OpenID Connect authentication.
8.  **opamp:** Enables remote management of the Collector via the OpAMP protocol.
9.  **pprof:** Exposes Go pprof endpoints for profiling and debugging.
10. **storage:** Provides persistent storage for queues (e.g., `filestorage`, `dbstorage`).
11. **zpages:** Exposes debugging pages (e.g., `/debug/tracez`) for analyzing latency, deadlocks, and errors.

## 7. Pipeline Configuration Patterns

A pipeline is defined in the `service.pipelines` section of the configuration. It consists of a type (traces, metrics, logs), receivers, processors, and exporters.

### 7.1 Basic Pipeline

```yaml
service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [otlp]
```

### 7.2 Complex Routing Pipeline (using Connectors)

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317

processors:
  memory_limiter:
    check_interval: 1s
    limit_mib: 1000
    spike_limit_mib: 200
  batch:
    send_batch_size: 10000
    timeout: 1s

exporters:
  otlp/jaeger:
    endpoint: jaeger:4317
    tls:
      insecure: true
  prometheus:
    endpoint: 0.0.0.0:8889

connectors:
  spanmetrics:
    histogram:
      explicit:
        buckets: [2ms, 4ms, 8ms, 16ms, 32ms, 64ms, 128ms, 256ms, 512ms, 1s, 2s, 4s, 8s]

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [spanmetrics, otlp/jaeger] # Route traces to spanmetrics connector and Jaeger
    metrics/spanmetrics:
      receivers: [spanmetrics] # Receive metrics from spanmetrics connector
      processors: [batch]
      exporters: [prometheus] # Export generated metrics to Prometheus
```

## 8. Deployment Modes

The Collector can be deployed in various modes depending on the requirements and environment.

### 8.1 Agent vs. Gateway

*   **Agent:** Deployed on the same host as the application (e.g., as a sidecar or DaemonSet). It collects telemetry locally, performs minimal processing (e.g., batching, adding host metadata), and forwards it to a Gateway.
*   **Gateway:** Deployed as a standalone cluster (e.g., Deployment or StatefulSet). It receives data from Agents, performs heavy processing (e.g., tail sampling, metric generation), and exports it to backend systems.

### 8.2 Kubernetes Deployment Modes (via Operator)

The OpenTelemetry Operator simplifies deployment in Kubernetes and supports several modes:

*   **Deployment (Default):** A standard replicated deployment. Suitable for stateless processing and routing.
*   **DaemonSet:** Deploys one Collector pod per node. Ideal for collecting host metrics and logs (Agent mode).
*   **StatefulSet:** Deploys a stateful cluster. Required for stateful components like tail sampling and span-to-metrics generation, often used in conjunction with the `loadbalancing` exporter.
*   **Sidecar:** Injects a Collector container into application pods via the annotation `sidecar.opentelemetry.io/inject: "true"`.
*   **Target Allocator:** A specialized component that shards Prometheus scrape targets across a StatefulSet of Collectors.

## 9. Distributions

The Collector is distributed in several pre-built binaries:

*   **otelcol (Core):** Contains only the core components. Minimal footprint.
*   **otelcol-contrib:** Contains all components from the core and contrib repositories. Large footprint but highly versatile.
*   **otelcol-k8s:** Tailored for Kubernetes environments, containing components relevant to K8s observability.
*   **otelcol-otlp:** Contains only OTLP receivers and exporters.
*   **Custom (via OCB):** You can build a custom distribution using the OpenTelemetry Collector Builder (OCB) to include only the specific components you need, reducing the attack surface and binary size.

## 10. OpenTelemetry Transformation Language (OTTL)

OTTL is a domain-specific language used for transforming telemetry data within processors (e.g., `transform`, `filter`, `routing`).

### 10.1 Syntax and Contexts

The syntax is generally `function(path) where condition`. OTTL operates within specific contexts (e.g., `resource`, `scope`, `span`, `spanevent`, `metric`, `datapoint`, `log`).

### 10.2 Common Functions

*   `set(target, value)`: Sets a value.
*   `delete_key(target, key)`: Deletes a key from a map.
*   `replace_match(target, pattern, replacement)`: Replaces matching strings.
*   `IsMatch(target, pattern)`: Checks if a string matches a regex.

### 10.3 Example (Transform Processor)

```yaml
processors:
  transform:
    error_mode: ignore
    trace_statements:
      - context: span
        statements:
          - set(status.code, 1) where attributes["http.status_code"] >= 500
          - replace_match(name, "/api/v1/users/.*", "/api/v1/users/{id}")
```

## 11. Production Configuration Examples

### 11.1 Prometheus and Jaeger

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
    limit_mib: 1000
    spike_limit_mib: 200
  batch:
    send_batch_size: 10000
    timeout: 1s

exporters:
  prometheus:
    endpoint: 0.0.0.0:8889
  otlp/jaeger:
    endpoint: jaeger:4317
    tls:
      insecure: true

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [otlp/jaeger]
    metrics:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [prometheus]
```

### 11.2 Elasticsearch (Logs)

```yaml
receivers:
  filelog:
    include: [ /var/log/*.log ]
    start_at: end

processors:
  memory_limiter:
    check_interval: 1s
    limit_mib: 1000
    spike_limit_mib: 200
  batch:
    send_batch_size: 10000
    timeout: 1s

exporters:
  elasticsearch:
    endpoints: [ "http://elasticsearch:9200" ]
    index: "logs-%Y.%m.%d"

service:
  pipelines:
    logs:
      receivers: [filelog]
      processors: [memory_limiter, batch]
      exporters: [elasticsearch]
```

### 11.3 Datadog

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317

processors:
  memory_limiter:
    check_interval: 1s
    limit_mib: 1000
    spike_limit_mib: 200
  batch:
    send_batch_size: 10000
    timeout: 1s
  resourcedetection:
    detectors: [system, env]
    timeout: 2s
    override: false

exporters:
  datadog:
    api:
      key: YOUR_DATADOG_API_KEY
    site: datadoghq.com

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, resourcedetection, batch]
      exporters: [datadog]
    metrics:
      receivers: [otlp]
      processors: [memory_limiter, resourcedetection, batch]
      exporters: [datadog]
```

### 11.4 Grafana Stack (Tempo, Loki, Mimir)

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317

processors:
  memory_limiter:
    check_interval: 1s
    limit_mib: 1000
    spike_limit_mib: 200
  batch:
    send_batch_size: 10000
    timeout: 1s

exporters:
  otlp/tempo:
    endpoint: tempo:4317
    tls:
      insecure: true
  loki:
    endpoint: http://loki:3100/loki/api/v1/push
  prometheusremotewrite/mimir:
    endpoint: http://mimir:9009/api/v1/push

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [otlp/tempo]
    metrics:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [prometheusremotewrite/mimir]
    logs:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [loki]
```

## 12. Troubleshooting and Operations

Operating the Collector in production requires a solid understanding of its internal metrics and troubleshooting tools.

### 12.1 Key Metrics for Scaling

Monitor these metrics to determine when to scale the Collector:

*   `otelcol_processor_refused_spans`: Indicates data is being dropped by processors (e.g., memory limiter).
*   `otelcol_exporter_queue_size` / `otelcol_exporter_queue_capacity`: Monitor the queue size. Scale up when the queue reaches 60-70% capacity.
*   `otelcol_exporter_enqueue_failed_spans`: Indicates the queue is full and data is being dropped.
*   `otelcol_exporter_send_failed_spans`: Indicates the exporter failed to send data to the backend (e.g., network issue, backend down).

**Important:** Do not scale the Collector if the backend cannot keep up. This will only exacerbate the problem.

### 12.2 Troubleshooting Tools

*   **Internal Telemetry:** The Collector exposes its own metrics (usually on port 8888). Scrape these metrics to monitor its health.
*   **Debug Exporter:** Use the `debug` exporter with `verbosity: detailed` to print the full payload to the console. This is invaluable for verifying data transformations.
*   **zpages Extension:** Enable the `zpages` extension (port 55679) and access `/debug/tracez` to analyze latency, deadlocks, and errors within the Collector's internal components.
*   **pprof Extension:** Enable the `pprof` extension (port 1777) for Go profiling (CPU, memory) to diagnose performance bottlenecks.
*   **`otelcol components` Command:** Run this command to list all available components and their stability levels in the current binary.

### 12.3 Common Issues and Solutions

*   **Dropping Data:**
    *   *Cause:* Improper sizing, destination unavailable, or memory limits reached.
    *   *Solution:* Configure `queued_retry` and `sending_queue` on exporters. Increase memory limits or scale out the Collector. Check backend health.
*   **Not Receiving Data:**
    *   *Cause:* Network issues, incorrect receiver configuration (e.g., wrong port), or the receiver is not included in a pipeline.
    *   *Solution:* Verify network connectivity. Check the receiver configuration and ensure it's listed in `service.pipelines.<type>.receivers`.
*   **Not Processing Data:**
    *   *Cause:* Misunderstanding processor scope (e.g., trying to modify a resource attribute with the `attributes` processor instead of the `resource` processor).
    *   *Solution:* Review OTTL syntax and ensure the correct processor is used for the target context. Use the `debug` exporter to verify transformations.
*   **Not Exporting Data:**
    *   *Cause:* Network issues, incorrect exporter configuration (e.g., wrong endpoint, missing authentication), destination unavailable, or the exporter is not in a pipeline.
    *   *Solution:* Verify network connectivity and backend health. Check exporter configuration (especially TLS and authentication settings). Ensure it's listed in `service.pipelines.<type>.exporters`.
*   **Configuration Issues (YAML Gotchas):**
    *   *Cause:* Null maps or incorrect indentation in YAML.
    *   *Solution:* Validate the YAML configuration file. The Collector will fail to start if the configuration is invalid.

### 12.4 Memory Management Best Practices

*   Set the `GOMEMLIMIT` environment variable to 80% of the container's hard memory limit.
*   Place the `memory_limiter` processor as the **FIRST** processor in every pipeline.
*   Set `spike_limit_mib` to 20% of the hard limit.
*   In containerized environments, use `limit_percentage` instead of absolute values for easier scaling.
*   Set `check_interval` to `1s`.

## 13. Security Best Practices

*   **Never run as root:** Run the Collector as a non-root user.
*   **Default to encrypted connections:** Ensure `insecure: false` (the default) is used for gRPC connections unless explicitly required (e.g., local testing).
*   **Use `configopaque.String`:** When developing custom components, use `configopaque.String` for sensitive fields (like API keys) to prevent them from being logged or exposed in zpages.
*   **Minimize privileged access:** Only grant the Collector the permissions it needs (e.g., RBAC for Kubernetes metadata enrichment).
*   **Don't expose health/telemetry externally:** Bind health check and internal telemetry endpoints to `localhost` or internal networks, not public interfaces.
