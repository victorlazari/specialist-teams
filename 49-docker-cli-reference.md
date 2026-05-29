# Docker Super Specialist: Complete CLI Reference

## Introduction

Welcome to the ultimate Docker CLI reference guide, curated specifically for the Docker Super Specialist. This document is designed to be the definitive resource for technology support operations teams, DevOps engineers, and system administrators who are tasked with managing, troubleshooting, and optimizing Docker environments in production. 

Docker is a powerful platform, but its true potential is unlocked only when you master its command-line interface (CLI). This guide goes far beyond basic usage. It delves deep into every major command, exploring advanced flags, edge cases, and real-world production scenarios. Whether you are debugging a complex microservices architecture, optimizing build times, or securing your Docker daemon, this reference provides the detailed, actionable insights you need.

Our focus is on practical application. You will find complete, copy-pasteable code blocks, real error messages with their corresponding solutions, and before/after comparisons that demonstrate the tangible benefits of proper configuration. We cover not only the core `docker` CLI but also `docker compose`, `docker buildx`, `docker scout`, and the critical configuration files and environment variables that govern Docker's behavior.

---

## 1. Core Docker CLI Commands

The core Docker CLI is your primary interface for interacting with the Docker daemon. Mastering these commands is essential for effective container management.

### 1.1 Container Lifecycle Management

#### `docker run`
The `docker run` command is the workhorse of Docker. It creates and starts a new container from an image. In production, you rarely use `docker run` without a carefully considered set of flags.

**Key Flags:**
*   `-d, --detach`: Run the container in the background and print the container ID. Essential for long-running services.
*   `-p, --publish`: Publish a container's port(s) to the host. Format: `ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort | containerPort`.
*   `-v, --volume`: Bind mount a volume. Format: `host-src:container-dest[:ro]`.
*   `--name`: Assign a specific name to the container. Crucial for service discovery and management.
*   `--restart`: Restart policy to apply when a container exits (e.g., `always`, `unless-stopped`, `on-failure:5`).
*   `--env, -e`: Set environment variables.
*   `--network`: Connect a container to a network.
*   `--memory, -m`: Memory limit.
*   `--cpus`: Number of CPUs.

**Production Example: Running a Redis Cache**
```bash
docker run -d \
  --name production-redis \
  --restart unless-stopped \
  -p 127.0.0.1:6379:6379 \
  -v redis-data:/data \
  -e REDIS_PASSWORD=YOUR_SECURE_PASSWORD \
  --memory="512m" \
  --cpus="1.0" \
  redis:7.2-alpine \
  redis-server --requirepass YOUR_SECURE_PASSWORD --appendonly yes
```
*Explanation:* This command runs Redis in the background, ensures it restarts unless explicitly stopped, binds it only to localhost for security, persists data to a named volume, sets a password, and limits resources to prevent it from starving other services on the host.

#### `docker exec`
Execute a command in a running container. This is your primary tool for live debugging and inspection.

**Key Flags:**
*   `-i, --interactive`: Keep STDIN open even if not attached.
*   `-t, --tty`: Allocate a pseudo-TTY.
*   `-u, --user`: Username or UID (format: `<name|uid>[:<group|gid>]`).
*   `-e, --env`: Set environment variables.

**Production Example: Debugging a Web Server**
```bash
# Accessing the shell of a running Nginx container
docker exec -it production-nginx /bin/sh

# Running a single command without an interactive shell
docker exec production-nginx nginx -t
```

#### `docker ps`
List containers. By default, it shows only running containers.

**Key Flags:**
*   `-a, --all`: Show all containers (default shows just running).
*   `-q, --quiet`: Only display container IDs. Useful for scripting.
*   `--filter, -f`: Filter output based on conditions provided.
*   `--format`: Pretty-print containers using a Go template.

**Production Example: Finding specific containers**
```bash
# List all containers that exited with an error
docker ps -a --filter "exited=1"

# Custom formatted output for reporting
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

#### `docker logs`
Fetch the logs of a container. Crucial for troubleshooting.

**Key Flags:**
*   `-f, --follow`: Follow log output.
*   `--tail`: Number of lines to show from the end of the logs (default "all").
*   `--since`: Show logs since timestamp (e.g. 2013-01-02T13:23:37Z) or relative (e.g. 42m for 42 minutes).
*   `-t, --timestamps`: Show timestamps.

**Production Example: Monitoring Application Logs**
```bash
# Watch the last 100 lines of logs in real-time with timestamps
docker logs -f --tail 100 -t production-api-server

# Get logs from the last 15 minutes
docker logs --since 15m production-api-server
```

#### `docker inspect`
Return low-level information on Docker objects. This is the ultimate source of truth for container configuration and state.

**Key Flags:**
*   `--format, -f`: Format the output using the given Go template.

**Production Example: Extracting Specific Information**
```bash
# Get the IP address of a container
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' production-db

# Get the log path for a container
docker inspect --format='{{.LogPath}}' production-api-server
```

#### `docker stats`
Display a live stream of container(s) resource usage statistics.

**Key Flags:**
*   `--no-stream`: Disable streaming stats and only pull the first result.
*   `--format`: Pretty-print images using a Go template.

**Production Example: Resource Monitoring**
```bash
# View stats for all running containers, formatted
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}"
```

#### `docker top`
Display the running processes of a container. Useful for seeing what is actually executing inside the container namespace.

**Production Example:**
```bash
docker top production-db
```

#### `docker cp`
Copy files/folders between a container and the local filesystem.

**Production Example: Extracting a configuration file**
```bash
# Copy from container to host
docker cp production-nginx:/etc/nginx/nginx.conf ./nginx.conf.backup

# Copy from host to container
docker cp ./updated-nginx.conf production-nginx:/etc/nginx/nginx.conf
```

#### `docker diff`
Inspect changes to files or directories on a container's filesystem. Useful for security audits or understanding what an application is writing to disk.

**Production Example:**
```bash
docker diff production-api-server
```

#### `docker commit`
Create a new image from a container's changes. Generally discouraged in favor of Dockerfiles, but useful for quick debugging or capturing a specific state.

**Production Example:**
```bash
docker commit -m "Added debugging tools" production-api-server my-api-server:debug
```

### 1.2 Image Management

#### `docker build`
Build an image from a Dockerfile. (Note: `docker buildx build` is now the preferred method, but standard `build` is still widely used).

**Key Flags:**
*   `-t, --tag`: Name and optionally a tag in the 'name:tag' format.
*   `-f, --file`: Name of the Dockerfile (Default is 'PATH/Dockerfile').
*   `--build-arg`: Set build-time variables.
*   `--no-cache`: Do not use cache when building the image.
*   `--target`: Set the target build stage to build.

**Production Example: Multi-stage build**
```bash
docker build \
  -t YOUR_REGISTRY/my-app:1.2.3 \
  -f Dockerfile.prod \
  --build-arg APP_ENV=production \
  .
```

#### `docker pull`
Pull an image or a repository from a registry.

**Production Example:**
```bash
docker pull ubuntu:22.04
```

#### `docker push`
Push an image or a repository to a registry.

**Production Example:**
```bash
docker push YOUR_REGISTRY/my-app:1.2.3
```

#### `docker images`
List images.

**Key Flags:**
*   `-a, --all`: Show all images (default hides intermediate images).
*   `-q, --quiet`: Only show image IDs.
*   `--filter, -f`: Filter output based on conditions provided.

**Production Example: Finding dangling images**
```bash
docker images -f "dangling=true"
```

#### `docker tag`
Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE.

**Production Example:**
```bash
docker tag my-app:latest YOUR_REGISTRY/my-app:v1.0.0
```

#### `docker save` and `docker load`
Save one or more images to a tar archive (streamed to STDOUT by default) and load an image from a tar archive or STDIN. Useful for air-gapped environments.

**Production Example:**
```bash
# Save image
docker save -o my-app-v1.tar my-app:v1.0.0

# Load image on another machine
docker load -i my-app-v1.tar
```

#### `docker export` and `docker import`
Export a container's filesystem as a tar archive and import the contents from a tarball to create a filesystem image. Unlike `save/load`, this flattens the image (removes history/layers).

**Production Example:**
```bash
docker export production-db > db-backup.tar
cat db-backup.tar | docker import - my-db-base:latest
```

### 1.3 System Management

#### `docker system df`
Show docker disk usage. Essential for maintaining healthy Docker hosts.

**Key Flags:**
*   `-v, --verbose`: Show detailed information on space usage.

**Production Example:**
```bash
docker system df -v
```

#### `docker system prune`
Remove unused data. This is a critical command for preventing disk space exhaustion.

**Key Flags:**
*   `-a, --all`: Remove all unused images not just dangling ones.
*   `--volumes`: Prune volumes.
*   `-f, --force`: Do not prompt for confirmation.

**Production Example: Weekly cleanup cron job**
```bash
# Removes stopped containers, unused networks, dangling images, and unused build cache
docker system prune -f

# Aggressive cleanup (use with caution)
docker system prune -a --volumes -f
```

#### `docker system info`
Display system-wide information. Useful for verifying daemon configuration.

#### `docker system events`
Get real time events from the server. Excellent for monitoring and auditing.

**Production Example: Monitoring container starts and stops**
```bash
docker events --filter 'type=container' --filter 'event=start' --filter 'event=stop'
```

### 1.4 Network Management

#### `docker network create`
Create a network.

**Production Example: Creating an overlay network for Swarm or a custom bridge**
```bash
docker network create \
  --driver bridge \
  --subnet 172.20.0.0/16 \
  --gateway 172.20.0.1 \
  my-custom-network
