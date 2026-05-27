# OpenTelemetry Collector Security Audit & Hardening Guide

## 1. Introduction

The OpenTelemetry Collector is a critical component in the observability pipeline, acting as a central hub for receiving, processing, and exporting telemetry data (traces, metrics, logs). Given its position, it often handles sensitive data, credentials, and network traffic from various sources. Securing the Collector is paramount to prevent data breaches, unauthorized access, and denial-of-service attacks.

This comprehensive guide covers security hardening practices for the OpenTelemetry Collector, focusing on a tech support operations team perspective. It includes detailed configurations, real-world examples, troubleshooting steps, and edge cases for production environments.

## 2. Core Security Principles

### 2.1. Never Run as Root

Running the Collector as root exposes the host system to significant risks if the Collector is compromised. Always run the Collector as a non-root user.

**Docker Example:**

```dockerfile
FROM otel/opentelemetry-collector-contrib:0.90.0

# Create a non-root user
RUN addgroup -S otel && adduser -S otel -G otel

# Set ownership of the config file
COPY config.yaml /etc/otelcol-contrib/config.yaml
RUN chown otel:otel /etc/otelcol-contrib/config.yaml

# Switch to the non-root user
USER otel

ENTRYPOINT ["/otelcol-contrib"]
CMD ["--config", "/etc/otelcol-contrib/config.yaml"]
```

**Kubernetes Security Context:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: otel-collector
spec:
  template:
    spec:
      securityContext:
        runAsUser: 10001
        runAsGroup: 10001
        fsGroup: 10001
        runAsNonRoot: true
      containers:
        - name: otel-collector
          image: otel/opentelemetry-collector-contrib:0.90.0
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL
```

**Troubleshooting:**
- **Error:** `listen tcp :443: bind: permission denied`
- **Cause:** Non-root users cannot bind to privileged ports (< 1024).
- **Solution:** Use ports > 1024 (e.g., 8443) or configure capabilities (`CAP_NET_BIND_SERVICE`) if absolutely necessary, though avoiding privileged ports is preferred.

### 2.2. TLS/mTLS Configuration

All communication to and from the Collector should be encrypted using TLS. Mutual TLS (mTLS) should be used where possible to authenticate both the client and the server.

**Server-Side TLS (Receivers):**

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
        tls:
          cert_file: /etc/certs/server.crt
          key_file: /etc/certs/server.key
          client_ca_file: /etc/certs/ca.crt # Required for mTLS
      http:
        endpoint: 0.0.0.0:4318
        tls:
          cert_file: /etc/certs/server.crt
          key_file: /etc/certs/server.key
```

**Client-Side TLS (Exporters):**

```yaml
exporters:
  otlp:
    endpoint: YOUR_ENDPOINT:4317
    tls:
      insecure: false
      ca_file: /etc/certs/ca.crt
      cert_file: /etc/certs/client.crt # Required for mTLS
      key_file: /etc/certs/client.key  # Required for mTLS
```

**Troubleshooting:**
- **Error:** `tls: first record does not look like a TLS handshake`
- **Cause:** A client is trying to connect via plain text to a TLS-enabled port.
- **Solution:** Ensure the client is configured to use TLS.
- **Error:** `x509: certificate signed by unknown authority`
- **Cause:** The client does not trust the server's certificate.
- **Solution:** Provide the correct CA certificate (`ca_file`) to the client.

### 2.3. Configopaque for Sensitive Fields

The `configopaque` type should be used for sensitive fields in the configuration to prevent them from being logged or exposed in memory dumps.

```yaml
extensions:
  basicauth/client:
    client_auth:
      username: myuser
      password: ${env:MY_PASSWORD} # Use environment variables for secrets
```

*Note: While `configopaque` is an internal Go type used by the Collector to mask secrets, operators should ensure secrets are passed via environment variables or secret management systems rather than hardcoded in the YAML.*

## 3. Authentication Extensions

The Collector supports various authentication mechanisms via extensions.

### 3.1. Basic Authentication

**Server-Side (Receiver):**

```yaml
extensions:
  basicauth/server:
    htpasswd:
      file: /etc/otel/htpasswd

receivers:
  otlp:
    protocols:
      http:
        endpoint: 0.0.0.0:4318
        auth:
          authenticator: basicauth/server

service:
  extensions: [basicauth/server]
```

**Client-Side (Exporter):**

```yaml
extensions:
  basicauth/client:
    client_auth:
      username: myuser
      password: ${env:MY_PASSWORD}

exporters:
  otlphttp:
    endpoint: https://YOUR_ENDPOINT:4318
    auth:
      authenticator: basicauth/client

service:
  extensions: [basicauth/client]
```

### 3.2. Bearer Token Authentication

**Server-Side:**

```yaml
extensions:
  bearertokenauth/server:
    scheme: Bearer
    filename: /etc/otel/token

receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
        auth:
          authenticator: bearertokenauth/server
```

**Client-Side:**

