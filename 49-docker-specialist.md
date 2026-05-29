# Specialist #49: Docker Super Specialist

## Introduction

Welcome to the comprehensive guide for the Docker Super Specialist. This document serves as the ultimate reference for tech support operations teams, DevOps engineers, and system administrators tasked with optimizing, troubleshooting, and securing Docker environments. As a Docker Super Specialist, your role is not merely to write functional Dockerfiles or basic `compose.yaml` files, but to engineer robust, flaw-proof, cost-effective, and highly optimized containerized architectures. This guide covers everything from the foundational architecture of the Docker Engine to advanced BuildKit features, comprehensive Docker Compose V2 specifications, and production-ready deployment strategies.

## 1. Docker Engine Architecture Deep Dive

To effectively troubleshoot and optimize Docker, one must first understand its underlying architecture. Docker is not a monolithic application; it is a complex ecosystem of interacting components designed to manage the lifecycle of containers.

### The Docker Daemon (dockerd)

The Docker daemon (`dockerd`) is the persistent background process that manages Docker objects such as images, containers, networks, and volumes. It listens for Docker API requests and processes them. The daemon can also communicate with other daemons to manage swarm services.

### containerd

`containerd` is an industry-standard core container runtime. Originally part of the Docker daemon, it was extracted and donated to the Cloud Native Computing Foundation (CNCF). `containerd` manages the complete container lifecycle of its host system, from image transfer and storage to container execution and supervision, to low-level storage and network attachments. It acts as a bridge between the Docker daemon and the lower-level runtime (`runc`).

### runc and the OCI Specification

`runc` is a lightweight, portable container runtime that implements the Open Container Initiative (OCI) specification. It is responsible for actually creating and running containers. When `containerd` needs to start a container, it uses `runc` to interface with the Linux kernel features (namespaces, cgroups, SELinux, AppArmor) that provide container isolation. The OCI specification ensures that container images and runtimes are standardized, allowing images built with Docker to run on any OCI-compliant runtime (like Podman or CRI-O).

### Image Layers and the Union Filesystem

Docker images are built from a series of layers. Each layer represents an instruction in the image's Dockerfile. These layers are stacked on top of each other to form the final image. Docker uses a Union Filesystem (such as OverlayFS, specifically the `overlay2` storage driver) to combine these layers into a single unified view.

When a container is created from an image, Docker adds a thin, writable "container layer" on top of the underlying image layers. All changes made to the running container (such as writing new files, modifying existing files, or deleting files) are written to this thin writable container layer. The underlying image layers remain read-only. This architecture is crucial for efficiency, as multiple containers can share the same underlying image layers, saving disk space and memory.

## 2. Dockerfile Mastery: Complete Instruction Reference

A Dockerfile is the blueprint for a Docker image. Mastering its instructions is essential for creating secure, efficient, and maintainable images. Below is a comprehensive reference of Dockerfile instructions, including best and worst practices.

### FROM

The `FROM` instruction initializes a new build stage and sets the Base Image for subsequent instructions. It must be the first non-comment instruction in the Dockerfile.

**Best Practices:**
- Always pin versions (e.g., `FROM node:18.17.0-alpine` instead of `FROM node:latest`).
- Use minimal base images like Alpine, Distroless, or Scratch to reduce attack surface and image size.
- Use multi-stage builds by naming stages (e.g., `FROM golang:1.20 AS builder`).

**Worst Practices:**
- Using `latest` tags, which leads to unpredictable builds and potential breaking changes.
- Using full OS images (like `ubuntu` or `debian`) when a minimal image would suffice.

### RUN

The `RUN` instruction executes commands in a new layer on top of the current image and commits the results.

**Best Practices:**
- Consolidate multiple `RUN` commands using `&&` to reduce the number of layers.
- Clean up package manager caches in the same `RUN` instruction to prevent cache files from being committed to the layer.
- Use `--no-install-recommends` with `apt-get` to avoid installing unnecessary dependencies.

**Example (Good):**
```dockerfile
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*
```

**Example (Bad):**
```dockerfile
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y ca-certificates
```

### COPY and ADD

Both `COPY` and `ADD` copy files from the host to the container. However, `ADD` has additional features: it can extract local tar files and download files from remote URLs.

**Best Practices:**
- Prefer `COPY` over `ADD` unless you specifically need `ADD`'s extraction capabilities. `COPY` is more transparent.
- Order `COPY` instructions carefully. Copy dependency files (like `package.json` or `requirements.txt`) before copying the rest of the source code to leverage Docker's layer caching.

**Worst Practices:**
- Using `ADD` to download remote files. It's better to use `RUN curl` or `wget` and clean up the downloaded archive in the same layer.
- Copying the entire directory (`COPY . .`) before installing dependencies, which invalidates the cache every time any file changes.

### CMD and ENTRYPOINT

`CMD` provides defaults for an executing container, while `ENTRYPOINT` configures a container that will run as an executable.

**Best Practices:**
- Use the exec form (JSON array) for both `CMD` and `ENTRYPOINT` (e.g., `CMD ["executable","param1","param2"]`). The shell form (`CMD command param1`) wraps the command in `/bin/sh -c`, which can cause issues with signal handling (like SIGTERM).
- Use `ENTRYPOINT` for the main executable and `CMD` for default arguments.

**Example:**
```dockerfile
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8080"]
```

### ENV and ARG

`ENV` sets environment variables that persist in the final image and running container. `ARG` defines variables that users can pass at build-time to the builder with the `docker build` command using the `--build-arg <varname>=<value>` flag. `ARG` variables do not persist in the final image.

**Best Practices:**
- Use `ENV` for configuration that the application needs at runtime.
- Use `ARG` for build-time configuration, such as specifying a version number to download.

**Worst Practices:**
- Storing secrets (passwords, API keys) in `ENV` or `ARG`. These values are visible in the image history (`docker history`). Use BuildKit secrets or Docker Compose secrets instead.

### EXPOSE

The `EXPOSE` instruction informs Docker that the container listens on the specified network ports at runtime. It does not actually publish the port.

**Best Practices:**
- Always document the intended ports using `EXPOSE`.
- Specify the protocol if necessary (e.g., `EXPOSE 80/udp`).

### VOLUME

The `VOLUME` instruction creates a mount point with the specified name and marks it as holding externally mounted volumes from native host or other containers.

**Best Practices:**
- Use `VOLUME` to declare directories that will hold persistent or shared data.

**Worst Practices:**
- Relying on `VOLUME` in the Dockerfile for critical data persistence without explicitly mapping it in `compose.yaml` or `docker run`. Data written to an anonymous volume is difficult to manage and can be lost if the container is removed.

### USER

The `USER` instruction sets the user name (or UID) and optionally the user group (or GID) to use when running the image and for any `RUN`, `CMD` and `ENTRYPOINT` instructions that follow it in the Dockerfile.

**Best Practices:**
- Always run containers as a non-root user for security.
- Use numeric UIDs instead of usernames to avoid issues with Kubernetes security contexts.

**Example:**
```dockerfile
RUN groupadd -r appgroup && useradd -r -g appgroup -u 1001 appuser
USER 1001:1001
```

### WORKDIR

The `WORKDIR` instruction sets the working directory for any `RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD` instructions that follow it in the Dockerfile.

**Best Practices:**
- Always use absolute paths for `WORKDIR`.
- Use `WORKDIR` instead of proliferating instructions like `RUN cd ... && do-something`.

### HEALTHCHECK

The `HEALTHCHECK` instruction tells Docker how to test a container to check that it is still working.

**Best Practices:**
- Always include a `HEALTHCHECK` to allow orchestrators (like Docker Swarm or Kubernetes) to know when a container is ready to receive traffic or needs to be restarted.

**Example:**
```dockerfile
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/health || exit 1
```

### SHELL

The `SHELL` instruction allows the default shell used for the shell form of commands to be overridden.

**Best Practices:**
- Use `SHELL` on Windows to switch between `cmd` and `powershell`.
- Use `SHELL ["/bin/bash", "-c"]` on Linux if you need bash-specific features in your `RUN` commands.

### LABEL

The `LABEL` instruction adds metadata to an image.

**Best Practices:**
- Use OCI standard labels (e.g., `org.opencontainers.image.source`, `org.opencontainers.image.version`) to provide consistent metadata.

### STOPSIGNAL

The `STOPSIGNAL` instruction sets the system call signal that will be sent to the container to exit.

**Best Practices:**
- Use `STOPSIGNAL` if your application requires a specific signal to shut down gracefully (e.g., `STOPSIGNAL SIGINT`).

### ONBUILD

The `ONBUILD` instruction adds to the image a trigger instruction to be executed at a later time, when the image is used as the base for another build.

**Best Practices:**
- Use `ONBUILD` sparingly, as it can make builds unpredictable for users of your base image. Document its usage clearly.

## 3. Multi-stage Builds: Patterns and Optimization

Multi-stage builds are a crucial technique for creating minimal, secure Docker images. They allow you to use multiple `FROM` statements in your Dockerfile. Each `FROM` instruction can use a different base, and each of them begins a new stage of the build. You can selectively copy artifacts from one stage to another, leaving behind everything you don't want in the final image.

### Go Pattern

Go applications compile to a single static binary, making them perfect candidates for multi-stage builds.

**Before (Single Stage):**
```dockerfile
FROM golang:1.20
WORKDIR /app
COPY . .
RUN go build -o main .
CMD ["./main"]
```
*Size: ~800MB*