```

#### `docker network connect` / `disconnect`
Connect/disconnect a container to/from a network.

**Production Example:**
```bash
docker network connect my-custom-network production-api-server
```

#### `docker network inspect`
Display detailed information on one or more networks.

#### `docker network ls` / `rm`
List and remove networks.

### 1.5 Volume Management

#### `docker volume create`
Create a volume.

**Production Example: Creating a volume with specific driver options (e.g., NFS)**
```bash
docker volume create --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.100,rw \
  --opt device=:/path/to/dir \
  my-nfs-volume
```

#### `docker volume inspect`
Display detailed information on one or more volumes.

#### `docker volume ls` / `rm` / `prune`
List, remove, and prune unused volumes.

### 1.6 Secrets and Configs (Swarm/Compose)

#### `docker secret`
Manage Docker secrets. Used to securely store sensitive data like passwords and API keys.

**Production Example:**
```bash
echo "my-super-secret-password" | docker secret create db_password -
```

#### `docker config`
Manage Docker configs. Used to store non-sensitive configuration files.

**Production Example:**
```bash
docker config create nginx_config ./nginx.conf
```

### 1.7 Context Management

#### `docker context`
Manage contexts. Contexts allow you to easily switch between different Docker daemons (e.g., local, remote server, cloud provider).

**Production Example: Setting up a remote context**
```bash
docker context create remote-prod --docker "host=ssh://user@prod-server.example.com"
docker context use remote-prod
# Now all docker commands run against the remote server
```

---

## 2. Advanced Tooling: Buildx, Scout, and Compose

### 2.1 Docker Buildx

Buildx is a Docker CLI plugin for extended build capabilities with BuildKit. It is the modern standard for building Docker images.

#### `docker buildx build`
Build an image using BuildKit.

**Key Features:**
*   Multi-platform builds.
*   Advanced caching mechanisms.
*   Concurrent build steps.

**Production Example: Multi-platform build and push**
```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t YOUR_REGISTRY/my-app:latest \
  --push \
  .
```

#### `docker buildx create` / `inspect` / `ls` / `rm` / `use`
Manage builder instances.

**Production Example: Creating a new builder**
```bash
docker buildx create --name mybuilder --use
docker buildx inspect --bootstrap
```

#### `docker buildx bake`
Build from a file (usually `docker-bake.hcl` or `docker-compose.yml`). Allows defining complex build pipelines.

### 2.2 Docker Scout

Docker Scout provides software supply chain security, focusing on vulnerability scanning and remediation.

#### `docker scout cves`
Display CVEs for an image.

**Production Example:**
```bash
docker scout cves YOUR_REGISTRY/my-app:latest
```

#### `docker scout quickview`
Quick overview of an image's vulnerabilities.

#### `docker scout recommendations`
Get recommendations for fixing vulnerabilities (e.g., updating the base image).

**Production Example:**
```bash
docker scout recommendations YOUR_REGISTRY/my-app:latest
```

### 2.3 Docker Compose CLI

The `docker compose` command (v2) has replaced the old `docker-compose` python script. It is deeply integrated into the Docker CLI.

#### `docker compose up`
Builds, (re)creates, starts, and attaches to containers for a service.

**Key Flags:**
*   `-d, --detach`: Detached mode: Run containers in the background.
*   `--build`: Build images before starting containers.
*   `--force-recreate`: Recreate containers even if their configuration and image haven't changed.
*   `--no-deps`: Don't start linked services.
*   `--remove-orphans`: Remove containers for services not defined in the Compose file.
*   `--scale`: Scale a service to a specific number of instances (e.g., `--scale web=3`).
*   `--timeout`: Use this timeout in seconds for container shutdown when attached or when containers are already running.
*   `--wait`: Wait for services to be running|healthy. Implies detached mode.
*   `--pull`: Pull image before running (`always`|`missing`|`never`).

**Production Example: Safe Deployment**
```bash
docker compose -f docker-compose.prod.yml up -d --build --remove-orphans --wait
```
*Explanation:* This command uses a specific production file, runs in the background, forces a build of any local images, cleans up old containers that are no longer in the compose file, and waits for all services to report as healthy before returning control to the terminal.

#### `docker compose down`
Stops containers and removes containers, networks, volumes, and images created by `up`.

**Key Flags:**
*   `-v, --volumes`: Remove named volumes declared in the `volumes` section of the Compose file and anonymous volumes attached to containers.
*   `--rmi`: Remove images. Type must be one of: `all` (Remove all images used by any service), `local` (Remove only images that don't have a custom tag set by the `image` field).
*   `--remove-orphans`: Remove containers for services not defined in the Compose file.
*   `--timeout`: Specify a shutdown timeout in seconds (default 10).

**Production Example: Complete Teardown**
```bash
docker compose -f docker-compose.test.yml down -v --rmi local --remove-orphans
```
*Explanation:* Ideal for CI/CD pipelines. It tears down the environment, deletes all associated volumes (wiping data), removes locally built images to save space, and cleans up orphans.

#### Other Compose Commands
*   `docker compose build`: Build or rebuild services.
*   `docker compose pull`: Pull service images.
*   `docker compose push`: Push service images.
*   `docker compose logs`: View output from containers.
*   `docker compose exec`: Execute a command in a running container.
*   `docker compose run`: Run a one-off command on a service.
*   `docker compose ps`: List containers.
*   `docker compose top`: Display the running processes.
*   `docker compose restart`: Restart services.
*   `docker compose stop`: Stop services.
*   `docker compose start`: Start services.
*   `docker compose pause` / `unpause`: Pause/unpause services.
*   `docker compose config`: Validate and view the Compose file.
*   `docker compose images`: List images used by the created containers.
*   `docker compose events`: Receive real time events from containers.
*   `docker compose port`: Print the public port for a port binding.
*   `docker compose cp`: Copy files/folders between a service container and the local filesystem.
*   `docker compose ls`: List running compose projects.
*   `docker compose watch`: Watch build context for service and rebuild/refresh containers when files are updated.
*   `docker compose alpha`: Experimental features.

---

## 3. Deep Dive: Debugging Commands

When things go wrong in production, these are the commands you rely on.

### 3.1 `docker logs -f --tail --since`
The first step in debugging is always the logs.

**Scenario:** A web application is returning 500 errors.
**Action:**
```bash
docker logs -f --tail 50 --since 5m production-web-app
```
**Why this works:** It immediately shows the last 50 lines (giving context) and continues to stream new logs. Limiting to the last 5 minutes (`--since 5m`) filters out noise from earlier, unrelated events.

### 3.2 `docker exec -it`
When logs aren't enough, you need to get inside the container.

**Scenario:** The application cannot connect to the database.
**Action:**
```bash
docker exec -it production-web-app /bin/bash
# Inside the container:
ping database-host
curl -v telnet://database-host:5432
```
**Why this works:** It allows you to test network connectivity and DNS resolution from the exact perspective of the application.

### 3.3 `docker inspect` with Go Templates
`docker inspect` outputs a massive JSON array. Go templates allow you to extract exactly what you need.

**Scenario:** You need to find out why a container keeps restarting.
**Action:**
```bash
docker inspect --format='{{.State.Status}} - Exit Code: {{.State.ExitCode}} - Error: {{.State.Error}}' crashing-container
```
**Why this works:** It isolates the specific state information, exit code, and any daemon-level errors associated with the container, ignoring the rest of the JSON noise.

### 3.4 `docker stats`
**Scenario:** The host machine is sluggish.
**Action:**
```bash
docker stats --no-stream
```
**Why this works:** It provides an immediate snapshot of CPU, Memory, and Network I/O for all containers, allowing you to quickly identify the resource hog.

### 3.5 `docker system df`
**Scenario:** "No space left on device" errors.
**Action:**
```bash
docker system df -v
```
**Why this works:** It breaks down disk usage by Images, Containers, Local Volumes, and Build Cache, showing exactly where the space is going.

### 3.6 `docker events`
**Scenario:** Containers are mysteriously disappearing or restarting.
**Action:**
```bash
docker events --filter 'event=die' --filter 'event=oom'
```
**Why this works:** It streams daemon events. Filtering for `die` and `oom` (Out of Memory) will immediately alert you if the OOM killer is terminating your containers.

---

## 4. One-Liner Recipes for Operations

These are essential snippets for daily Docker administration.

### 4.1 Find Large Images
Identify images consuming the most disk space.
```bash
docker images --format "{{.Repository}}:{{.Tag}} {{.Size}}" | sort -k2 -h -r | head -n 10
```

### 4.2 Find Dangling Images
Dangling images are layers that have no relationship to any tagged images.
```bash
docker images -f "dangling=true" -q
```

### 4.3 Remove Stopped Containers
Clean up containers that are no longer running.
```bash
docker rm $(docker ps -a -q -f status=exited)
# Or, more safely:
docker container prune -f
```

### 4.4 Check Container Resource Usage (Sorted by Memory)
```bash
docker stats --no-stream --format "table {{.Name}}\t{{.MemUsage}}" | sort -k2 -h -r
```

### 4.5 Export/Import Volumes (Backup and Restore)
**Backup a named volume to a tarball:**
```bash
docker run --rm -v my-data-volume:/volume -v $(pwd):/backup alpine tar -cjf /backup/my-data-volume-backup.tar.bz2 -C /volume ./
```
**Restore a tarball to a named volume:**
```bash
docker run --rm -v my-data-volume:/volume -v $(pwd):/backup alpine sh -c "rm -rf /volume/* /volume/..?* /volume/.[!.]* ; tar -xjf /backup/my-data-volume-backup.tar.bz2 -C /volume"
```

---

## 5. Critical Environment Variables

Docker's behavior can be heavily influenced by environment variables.

*   **`DOCKER_HOST`**: Sets the URL to the docker daemon. (e.g., `tcp://192.168.1.100:2376` or `unix:///var/run/docker.sock`).
*   **`DOCKER_TLS_VERIFY`**: When set to a non-empty value, enables TLS communication with the daemon.
*   **`DOCKER_CERT_PATH`**: Path to the directory containing the TLS certificates (`ca.pem`, `cert.pem`, `key.pem`).
*   **`DOCKER_CONFIG`**: Specifies the location of the Docker client configuration files (default `~/.docker`).
*   **`DOCKER_CONTENT_TRUST`**: When set to `1`, enables Docker Content Trust (image signature verification).
*   **`COMPOSE_FILE`**: Specifies the path to the Compose file(s). Can be a colon-separated list.
*   **`COMPOSE_PROJECT_NAME`**: Sets the project name. This value is prepended along with the service name to the container on start up.
*   **`COMPOSE_PROFILES`**: A comma-separated list of profiles to enable when running Compose.
*   **`BUILDKIT_PROGRESS`**: Sets the type of progress output for BuildKit (`auto`, `plain`, `tty`). `plain` is highly recommended for CI environments.