```yaml
extensions:
  bearertokenauth/client:
    scheme: Bearer
    token: ${env:MY_BEARER_TOKEN}

exporters:
  otlp:
    endpoint: YOUR_ENDPOINT:4317
    auth:
      authenticator: bearertokenauth/client
```

### 3.3. OIDC Authentication

```yaml
extensions:
  oidc:
    issuer_url: https://YOUR_OIDC_ISSUER
    audience: YOUR_AUDIENCE

receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
        auth:
          authenticator: oidc
```

**Troubleshooting Authentication:**
- **Error:** `rpc error: code = Unauthenticated desc = unauthorized`
- **Cause:** The client provided invalid or missing credentials.
- **Solution:** Verify the credentials and ensure the correct authentication extension is configured on both sides.

## 4. Kubernetes Security

### 4.1. RBAC for Kubernetes Components

Components like `k8sattributes` processor and `k8scluster` receiver require access to the Kubernetes API. Grant only the minimum necessary permissions.

**ClusterRole Example:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: otel-collector
rules:
  - apiGroups: [""]
    resources: ["pods", "namespaces", "nodes", "endpoints", "services"]
    verbs: ["get", "watch", "list"]
  - apiGroups: ["apps"]
    resources: ["replicasets", "deployments", "daemonsets", "statefulsets"]
    verbs: ["get", "watch", "list"]
```

### 4.2. Network Policies

Restrict traffic to and from the Collector pods using Kubernetes Network Policies.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: otel-collector-policy
spec:
  podSelector:
    matchLabels:
      app: otel-collector
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: my-app-namespace
      ports:
        - protocol: TCP
          port: 4317
        - protocol: TCP
          port: 4318
  egress:
    - to:
        - ipBlock:
            cidr: YOUR_BACKEND_IP/32
      ports:
        - protocol: TCP
          port: 443
```

## 5. Data Redaction and PII Removal

Telemetry data often contains Personally Identifiable Information (PII) or sensitive data. Use processors to redact this information before exporting.

### 5.1. Redaction Processor

```yaml
processors:
  redaction:
    allow_all_keys: false
    allowed_keys:
      - http.method
      - http.status_code
    blocked_values:
      - "4[0-9]{12}(?:[0-9]{3})?" # Visa credit card regex
```

### 5.2. Attributes Processor

```yaml
processors:
  attributes/pii:
    actions:
      - key: user.email
        action: delete
      - key: http.request.header.authorization
        action: hash
```

**Troubleshooting:**
- **Issue:** Sensitive data is still being exported.
- **Cause:** The processor is not included in the pipeline, or the regex/key matching is incorrect.
- **Solution:** Verify the pipeline configuration and test the regex patterns. Use the `debug` exporter to inspect the data before it leaves the Collector.

## 6. Access Control for Endpoints

The Collector exposes several endpoints for health checks, profiling, and debugging. These should not be exposed externally.

### 6.1. Health Check Extension

```yaml
extensions:
  health_check:
    endpoint: localhost:13133 # Bind to localhost only
```

### 6.2. pprof Extension

```yaml
extensions:
  pprof:
    endpoint: localhost:1777 # Bind to localhost only
```

### 6.3. zPages Extension

```yaml
extensions:
  zpages:
    endpoint: localhost:55679 # Bind to localhost only
```

**Troubleshooting:**
- **Issue:** Endpoints are accessible from outside the pod/host.
- **Cause:** The endpoint is bound to `0.0.0.0`.
- **Solution:** Change the endpoint to `localhost:<port>` or `127.0.0.1:<port>`.

## 7. Proxy Support

If the Collector needs to communicate through a proxy, configure the standard environment variables.

```bash
export HTTP_PROXY="http://YOUR_PROXY:8080"
export HTTPS_PROXY="http://YOUR_PROXY:8080"
export NO_PROXY="localhost,127.0.0.1,.svc.cluster.local"
```

**Troubleshooting:**
- **Error:** `context deadline exceeded` or connection timeouts.
- **Cause:** The proxy is blocking the connection or is misconfigured.
- **Solution:** Verify the proxy settings and ensure the proxy allows traffic to the destination endpoint.

## 8. Supply Chain Security

### 8.1. Signed Images

Always use signed images from trusted registries. Verify the signatures before deployment.

### 8.2. SBOM (Software Bill of Materials)

Maintain an SBOM for the Collector image to track vulnerabilities in dependencies. Use tools like `syft` or `trivy` to generate and scan the SBOM.

## 9. Audit Logging

Enable audit logging to track configuration changes and access to the Collector.

```yaml
service:
  telemetry:
    logs:
      level: info
      development: false
      encoding: json
```

Monitor the Collector's logs for unauthorized access attempts or configuration errors.

## 10. Conclusion

Securing the OpenTelemetry Collector requires a multi-layered approach, encompassing network security, authentication, data redaction, and secure deployment practices. By following this guide, operations teams can ensure the Collector operates securely and reliably in production environments.