**After (Multi-stage with Scratch):**
```dockerfile
# Stage 1: Build
FROM golang:1.20-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
# Build a statically linked binary
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# Stage 2: Runtime
FROM scratch
WORKDIR /app
# Copy CA certificates for HTTPS requests
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=builder /app/main .
ENTRYPOINT ["./main"]
```
*Size: ~15MB (Massive reduction, no OS vulnerabilities)*

### Node.js Pattern

Node.js applications require the Node runtime, but you don't need the build tools (like `node-gyp` or Python) in the final image.

**Before (Single Stage):**
```dockerfile
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
CMD ["npm", "start"]
```
*Size: ~1GB*

**After (Multi-stage with Alpine):**
```dockerfile
# Stage 1: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Runtime
FROM node:18-alpine
WORKDIR /app
ENV NODE_ENV=production
COPY package*.json ./
# Install only production dependencies
RUN npm ci --only=production && npm cache clean --force
COPY --from=builder /app/dist ./dist
USER node
CMD ["node", "dist/main.js"]
```
*Size: ~150MB*

### Python Pattern

Python applications often require C compilers for dependencies (like `psycopg2` or `numpy`), which shouldn't be in the final image.

**Before (Single Stage):**
```dockerfile
FROM python:3.11
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```
*Size: ~900MB*

**After (Multi-stage with Slim and Wheels):**
```dockerfile
# Stage 1: Build wheels
FROM python:3.11-slim AS builder
WORKDIR /app
RUN apt-get update && apt-get install -y --no-install-recommends gcc libpq-dev
COPY requirements.txt .
RUN pip wheel --no-cache-dir --no-deps --wheel-dir /app/wheels -r requirements.txt

# Stage 2: Runtime
FROM python:3.11-slim
WORKDIR /app
RUN apt-get update && apt-get install -y --no-install-recommends libpq5 \
    && rm -rf /var/lib/apt/lists/*
COPY --from=builder /app/wheels /wheels
COPY --from=builder /app/requirements.txt .
RUN pip install --no-cache /wheels/*
COPY . .
RUN useradd -m appuser
USER appuser
CMD ["python", "app.py"]
```
*Size: ~180MB*

### Java Pattern

Java applications require a JDK to build but only a JRE to run.

**Before (Single Stage):**
```dockerfile
FROM maven:3.8-openjdk-17
WORKDIR /app
COPY . .
RUN mvn clean package
CMD ["java", "-jar", "target/app.jar"]
```
*Size: ~600MB*

**After (Multi-stage with JRE):**
```dockerfile
# Stage 1: Build
FROM maven:3.8-eclipse-temurin-17 AS builder
WORKDIR /app
COPY pom.xml .
# Cache dependencies
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Runtime
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
CMD ["java", "-jar", "app.jar"]
```
*Size: ~120MB*

### Rust Pattern

Similar to Go, Rust compiles to a static binary.

**Before (Single Stage):**
```dockerfile
FROM rust:1.70
WORKDIR /app
COPY . .
RUN cargo build --release
CMD ["./target/release/myapp"]
```
*Size: ~1.5GB*

**After (Multi-stage with Distroless):**
```dockerfile
# Stage 1: Build
FROM rust:1.70 AS builder
WORKDIR /app
COPY Cargo.toml Cargo.lock ./
# Dummy build to cache dependencies
RUN mkdir src && echo "fn main() {}" > src/main.rs
RUN cargo build --release
RUN rm -rf src
COPY src ./src
# Touch main.rs to force rebuild
RUN touch src/main.rs
RUN cargo build --release

# Stage 2: Runtime
FROM gcr.io/distroless/cc-debian12
WORKDIR /app
COPY --from=builder /app/target/release/myapp .
CMD ["./myapp"]
```
*Size: ~30MB*

## 4. Base Image Selection Guide

Choosing the right base image is the foundation of a secure and optimized Docker container. The choice impacts image size, security posture, debugging capabilities, and application compatibility.

### Comparison Table

| Base Image Type | Typical Size | Security Surface | Debugging Tools | Compatibility | Best For |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Scratch** | 0 MB | None (Empty) | None | Statically linked binaries only | Go, Rust, C/C++ (static) |
| **Distroless** | 2-20 MB | Extremely Low | Minimal (no shell) | Language-specific runtimes | Java, Node.js, Python, Go |
| **Alpine** | ~5 MB | Low | `sh`, `apk` | `musl` libc (can cause issues) | General purpose, minimal |
| **Slim (Debian)** | ~30-50 MB | Medium | `bash`, `apt` | `glibc` (standard) | Python, Node.js, Ruby |
| **Full (Ubuntu/Debian)** | 100-200+ MB | High | Full OS tools | Maximum compatibility | Legacy apps, complex deps |

### Scratch

`scratch` is an explicitly empty image. It is the starting point for building all other images.

- **Pros:** Zero size, zero attack surface.
- **Cons:** No shell, no package manager, no CA certificates (unless copied).
- **When to use:** When you have a statically compiled binary (like Go or Rust) that has no external dependencies.

### Distroless

Distroless images (maintained by Google) contain only your application and its runtime dependencies. They do not contain package managers, shells, or any other programs you would expect to find in a standard Linux distribution.

- **Pros:** Extremely secure (no shell means attackers cannot easily execute commands if they compromise the app), very small size.
- **Cons:** Difficult to debug (no `sh` or `bash` to `docker exec` into).
- **When to use:** Production deployments of applications written in Java, Node.js, Python, or Go where security is paramount.

### Alpine

Alpine Linux is a security-oriented, lightweight Linux distribution based on `musl` libc and `busybox`.

- **Pros:** Very small size (~5MB), includes a package manager (`apk`).
- **Cons:** Uses `musl` instead of `glibc`. This can cause compatibility issues with applications or libraries compiled against `glibc` (common in Python data science libraries like `numpy` or `pandas`).
- **When to use:** General-purpose minimal images where `musl` compatibility is not an issue.

### Slim

Slim images (e.g., `debian:bullseye-slim`, `python:3.11-slim`) are stripped-down versions of standard distributions.

- **Pros:** Standard `glibc` compatibility, smaller than full images, includes standard tools (`apt`, `bash`).
- **Cons:** Larger than Alpine or Distroless, larger attack surface.
- **When to use:** When you need `glibc` compatibility (e.g., Python applications with C extensions) but want to minimize size.

### Full

Full images (e.g., `ubuntu:22.04`, `node:18`) contain a complete operating system environment.

- **Pros:** Maximum compatibility, all debugging tools available.
- **Cons:** Very large size, massive attack surface (many unnecessary packages with potential vulnerabilities).
- **When to use:** Only during development or for legacy applications that require a full OS environment. Never recommended for production.

## 5. Layer Optimization Strategies

Docker images are built layer by layer. Optimizing these layers is critical for reducing build times, minimizing image size, and maximizing cache hits.

### Ordering Strategy

Docker caches each layer. If a layer changes, Docker must rebuild that layer and all subsequent layers. Therefore, the golden rule of layer ordering is: **Order instructions from least frequently changed to most frequently changed.**

1. **Base Image:** `FROM` (Rarely changes)
2. **System Dependencies:** `RUN apt-get install...` (Infrequently changes)
3. **Application Dependencies:** `COPY package.json` -> `RUN npm install` (Changes occasionally)
4. **Application Code:** `COPY . .` (Changes constantly)

### Cache Invalidation Rules

- For `RUN` instructions, the cache is invalidated if the command string changes.
- For `COPY` and `ADD` instructions, Docker calculates a checksum of the files being copied. If the checksum changes (i.e., a file was modified), the cache is invalidated.
- Once a layer's cache is invalidated, all subsequent layers are also invalidated.

### RUN Consolidation

Every `RUN` instruction creates a new layer. To minimize the number of layers and reduce image size, consolidate related commands using `&&`.

**Bad (Creates 3 layers):**
```dockerfile
RUN apt-get update
RUN apt-get install -y curl
RUN rm -rf /var/lib/apt/lists/*
```

**Good (Creates 1 layer):**
```dockerfile
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
```

### .dockerignore Patterns

The `.dockerignore` file is essential for preventing unnecessary files from being sent to the Docker daemon (the build context) and copied into the image. This speeds up the build process and reduces image size.

**Standard `.dockerignore` template:**
```text
# Git
.git
.gitignore

# Node
node_modules
npm-debug.log

# Python
__pycache__
*.pyc
*.pyo
*.pyd
.Python
env/
venv/

# Docker
Dockerfile
.dockerignore
compose.yaml

# Secrets
.env
*.pem
*.key
```

## 6. BuildKit Features

BuildKit is the modern build subsystem for Docker (default since Docker 23.0). It offers significant performance improvements and advanced features over the legacy builder.

### Cache Mounts

Cache mounts allow you to cache directories between builds, which is incredibly useful for package managers like `pip`, `npm`, or `apt`. This prevents re-downloading dependencies if the cache is invalidated.

**Example (npm cache):**
```dockerfile
# syntax=docker/dockerfile:1
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
# Mount the npm cache directory
RUN --mount=type=cache,target=/root/.npm \
    npm ci
COPY . .
CMD ["npm", "start"]
```

### Build Secrets

Never pass secrets (like SSH keys or API tokens) using `ARG` or `ENV`, as they will be baked into the image history. Use BuildKit secrets instead.