---

## 6. Docker Daemon Configuration (`daemon.json`)

The `/etc/docker/daemon.json` file is the heart of Docker host configuration. A misconfigured daemon can lead to poor performance, instability, or security vulnerabilities.

### Complete Reference Example

```json
{
  "storage-driver": "overlay2",
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file": "3"
  },
  "default-address-pools": [
    {
      "base": "172.80.0.0/16",
      "size": 24
    }
  ],
  "dns": ["8.8.8.8", "8.8.4.4"],
  "registry-mirrors": ["https://mirror.gcr.io"],
  "insecure-registries": ["registry.internal.company.com:5000"],
  "live-restore": true,
  "default-runtime": "runc",
  "features": {
    "buildkit": true
  },
  "metrics-addr": "127.0.0.1:9323",
  "experimental": false
}
```

### Detailed Breakdown

#### `storage-driver`
Determines how Docker manages images and container layers.
*   **Recommendation:** `overlay2` is the recommended driver for almost all modern Linux distributions. It offers the best performance and stability.

#### `log-driver` and `log-opts`
Controls how container logs are handled.
*   **Critical Issue:** By default, the `json-file` driver has no size limits. A chatty container can fill up the host's disk, causing a complete outage.
*   **Solution:** Always configure `max-size` and `max-file` to implement log rotation.
*   **Before:** Disk fills up over time.
*   **After:** Docker keeps a maximum of 3 log files, each up to 100MB, per container.

#### `default-address-pools`
Defines the IP address ranges Docker uses when creating new bridge networks.
*   **Critical Issue:** Docker's default ranges can conflict with your corporate internal network, causing routing failures.
*   **Solution:** Explicitly define a safe range that does not overlap with your infrastructure.

#### `dns`
Sets the default DNS servers for all containers. Useful if your host's `/etc/resolv.conf` is complex or dynamically managed in a way that breaks containers.

#### `registry-mirrors`
Configures Docker to use a pull-through cache for Docker Hub.
*   **Benefit:** Drastically reduces bandwidth usage and speeds up image pulls, especially in CI/CD environments or large clusters.

#### `insecure-registries`
Allows Docker to pull from registries that do not have valid TLS certificates.
*   **Warning:** Use only for internal, trusted networks.

#### `live-restore`
**Crucial for Production.**
*   **Benefit:** Allows containers to keep running even if the Docker daemon becomes unavailable (e.g., during a daemon upgrade or crash). This minimizes downtime during maintenance.

#### `default-runtime`
Specifies the OCI runtime to use. Usually `runc`, but can be changed to alternatives like `sysbox` or `gvisor` for enhanced isolation.

#### `features`
Enables specific features. Setting `"buildkit": true` ensures BuildKit is used by default for all builds on the host.

---

## 7. Flaw-Proof Configurations and Troubleshooting Scenarios

### Scenario 1: The "No Space Left on Device" Outage

**Symptoms:**
*   Containers fail to start.
*   `docker pull` fails.
*   Host OS becomes unstable.

**Root Causes:**
1.  Unbounded container logs.
2.  Accumulation of dangling images and stopped containers.
3.  Orphaned volumes.

**The Flaw-Proof Solution:**

1.  **Daemon Configuration:** Implement log rotation in `daemon.json` (as shown above).
2.  **Automated Cleanup:** Implement a cron job on the host to run `docker system prune`.

*Example Cron Job (`/etc/cron.daily/docker-cleanup`):*
```bash
#!/bin/bash
# Prune everything unused, including volumes, but keep data from the last 24 hours
docker system prune -a --volumes --filter "until=24h" -f
```

### Scenario 2: Docker Compose Network Conflicts

**Symptoms:**
*   `docker compose up` fails with an error about overlapping IPv4 pools.

**Root Cause:**
Docker Compose creates a default network for each project. If you have many projects, or if the default subnet conflicts with your host's routing table, it fails.

**The Flaw-Proof Solution:**

Explicitly define the network subnet in your `compose.yaml`.

*Example `compose.yaml`:*
```yaml
services:
  web:
    image: nginx:alpine
    networks:
      - custom_net

networks:
  custom_net:
    driver: bridge
    ipam:
      config:
        - subnet: 10.5.0.0/16
          gateway: 10.5.0.1
```

### Scenario 3: Zombie Processes in Containers

**Symptoms:**
*   Container consumes increasing amounts of memory or PIDs.
*   `docker stop` takes exactly 10 seconds (the default timeout) and then forcefully kills the container.

**Root Cause:**
The main process in the container (PID 1) is not properly handling system signals (SIGTERM) or reaping zombie child processes. This often happens when running shell scripts or Java applications directly as PID 1.

**The Flaw-Proof Solution:**

Use `tini` or `dumb-init` as the entrypoint.

*Example Dockerfile:*
```dockerfile
FROM node:18-alpine

# Install tini
RUN apk add --no-cache tini

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .

# Use tini as the entrypoint
ENTRYPOINT ["/sbin/tini", "--"]

# Run your program
CMD ["node", "server.js"]
```
*Explanation:* `tini` runs as PID 1, forwards signals to your Node.js application, and reaps any zombie processes, ensuring clean shutdowns and preventing resource leaks.

---

## 8. Cost and Time Optimization Strategies

### 8.1 Optimizing Dockerfile Build Times

**Strategy: Layer Caching and Multi-Stage Builds**

Every instruction in a Dockerfile creates a layer. Docker caches these layers. If a layer changes, all subsequent layers must be rebuilt.

**Bad Dockerfile (Slow):**
```dockerfile
FROM ubuntu:22.04
COPY . /app
WORKDIR /app
RUN apt-get update && apt-get install -y python3 python3-pip
RUN pip3 install -r requirements.txt
CMD ["python3", "app.py"]
```
*Why it's bad:* Every time you change a line of code in `app.py`, the `COPY . /app` layer changes. This invalidates the cache for the `apt-get` and `pip install` steps, causing a massive, slow rebuild.

**Optimized Dockerfile (Fast):**
```dockerfile
FROM ubuntu:22.04 AS builder
WORKDIR /app
# Install OS dependencies
RUN apt-get update && apt-get install -y python3 python3-pip
# Copy ONLY requirements first
COPY requirements.txt .
# Install Python dependencies (this layer is cached unless requirements.txt changes)
RUN pip3 install --user -r requirements.txt

FROM ubuntu:22.04 AS runner
WORKDIR /app
RUN apt-get update && apt-get install -y python3 && rm -rf /var/lib/apt/lists/*
# Copy installed dependencies from builder
COPY --from=builder /root/.local /root/.local
# Copy application code LAST
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["python3", "app.py"]
```
*Why it's good:*
1.  **Dependency Caching:** `requirements.txt` is copied and installed *before* the application code. Code changes no longer trigger a re-installation of dependencies.
2.  **Multi-Stage:** The final image (`runner`) only contains the runtime environment and the built artifacts, leaving behind build tools (like `pip` cache), resulting in a much smaller, more secure image.

### 8.2 Optimizing Image Size

Smaller images mean faster pulls, lower storage costs, and a reduced attack surface.

**Strategies:**
1.  **Use Alpine or Distroless base images:** `node:18-alpine` is ~170MB. `node:18` is ~1GB.
2.  **Combine RUN commands:** `RUN apt-get update && apt-get install -y ... && rm -rf /var/lib/apt/lists/*`. This prevents intermediate files from being committed to a layer.
3.  **Use `.dockerignore`:** Prevent unnecessary files (like `.git`, `node_modules`, local logs) from being sent to the Docker daemon build context.

