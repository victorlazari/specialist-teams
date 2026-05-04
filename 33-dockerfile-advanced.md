# Dockerfile Mastery — Advanced Patterns

> **Role:** Dockerfile Architect & Container Optimization Specialist
> **Domain:** BuildKit advanced features, CI/CD linting integration, vulnerability scanning, ephemeral containers
> **Official Documentation:** [Docker Build Best Practices](https://docs.docker.com/build/building/best-practices/)

---

## 1. Advanced BuildKit Features

BuildKit is the modern execution engine for Docker builds. Beyond parallel execution, it unlocks powerful advanced features that transform how Dockerfiles are structured.

### 1.1 Cache Mounts for Package Managers

Standard Docker layer caching is binary — if a `package.json` changes, the entire `npm ci` layer is invalidated, and all packages are downloaded again from the internet. BuildKit introduces cache mounts, which preserve package manager caches across builds, even when the Docker layer is invalidated.

```dockerfile
# syntax=docker/dockerfile:1
FROM node:22-alpine AS deps
WORKDIR /app
COPY package*.json ./
# The cache mount persists the npm cache directory between builds
RUN --mount=type=cache,target=/root/.npm \
    npm ci
```

When a dependency is added, the layer is invalidated, but `npm ci` will find the previously downloaded packages in the `/root/.npm` cache mount, drastically reducing network I/O and build time.

### 1.2 Bind Mounts for Source Code

Traditionally, source code is copied into the image using `COPY src/ ./` before a build step. This creates a permanent layer containing the raw source code. If the final image only needs the compiled binary, copying the source code creates unnecessary intermediate layers.

BuildKit allows bind-mounting the build context directly into a `RUN` instruction:

```dockerfile
# syntax=docker/dockerfile:1
FROM golang:1.21-alpine AS build
WORKDIR /app
# Mount the source code temporarily without creating a layer
RUN --mount=type=bind,target=. \
    --mount=type=cache,target=/root/.cache/go-build \
    go build -o /bin/server ./cmd/server
```

This pattern ensures the raw source code never exists as a layer in the image history, saving disk space and preventing source code leakage in intermediate builder images.

---

## 2. Dockerfile Linting and Code Review

A Dockerfile should be treated as production code, subject to static analysis, linting, and peer review. Integrating automated linting into the CI/CD pipeline prevents anti-patterns and security vulnerabilities from reaching production.

### 2.1 Hadolint Integration

[Hadolint](https://github.com/hadolint/hadolint) is the industry-standard Haskell-based Dockerfile linter. It parses the Dockerfile into an Abstract Syntax Tree (AST) and applies rules based on official Docker best practices.

Hadolint enforces rules such as:
- **DL3008:** Pin versions in `apt-get install` (prevents unrepeatable builds).
- **DL3009:** Pin versions in `apk add` (prevents unrepeatable builds).
- **DL3002:** Last `USER` should not be root (enforces non-root execution).
- **DL3020:** Use `COPY` instead of `ADD` (reduces attack surface).

Hadolint should be integrated into GitHub Actions or GitLab CI to fail pull requests that violate best practices:

```yaml
# GitHub Actions Example
name: Dockerfile Lint
on: [push, pull_request]
jobs:
  hadolint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Hadolint
        uses: hadolint/hadolint-action@v3.1.0
        with:
          dockerfile: Dockerfile
          failure-threshold: warning
```

### 2.2 Container Vulnerability Scanning

Linting analyzes the Dockerfile syntax, but vulnerability scanning analyzes the resulting image for known CVEs (Common Vulnerabilities and Exposures) in OS packages and application dependencies.

**Trivy** (by Aqua Security) and **Docker Scout** are the leading tools for this purpose. They generate a Software Bill of Materials (SBOM) and check it against vulnerability databases.

The CI pipeline should build the image, scan it with Trivy, and block the deployment if critical or high vulnerabilities are detected:

```yaml
# GitHub Actions Example with Trivy
      - name: Build image
        run: docker build -t my-app:${{ github.sha }} .
        
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'my-app:${{ github.sha }}'
          format: 'table'
          exit-code: '1'
          ignore-unfixed: true
          vuln-type: 'os,library'
          severity: 'CRITICAL,HIGH'
```

---

## 3. The Ephemeral Container Paradigm

The image defined by a Dockerfile should generate containers that are as ephemeral as possible [1]. Ephemeral means that the container can be stopped, destroyed, rebuilt, and replaced with an absolute minimum of setup and configuration [1].

### 3.1 Stateless Architecture

Containers must not store persistent state within their own filesystem. Any data that must survive a container restart (databases, user uploads, logs) must be written to external volumes, object storage (like S3), or external logging services. The Dockerfile should configure the application to output logs to `stdout` and `stderr` rather than local files, allowing the container runtime to handle log routing.

### 3.2 Single Concern Principle

Each container should have only one concern [1]. Decoupling applications into multiple containers makes it easier to scale horizontally and reuse containers [1]. 

A Dockerfile should not attempt to run a web server, a database, and a background worker simultaneously using process managers like `supervisord`. Instead, these should be split into three separate Dockerfiles (or three different entrypoints using the same image), managed via Docker Compose or Kubernetes.

### 3.3 Graceful Shutdown Handling

A robust Dockerfile must ensure the application handles termination signals correctly. When a container orchestrator (like Kubernetes) stops a container, it sends a `SIGTERM` signal. If the application does not exit within a grace period, it sends a `SIGKILL`, forcefully terminating the process and potentially corrupting data or dropping active requests.

The `CMD` instruction must use the **exec form** (`CMD ["node", "server.js"]`) rather than the **shell form** (`CMD node server.js`). The shell form wraps the command in `/bin/sh -c`, which does not pass signals to the underlying child process, preventing graceful shutdowns.

Furthermore, applications like Node.js do not handle `SIGTERM` automatically. Tools like `tini` or `dumb-init` should be used as the `ENTRYPOINT` to act as a lightweight init system, reaping zombie processes and properly forwarding signals to the application.

```dockerfile
RUN apk add --no-cache dumb-init
ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["node", "server.js"]
```

---

## References

[1] [Docker Docs: Building best practices](https://docs.docker.com/build/building/best-practices/)
[2] [Docker Docs: Multi-stage builds](https://docs.docker.com/build/building/multi-stage/)
[3] [Hadolint GitHub Repository](https://github.com/hadolint/hadolint)