**Dockerfile:**
```dockerfile
# syntax=docker/dockerfile:1
FROM alpine
# Mount the secret and use it in a command
RUN --mount=type=secret,id=mysecret \
    cat /run/secrets/mysecret > /tmp/secret_copy
```

**Build Command:**
```bash
docker build --secret id=mysecret,src=mysecret.txt .
```

### SSH Forwarding

If your build needs to access private repositories (e.g., private GitHub repos), use SSH forwarding instead of copying SSH keys into the image.

**Dockerfile:**
```dockerfile
# syntax=docker/dockerfile:1
FROM alpine
RUN apk add --no-cache openssh-client git
# Add github to known hosts
RUN mkdir -p -m 0700 ~/.ssh && ssh-keyscan github.com >> ~/.ssh/known_hosts
# Mount the SSH agent socket
RUN --mount=type=ssh \
    git clone git@github.com:myorg/myprivaterepo.git
```

**Build Command:**
```bash
docker build --ssh default .
```

### Multi-platform Builds with buildx

BuildKit allows you to build images for multiple architectures (e.g., `amd64` and `arm64`) simultaneously using `docker buildx`.

**Command:**
```bash
docker buildx build --platform linux/amd64,linux/arm64 -t myrepo/myimage:latest --push .
```

### Inline Cache

You can embed cache metadata into the image itself, allowing subsequent builds (even on different machines, like in CI/CD) to use the image as a cache source.

**Build Command (to create the cache):**
```bash
docker build --build-arg BUILDKIT_INLINE_CACHE=1 -t myrepo/myimage:latest .
```

**Build Command (to use the cache):**
```bash
docker build --cache-from myrepo/myimage:latest -t myrepo/myimage:latest .
```

### Named Contexts

Named contexts allow you to pass additional build contexts (like other images or local directories) to your build.

**Command:**
```bash
docker build --build-context project2=../project2 .
```

**Dockerfile:**
```dockerfile
FROM alpine
COPY --from=project2 . /project2
```

## 7. Docker Compose V2 Complete Specification

Docker Compose V2 is a complete rewrite of the original Python-based `docker-compose` in Go. It is now integrated directly into the Docker CLI as `docker compose`.

### Top-Level Elements

A `compose.yaml` file consists of several top-level elements:

- **`services`:** Defines the containers to be run.
- **`networks`:** Defines the networks the services will connect to.
- **`volumes`:** Defines the persistent volumes to be used by the services.
- **`configs`:** Defines configuration files to be mounted into containers.
- **`secrets`:** Defines sensitive data to be securely mounted into containers.
- **`name`:** (Optional) Sets the project name, overriding the directory name.

*Note: The `version` field is deprecated in Compose V2 and should be omitted.*

### Advanced Compose Features

- **Fragments (YAML Anchors):** Use `&` to define an anchor and `*` to reference it, allowing you to reuse configuration blocks.
- **Extensions (`x-`):** Any top-level field starting with `x-` is ignored by Compose. This is useful for defining anchors that don't belong to a specific service.
- **Interpolation:** Use `${VAR}` to inject environment variables from the host or a `.env` file into the Compose file.
- **Merge (`<<`):** Merge an anchor into a dictionary.
- **Include:** Use the `include` top-level element to pull in other Compose files, making it easier to manage large projects.
- **Profiles:** Assign services to profiles (e.g., `profiles: ["dev", "debug"]`). Services with profiles are only started if that profile is explicitly activated via the `--profile` flag or the `COMPOSE_PROFILES` environment variable.

### Service Attributes (Complete List)

Here is a comprehensive list of service attributes available in Compose V2:

`annotations`, `attach`, `build`, `blkio_config`, `cpu_count`, `cpu_percent`, `cpu_shares`, `cpu_period`, `cpu_quota`, `cpu_rt_runtime`, `cpu_rt_period`, `cpus`, `cpuset`, `cap_add`, `cap_drop`, `cgroup`, `cgroup_parent`, `command`, `configs`, `container_name`, `credential_spec`, `depends_on`, `deploy`, `develop`, `device_cgroup_rules`, `devices`, `dns`, `dns_opt`, `dns_search`, `domainname`, `entrypoint`, `env_file`, `environment`, `expose`, `extends`, `external_links`, `extra_hosts`, `group_add`, `healthcheck`, `hostname`, `image`, `init`, `ipc`, `isolation`, `labels`, `links`, `logging`, `mac_address`, `mem_limit`, `mem_reservation`, `mem_swappiness`, `memswap_limit`, `network_mode`, `networks`, `oom_kill_disable`, `oom_score_adj`, `pid`, `pids_limit`, `platform`, `ports`, `privileged`, `profiles`, `pull_policy`, `read_only`, `restart`, `runtime`, `scale`, `secrets`, `security_opt`, `shm_size`, `stdin_open`, `stop_grace_period`, `stop_signal`, `storage_opt`, `sysctls`, `tmpfs`, `tty`, `ulimits`, `user`, `userns_mode`, `uts`, `volumes`, `volumes_from`, `working_dir`.

## 8. Docker Compose Flaw-Proof Production Template

This template demonstrates a highly secure, production-ready `compose.yaml` file incorporating all best practices: non-root execution, read-only filesystems, capability dropping, resource limits, health checks, and secure networking.

```yaml
# compose.yaml
name: production-app

# Extension block for common security settings
x-security-opts: &security-opts
  security_opt:
    - no-new-privileges:true
  cap_drop:
    - ALL
  read_only: true

# Extension block for common logging settings
x-logging-opts: &logging-opts
  logging:
    driver: "json-file"
    options:
      max-size: "10m"
      max-file: "3"

services:
  api:
    image: YOUR_REGISTRY/api:v1.2.3
    container_name: prod-api
    restart: unless-stopped
    init: true # Use init process to handle signals properly
    user: "1001:1001" # Run as non-root
    <<: *security-opts
    <<: *logging-opts
    ports:
      - "8080:8080"
    environment:
      - NODE_ENV=production
      - DB_HOST=db
    secrets:
      - db_password
    networks:
      - backend
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
    stop_grace_period: 30s

  db:
    image: postgres:15-alpine
    container_name: prod-db
    restart: unless-stopped
    user: "70:70" # Postgres user in alpine
    <<: *security-opts
    <<: *logging-opts
    environment:
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
      POSTGRES_DB: appdb
    secrets:
      - db_password
    volumes:
      - db_data:/var/lib/postgresql/data
      # Mount run directory as tmpfs for postgres lock files when read_only is true
      - type: tmpfs
        target: /var/run/postgresql
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 1G

networks:
  backend:
    driver: bridge
    internal: true # No external access to this network

volumes:
  db_data:
    driver: local

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

## 9. Health Check Patterns for Common Services

Health checks are critical for ensuring that dependent services only start when their dependencies are truly ready, and for orchestrators to know when to restart a failing container.

### PostgreSQL
```yaml
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
  interval: 10s
  timeout: 5s
  retries: 5
```

### MongoDB
```yaml
healthcheck:
  test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
  interval: 10s
  timeout: 5s
  retries: 5
```

### Redis
```yaml
healthcheck:
  test: ["CMD", "redis-cli", "ping"]
  interval: 10s
  timeout: 5s
  retries: 5
```

### RabbitMQ
```yaml
healthcheck:
  test: ["CMD", "rabbitmq-diagnostics", "-q", "ping"]
  interval: 30s
  timeout: 10s
  retries: 3
```

### Nginx
```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost/"]
  interval: 30s
  timeout: 10s
  retries: 3
```

### Node.js / Go / Java (HTTP API)
```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 15s # Allow time for JVM/Node startup
```

## 10. Networking Deep Dive

Docker's networking subsystem is pluggable, using drivers. Understanding these drivers is essential for designing secure and performant container architectures.

### Network Drivers

- **`bridge`:** The default network driver. If you don't specify a driver, this is what you're creating. Bridge networks are usually used when your applications run in standalone containers that need to communicate. Docker Compose creates a custom bridge network for your project by default, providing automatic DNS resolution between containers using their service names.
- **`host`:** For standalone containers, remove network isolation between the container and the Docker host, and use the host's networking directly. Port mapping (`-p`) has no effect. This is useful for optimizing performance (bypassing NAT) or for applications that need to manage a large range of ports.
- **`overlay`:** Overlay networks connect multiple Docker daemons together and enable swarm services to communicate with each other. You can also use overlay networks to facilitate communication between a swarm service and a standalone container, or between two standalone containers on different Docker daemons.
- **`macvlan`:** Macvlan networks allow you to assign a MAC address to a container, making it appear as a physical device on your network. The Docker daemon routes traffic to containers by their MAC addresses. This is useful for legacy applications that expect to be directly connected to the physical network.
- **`ipvlan`:** Similar to `macvlan`, but containers share the same MAC address as the host interface. This is useful in environments where the network switch restricts the number of MAC addresses per port.
- **`none`:** Disables all networking for the container. Usually used in conjunction with a custom network driver.

### Internal Networks

For enhanced security, you can define a network as `internal: true`. This creates a bridge network that has no default gateway, meaning containers on this network cannot access the internet, nor can they be accessed from the internet. This is perfect for backend databases or internal microservices.

```yaml
networks:
  backend:
    internal: true