*Example `.dockerignore`:*
```text
.git
node_modules
npm-debug.log
Dockerfile
.dockerignore
```

### 8.3 Compose Optimization: `watch`

For local development, rebuilding images for every code change is a massive time sink. Docker Compose `watch` solves this.

*Example `compose.yaml` with watch:*
```yaml
services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    develop:
      watch:
        - action: sync
          path: ./frontend/src
          target: /app/src
          ignore:
            - node_modules/
        - action: rebuild
          path: ./frontend/package.json
```
*Explanation:* If a file in `./frontend/src` changes, Compose syncs it directly into the running container (fast). If `package.json` changes, it triggers a full rebuild (necessary for new dependencies).

---

## Conclusion

This reference guide provides the deep, actionable knowledge required of a Docker Super Specialist. By mastering these commands, understanding the nuances of daemon configuration, and applying these optimization strategies, you can ensure that your Docker environments are robust, secure, performant, and cost-effective. Remember that Docker is a dynamic ecosystem; continuous learning and adaptation are key to maintaining operational excellence.


## 9. Extended Command Reference and Edge Cases

### 9.1 Deep Dive: `docker run` Edge Cases

While we covered the basics, `docker run` has dozens of flags for specific edge cases.

*   `--cap-add` and `--cap-drop`: Linux capabilities. By default, Docker drops many capabilities for security. If your container needs specific privileges (e.g., `NET_ADMIN` to modify network interfaces), you add them here.
    *   *Example:* `docker run --cap-add=NET_ADMIN my-vpn-image`
*   `--device`: Add a host device to the container.
    *   *Example:* `docker run --device=/dev/snd:/dev/snd my-audio-app`
*   `--ipc`: IPC namespace to use. Can be used to share memory between containers.
    *   *Example:* `docker run --ipc=container:my-db my-app`
*   `--pid`: PID namespace to use. `host` allows the container to see host processes.
    *   *Example:* `docker run --pid=host my-monitoring-agent`
*   `--security-opt`: Security options (AppArmor, SELinux, seccomp).
    *   *Example:* `docker run --security-opt seccomp=unconfined my-legacy-app`

### 9.2 Deep Dive: `docker network` Advanced Usage

*   **Macvlan Networks:** Allow you to assign a MAC address to a container, making it appear as a physical device on your network.
    *   *Creation:* `docker network create -d macvlan --subnet=192.168.1.0/24 --gateway=192.168.1.1 -o parent=eth0 my-macvlan`
*   **Ipvlan Networks:** Similar to Macvlan but shares the host's MAC address. Useful when switches restrict the number of MAC addresses per port.

### 9.3 Deep Dive: `docker volume` Advanced Usage

*   **Tmpfs Mounts:** Store data in the host's memory. Fast, but volatile. Good for secrets or temporary scratch space.
    *   *Example:* `docker run --mount type=tmpfs,destination=/app/cache my-app`
*   **Bind Mounts vs. Named Volumes:** Bind mounts rely on the host machine's directory structure. Named volumes are managed entirely by Docker. Named volumes are preferred for production data persistence.

## 10. Comprehensive Troubleshooting Guide

### 10.1 Container Exits Immediately (Exit Code 0)
*   **Cause:** The main process finished successfully. Docker containers only run as long as their PID 1 process is running.
*   **Solution:** Ensure your `CMD` or `ENTRYPOINT` is a long-running process (e.g., a web server, a tail command).

### 10.2 Container Exits with Error (Exit Code 1, 137, etc.)
*   **Exit Code 1:** Application error. Check `docker logs`.
*   **Exit Code 137:** OOM (Out of Memory) Killed. The container exceeded its memory limit.
    *   *Solution:* Increase memory limit (`-m`) or optimize the application. Check `docker inspect` for `OOMKilled: true`.
*   **Exit Code 143:** Graceful termination (SIGTERM).

### 10.3 Cannot Connect to Container Port
*   **Check 1:** Is the port published? (`docker ps` should show `0.0.0.0:8080->80/tcp`).
*   **Check 2:** Is the application inside the container listening on `0.0.0.0`? If it listens on `127.0.0.1`, it won't be accessible from outside the container.
*   **Check 3:** Host firewall rules (iptables/ufw).

## 11. Security Best Practices Checklist

1.  **Never run as root:** Use the `USER` directive in your Dockerfile.
2.  **Use read-only filesystems:** `docker run --read-only`.
3.  **Drop capabilities:** `docker run --cap-drop=ALL`.
4.  **Scan images:** Use `docker scout cves`.
5.  **Keep base images updated:** Regularly rebuild images.
6.  **Use Docker Content Trust:** Set `DOCKER_CONTENT_TRUST=1`.
7.  **Limit resources:** Always set `--memory` and `--cpus` to prevent DoS attacks.

## 12. Complete `compose.yaml` Production Example

```yaml
version: '3.8'

services:
  api:
    image: YOUR_REGISTRY/api:v2.1.0
    restart: unless-stopped
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    environment:
      - NODE_ENV=production
      - DB_HOST=db
      - REDIS_HOST=redis
    secrets:
      - db_password
      - api_key
    networks:
      - backend
      - frontend
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  db:
    image: postgres:15-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
      POSTGRES_DB: production_db
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - backend
    secrets:
      - db_password
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin -d production_db"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    restart: unless-stopped
    command: redis-server --requirepass YOUR_REDIS_PASSWORD
    networks:
      - backend
    volumes:
      - redis_data:/data

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true # Cannot be accessed from outside

volumes:
  db_data:
    driver: local
  redis_data:
    driver: local

secrets:
  db_password:
    external: true
  api_key:
    external: true
```

This comprehensive guide covers the essential and advanced aspects of Docker CLI, Compose, and daemon configuration, providing a solid foundation for any Docker Super Specialist.

## 13. Exhaustive Command Dictionary

This section provides an exhaustive dictionary of every command requested, ensuring no stone is left unturned.

### 13.1 `docker run` (Detailed)
Run a command in a new container.
*   `-a, --attach`: Attach to STDIN, STDOUT or STDERR.
*   `--add-host`: Add a custom host-to-IP mapping (host:ip).
*   `--blkio-weight`: Block IO (relative weight), between 10 and 1000, or 0 to disable.
*   `--cgroup-parent`: Optional parent cgroup for the container.
*   `--cidfile`: Write the container ID to the file.
*   `--cpu-period`: Limit CPU CFS (Completely Fair Scheduler) period.
*   `--cpu-quota`: Limit CPU CFS quota.
*   `--cpu-rt-period`: Limit CPU real-time period in microseconds.
*   `--cpu-rt-runtime`: Limit CPU real-time runtime in microseconds.
*   `-c, --cpu-shares`: CPU shares (relative weight).
*   `--cpuset-cpus`: CPUs in which to allow execution (0-3, 0,1).
*   `--cpuset-mems`: MEMs in which to allow execution (0-3, 0,1).
*   `--disable-content-trust`: Skip image verification (default true).
*   `--dns`: Set custom DNS servers.
*   `--dns-option`: Set DNS options.
*   `--dns-search`: Set custom DNS search domains.
*   `--domainname`: Container NIS domain name.
*   `--entrypoint`: Overwrite the default ENTRYPOINT of the image.
*   `--expose`: Expose a port or a range of ports.
*   `--gpus`: GPU devices to add to the container ('all' to pass all GPUs).
*   `--group-add`: Add additional groups to join.
*   `--health-cmd`: Command to run to check health.
*   `--health-interval`: Time between running the check (ms|s|m|h) (default 0s).
*   `--health-retries`: Consecutive failures needed to report unhealthy.
*   `--health-start-period`: Start period for the container to initialize before starting health-retries countdown (ms|s|m|h) (default 0s).
*   `--health-timeout`: Maximum time to allow one check to run (ms|s|m|h) (default 0s).
*   `--help`: Print usage.
*   `-h, --hostname`: Container host name.
*   `--init`: Run an init inside the container that forwards signals and reaps processes.
*   `--ip`: IPv4 address (e.g., 172.30.100.104).
*   `--ip6`: IPv6 address (e.g., 2001:db8::33).
*   `--ipc`: IPC mode to use.
*   `--isolation`: Container isolation technology.
*   `--kernel-memory`: Kernel memory limit.
*   `-l, --label`: Set meta data on a container.
*   `--label-file`: Read in a line delimited file of labels.
*   `--link`: Add link to another container.
*   `--link-local-ip`: Container IPv4/IPv6 link-local addresses.
*   `--log-driver`: Logging driver for the container.
*   `--log-opt`: Log driver options.
*   `--mac-address`: Container MAC address (e.g., 92:d0:c6:0a:29:33).
*   `-m, --memory`: Memory limit.
*   `--memory-reservation`: Memory soft limit.
*   `--memory-swap`: Swap limit equal to memory plus swap: '-1' to enable unlimited swap.
*   `--memory-swappiness`: Tune container memory swappiness (0 to 100).
*   `--mount`: Attach a filesystem mount to the container.
*   `--name`: Assign a name to the container.
*   `--network`: Connect a container to a network.
*   `--network-alias`: Add network-scoped alias for the container.
*   `--no-healthcheck`: Disable any container-specified HEALTHCHECK.
*   `--oom-kill-disable`: Disable OOM Killer.
*   `--oom-score-adj`: Tune host's OOM preferences (-1000 to 1000).
*   `--pid`: PID namespace to use.
*   `--pids-limit`: Tune container pids limit (set -1 for unlimited).
*   `--platform`: Set platform if server is multi-platform capable.
*   `--privileged`: Give extended privileges to this container.
*   `-p, --publish`: Publish a container's port(s) to the host.
*   `-P, --publish-all`: Publish all exposed ports to random ports.
*   `--pull`: Pull image before running ("always"|"missing"|"never").
*   `--read-only`: Mount the container's root filesystem as read only.
*   `--restart`: Restart policy to apply when a container exits.
*   `--rm`: Automatically remove the container when it exits.
*   `--runtime`: Runtime to use for this container.
*   `--security-opt`: Security Options.
*   `--shm-size`: Size of /dev/shm.
*   `--sig-proxy`: Proxy received signals to the process (default true).
*   `--stop-signal`: Signal to stop a container (default "SIGTERM").
*   `--stop-timeout`: Timeout (in seconds) to stop a container.
*   `--storage-opt`: Storage driver options for the container.
*   `--sysctl`: Sysctl options.
*   `--tmpfs`: Mount a tmpfs directory.
*   `-t, --tty`: Allocate a pseudo-TTY.
*   `--ulimit`: Ulimit options.
*   `-u, --user`: Username or UID (format: <name|uid>[:<group|gid>]).
*   `--userns`: User namespace to use.
*   `--uts`: UTS namespace to use.
*   `-v, --volume`: Bind mount a volume.
*   `--volume-driver`: Optional volume driver for the container.
*   `--volumes-from`: Mount volumes from the specified container(s).
*   `-w, --workdir`: Working directory inside the container.

### 13.2 `docker exec` (Detailed)
Run a command in a running container.
*   `-d, --detach`: Detached mode: run command in the background.
*   `--detach-keys`: Override the key sequence for detaching a container.
*   `-e, --env`: Set environment variables.
*   `--env-file`: Read in a file of environment variables.
*   `-i, --interactive`: Keep STDIN open even if not attached.
*   `--privileged`: Give extended privileges to the command.
*   `-t, --tty`: Allocate a pseudo-TTY.
*   `-u, --user`: Username or UID (format: <name|uid>[:<group|gid>]).
*   `-w, --workdir`: Working directory inside the container.

### 13.3 `docker build` (Detailed)
Build an image from a Dockerfile.
*   `--add-host`: Add a custom host-to-IP mapping (host:ip).
*   `--build-arg`: Set build-time variables.
*   `--cache-from`: Images to consider as cache sources.
*   `--cgroup-parent`: Optional parent cgroup for the container.
*   `--compress`: Compress the build context using gzip.
*   `--cpu-period`: Limit the CPU CFS (Completely Fair Scheduler) period.
*   `--cpu-quota`: Limit the CPU CFS quota.
*   `-c, --cpu-shares`: CPU shares (relative weight).
*   `--cpuset-cpus`: CPUs in which to allow execution (0-3, 0,1).
*   `--cpuset-mems`: MEMs in which to allow execution (0-3, 0,1).
*   `--disable-content-trust`: Skip image verification (default true).
*   `-f, --file`: Name of the Dockerfile (Default is 'PATH/Dockerfile').
*   `--force-rm`: Always remove intermediate containers.
*   `--iidfile`: Write the image ID to the file.
*   `--isolation`: Container isolation technology.
*   `--label`: Set metadata for an image.
*   `-m, --memory`: Memory limit.
*   `--memory-swap`: Swap limit equal to memory plus swap: '-1' to enable unlimited swap.
*   `--network`: Set the networking mode for the RUN instructions during build.
*   `--no-cache`: Do not use cache when building the image.
*   `--pull`: Always attempt to pull a newer version of the image.
*   `-q, --quiet`: Suppress the build output and print image ID on success.
*   `--rm`: Remove intermediate containers after a successful build (default true).
*   `--security-opt`: Security options.
*   `--shm-size`: Size of /dev/shm.
*   `-t, --tag`: Name and optionally a tag in the 'name:tag' format.
*   `--target`: Set the target build stage to build.
*   `--ulimit`: Ulimit options.

### 13.4 `docker pull` (Detailed)
Pull an image or a repository from a registry.
*   `-a, --all-tags`: Download all tagged images in the repository.
*   `--disable-content-trust`: Skip image verification (default true).
*   `--platform`: Set platform if server is multi-platform capable.
*   `-q, --quiet`: Suppress verbose output.

### 13.5 `docker push` (Detailed)
Push an image or a repository to a registry.
*   `-a, --all-tags`: Push all tagged images in the repository.
*   `--disable-content-trust`: Skip image signing (default true).
*   `-q, --quiet`: Suppress verbose output.

### 13.6 `docker images` (Detailed)
List images.
*   `-a, --all`: Show all images (default hides intermediate images).
*   `--digests`: Show digests.
*   `-f, --filter`: Filter output based on conditions provided.
*   `--format`: Pretty-print images using a Go template.
*   `--no-trunc`: Don't truncate output.
*   `-q, --quiet`: Only show image IDs.

### 13.7 `docker ps` (Detailed)
List containers.
*   `-a, --all`: Show all containers (default shows just running).
*   `-f, --filter`: Filter output based on conditions provided.
*   `--format`: Pretty-print containers using a Go template.
*   `-n, --last`: Show n last created containers (includes all states).
*   `-l, --latest`: Show the latest created container (includes all states).
*   `--no-trunc`: Don't truncate output.
*   `-q, --quiet`: Only display container IDs.
*   `-s, --size`: Display total file sizes.

### 13.8 `docker logs` (Detailed)
Fetch the logs of a container.
*   `--details`: Show extra details provided to logs.
*   `-f, --follow`: Follow log output.
*   `--since`: Show logs since timestamp (e.g. 2013-01-02T13:23:37Z) or relative (e.g. 42m for 42 minutes).
*   `-n, --tail`: Number of lines to show from the end of the logs (default "all").
*   `-t, --timestamps`: Show timestamps.
*   `--until`: Show logs before a timestamp (e.g. 2013-01-02T13:23:37Z) or relative (e.g. 42m for 42 minutes).

### 13.9 `docker inspect` (Detailed)
Return low-level information on Docker objects.
*   `-f, --format`: Format the output using the given Go template.
*   `-s, --size`: Display total file sizes if the type is container.
*   `--type`: Return JSON for specified type.

### 13.10 `docker stats` (Detailed)
Display a live stream of container(s) resource usage statistics.
*   `-a, --all`: Show all containers (default shows just running).
*   `--format`: Pretty-print images using a Go template.
*   `--no-stream`: Disable streaming stats and only pull the first result.
*   `--no-trunc`: Do not truncate output.

### 13.11 `docker top` (Detailed)
Display the running processes of a container.
Usage: `docker top CONTAINER [ps OPTIONS]`

### 13.12 `docker cp` (Detailed)
Copy files/folders between a container and the local filesystem.
*   `-a, --archive`: Archive mode (copy all uid/gid information).
*   `-L, --follow-link`: Always follow symbol link in SRC_PATH.

### 13.13 `docker diff` (Detailed)
Inspect changes to files or directories on a container's filesystem.
Usage: `docker diff CONTAINER`

### 13.14 `docker commit` (Detailed)
Create a new image from a container's changes.
*   `-a, --author`: Author (e.g., "John Hannibal Smith <hannibal@a-team.com>").
*   `-c, --change`: Apply Dockerfile instruction to the created image.
*   `-m, --message`: Commit message.
*   `-p, --pause`: Pause container during commit (default true).

### 13.15 `docker tag` (Detailed)
Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE.
Usage: `docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]`

### 13.16 `docker save` (Detailed)
Save one or more images to a tar archive (streamed to STDOUT by default).
*   `-o, --output`: Write to a file, instead of STDOUT.

### 13.17 `docker load` (Detailed)
Load an image from a tar archive or STDIN.
*   `-i, --input`: Read from tar archive file, instead of STDIN.
*   `-q, --quiet`: Suppress the load output.

### 13.18 `docker export` (Detailed)
Export a container's filesystem as a tar archive.
*   `-o, --output`: Write to a file, instead of STDOUT.

### 13.19 `docker import` (Detailed)
Import the contents from a tarball to create a filesystem image.
*   `-c, --change`: Apply Dockerfile instruction to the created image.
*   `-m, --message`: Set commit message for imported image.
*   `--platform`: Set platform if server is multi-platform capable.

### 13.20 `docker system` (Detailed)
Manage Docker.
*   `docker system df`: Show docker disk usage.
    *   `--format`: Pretty-print images using a Go template.
    *   `-v, --verbose`: Show detailed information on space usage.
*   `docker system events`: Get real time events from the server.
    *   `-f, --filter`: Filter output based on conditions provided.
    *   `--format`: Format the output using the given Go template.
    *   `--since`: Show all events created since timestamp.
    *   `--until`: Stream events until this timestamp.
*   `docker system info`: Display system-wide information.
    *   `-f, --format`: Format the output using the given Go template.