```

## 11. Volume Management

Data persistence in Docker is handled through volumes and bind mounts.

### Named Volumes

Named volumes are managed entirely by Docker (usually stored in `/var/lib/docker/volumes/` on Linux). They are the preferred mechanism for persisting data generated by and used by Docker containers.

- **Pros:** Easy to back up or migrate, managed via Docker CLI, work on both Linux and Windows containers, can be safely shared among multiple containers.
- **Cons:** Abstracted from the host filesystem, making direct manipulation slightly harder.

### Bind Mounts

Bind mounts map a specific file or directory on the host machine into a container.

- **Pros:** Very performant, easy to access files from the host (useful for development).
- **Cons:** Ties the container to a specific host filesystem structure, potential permission issues (especially between Linux hosts and containers running as non-root).

### tmpfs Mounts

A `tmpfs` mount is temporary and only stored in the host's memory. When the container stops, the `tmpfs` mount is removed, and files written there won't be persisted.

- **Pros:** Fast, secure (data is never written to disk).
- **Cons:** Data is lost when the container stops.
- **When to use:** For sensitive data (like secrets or keys) that you don't want persisted, or for applications that write a lot of temporary state.

## 12. Logging Configuration

By default, Docker uses the `json-file` logging driver, which captures standard output and standard error and writes them to a JSON file on the host. Without configuration, these files can grow indefinitely and consume all host disk space.

### Production Logging Configuration

Always configure log rotation for the `json-file` driver, either globally in `/etc/docker/daemon.json` or per-service in `compose.yaml`.

```yaml
logging:
  driver: "json-file"
  options:
    max-size: "10m" # Rotate after 10 megabytes
    max-file: "3"   # Keep a maximum of 3 files
```

### Alternative Logging Drivers

For centralized logging, Docker supports several other drivers:
- **`syslog`:** Routes logs to a syslog server.
- **`journald`:** Writes logs to the `systemd` journal.
- **`fluentd`:** Sends logs to a Fluentd daemon.
- **`awslogs`:** Sends logs to Amazon CloudWatch Logs.
- **`gcplogs`:** Sends logs to Google Cloud Logging.

## 13. Cost and Time Optimization Strategies

Optimizing Docker environments saves both compute costs (storage, bandwidth) and developer time (build speed, CI/CD pipeline duration).

### Image Size Reduction Checklist

1. **Use Minimal Base Images:** Switch from `ubuntu` to `alpine` or `distroless`.
2. **Multi-stage Builds:** Never ship build tools (compilers, headers) in production images.
3. **Consolidate RUN Commands:** Chain commands with `&&` to reduce layers.
4. **Clean Up Caches:** Remove `apt` or `apk` caches in the same layer they are used.
5. **Use .dockerignore:** Prevent unnecessary files from entering the build context.

### Build Time Optimization

1. **Layer Ordering:** Put the most frequently changing instructions (like `COPY . .`) at the very end of the Dockerfile.
2. **BuildKit Cache Mounts:** Use `--mount=type=cache` for package managers to avoid re-downloading dependencies.
3. **Parallel Builds:** Use `docker compose build --parallel` to build multiple services concurrently.
4. **CI/CD Caching:** Use inline caching (`--build-arg BUILDKIT_INLINE_CACHE=1`) or registry caching (`--cache-from`) in your CI pipelines.

### Registry Optimization

1. **Pull-Through Cache:** Set up a local registry as a pull-through cache to reduce bandwidth usage and speed up image pulls across your infrastructure.
2. **Image Pull Policy:** Use `pull_policy: if_not_present` in Compose to avoid unnecessary API calls to the registry.

## 14. Docker Compose for Databases

Running databases in Docker requires careful configuration of persistence, networking, and health checks.

### PostgreSQL with Replication (Example)

```yaml
services:
  postgres-primary:
    image: postgres:15
    environment:
      POSTGRES_USER: repl_user
      POSTGRES_PASSWORD: repl_password
      POSTGRES_DB: myapp
    volumes:
      - pg_primary_data:/var/lib/postgresql/data
      - ./init-primary.sh:/docker-entrypoint-initdb.d/init.sh
    networks:
      - db_net

  postgres-replica:
    image: postgres:15
    environment:
      POSTGRES_USER: repl_user
      POSTGRES_PASSWORD: repl_password
    volumes:
      - pg_replica_data:/var/lib/postgresql/data
      - ./init-replica.sh:/docker-entrypoint-initdb.d/init.sh
    depends_on:
      - postgres-primary
    networks:
      - db_net

volumes:
  pg_primary_data:
  pg_replica_data:

networks:
  db_net:
    internal: true
```

### Redis Sentinel/Cluster

For high availability, Redis should be deployed using Sentinel or Cluster mode. This typically involves multiple Redis nodes and Sentinel nodes monitoring them, all communicating over an internal Docker network.

## 15. Complete Multi-Stage Build Patterns by Language

Multi-stage builds are the single most impactful optimization technique. Below are complete, production-ready Dockerfiles for every major language, showing the before (naive) and after (optimized) approach.

### Go Application

**Before (Naive) — 1.2 GB:**
```dockerfile
FROM golang:1.22
WORKDIR /app
COPY . .
RUN go build -o server .
CMD ["./server"]
```

**After (Optimized) — 12 MB:**
```dockerfile
# syntax=docker/dockerfile:1
FROM golang:1.22-alpine AS builder
WORKDIR /app

# Copy dependency files first for cache
COPY go.mod go.sum ./
RUN --mount=type=cache,target=/go/pkg/mod \
    go mod download

# Copy source and build
COPY . .
RUN --mount=type=cache,target=/go/pkg/mod \
    --mount=type=cache,target=/root/.cache/go-build \
    CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
    go build -ldflags='-w -s -extldflags "-static"' -o /server .

# Runtime stage — scratch for smallest possible image
FROM scratch
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=builder /server /server
USER 65534:65534
EXPOSE 8080
ENTRYPOINT ["/server"]
```

**Key optimizations:** CGO_ENABLED=0 for static binary, ldflags `-w -s` to strip debug info, scratch base (no OS), cache mounts for Go module and build caches, non-root user via numeric UID.

### Node.js Application

**Before (Naive) — 1.1 GB:**
```dockerfile
FROM node:20
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "server.js"]
```

**After (Optimized) — 85 MB:**
```dockerfile
# syntax=docker/dockerfile:1
FROM node:20-alpine AS builder
WORKDIR /app

COPY package.json package-lock.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm ci --only=production

# If you have a build step (TypeScript, webpack, etc.)
COPY . .
RUN npm run build

# Runtime stage
FROM node:20-alpine AS runtime
WORKDIR /app
ENV NODE_ENV=production

# Copy only production dependencies and built output
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package.json ./

# Non-root user
RUN addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G appgroup
USER 1001:1001

EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1
CMD ["node", "dist/server.js"]
```

**Key optimizations:** `npm ci` for reproducible installs, cache mount for npm, separate build and runtime stages, only production dependencies in final image, `wget` instead of `curl` for healthcheck (available in alpine), non-root user.

### Python Application

**Before (Naive) — 1.0 GB:**
```dockerfile
FROM python:3.12
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

**After (Optimized) — 120 MB:**
```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.12-slim AS builder
WORKDIR /app

# Install build dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc libpq-dev && rm -rf /var/lib/apt/lists/*

# Install Python dependencies into a virtual environment
COPY requirements.txt .
RUN --mount=type=cache,target=/root/.cache/pip \
    python -m venv /opt/venv && \
    /opt/venv/bin/pip install --no-compile -r requirements.txt

# Runtime stage
FROM python:3.12-slim AS runtime
WORKDIR /app

# Install only runtime libraries (no gcc)
RUN apt-get update && apt-get install -y --no-install-recommends \
    libpq5 curl && rm -rf /var/lib/apt/lists/*

# Copy virtual environment from builder
COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

COPY . .

RUN adduser --disabled-password --gecos '' --uid 1001 appuser
USER 1001:1001

EXPOSE 8000
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "app:app"]
```

**Key optimizations:** Slim base (not alpine, avoids musl issues with C extensions), virtual environment copied between stages, build tools only in builder stage, pip cache mount, gunicorn for production.

### Java (Spring Boot) Application

**Before (Naive) — 800 MB:**
```dockerfile
FROM openjdk:17
COPY target/app.jar /app.jar
CMD ["java", "-jar", "/app.jar"]
```

**After (Optimized) — 200 MB:**
```dockerfile
# syntax=docker/dockerfile:1
FROM eclipse-temurin:17-jdk-alpine AS builder
WORKDIR /app

COPY pom.xml mvnw ./
COPY .mvn .mvn
RUN --mount=type=cache,target=/root/.m2 \
    ./mvnw dependency:go-offline

COPY src src
RUN --mount=type=cache,target=/root/.m2 \
    ./mvnw package -DskipTests && \
    java -Djarmode=layertools -jar target/*.jar extract --destination /extracted

# Runtime stage with JRE only
FROM eclipse-temurin:17-jre-alpine AS runtime
WORKDIR /app

# Copy Spring Boot layers in order of change frequency
COPY --from=builder /extracted/dependencies/ ./
COPY --from=builder /extracted/spring-boot-loader/ ./
COPY --from=builder /extracted/snapshot-dependencies/ ./
COPY --from=builder /extracted/application/ ./

RUN addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G appgroup
USER 1001:1001

EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=10s --retries=3 --start-period=60s \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1
ENTRYPOINT ["java", "-XX:+UseContainerSupport", "-XX:MaxRAMPercentage=75.0", "org.springframework.boot.loader.launch.JarLauncher"]
```