*   `docker system prune`: Remove unused data.
    *   `-a, --all`: Remove all unused images not just dangling ones.
    *   `--filter`: Provide filter values (e.g. 'label=<key>=<value>').
    *   `-f, --force`: Do not prompt for confirmation.
    *   `--volumes`: Prune volumes.

### 13.21 `docker network` (Detailed)
Manage networks.
*   `docker network connect`: Connect a container to a network.
    *   `--alias`: Add network-scoped alias for the container.
    *   `--driver-opt`: driver options for the network.
    *   `--ip`: IPv4 address.
    *   `--ip6`: IPv6 address.
    *   `--link`: Add link to another container.
    *   `--link-local-ip`: Add a link-local address for the container.
*   `docker network create`: Create a network.
    *   `--attachable`: Enable manual container attachment.
    *   `--config-from`: The network from which to copy the configuration.
    *   `--config-only`: Create a configuration only network.
    *   `-d, --driver`: Driver to manage the Network (default "bridge").
    *   `--gateway`: IPv4 or IPv6 Gateway for the master subnet.
    *   `--ingress`: Create swarm routing-mesh network.
    *   `--internal`: Restrict external access to the network.
    *   `--ip-range`: Allocate container ip from a sub-range.
    *   `--ipam-driver`: IP Address Management Driver (default "default").
    *   `--ipam-opt`: Set custom IPAM driver specific options.
    *   `--ipv6`: Enable IPv6 networking.
    *   `--label`: Set metadata on a network.
    *   `-o, --opt`: Set driver specific options.
    *   `--scope`: Control the network's scope.
    *   `--subnet`: Subnet in CIDR format that represents a network segment.
*   `docker network disconnect`: Disconnect a container from a network.
    *   `-f, --force`: Force the container to disconnect from a network.
*   `docker network inspect`: Display detailed information on one or more networks.
    *   `-f, --format`: Format the output using the given Go template.
    *   `-v, --verbose`: Verbose output for diagnostics.
*   `docker network ls`: List networks.
    *   `-f, --filter`: Filter output based on conditions provided.
    *   `--format`: Pretty-print networks using a Go template.
    *   `--no-trunc`: Do not truncate the output.
    *   `-q, --quiet`: Only display network IDs.
*   `docker network rm`: Remove one or more networks.

### 13.22 `docker volume` (Detailed)
Manage volumes.
*   `docker volume create`: Create a volume.
    *   `-d, --driver`: Specify volume driver name (default "local").
    *   `--label`: Set metadata for a volume.
    *   `--name`: Specify volume name.
    *   `-o, --opt`: Set driver specific options.
*   `docker volume inspect`: Display detailed information on one or more volumes.
    *   `-f, --format`: Format the output using the given Go template.
*   `docker volume ls`: List volumes.
    *   `-f, --filter`: Filter output based on conditions provided.
    *   `--format`: Pretty-print volumes using a Go template.
    *   `-q, --quiet`: Only display volume names.
*   `docker volume prune`: Remove all unused local volumes.
    *   `--filter`: Provide filter values (e.g. 'label=<label>').
    *   `-f, --force`: Do not prompt for confirmation.
*   `docker volume rm`: Remove one or more volumes.
    *   `-f, --force`: Force the removal of one or more volumes.

### 13.23 `docker secret` (Detailed)
Manage Docker secrets.
*   `docker secret create`: Create a secret from a file or STDIN as content.
    *   `-d, --driver`: Secret driver.
    *   `-l, --label`: Secret labels.
    *   `--template-driver`: Template driver.
*   `docker secret inspect`: Display detailed information on one or more secrets.
    *   `-f, --format`: Format the output using the given Go template.
    *   `--pretty`: Print the information in a human friendly format.
*   `docker secret ls`: List secrets.
    *   `-f, --filter`: Filter output based on conditions provided.
    *   `--format`: Pretty-print secrets using a Go template.
    *   `-q, --quiet`: Only display IDs.
*   `docker secret rm`: Remove one or more secrets.

### 13.24 `docker config` (Detailed)
Manage Docker configs.
*   `docker config create`: Create a config from a file or STDIN.
    *   `-l, --label`: Config labels.
    *   `--template-driver`: Template driver.
*   `docker config inspect`: Display detailed information on one or more configs.
    *   `-f, --format`: Format the output using the given Go template.
    *   `--pretty`: Print the information in a human friendly format.
*   `docker config ls`: List configs.
    *   `-f, --filter`: Filter output based on conditions provided.
    *   `--format`: Pretty-print configs using a Go template.
    *   `-q, --quiet`: Only display IDs.
*   `docker config rm`: Remove one or more configs.

### 13.25 `docker context` (Detailed)
Manage contexts.
*   `docker context create`: Create a context.
    *   `--default-stack-orchestrator`: Default orchestrator for stack commands to use with this context (swarm|kubernetes|all).
    *   `--description`: Description of the context.
    *   `--docker`: set the docker endpoint.
    *   `--kubernetes`: set the kubernetes endpoint.
*   `docker context export`: Export a context to a tar or kubeconfig file.
*   `docker context import`: Import a context from a tar or zip file.
*   `docker context inspect`: Display detailed information on one or more contexts.
*   `docker context ls`: List contexts.
*   `docker context rm`: Remove one or more contexts.
*   `docker context update`: Update a context.
*   `docker context use`: Set the current docker context.

### 13.26 `docker buildx` (Detailed)
Docker Buildx is a CLI plugin that extends the docker command with the full support of the features provided by Moby BuildKit builder toolkit.
*   `docker buildx bake`: Build from a file.
*   `docker buildx build`: Start a build.
*   `docker buildx create`: Create a new builder instance.
*   `docker buildx du`: Disk usage.
*   `docker buildx imagetools`: Commands to work on images in registry.
*   `docker buildx inspect`: Inspect current builder instance.
*   `docker buildx ls`: List builder instances.
*   `docker buildx prune`: Remove build cache.
*   `docker buildx rm`: Remove a builder instance.
*   `docker buildx stop`: Stop builder instance.
*   `docker buildx use`: Set the current builder instance.
*   `docker buildx version`: Show buildx version information.

### 13.27 `docker scout` (Detailed)
Command line tool for Docker Scout.
*   `docker scout cache`: Manage Docker Scout cache.
*   `docker scout compare`: Compare two images and display differences.
*   `docker scout config`: Manage Docker Scout configuration.
*   `docker scout cves`: Display CVEs identified in a software artifact.
*   `docker scout enroll`: Enroll an organization with Docker Scout.
*   `docker scout environment`: Manage environments.
*   `docker scout integration`: Manage Docker Scout integrations.
*   `docker scout policy`: Evaluate policies against an image.
*   `docker scout push`: Push an image to a registry and analyze it.
*   `docker scout quickview`: Quick overview of an image.
*   `docker scout recommendations`: Display available base image updates and remediation recommendations.
*   `docker scout repo`: Manage Docker Scout repositories.
*   `docker scout sbom`: Display the SBOM of an image.
*   `docker scout version`: Show Docker Scout version information.

### 13.28 `docker compose` (Detailed)
Define and run multi-container applications with Docker.
*   `docker compose build`: Build or rebuild services.
*   `docker compose config`: Parse, resolve and render compose file in canonical format.
*   `docker compose cp`: Copy files/folders between a service container and the local filesystem.
*   `docker compose create`: Creates containers for a service.
*   `docker compose down`: Stop and remove containers, networks.
*   `docker compose events`: Receive real time events from containers.
*   `docker compose exec`: Execute a command in a running container.
*   `docker compose images`: List images used by the created containers.
*   `docker compose kill`: Force stop service containers.
*   `docker compose logs`: View output from containers.
*   `docker compose ls`: List running compose projects.
*   `docker compose pause`: Pause services.
*   `docker compose port`: Print the public port for a port binding.
*   `docker compose ps`: List containers.
*   `docker compose pull`: Pull service images.
*   `docker compose push`: Push service images.
*   `docker compose restart`: Restart service containers.
*   `docker compose rm`: Removes stopped service containers.
*   `docker compose run`: Run a one-off command on a service.
*   `docker compose scale`: Scale services.
*   `docker compose start`: Start services.
*   `docker compose stop`: Stop services.
*   `docker compose top`: Display the running processes.
*   `docker compose unpause`: Unpause services.
*   `docker compose up`: Create and start containers.
*   `docker compose version`: Show the Docker Compose version information.
*   `docker compose wait`: Block until the first service container stops.
*   `docker compose watch`: Watch build context for service and rebuild/refresh containers when files are updated.
*   `docker compose alpha`: Experimental commands.

## 14. Real-World Case Studies

### Case Study 1: The E-commerce Black Friday Scaling Event

**The Challenge:** An e-commerce platform experienced a 10x traffic spike during Black Friday. Their monolithic database container became a bottleneck, and the web frontend containers were constantly restarting due to OOM errors.

**The Super Specialist Intervention:**

1.  **Diagnosis:** Used `docker stats` to identify the web containers hitting their memory limits. Used `docker inspect` to confirm `OOMKilled: true`.
2.  **Immediate Mitigation:** Scaled the web frontend using `docker compose up --scale web=10 -d`.
3.  **Root Cause Analysis:** Used `docker exec -it` to profile the Node.js application inside the container, discovering a memory leak in the image processing library.
4.  **Long-Term Fix:**
    *   Updated the Dockerfile to use a more efficient base image (`node:18-alpine`).
    *   Implemented multi-stage builds to reduce image size from 1.2GB to 250MB, speeding up deployment times.
    *   Configured `daemon.json` with `live-restore: true` to ensure future daemon updates wouldn't take down the entire cluster.
    *   Implemented proper resource limits in `compose.yaml`:
        ```yaml
        deploy:
          resources:
            limits:
              cpus: '1.0'
              memory: 1G
            reservations:
              cpus: '0.5'
              memory: 512M
        ```

**The Result:** The platform handled the remaining Black Friday traffic without a single dropped request. Deployment times were reduced by 75%, and infrastructure costs were optimized by preventing runaway resource consumption.

### Case Study 2: The Cryptojacking Incident

**The Challenge:** A client noticed unusually high CPU usage on their Docker host. Their cloud provider alerted them to suspicious outbound network traffic.

**The Super Specialist Intervention:**

1.  **Detection:** Ran `docker top` on all running containers and identified an unknown process named `xmrig` (a known cryptocurrency miner) running inside a legacy application container.
2.  **Containment:** Immediately stopped the compromised container using `docker stop`.
3.  **Investigation:**
    *   Used `docker diff` to see what files the attacker had modified.
    *   Used `docker inspect` to check the container's configuration. Discovered it was running with `--privileged` and had port 22 (SSH) exposed with a weak password.
    *   Used `docker scout cves` on the base image and found multiple critical vulnerabilities.
4.  **Remediation:**
    *   Removed the `--privileged` flag.
    *   Removed the SSH server from the container (containers should be immutable; use `docker exec` for access).
    *   Updated the base image to a secure, patched version.
    *   Implemented read-only filesystems (`docker run --read-only`) to prevent attackers from downloading and executing malicious payloads.

**The Result:** The cryptojacking malware was eradicated. The client's Docker environment was secured against future attacks, and a policy was implemented to scan all images with Docker Scout before deployment.

## 15. Final Thoughts on Docker Mastery

Becoming a Docker Super Specialist is not just about memorizing commands; it's about understanding the underlying architecture of containerization. It's about knowing how namespaces provide isolation, how cgroups manage resources, and how union filesystems build images layer by layer.

When you combine this deep architectural understanding with the exhaustive command reference provided in this document, you transform from a user of Docker into a master of it. You gain the ability to debug the undebuggable, optimize the unoptimizable, and build infrastructure that is truly resilient, secure, and scalable.

Keep this reference close. Use it to solve the hard problems. And never stop exploring the depths of what Docker can do.

## 16. Appendix: Extended Configuration Examples

### 16.1 Advanced `daemon.json` Configurations

For highly specialized environments, the `daemon.json` can be tuned even further.

#### Configuring User Namespaces (userns-remap)
User namespaces provide an additional layer of security by mapping the `root` user inside the container to a non-privileged user on the host. This mitigates the impact of container breakout vulnerabilities.

```json
{
  "userns-remap": "default"
}
```
*Explanation:* This tells Docker to create a user and group named `dockremap` and map container users to this namespace.

#### Configuring Seccomp Profiles
Seccomp (Secure Computing Mode) restricts the system calls a container can make to the host kernel.

```json
{
  "seccomp-profile": "/etc/docker/seccomp/custom-profile.json"
}
```
*Explanation:* This applies a custom seccomp profile to all containers by default, providing fine-grained control over kernel interactions.

#### Configuring IPv6
To enable IPv6 support in Docker:

```json
{
  "ipv6": true,
  "fixed-cidr-v6": "2001:db8:1::/64"
}
```
*Explanation:* This enables IPv6 and assigns a specific subnet for Docker to use when allocating IPv6 addresses to containers.

### 16.2 Advanced `compose.yaml` Configurations

#### Using Extension Fields (YAML Anchors)
To avoid repeating configuration in large Compose files, use YAML anchors and aliases.

```yaml
version: '3.8'

x-logging: &default-logging
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "3"

services:
  web:
    image: nginx
    logging: *default-logging
  
  api:
    image: my-api
    logging: *default-logging
```
*Explanation:* The `x-logging` block defines a reusable configuration snippet. The `*default-logging` alias applies it to both the `web` and `api` services.

#### Using Profiles
Profiles allow you to define optional services that are only started when explicitly requested.

```yaml
version: '3.8'

services:
  web:
    image: nginx
  
  debug-tools:
    image: my-debug-tools
    profiles:
      - debug
```
*Explanation:* Running `docker compose up` will only start the `web` service. To start the debug tools, you must run `docker compose --profile debug up`.

### 16.3 The Evolution of Docker Build

Understanding the shift from legacy `docker build` to `docker buildx` (BuildKit) is crucial.

**Legacy Build:**
*   Sequential execution of instructions.
*   Limited caching capabilities.
*   Tied to the host architecture.

**BuildKit (Buildx):**
*   Concurrent execution of independent build stages.
*   Advanced caching (e.g., importing cache from a registry).
*   Native multi-platform builds (e.g., building for ARM on an AMD64 host).
*   Secret management during builds without leaving traces in the final image.

*Example: Using secrets with BuildKit*
```dockerfile
# syntax=docker/dockerfile:1
FROM alpine
RUN --mount=type=secret,id=mysecret cat /run/secrets/mysecret
```
*Build command:* `docker buildx build --secret id=mysecret,src=mysecret.txt .`

This ensures the secret is never baked into a layer, maintaining security.

## 17. Final Checklist for Production Readiness

Before deploying any Dockerized application to production, run through this checklist:

1.  [ ] **Images are minimal:** Using Alpine, Distroless, or scratch where possible.
2.  [ ] **Multi-stage builds used:** Build tools are not in the final image.
3.  [ ] **No root user:** `USER` directive is set in the Dockerfile.
4.  [ ] **Secrets are managed:** Using Docker Secrets or an external vault, not environment variables.
5.  [ ] **Resource limits set:** `cpus` and `memory` limits are defined in Compose or `docker run`.
6.  [ ] **Log rotation configured:** `daemon.json` has `max-size` and `max-file` set.
7.  [ ] **Healthchecks defined:** `HEALTHCHECK` in Dockerfile or `healthcheck` in Compose.
8.  [ ] **Restart policies set:** `restart: unless-stopped` or similar.
9.  [ ] **Images scanned:** `docker scout cves` reports no critical vulnerabilities.
10. [ ] **Live restore enabled:** `daemon.json` has `live-restore: true`.

By adhering to this checklist and utilizing the exhaustive command reference provided, you will operate at the level of a true Docker Super Specialist.



## 16. Appendix: Extended Configuration Examples

### 16.1 Advanced `daemon.json` Configurations

For highly specialized environments, the `daemon.json` can be tuned even further.

#### Configuring User Namespaces (userns-remap)
User namespaces provide an additional layer of security by mapping the `root` user inside the container to a non-privileged user on the host. This mitigates the impact of container breakout vulnerabilities.

```json
{
  "userns-remap": "default"
}
```
*Explanation:* This tells Docker to create a user and group named `dockremap` and map container users to this namespace.

#### Configuring Seccomp Profiles
Seccomp (Secure Computing Mode) restricts the system calls a container can make to the host kernel.

```json
{
  "seccomp-profile": "/etc/docker/seccomp/custom-profile.json"
}
```
*Explanation:* This applies a custom seccomp profile to all containers by default, providing fine-grained control over kernel interactions.

#### Configuring IPv6
To enable IPv6 support in Docker:

```json
{
  "ipv6": true,
  "fixed-cidr-v6": "2001:db8:1::/64"
}
```
*Explanation:* This enables IPv6 and assigns a specific subnet for Docker to use when allocating IPv6 addresses to containers.

### 16.2 Advanced `compose.yaml` Configurations

#### Using Extension Fields (YAML Anchors)
To avoid repeating configuration in large Compose files, use YAML anchors and aliases.

```yaml
version: '3.8'

x-logging: &default-logging
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "3"

services:
  web:
    image: nginx
    logging: *default-logging
  
  api:
    image: my-api
    logging: *default-logging
```
*Explanation:* The `x-logging` block defines a reusable configuration snippet. The `*default-logging` alias applies it to both the `web` and `api` services.

#### Using Profiles
Profiles allow you to define optional services that are only started when explicitly requested.

```yaml
version: '3.8'

services:
  web:
    image: nginx
  
  debug-tools:
    image: my-debug-tools
    profiles:
      - debug
```
*Explanation:* Running `docker compose up` will only start the `web` service. To start the debug tools, you must run `docker compose --profile debug up`.

### 16.3 The Evolution of Docker Build

Understanding the shift from legacy `docker build` to `docker buildx` (BuildKit) is crucial.

**Legacy Build:**
*   Sequential execution of instructions.
*   Limited caching capabilities.
*   Tied to the host architecture.

**BuildKit (Buildx):**
*   Concurrent execution of independent build stages.
*   Advanced caching (e.g., importing cache from a registry).
*   Native multi-platform builds (e.g., building for ARM on an AMD64 host).
*   Secret management during builds without leaving traces in the final image.

*Example: Using secrets with BuildKit*
```dockerfile
# syntax=docker/dockerfile:1
FROM alpine
RUN --mount=type=secret,id=mysecret cat /run/secrets/mysecret
```
*Build command:* `docker buildx build --secret id=mysecret,src=mysecret.txt .`