**Key optimizations:** JDK for build, JRE for runtime, Spring Boot layered extraction for optimal caching, Maven cache mount, container-aware JVM flags (`UseContainerSupport`, `MaxRAMPercentage`), long `start_period` for JVM warmup.

### Rust Application

**Before (Naive) — 1.5 GB:**
```dockerfile
FROM rust:1.77
WORKDIR /app
COPY . .
RUN cargo build --release
CMD ["./target/release/myapp"]
```

**After (Optimized) — 8 MB:**
```dockerfile
# syntax=docker/dockerfile:1
FROM rust:1.77-alpine AS builder
RUN apk add --no-cache musl-dev
WORKDIR /app

# Cache dependencies by building a dummy project first
RUN cargo init --name myapp
COPY Cargo.toml Cargo.lock ./
RUN --mount=type=cache,target=/usr/local/cargo/registry \
    --mount=type=cache,target=/app/target \
    cargo build --release

# Now copy real source and rebuild (only app code recompiles)
COPY src src
RUN --mount=type=cache,target=/usr/local/cargo/registry \
    --mount=type=cache,target=/app/target \
    cargo build --release && \
    cp target/release/myapp /myapp

FROM scratch
COPY --from=builder /myapp /myapp
USER 65534:65534
ENTRYPOINT ["/myapp"]
```

**Key optimizations:** Alpine with musl for static linking, dependency caching trick (build dummy project first), cargo registry and target caches, scratch final image.

### Base Image Comparison Table

| Base Image | Size | Shell | Package Manager | glibc | Security Score | Best For |
|---|---|---|---|---|---|---|
| `scratch` | 0 MB | No | No | No | Excellent | Go, Rust static binaries |
| `gcr.io/distroless/static` | 2 MB | No | No | No | Excellent | Go, Rust static binaries |
| `gcr.io/distroless/base` | 20 MB | No | No | Yes | Excellent | C/C++ apps |
| `gcr.io/distroless/java17` | 220 MB | No | No | Yes | Excellent | Java apps |
| `gcr.io/distroless/nodejs20` | 130 MB | No | No | Yes | Excellent | Node.js apps |
| `gcr.io/distroless/python3` | 50 MB | No | No | Yes | Excellent | Python apps |
| `alpine:3.19` | 7 MB | Yes | apk | No (musl) | Very Good | General purpose |
| `debian:bookworm-slim` | 80 MB | Yes | apt | Yes | Good | Python with C extensions |
| `ubuntu:24.04` | 78 MB | Yes | apt | Yes | Fair | Development, legacy |
| `node:20-alpine` | 180 MB | Yes | apk+npm | No (musl) | Good | Node.js |
| `python:3.12-slim` | 150 MB | Yes | apt+pip | Yes | Good | Python |
| `golang:1.22-alpine` | 260 MB | Yes | apk+go | No (musl) | Good | Go (build stage only) |

## 16. Docker Compose depends_on Deep Dive

The `depends_on` attribute controls service startup and shutdown order. In Compose V2, it supports conditions that make it far more powerful than the simple ordering of V1.

### Short Syntax (Order Only)

```yaml
services:
  web:
    depends_on:
      - db
      - redis
```

This only ensures `db` and `redis` containers are **started** before `web`. It does NOT wait for them to be **ready**.

### Long Syntax (With Conditions)

```yaml
services:
  web:
    depends_on:
      db:
        condition: service_healthy
        restart: true
      redis:
        condition: service_healthy
      migrations:
        condition: service_completed_successfully
```

**Conditions explained:**

| Condition | Behavior |
|---|---|
| `service_started` | Default. Waits for the container to start (not ready). |
| `service_healthy` | Waits for the container's healthcheck to report healthy. Requires a `healthcheck` on the dependency. |
| `service_completed_successfully` | Waits for the container to run and exit with code 0. Perfect for init/migration containers. |

**The `restart: true` flag** (Compose 2.22+): When set, if the dependency restarts, the dependent service is also restarted. This is critical for database connections — if PostgreSQL restarts, your API service should also restart to re-establish connections.

### Init Container Pattern

Use `service_completed_successfully` to run database migrations before starting the application:

```yaml
services:
  migrations:
    image: YOUR_REGISTRY/api:v1.2.3
    command: ["./migrate", "up"]
    environment:
      DATABASE_URL: postgres://user:pass@db:5432/myapp
    depends_on:
      db:
        condition: service_healthy
    # No restart — this is a one-shot container
    restart: "no"

  api:
    image: YOUR_REGISTRY/api:v1.2.3
    depends_on:
      migrations:
        condition: service_completed_successfully
      db:
        condition: service_healthy
    restart: unless-stopped
```

## 17. Docker Compose Secrets and Configs

### Secrets

Secrets provide a mechanism to securely pass sensitive data to containers without exposing them in environment variables or the Compose file.

```yaml
services:
  api:
    image: myapi:latest
    secrets:
      - db_password
      - api_key
    environment:
      # Reference the secret file path, not the value
      DB_PASSWORD_FILE: /run/secrets/db_password

secrets:
  db_password:
    file: ./secrets/db_password.txt    # From a file
  api_key:
    environment: "API_KEY"              # From host env var (Compose 2.23+)
```

Inside the container, secrets are mounted at `/run/secrets/<secret_name>` as read-only files. Applications must be designed to read from files (many databases support `*_FILE` environment variables natively, like PostgreSQL and MySQL).

### Configs

Configs are similar to secrets but for non-sensitive configuration files:

```yaml
services:
  nginx:
    image: nginx:alpine
    configs:
      - source: nginx_conf
        target: /etc/nginx/nginx.conf
        mode: 0444

configs:
  nginx_conf:
    file: ./nginx/nginx.conf
```

## 18. Docker Compose Environment Variable Management

Environment variables are the primary configuration mechanism for containerized applications. Docker Compose provides multiple ways to set them, with a clear precedence order.

### Precedence Order (Highest to Lowest)

1. `docker compose run -e` (CLI override)
2. `environment` attribute in compose.yaml
3. `--env-file` flag on CLI
4. `env_file` attribute in compose.yaml
5. `.env` file in project directory
6. Host environment variables

### Best Practices

**Use `.env` for defaults and local development:**
```env
# .env
COMPOSE_PROJECT_NAME=myapp
POSTGRES_VERSION=15
API_PORT=8080
```

**Use `env_file` for service-specific configuration:**
```yaml
services:
  api:
    env_file:
      - ./envs/common.env
      - ./envs/api.env
      - path: ./envs/api.local.env
        required: false  # Optional override (Compose 2.24+)
```

**Use `environment` for values that reference other variables:**
```yaml
services:
  api:
    environment:
      DATABASE_URL: "postgres://${DB_USER}:${DB_PASS}@db:5432/${DB_NAME}"
```

**Never put secrets in environment variables.** Use Docker secrets instead.

## 19. Complete Production Compose Templates

### Full-Stack Web Application

```yaml
name: production-fullstack

x-common: &common
  restart: unless-stopped
  init: true
  security_opt:
    - no-new-privileges:true
  cap_drop:
    - ALL
  logging:
    driver: "json-file"
    options:
      max-size: "10m"
      max-file: "5"

services:
  # Reverse Proxy
  nginx:
    <<: *common
    image: nginx:1.25-alpine
    read_only: true
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/certs:/etc/nginx/certs:ro
      - type: tmpfs
        target: /var/cache/nginx
      - type: tmpfs
        target: /var/run
    networks:
      - frontend
    depends_on:
      api:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost/health"]
      interval: 30s
      timeout: 5s
      retries: 3
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 256M

  # API Service
  api:
    <<: *common
    image: YOUR_REGISTRY/api:${API_VERSION}
    read_only: true
    user: "1001:1001"
    expose:
      - "8080"
    environment:
      NODE_ENV: production
      DB_HOST: postgres
      DB_PORT: "5432"
      DB_NAME: ${DB_NAME}
      REDIS_URL: redis://redis:6379/0
      RABBITMQ_URL: amqp://guest:guest@rabbitmq:5672/
    secrets:
      - db_password
    volumes:
      - type: tmpfs
        target: /tmp
    networks:
      - frontend
      - backend
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy
      migrations:
        condition: service_completed_successfully
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 15s
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 128M

  # Worker Service
  worker:
    <<: *common
    image: YOUR_REGISTRY/api:${API_VERSION}
    read_only: true
    user: "1001:1001"
    command: ["node", "dist/worker.js"]
    environment:
      NODE_ENV: production
      DB_HOST: postgres
      REDIS_URL: redis://redis:6379/0
      RABBITMQ_URL: amqp://guest:guest@rabbitmq:5672/
    secrets:
      - db_password
    volumes:
      - type: tmpfs
        target: /tmp
    networks:
      - backend
    depends_on:
      postgres:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 256M

  # Database Migrations (init container)
  migrations:
    image: YOUR_REGISTRY/api:${API_VERSION}
    command: ["npx", "prisma", "migrate", "deploy"]
    user: "1001:1001"
    environment:
      DATABASE_URL: postgres://${DB_USER}:${DB_PASS}@postgres:5432/${DB_NAME}
    networks:
      - backend
    depends_on:
      postgres:
        condition: service_healthy
    restart: "no"

  # PostgreSQL
  postgres:
    <<: *common
    image: postgres:16-alpine
    read_only: true
    user: "70:70"
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
      POSTGRES_DB: ${DB_NAME}
      PGDATA: /var/lib/postgresql/data/pgdata
    secrets:
      - db_password
    volumes:
      - pg_data:/var/lib/postgresql/data
      - type: tmpfs
        target: /var/run/postgresql
      - type: tmpfs
        target: /tmp
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER} -d ${DB_NAME}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 2G
        reservations:
          cpus: '0.50'
          memory: 512M
    shm_size: '256m'

  # Redis
  redis:
    <<: *common
    image: redis:7-alpine
    read_only: true
    user: "999:999"
    command: ["redis-server", "--maxmemory", "256mb", "--maxmemory-policy", "allkeys-lru", "--appendonly", "yes"]
    volumes:
      - redis_data:/data
      - type: tmpfs
        target: /tmp
    networks:
      - backend
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 300M

  # RabbitMQ
  rabbitmq:
    <<: *common
    image: rabbitmq:3.13-management-alpine
    hostname: rabbitmq
    environment:
      RABBITMQ_DEFAULT_USER: ${RABBITMQ_USER}
      RABBITMQ_DEFAULT_PASS: ${RABBITMQ_PASS}
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    networks:
      - backend
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "-q", "ping"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 30s
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true

volumes:
  pg_data:
    driver: local
  redis_data:
    driver: local
  rabbitmq_data:
    driver: local

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

### Monitoring Stack (Prometheus + Grafana)

```yaml
name: monitoring