This ensures the secret is never baked into a layer, maintaining security.

## 17. Final Checklist for Production Readiness

Before deploying any Dockerized application to production, run through this checklist:

1.  [ ] **Images are minimal:** Using Alpine, Distroless, or scratch where possible.
2.  [ ] **Multi-stage builds used:** Build tools are not in the final image.
3.  [ ] **No root user:** `USER` directive is set in the Dockerfile.
4.  [ ] **Secrets are managed:** Using Docker Secrets or an external vault, not environment variables.
5.  [ ] **Resource limits set:** `cpus` and `memory` limits are defined in Compose or `docker run`.
6.  [ ] **Log rotation configured:** `daemon.json` has `max-size` and `max-file` set.
7.  [ ] **Healthchecks defined:** `HEALTHCHECK` in Dockerfile or `healthcheck` in Compose.
8.  [ ] **Restart policies set:** `restart: unless-stopped` or similar.
9.  [ ] **Images scanned:** `docker scout cves` reports no critical vulnerabilities.
10. [ ] **Live restore enabled:** `daemon.json` has `live-restore: true`.

By adhering to this checklist and utilizing the exhaustive command reference provided, you will operate at the level of a true Docker Super Specialist.

## 16. Appendix: Extended Configuration Examples

### 16.1 Advanced `daemon.json` Configurations

For highly specialized environments, the `daemon.json` can be tuned even further.

#### Configuring User Namespaces (userns-remap)
User namespaces provide an additional layer of security by mapping the `root` user inside the container to a non-privileged user on the host. This mitigates the impact of container breakout vulnerabilities.

```json
{
  "userns-remap": "default"
}
```
*Explanation:* This tells Docker to create a user and group named `dockremap` and map container users to this namespace.

#### Configuring Seccomp Profiles
Seccomp (Secure Computing Mode) restricts the system calls a container can make to the host kernel.

```json
{
  "seccomp-profile": "/etc/docker/seccomp/custom-profile.json"
}
```
*Explanation:* This applies a custom seccomp profile to all containers by default, providing fine-grained control over kernel interactions.

#### Configuring IPv6
To enable IPv6 support in Docker:

```json
{
  "ipv6": true,
  "fixed-cidr-v6": "2001:db8:1::/64"
}
```
*Explanation:* This enables IPv6 and assigns a specific subnet for Docker to use when allocating IPv6 addresses to containers.

### 16.2 Advanced `compose.yaml` Configurations

#### Using Extension Fields (YAML Anchors)
To avoid repeating configuration in large Compose files, use YAML anchors and aliases.

```yaml
version: '3.8'

x-logging: &default-logging
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "3"

services:
  web:
    image: nginx
    logging: *default-logging
  
  api:
    image: my-api
    logging: *default-logging
```
*Explanation:* The `x-logging` block defines a reusable configuration snippet. The `*default-logging` alias applies it to both the `web` and `api` services.

#### Using Profiles
Profiles allow you to define optional services that are only started when explicitly requested.

```yaml
version: '3.8'

services:
  web:
    image: nginx
  
  debug-tools:
    image: my-debug-tools
    profiles:
      - debug
```
*Explanation:* Running `docker compose up` will only start the `web` service. To start the debug tools, you must run `docker compose --profile debug up`.

### 16.3 The Evolution of Docker Build

Understanding the shift from legacy `docker build` to `docker buildx` (BuildKit) is crucial.

**Legacy Build:**
*   Sequential execution of instructions.
*   Limited caching capabilities.
*   Tied to the host architecture.

**BuildKit (Buildx):**
*   Concurrent execution of independent build stages.
*   Advanced caching (e.g., importing cache from a registry).
*   Native multi-platform builds (e.g., building for ARM on an AMD64 host).
*   Secret management during builds without leaving traces in the final image.

*Example: Using secrets with BuildKit*
```dockerfile
# syntax=docker/dockerfile:1
FROM alpine
RUN --mount=type=secret,id=mysecret cat /run/secrets/mysecret
```
*Build command:* `docker buildx build --secret id=mysecret,src=mysecret.txt .`

This ensures the secret is never baked into a layer, maintaining security.

## 17. Final Checklist for Production Readiness

Before deploying any Dockerized application to production, run through this checklist:

1.  [ ] **Images are minimal:** Using Alpine, Distroless, or scratch where possible.
2.  [ ] **Multi-stage builds used:** Build tools are not in the final image.
3.  [ ] **No root user:** `USER` directive is set in the Dockerfile.
4.  [ ] **Secrets are managed:** Using Docker Secrets or an external vault, not environment variables.
5.  [ ] **Resource limits set:** `cpus` and `memory` limits are defined in Compose or `docker run`.
6.  [ ] **Log rotation configured:** `daemon.json` has `max-size` and `max-file` set.
7.  [ ] **Healthchecks defined:** `HEALTHCHECK` in Dockerfile or `healthcheck` in Compose.
8.  [ ] **Restart policies set:** `restart: unless-stopped` or similar.
9.  [ ] **Images scanned:** `docker scout cves` reports no critical vulnerabilities.
10. [ ] **Live restore enabled:** `daemon.json` has `live-restore: true`.

By adhering to this checklist and utilizing the exhaustive command reference provided, you will operate at the level of a true Docker Super Specialist.

## 16. Appendix: Extended Configuration Examples

### 16.1 Advanced `daemon.json` Configurations

For highly specialized environments, the `daemon.json` can be tuned even further.

#### Configuring User Namespaces (userns-remap)
User namespaces provide an additional layer of security by mapping the `root` user inside the container to a non-privileged user on the host. This mitigates the impact of container breakout vulnerabilities.

```json
{
  "userns-remap": "default"
}
```
*Explanation:* This tells Docker to create a user and group named `dockremap` and map container users to this namespace.

#### Configuring Seccomp Profiles
Seccomp (Secure Computing Mode) restricts the system calls a container can make to the host kernel.

```json
{
  "seccomp-profile": "/etc/docker/seccomp/custom-profile.json"
}
```
*Explanation:* This applies a custom seccomp profile to all containers by default, providing fine-grained control over kernel interactions.

#### Configuring IPv6
To enable IPv6 support in Docker:

```json
{
  "ipv6": true,
  "fixed-cidr-v6": "2001:db8:1::/64"
}
```
*Explanation:* This enables IPv6 and assigns a specific subnet for Docker to use when allocating IPv6 addresses to containers.

### 16.2 Advanced `compose.yaml` Configurations

#### Using Extension Fields (YAML Anchors)
To avoid repeating configuration in large Compose files, use YAML anchors and aliases.

```yaml
version: '3.8'

x-logging: &default-logging
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "3"

services:
  web:
    image: nginx
    logging: *default-logging
  
  api:
    image: my-api
    logging: *default-logging
```
*Explanation:* The `x-logging` block defines a reusable configuration snippet. The `*default-logging` alias applies it to both the `web` and `api` services.

#### Using Profiles
Profiles allow you to define optional services that are only started when explicitly requested.

```yaml
version: '3.8'

services:
  web:
    image: nginx
  
  debug-tools:
    image: my-debug-tools
    profiles:
      - debug
```
*Explanation:* Running `docker compose up` will only start the `web` service. To start the debug tools, you must run `docker compose --profile debug up`.

### 16.3 The Evolution of Docker Build

Understanding the shift from legacy `docker build` to `docker buildx` (BuildKit) is crucial.

**Legacy Build:**
*   Sequential execution of instructions.
*   Limited caching capabilities.
*   Tied to the host architecture.

**BuildKit (Buildx):**
*   Concurrent execution of independent build stages.
*   Advanced caching (e.g., importing cache from a registry).
*   Native multi-platform builds (e.g., building for ARM on an AMD64 host).
*   Secret management during builds without leaving traces in the final image.

*Example: Using secrets with BuildKit*
```dockerfile
# syntax=docker/dockerfile:1
FROM alpine
RUN --mount=type=secret,id=mysecret cat /run/secrets/mysecret
```
*Build command:* `docker buildx build --secret id=mysecret,src=mysecret.txt .`

This ensures the secret is never baked into a layer, maintaining security.

## 17. Final Checklist for Production Readiness

Before deploying any Dockerized application to production, run through this checklist:

1.  [ ] **Images are minimal:** Using Alpine, Distroless, or scratch where possible.
2.  [ ] **Multi-stage builds used:** Build tools are not in the final image.
3.  [ ] **No root user:** `USER` directive is set in the Dockerfile.
4.  [ ] **Secrets are managed:** Using Docker Secrets or an external vault, not environment variables.
5.  [ ] **Resource limits set:** `cpus` and `memory` limits are defined in Compose or `docker run`.
6.  [ ] **Log rotation configured:** `daemon.json` has `max-size` and `max-file` set.
7.  [ ] **Healthchecks defined:** `HEALTHCHECK` in Dockerfile or `healthcheck` in Compose.
8.  [ ] **Restart policies set:** `restart: unless-stopped` or similar.
9.  [ ] **Images scanned:** `docker scout cves` reports no critical vulnerabilities.
10. [ ] **Live restore enabled:** `daemon.json` has `live-restore: true`.

By adhering to this checklist and utilizing the exhaustive command reference provided, you will operate at the level of a true Docker Super Specialist.