x-common: &common
  restart: unless-stopped
  security_opt:
    - no-new-privileges:true
  cap_drop:
    - ALL
  logging:
    driver: "json-file"
    options:
      max-size: "10m"
      max-file: "3"

services:
  prometheus:
    <<: *common
    image: prom/prometheus:v2.51.0
    read_only: true
    user: "65534:65534"
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=30d'
      - '--web.console.libraries=/usr/share/prometheus/console_libraries'
      - '--web.console.templates=/usr/share/prometheus/consoles'
      - '--web.enable-lifecycle'
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus_data:/prometheus
    ports:
      - "9090:9090"
    networks:
      - monitoring
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:9090/-/healthy"]
      interval: 30s
      timeout: 5s
      retries: 3
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 2G

  grafana:
    <<: *common
    image: grafana/grafana:10.4.0
    user: "472:472"
    environment:
      GF_SECURITY_ADMIN_PASSWORD__FILE: /run/secrets/grafana_admin_password
      GF_INSTALL_PLUGINS: grafana-clock-panel
    secrets:
      - grafana_admin_password
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning:ro
    ports:
      - "3000:3000"
    networks:
      - monitoring
    depends_on:
      prometheus:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000/api/health"]
      interval: 30s
      timeout: 5s
      retries: 3
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M

  node-exporter:
    <<: *common
    image: prom/node-exporter:v1.7.0
    read_only: true
    pid: host
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--path.rootfs=/rootfs'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    networks:
      - monitoring
    deploy:
      resources:
        limits:
          cpus: '0.25'
          memory: 128M

  cadvisor:
    <<: *common
    image: gcr.io/cadvisor/cadvisor:v0.49.1
    read_only: true
    privileged: true
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /dev/disk/:/dev/disk:ro
    networks:
      - monitoring
    deploy:
      resources:
        limits:
          cpus: '0.25'
          memory: 256M

networks:
  monitoring:
    driver: bridge

volumes:
  prometheus_data:
  grafana_data:

secrets:
  grafana_admin_password:
    file: ./secrets/grafana_password.txt
```

## 20. Docker Compose Override and Multi-Environment Patterns

Docker Compose supports multiple configuration files that are merged together. This is the recommended approach for managing different environments.

### File Structure

```text
project/
  compose.yaml          # Base configuration (shared)
  compose.override.yaml # Development overrides (auto-loaded)
  compose.prod.yaml     # Production overrides
  compose.test.yaml     # Test overrides
  .env                  # Default environment variables
  .env.prod             # Production environment variables
```

### Base Configuration (compose.yaml)

```yaml
services:
  api:
    image: YOUR_REGISTRY/api:${API_VERSION:-latest}
    environment:
      DB_HOST: postgres
    depends_on:
      postgres:
        condition: service_healthy

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: myapp
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 10s
      timeout: 5s
      retries: 5
```

### Development Override (compose.override.yaml)

```yaml
# Automatically loaded when running `docker compose up`
services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
      - "9229:9229"  # Node.js debugger
    volumes:
      - .:/app:cached
      - /app/node_modules
    environment:
      NODE_ENV: development
      LOG_LEVEL: debug
    command: ["npm", "run", "dev"]

  postgres:
    ports:
      - "5432:5432"  # Expose for local tools
    environment:
      POSTGRES_PASSWORD: devpassword
```

### Production Override (compose.prod.yaml)

```yaml
# Used with: docker compose -f compose.yaml -f compose.prod.yaml up -d
services:
  api:
    restart: unless-stopped
    read_only: true
    user: "1001:1001"
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "5"

  postgres:
    restart: unless-stopped
    read_only: true
    volumes:
      - pg_data:/var/lib/postgresql/data
      - type: tmpfs
        target: /var/run/postgresql
    secrets:
      - db_password
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 2G
    shm_size: '256m'

volumes:
  pg_data:

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

### Usage Commands

```bash
# Development (auto-loads compose.override.yaml)
docker compose up

# Production (explicitly specify files)
docker compose -f compose.yaml -f compose.prod.yaml up -d

# Testing
docker compose -f compose.yaml -f compose.test.yaml up --abort-on-container-exit

# Using env file
docker compose --env-file .env.prod -f compose.yaml -f compose.prod.yaml up -d
```

## 21. Docker System Maintenance and Cleanup

Docker can consume significant disk space over time. Regular maintenance is essential.

### Disk Usage Analysis

```bash
# Show Docker disk usage summary
docker system df

# Detailed breakdown
docker system df -v

# Check overlay2 storage
du -sh /var/lib/docker/overlay2/
```

### Cleanup Commands

```bash
# Remove all stopped containers, unused networks, dangling images, and build cache
docker system prune

# Also remove unused images (not just dangling)
docker system prune -a

# Also remove unused volumes (DANGEROUS — data loss)
docker system prune -a --volumes

# Remove only specific types
docker container prune    # Stopped containers
docker image prune         # Dangling images
docker image prune -a      # All unused images
docker volume prune        # Unused volumes
docker network prune       # Unused networks
docker builder prune       # Build cache

# Remove images older than 24 hours
docker image prune -a --filter "until=24h"

# Remove build cache older than 7 days, keep 10GB
docker builder prune --filter "until=168h" --keep-storage 10GB
```

### Automated Cleanup (Cron)

```bash
# Add to crontab: clean up every Sunday at 3 AM
0 3 * * 0 docker system prune -af --filter "until=168h" 2>&1 | logger -t docker-cleanup
```

## 22. Signal Handling and Graceful Shutdown

Proper signal handling is critical for zero-downtime deployments and data integrity.

### The PID 1 Problem

When Docker stops a container, it sends SIGTERM to PID 1 inside the container. If PID 1 is a shell (because you used the shell form of CMD/ENTRYPOINT), the signal is not forwarded to the actual application process, and Docker must resort to SIGKILL after the grace period.

**Solution 1: Use exec form**
```dockerfile
# Good — node is PID 1, receives SIGTERM directly
CMD ["node", "server.js"]

# Bad — sh is PID 1, node never receives SIGTERM
CMD node server.js
```

**Solution 2: Use init: true in Compose**
```yaml
services:
  api:
    init: true  # Uses tini as PID 1, forwards signals properly
    command: ["node", "server.js"]
```

**Solution 3: Use tini in Dockerfile**
```dockerfile
RUN apk add --no-cache tini
ENTRYPOINT ["/sbin/tini", "--"]
CMD ["node", "server.js"]
```

### Stop Grace Period

Configure how long Docker waits between SIGTERM and SIGKILL:

```yaml
services:
  api:
    stop_grace_period: 30s  # Default is 10s
    stop_signal: SIGTERM     # Default signal
```

For databases, use a longer grace period to allow transactions to complete:

```yaml
services:
  postgres:
    stop_grace_period: 120s  # 2 minutes for DB shutdown
```

## Conclusion

This Docker Super Specialist guide provides the complete knowledge base needed by tech support operations teams to optimize, secure, troubleshoot, and maintain Docker environments at any scale. From Dockerfile construction patterns that reduce image sizes by 90%+ to flaw-proof production Compose templates with full security hardening, every section is designed to be directly applicable to real-world scenarios. The multi-stage build patterns, health check configurations, environment management strategies, and maintenance procedures documented here represent the current state of the art for containerized application deployment.

## 23. Docker Compose Profiles for Selective Service Activation

Profiles allow you to define optional services that are only started when explicitly activated. This is essential for managing development tools, debugging utilities, and optional infrastructure components.

```yaml
services:
  api:
    image: YOUR_REGISTRY/api:latest
    # No profile = always started

  postgres:
    image: postgres:16-alpine
    # No profile = always started

  # Only started with --profile debug
  pgadmin:
    image: dpage/pgadmin4:latest
    profiles: ["debug"]
    ports:
      - "5050:80"
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@local.dev
      PGADMIN_DEFAULT_PASSWORD: admin
    depends_on:
      postgres:
        condition: service_healthy

  # Only started with --profile monitoring
  prometheus:
    image: prom/prometheus:latest
    profiles: ["monitoring"]
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana:latest
    profiles: ["monitoring"]
    ports:
      - "3000:3000"
    depends_on:
      - prometheus

  # Only started with --profile test
  test-runner:
    image: YOUR_REGISTRY/api:latest
    profiles: ["test"]
    command: ["npm", "test"]
    depends_on:
      postgres:
        condition: service_healthy
```

**Usage:**
```bash
# Start only core services (api + postgres)
docker compose up -d

# Start with debugging tools
docker compose --profile debug up -d

# Start with monitoring
docker compose --profile monitoring up -d

# Start with multiple profiles
docker compose --profile debug --profile monitoring up -d

# Or via environment variable
COMPOSE_PROFILES=debug,monitoring docker compose up -d
```

## 24. Docker Compose Watch for Development Hot-Reload

Docker Compose Watch (introduced in Compose 2.22) provides file-watching capabilities that automatically sync changes or rebuild services during development.

```yaml
services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
    develop:
      watch:
        # Sync source files without rebuild
        - action: sync
          path: ./src
          target: /app/src
          ignore:
            - "**/*.test.ts"

        # Sync and restart when config changes
        - action: sync+restart
          path: ./config
          target: /app/config

        # Full rebuild when dependencies change
        - action: rebuild
          path: ./package.json

        # Full rebuild when Dockerfile changes
        - action: rebuild
          path: ./Dockerfile
```

**Usage:**
```bash
# Start with file watching
docker compose watch

# Or in background
docker compose up --watch
```

**Watch actions explained:**

| Action | Behavior | Use Case |
|---|---|---|
| `sync` | Copies changed files into the running container | Source code with hot-reload (nodemon, webpack-dev-server) |
| `sync+restart` | Copies files and restarts the container | Configuration files, environment changes |
| `rebuild` | Triggers a full image rebuild and container recreation | Dependency changes (package.json, requirements.txt, go.mod) |

## 25. Docker Compose Include for Modular Architecture

For large projects with many services, use `include` to split your Compose configuration into manageable modules.

```yaml
# compose.yaml (main entry point)
include:
  - path: ./infra/compose.db.yaml
  - path: ./infra/compose.cache.yaml
  - path: ./infra/compose.queue.yaml
  - path: ./services/compose.api.yaml
  - path: ./services/compose.worker.yaml
  - path:
      - ./monitoring/compose.monitoring.yaml
      - ./monitoring/compose.monitoring.override.yaml
```

**Each included file is a complete Compose file:**
```yaml
# infra/compose.db.yaml
services:
  postgres:
    image: postgres:16-alpine
    volumes:
      - pg_data:/var/lib/postgresql/data
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  pg_data:

networks:
  backend:
    external: true
```

This pattern keeps each file focused and maintainable, while the main `compose.yaml` orchestrates everything.

## 26. Docker Compose Resource Management and Deploy Configuration

The `deploy` section provides fine-grained control over resource allocation and deployment behavior. While originally designed for Docker Swarm, many attributes now work with `docker compose up` when using the `--compatibility` flag or Compose V2.

### Resource Limits and Reservations

```yaml
services:
  api:
    deploy:
      resources:
        limits:
          cpus: '2.0'        # Maximum 2 CPU cores
          memory: 1G          # Maximum 1 GB RAM
          pids: 100           # Maximum 100 processes
        reservations:
          cpus: '0.5'        # Guaranteed 0.5 CPU cores
          memory: 256M        # Guaranteed 256 MB RAM
          devices:
            - driver: nvidia  # GPU reservation
              count: 1
              capabilities: [gpu]
```

### Restart Policies

```yaml
services:
  api:
    restart: unless-stopped  # Recommended for production

  migrations:
    restart: "no"            # One-shot containers

  critical-service:
    restart: always          # Always restart, even after daemon restart
    deploy:
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
```

**Restart policy comparison:**

| Policy | Behavior | Use Case |
|---|---|---|
| `no` | Never restart | Init containers, migrations, one-shot tasks |
| `always` | Always restart, including after daemon restart | Critical services that must always run |
| `unless-stopped` | Restart unless explicitly stopped by user | Standard production services |
| `on-failure` | Restart only on non-zero exit code | Services where clean exit means "done" |

## 27. Docker Compose YAML Anchors and Extensions

YAML anchors and Compose extensions eliminate configuration duplication across services.

### YAML Anchors and Merge Keys

```yaml
# Define anchors in x- extensions (ignored by Compose)
x-common-env: &common-env
  LOG_LEVEL: info
  TZ: UTC
  NODE_ENV: production

x-common-deploy: &common-deploy
  deploy:
    resources:
      limits:
        cpus: '1.0'
        memory: 512M
    restart_policy:
      condition: on-failure
      delay: 5s
      max_attempts: 3

x-common-security: &common-security
  security_opt:
    - no-new-privileges:true
  cap_drop:
    - ALL
  read_only: true

x-common-logging: &common-logging
  logging:
    driver: "json-file"
    options:
      max-size: "10m"
      max-file: "5"

services:
  api:
    image: YOUR_REGISTRY/api:v1
    <<: [*common-deploy, *common-security, *common-logging]
    environment:
      <<: *common-env
      API_PORT: "8080"
      DB_HOST: postgres

  worker:
    image: YOUR_REGISTRY/worker:v1
    <<: [*common-deploy, *common-security, *common-logging]
    environment:
      <<: *common-env
      WORKER_CONCURRENCY: "4"
      QUEUE_NAME: default

  scheduler:
    image: YOUR_REGISTRY/scheduler:v1
    <<: [*common-deploy, *common-security, *common-logging]
    environment:
      <<: *common-env
      CRON_EXPRESSION: "*/5 * * * *"
```

This pattern ensures that all services share identical security, logging, and resource configurations, making it impossible to accidentally deploy a service without proper hardening.

## 28. Docker Image Tagging Strategy

A proper tagging strategy is essential for traceability, rollback capability, and CI/CD integration.

### Recommended Tagging Convention

```bash
# Semantic version (primary tag for releases)
YOUR_REGISTRY/api:1.2.3

# Git SHA (for traceability to exact commit)
YOUR_REGISTRY/api:sha-a1b2c3d

# Branch-based (for development/staging)
YOUR_REGISTRY/api:main
YOUR_REGISTRY/api:develop

# Date-based (for nightly builds)
YOUR_REGISTRY/api:2026-05-29

# Combined (best practice for CI/CD)
YOUR_REGISTRY/api:1.2.3
YOUR_REGISTRY/api:1.2
YOUR_REGISTRY/api:1
YOUR_REGISTRY/api:sha-a1b2c3d
YOUR_REGISTRY/api:latest
```

### CI/CD Tagging Script

```bash
#!/bin/bash
set -euo pipefail

REGISTRY="YOUR_REGISTRY"
IMAGE="api"
VERSION=$(cat VERSION)
GIT_SHA=$(git rev-parse --short HEAD)
DATE=$(date +%Y-%m-%d)

# Build with BuildKit
DOCKER_BUILDKIT=1 docker build \
  --build-arg BUILDKIT_INLINE_CACHE=1 \
  --cache-from "${REGISTRY}/${IMAGE}:latest" \
  -t "${REGISTRY}/${IMAGE}:${VERSION}" \
  -t "${REGISTRY}/${IMAGE}:sha-${GIT_SHA}" \
  -t "${REGISTRY}/${IMAGE}:${DATE}" \
  -t "${REGISTRY}/${IMAGE}:latest" \
  .

# Push all tags
for tag in "${VERSION}" "sha-${GIT_SHA}" "${DATE}" "latest"; do
  docker push "${REGISTRY}/${IMAGE}:${tag}"
done
```

### Never Use `latest` in Production

The `latest` tag is mutable and provides no guarantee about what version is actually running. Always use explicit version tags in production Compose files:

```yaml
# BAD — unpredictable
services:
  api:
    image: YOUR_REGISTRY/api:latest

# GOOD — deterministic
services:
  api:
    image: YOUR_REGISTRY/api:1.2.3
```

## 29. Docker Compose Networking Best Practices

### Network Segmentation

Always separate frontend (internet-facing) and backend (internal) networks:

```yaml
services:
  nginx:
    networks:
      - frontend        # Internet-facing

  api:
    networks:
      - frontend        # Receives traffic from nginx
      - backend         # Connects to databases

  postgres:
    networks:
      - backend         # Only accessible from backend network

  redis:
    networks:
      - backend

networks:
  frontend:
    driver: bridge

  backend:
    driver: bridge
    internal: true      # No internet access, no external access
```

### DNS Resolution

Within a Docker Compose network, services can reach each other by service name. Docker's embedded DNS server (127.0.0.11) handles resolution automatically. Important rules to remember:

1. Service names are resolved to the container's IP on the shared network.
2. If a service is on multiple networks, it can only reach services on the same network(s).
3. Network aliases allow a service to be reachable by multiple names.
4. The `container_name` does NOT affect DNS resolution — the service name does.

```yaml
services:
  api:
    networks:
      backend:
        aliases:
          - api-service
          - app-backend
```

### Custom Network Configuration

```yaml
networks:
  backend:
    driver: bridge
    internal: true
    ipam:
      config:
        - subnet: 172.28.0.0/16
          ip_range: 172.28.5.0/24
          gateway: 172.28.5.254
    driver_opts:
      com.docker.network.bridge.name: br-backend
      com.docker.network.bridge.enable_icc: "true"
      com.docker.network.bridge.enable_ip_masquerade: "false"
```

## 30. Docker Backup and Disaster Recovery

### Volume Backup

```bash
# Backup a named volume to a tar file
docker run --rm \
  -v myapp_pg_data:/source:ro \
  -v $(pwd)/backups:/backup \
  alpine tar czf /backup/pg_data_$(date +%Y%m%d_%H%M%S).tar.gz -C /source .

# Restore a volume from backup
docker run --rm \
  -v myapp_pg_data:/target \
  -v $(pwd)/backups:/backup \
  alpine sh -c "rm -rf /target/* && tar xzf /backup/pg_data_20260529_120000.tar.gz -C /target"
```

### PostgreSQL Backup with Docker

```bash
# Logical backup (pg_dump)
docker compose exec -T postgres pg_dump -U appuser -d appdb -Fc > backup_$(date +%Y%m%d).dump

# Restore
docker compose exec -T postgres pg_restore -U appuser -d appdb --clean < backup_20260529.dump

# Automated daily backup via cron
0 2 * * * cd /opt/myapp && docker compose exec -T postgres pg_dump -U appuser -d appdb -Fc > /backups/daily_$(date +\%Y\%m\%d).dump 2>&1 | logger -t pg-backup
```

### Full Compose Stack Backup Script

```bash
#!/bin/bash
set -euo pipefail

BACKUP_DIR="/backups/$(date +%Y%m%d_%H%M%S)"
mkdir -p "$BACKUP_DIR"

# 1. Stop the stack gracefully
docker compose stop

# 2. Backup all named volumes
for volume in $(docker compose config --volumes); do
  echo "Backing up volume: $volume"
  docker run --rm \
    -v "${COMPOSE_PROJECT_NAME}_${volume}:/source:ro" \
    -v "$BACKUP_DIR:/backup" \
    alpine tar czf "/backup/${volume}.tar.gz" -C /source .
done

# 3. Backup compose files and configs
cp compose.yaml "$BACKUP_DIR/"
cp .env "$BACKUP_DIR/" 2>/dev/null || true
cp -r secrets/ "$BACKUP_DIR/secrets/" 2>/dev/null || true

# 4. Restart the stack
docker compose start

echo "Backup completed: $BACKUP_DIR"
```

## 31. Docker Compose Upgrade and Rolling Update Strategies

### Zero-Downtime Update Procedure

```bash
# 1. Pull new images
docker compose pull

# 2. Recreate only changed services (no downtime for unchanged ones)
docker compose up -d --no-deps --build api

# 3. Or recreate with force (ensures clean state)
docker compose up -d --force-recreate --no-deps api

# 4. Verify health
docker compose ps
docker compose logs --tail=50 api

# 5. If something went wrong, rollback
docker compose up -d --no-deps api  # with previous image tag in compose.yaml
```

### Blue-Green Deployment Pattern

```bash
# 1. Start new version alongside old
docker compose -p myapp-blue up -d   # Current (blue)
docker compose -p myapp-green up -d  # New version (green)

# 2. Test green deployment
curl http://localhost:8081/health

# 3. Switch traffic (update nginx/load balancer config)
# 4. Stop blue deployment
docker compose -p myapp-blue down
```

### Database Migration Safety

When upgrading services that require database migrations, always follow this order:

1. **Backup the database** before any migration.
2. **Run migrations in a separate container** with `service_completed_successfully`.
3. **Verify migration success** before starting the application.
4. **Set timeouts** on migration containers to prevent hanging.

```yaml
services:
  migrations:
    image: YOUR_REGISTRY/api:v2.0.0
    command: ["./migrate", "up"]
    restart: "no"
    environment:
      DATABASE_URL: postgres://user:pass@postgres:5432/myapp
      MIGRATION_TIMEOUT: "300"
    depends_on:
      postgres:
        condition: service_healthy
    deploy:
      resources:
        limits:
          memory: 256M
    # Fail fast if migration takes too long
    stop_grace_period: 5m
```

## Conclusion

This Docker Super Specialist guide provides the complete knowledge base needed by tech support operations teams to optimize, secure, troubleshoot, and maintain Docker environments at any scale. From Dockerfile construction patterns that reduce image sizes by 90%+ to flaw-proof production Compose templates with full security hardening, every section is designed to be directly applicable to real-world scenarios. The multi-stage build patterns, health check configurations, environment management strategies, dependency handling, network segmentation, backup procedures, and upgrade strategies documented here represent the current state of the art for containerized application deployment. Combined with the supporting specialist files (advanced patterns, CLI reference, troubleshooting, security audit, configuration schemas, and architecture deep dive), this forms a comprehensive operational playbook for any Docker-based infrastructure.

## Appendix A: Docker Compose Quick Reference Cheat Sheet

### Service Lifecycle Commands

| Command | Description |
|---|---|
| `docker compose up -d` | Start all services in background |
| `docker compose up -d --build` | Build images and start services |
| `docker compose up -d --force-recreate` | Recreate all containers even if unchanged |
| `docker compose up -d --no-deps api` | Start only `api` without its dependencies |
| `docker compose down` | Stop and remove containers and networks |
| `docker compose down -v` | Also remove named volumes (data loss) |
| `docker compose down --rmi all` | Also remove all images |
| `docker compose stop` | Stop containers without removing them |
| `docker compose start` | Start previously stopped containers |
| `docker compose restart api` | Restart a specific service |
| `docker compose pause api` | Pause a running service |
| `docker compose unpause api` | Unpause a paused service |

### Debugging Commands

| Command | Description |
|---|---|
| `docker compose ps` | List running containers with status |
| `docker compose ps -a` | List all containers including stopped |
| `docker compose logs -f api` | Follow logs for a specific service |
| `docker compose logs --tail=100 --since=1h` | Last 100 lines from the past hour |
| `docker compose exec api sh` | Open a shell in a running container |
| `docker compose exec -T postgres psql -U user db` | Run command without TTY (for scripts) |
| `docker compose top` | Show running processes in all containers |
| `docker compose config` | Validate and display the merged config |
| `docker compose config --services` | List all service names |
| `docker compose config --volumes` | List all volume names |
| `docker compose images` | List images used by services |
| `docker compose events` | Stream real-time container events |

### Build Commands

| Command | Description |
|---|---|
| `docker compose build` | Build all services with build context |
| `docker compose build --no-cache api` | Build without cache |
| `docker compose build --pull` | Always pull base images before building |
| `docker compose build --parallel` | Build services in parallel |
| `docker compose push` | Push built images to registry |

## Appendix B: Dockerfile Instruction Quick Reference

| Instruction | Purpose | Exec Form | Shell Form |
|---|---|---|---|
| `FROM` | Set base image | `FROM image:tag AS name` | N/A |
| `RUN` | Execute build command | `RUN ["cmd", "arg"]` | `RUN cmd arg` |
| `CMD` | Default container command | `CMD ["cmd", "arg"]` | `CMD cmd arg` |
| `ENTRYPOINT` | Container executable | `ENTRYPOINT ["cmd"]` | `ENTRYPOINT cmd` |
| `COPY` | Copy files from context | `COPY [--chown=u:g] src dst` | N/A |
| `ADD` | Copy with extraction/URL | `ADD src dst` | N/A |
| `ENV` | Set environment variable | `ENV KEY=value` | N/A |
| `ARG` | Build-time variable | `ARG NAME=default` | N/A |
| `EXPOSE` | Document port | `EXPOSE 8080/tcp` | N/A |
| `VOLUME` | Create mount point | `VOLUME ["/data"]` | `VOLUME /data` |
| `WORKDIR` | Set working directory | `WORKDIR /app` | N/A |
| `USER` | Set runtime user | `USER 1001:1001` | N/A |
| `HEALTHCHECK` | Define health check | `HEALTHCHECK CMD curl -f ...` | N/A |
| `LABEL` | Add metadata | `LABEL key="value"` | N/A |
| `STOPSIGNAL` | Set stop signal | `STOPSIGNAL SIGTERM` | N/A |
| `SHELL` | Override default shell | `SHELL ["/bin/bash", "-c"]` | N/A |
| `ONBUILD` | Trigger on child build | `ONBUILD RUN cmd` | N/A |

## Appendix C: Common Docker Error Messages and Solutions

| Error Message | Cause | Solution |
|---|---|---|
| `exec format error` | Wrong platform (e.g., arm64 image on amd64) | Rebuild with `--platform linux/amd64` or use `platform:` in compose |
| `port is already allocated` | Another process using the port | `lsof -i :PORT` to find and stop it, or change the port mapping |
| `no space left on device` | Docker storage full | `docker system prune -a` and check `/var/lib/docker` |
| `OOMKilled` | Container exceeded memory limit | Increase `memory` limit or optimize application memory usage |
| `connection refused` | Service not ready or wrong network | Check `depends_on` conditions and verify network connectivity |
| `permission denied` | File ownership mismatch | Match container USER UID with volume file ownership |
| `name already in use` | Container name conflict | `docker compose down --remove-orphans` or remove the old container |
| `network not found` | Stale network reference | `docker network prune` and recreate |
| `manifest unknown` | Image tag doesn't exist in registry | Verify the tag exists: `docker manifest inspect image:tag` |
| `unauthorized: authentication required` | Registry login needed | `docker login YOUR_REGISTRY` |
| `context deadline exceeded` | Network timeout pulling image | Check DNS, proxy settings, or use a registry mirror |
| `max depth exceeded` | Too many image layers | Consolidate RUN instructions, use multi-stage builds |
