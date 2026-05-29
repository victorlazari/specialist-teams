# Docker Super Specialist: Complete Configuration Reference

## 1. `compose.yaml` Complete Schema Reference

The `compose.yaml` file is the heart of Docker Compose. Below is the exhaustive reference for every top-level element and nested option.

### 1.1 Top-Level Elements

- `version`: (Deprecated) No longer required in Compose V2.
- `name`: Sets the project name. Overrides the directory name.
- `services`: Defines the containers to run.
- `networks`: Defines the networks to be created or used.
- `volumes`: Defines the persistent volumes.
- `configs`: Defines configuration files to be mounted.
- `secrets`: Defines sensitive data to be mounted securely.

### 1.2 `services` Attributes

Every service can have the following attributes:

| Attribute | Type | Default | Description | Example |
|---|---|---|---|---|
| `annotations` | map/list | None | Metadata for the container. | `annotations: { "com.example.foo": "bar" }` |
| `attach` | boolean | `true` | Whether to attach to the container's output. | `attach: false` |
| `build` | string/object | None | Configuration for building the image. | `build: ./dir` or `build: { context: ., dockerfile: Dockerfile.alt }` |
| `blkio_config` | object | None | Block IO configuration. | `blkio_config: { weight: 300 }` |
| `cpu_count` | integer | None | Number of usable CPUs. | `cpu_count: 2` |
| `cpu_percent` | integer | None | Usable percentage of available CPUs. | `cpu_percent: 50` |
| `cpu_shares` | integer | None | CPU shares (relative weight). | `cpu_shares: 73` |
| `cpu_period` | integer | None | CPU CFS (Completely Fair Scheduler) period. | `cpu_period: 100000` |
| `cpu_quota` | integer | None | CPU CFS quota. | `cpu_quota: 50000` |
| `cpu_rt_runtime` | integer | None | CPU real-time runtime. | `cpu_rt_runtime: 95000` |
| `cpu_rt_period` | integer | None | CPU real-time period. | `cpu_rt_period: 100000` |
| `cpus` | float | None | Number of CPUs. | `cpus: 1.5` |
| `cpuset` | string | None | CPUs in which to allow execution. | `cpuset: "0,1"` |
| `cap_add` | list | None | Add Linux capabilities. | `cap_add: ["SYS_ADMIN"]` |
| `cap_drop` | list | None | Drop Linux capabilities. | `cap_drop: ["ALL"]` |
| `cgroup` | string | None | Cgroup namespace mode. | `cgroup: "host"` |
| `cgroup_parent` | string | None | Optional parent cgroup. | `cgroup_parent: "m-executor-abcd"` |
| `command` | string/list | None | Override the default command. | `command: ["bundle", "exec", "thin", "-p", "3000"]` |
| `configs` | list | None | Grant access to configs. | `configs: ["my_config"]` |
| `container_name` | string | None | Custom container name. | `container_name: my-web-container` |
| `credential_spec` | object | None | Credential spec for managed service accounts (Windows). | `credential_spec: { file: "my-spec.json" }` |
| `depends_on` | list/object | None | Express dependency between services. | `depends_on: { db: { condition: service_healthy } }` |
| `deploy` | object | None | Configuration for deployment and resource limits. | `deploy: { replicas: 6 }` |
| `develop` | object | None | Configuration for development (Compose Watch). | `develop: { watch: [...] }` |
| `device_cgroup_rules` | list | None | Add rules to the cgroup allowed devices list. | `device_cgroup_rules: ["c 1:3 mr"]` |
| `devices` | list | None | Device mappings. | `devices: ["/dev/ttyUSB0:/dev/ttyUSB0"]` |
| `dns` | string/list | None | Custom DNS servers. | `dns: ["8.8.8.8", "9.9.9.9"]` |
| `dns_opt` | list | None | Custom DNS options. | `dns_opt: ["use-vc", "no-tld-query"]` |
| `dns_search` | string/list | None | Custom DNS search domains. | `dns_search: ["dc1.example.com"]` |
| `domainname` | string | None | Custom domain name. | `domainname: foo.com` |
| `entrypoint` | string/list | None | Override the default entrypoint. | `entrypoint: /code/entrypoint.sh` |
| `env_file` | string/list | None | Add environment variables from a file. | `env_file: .env` |
| `environment` | map/list | None | Add environment variables. | `environment: { RACK_ENV: development }` |
| `expose` | list | None | Expose ports without publishing them to the host. | `expose: ["3000"]` |
| `extends` | string/object | None | Extend another service. | `extends: { file: common.yml, service: webapp }` |
| `external_links` | list | None | Link to containers started outside this compose. | `external_links: ["redis_1", "project_db_1:mysql"]` |
| `extra_hosts` | list/map | None | Add hostname mappings. | `extra_hosts: ["somehost:162.242.195.82"]` |
| `group_add` | list | None | Add additional groups. | `group_add: ["mail"]` |
| `healthcheck` | object | None | Configure a check that's run to determine whether or not containers for this service are "healthy". | `healthcheck: { test: ["CMD", "curl", "-f", "http://localhost"] }` |
| `hostname` | string | None | Custom host name. | `hostname: foo` |
| `image` | string | None | Specify the image to start the container from. | `image: redis:alpine` |
| `init` | boolean | `false` | Run an init inside the container that forwards signals and reaps processes. | `init: true` |
| `ipc` | string | None | IPC namespace to use. | `ipc: host` |
| `isolation` | string | None | Specify a container's isolation technology. | `isolation: default` |
| `labels` | map/list | None | Add metadata to containers. | `labels: { com.example.description: "Accounting webapp" }` |
| `links` | list | None | Link to containers in another service. | `links: ["db", "db:database"]` |
| `logging` | object | None | Logging configuration for the service. | `logging: { driver: syslog, options: { syslog-address: "tcp://192.168.0.42:123" } }` |
| `mac_address` | string | None | MAC address. | `mac_address: 02:42:ac:11:65:43` |
| `mem_limit` | string | None | Memory limit. | `mem_limit: 1g` |
| `mem_reservation` | string | None | Memory soft limit. | `mem_reservation: 512m` |
| `mem_swappiness` | integer | None | Tune a container's memory swappiness behavior. | `mem_swappiness: 60` |
| `memswap_limit` | string | None | Swap limit equal to memory plus swap. | `memswap_limit: 2g` |
| `network_mode` | string | None | Network mode. | `network_mode: "host"` |
| `networks` | list/map | None | Networks to join. | `networks: ["frontend", "backend"]` |
| `oom_kill_disable` | boolean | `false` | Disable OOM Killer. | `oom_kill_disable: true` |
| `oom_score_adj` | integer | None | Tune the host's OOM preferences for containers. | `oom_score_adj: 500` |
| `pid` | string | None | PID namespace to use. | `pid: "host"` |
| `pids_limit` | integer | None | Tune a container's pids limit. | `pids_limit: 100` |
| `platform` | string | None | Target platform containers for this service will run on. | `platform: linux/amd64` |
| `ports` | list | None | Expose ports. | `ports: ["3000", "8000:8000", "9000:8080"]` |
| `privileged` | boolean | `false` | Give extended privileges to this container. | `privileged: true` |
| `profiles` | list | None | Define a list of named profiles for the service to be enabled under. | `profiles: ["frontend", "debug"]` |
| `pull_policy` | string | `always` | Define the decisions Compose makes when it starts to pull images. | `pull_policy: missing` |
| `read_only` | boolean | `false` | Mount the container's root filesystem as read only. | `read_only: true` |
| `restart` | string | `no` | Restart policy. | `restart: always` |
| `runtime` | string | None | Specify the runtime to use for the container. | `runtime: runc` |
| `scale` | integer | `1` | Specify the default number of containers to deploy for this service. | `scale: 3` |
| `secrets` | list | None | Grant access to secrets on a per-service basis. | `secrets: ["my_secret", "my_other_secret"]` |
| `security_opt` | list | None | Override the default labeling scheme for each container. | `security_opt: ["label:user:USER", "label:role:ROLE"]` |
| `shm_size` | string | None | Size of `/dev/shm`. | `shm_size: '2gb'` |
| `stdin_open` | boolean | `false` | Keep STDIN open even if not attached. | `stdin_open: true` |
| `stop_grace_period` | string | `10s` | Specify how long to wait when attempting to stop a container if it doesn't handle SIGTERM. | `stop_grace_period: 1m30s` |
| `stop_signal` | string | `SIGTERM` | Set an alternative signal to stop the container. | `stop_signal: SIGUSR1` |
| `storage_opt` | map | None | Storage driver options for this service. | `storage_opt: { size: '120G' }` |
| `sysctls` | map/list | None | Kernel parameters to set in the container. | `sysctls: { net.core.somaxconn: 1024 }` |
| `tmpfs` | string/list | None | Mount a temporary file system inside the container. | `tmpfs: /run` |
| `tty` | boolean | `false` | Allocate a pseudo-TTY. | `tty: true` |
| `ulimits` | map | None | Override the default ulimits for a container. | `ulimits: { nproc: 65535, nofile: { soft: 20000, hard: 40000 } }` |
| `user` | string | None | Override the user used to run the container process. | `user: "1000:1000"` |
| `userns_mode` | string | None | Disable the user namespace for this service, if Docker daemon is configured with user namespaces. | `userns_mode: "host"` |
| `uts` | string | None | UTS namespace to use. | `uts: "host"` |
| `volumes` | list | None | Mount host paths or named volumes, specified as sub-options to a service. | `volumes: ["/var/lib/mysql", "./cache:/tmp/cache", "datavolume:/var/lib/mysql"]` |
| `volumes_from` | list | None | Mount all of the volumes from another service or container. | `volumes_from: ["service_name", "container_name"]` |
| `working_dir` | string | None | Override the container's working directory. | `working_dir: /code` |

### 1.3 `networks` Attributes

| Attribute | Type | Default | Description | Example |
|---|---|---|---|---|
| `driver` | string | `bridge` | Specify which driver should be used for this network. | `driver: overlay` |
| `driver_opts` | map | None | Specify a list of options as key-value pairs to pass to the driver. | `driver_opts: { com.docker.network.bridge.name: br_1 }` |
| `attachable` | boolean | `false` | Only used when the driver is set to `overlay`. If set to `true`, then standalone containers can attach to this network. | `attachable: true` |
| `enable_ipv6` | boolean | `false` | Enable IPv6 networking. | `enable_ipv6: true` |
| `internal` | boolean | `false` | By default, Docker also connects a bridge network to it to provide external connectivity. If you want to create an externally isolated overlay network, you can set this option to `true`. | `internal: true` |
| `labels` | map/list | None | Add metadata to containers. | `labels: { com.example.description: "Financial transaction network" }` |
| `external` | boolean | `false` | If set to `true`, specifies that this network has been created outside of Compose. | `external: true` |
| `name` | string | None | Set a custom name for this network. | `name: my-app-net` |
| `ipam` | object | None | Specify custom IPAM config. | `ipam: { driver: default, config: [{ subnet: "172.28.0.0/16" }] }` |

### 1.4 `volumes` Attributes

| Attribute | Type | Default | Description | Example |
|---|---|---|---|---|
| `driver` | string | `local` | Specify which volume driver should be used for this volume. | `driver: foobar` |
| `driver_opts` | map | None | Specify a list of options as key-value pairs to pass to the driver for this volume. | `driver_opts: { type: "nfs", o: "addr=10.40.0.199,nolock,soft,rw", device: ":/docker/example" }` |
| `external` | boolean | `false` | If set to `true`, specifies that this volume has been created outside of Compose. | `external: true` |
| `labels` | map/list | None | Add metadata to containers. | `labels: { com.example.description: "Database volume" }` |
| `name` | string | None | Set a custom name for this volume. | `name: my-app-data` |

### 1.5 `configs` Attributes

| Attribute | Type | Default | Description | Example |
|---|---|---|---|---|
| `file` | string | None | The config is created with the contents of the file at the specified path. | `file: ./my_config.txt` |
| `external` | boolean | `false` | If set to `true`, specifies that this config has already been created. | `external: true` |
| `name` | string | None | The name of the config object in Docker. | `name: my_config` |
| `content` | string | None | The content of the config. | `content: | 
  server {
    listen 80;
  }` |

### 1.6 `secrets` Attributes

| Attribute | Type | Default | Description | Example |
|---|---|---|---|---|
| `file` | string | None | The secret is created with the contents of the file at the specified path. | `file: ./my_secret.txt` |
| `environment` | string | None | The secret is created with the value of an environment variable. | `environment: "MY_SECRET"` |
| `external` | boolean | `false` | If set to `true`, specifies that this secret has already been created. | `external: true` |
| `name` | string | None | The name of the secret object in Docker. | `name: my_secret` |

## 2. Dockerfile Instruction Reference

A complete reference for every Dockerfile instruction.

### `FROM`
Initializes a new build stage and sets the Base Image for subsequent instructions.
- Syntax: `FROM [--platform=<platform>] <image> [AS <name>]` or `FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]` or `FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]`
- Example: `FROM --platform=linux/amd64 ubuntu:22.04 AS builder`

### `RUN`
Executes any commands in a new layer on top of the current image and commits the results.
- Syntax: `RUN <command>` (shell form) or `RUN ["executable", "param1", "param2"]` (exec form)
- Flags: `--mount=type=cache|bind|secret|ssh`, `--network=default|none|host`
- Example: `RUN --mount=type=cache,target=/var/cache/apt apt-get update && apt-get install -y curl`

### `CMD`
Provides defaults for an executing container. There can only be one `CMD` instruction in a Dockerfile.
- Syntax: `CMD ["executable","param1","param2"]` (exec form, preferred) or `CMD ["param1","param2"]` (as default parameters to ENTRYPOINT) or `CMD command param1 param2` (shell form)
- Example: `CMD ["node", "server.js"]`

### `LABEL`
Adds metadata to an image.
- Syntax: `LABEL <key>=<value> <key>=<value> <key>=<value> ...`
- Example: `LABEL org.opencontainers.image.authors="team@example.com"`

### `EXPOSE`
Informs Docker that the container listens on the specified network ports at runtime.
- Syntax: `EXPOSE <port> [<port>/<protocol>...]`
- Example: `EXPOSE 80/tcp 80/udp`

### `ENV`
Sets the environment variable `<key>` to the value `<value>`.
- Syntax: `ENV <key>=<value> ...`
- Example: `ENV NODE_ENV=production PORT=3000`

### `ADD`
Copies new files, directories or remote file URLs from `<src>` and adds them to the filesystem of the image at the path `<dest>`.
- Syntax: `ADD [--chown=<user>:<group>] [--chmod=<perms>] [--checksum=<checksum>] <src>... <dest>`
- Example: `ADD https://example.com/big.tar.xz /usr/src/things/`

### `COPY`
Copies new files or directories from `<src>` and adds them to the filesystem of the container at the path `<dest>`.
- Syntax: `COPY [--chown=<user>:<group>] [--chmod=<perms>] <src>... <dest>`
- Example: `COPY --chown=node:node package*.json ./`

### `ENTRYPOINT`
Allows you to configure a container that will run as an executable.
- Syntax: `ENTRYPOINT ["executable", "param1", "param2"]` (exec form, preferred) or `ENTRYPOINT command param1 param2` (shell form)
- Example: `ENTRYPOINT ["docker-entrypoint.sh"]`

### `VOLUME`
Creates a mount point with the specified name and marks it as holding externally mounted volumes from native host or other containers.
- Syntax: `VOLUME ["/data"]`
- Example: `VOLUME /var/lib/mysql`

### `USER`
Sets the user name (or UID) and optionally the user group (or GID) to use when running the image and for any `RUN`, `CMD` and `ENTRYPOINT` instructions that follow it in the Dockerfile.
- Syntax: `USER <user>[:<group>]` or `USER <UID>[:<GID>]`
- Example: `USER 1000:1000`

### `WORKDIR`
Sets the working directory for any `RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD` instructions that follow it in the Dockerfile.
- Syntax: `WORKDIR /path/to/workdir`
- Example: `WORKDIR /app`

### `ARG`
Defines a variable that users can pass at build-time to the builder with the `docker build` command using the `--build-arg <varname>=<value>` flag.
- Syntax: `ARG <name>[=<default value>]`
- Example: `ARG VERSION=latest`

### `ONBUILD`
Adds to the image a trigger instruction to be executed at a later time, when the image is used as the base for another build.
- Syntax: `ONBUILD <INSTRUCTION>`
- Example: `ONBUILD COPY . /app/src`

### `STOPSIGNAL`
Sets the system call signal that will be sent to the container to exit.
- Syntax: `STOPSIGNAL signal`
- Example: `STOPSIGNAL SIGKILL`

### `HEALTHCHECK`
Tells Docker how to test a container to check that it is still working.
- Syntax: `HEALTHCHECK [OPTIONS] CMD command` or `HEALTHCHECK NONE`
- Options: `--interval=DURATION` (default: 30s), `--timeout=DURATION` (default: 30s), `--start-period=DURATION` (default: 0s), `--retries=N` (default: 3)
- Example: `HEALTHCHECK --interval=5m --timeout=3s CMD curl -f http://localhost/ || exit 1`

### `SHELL`
Allows the default shell used for the shell form of commands to be overridden.
- Syntax: `SHELL ["executable", "parameters"]`
- Example: `SHELL ["powershell", "-command"]`

## 3. `daemon.json` Complete Reference

The `daemon.json` file configures the Docker daemon.

| Field | Type | Default | Description |
|---|---|---|---|
| `storage-driver` | string | `overlay2` | The storage driver to use. |
| `log-driver` | string | `json-file` | The default logging driver. |
| `log-opts` | map | None | Options for the logging driver. |
| `default-address-pools` | list | None | Default address pools for node networks. |
| `dns` | list | None | DNS servers to use. |
| `registry-mirrors` | list | None | Registry mirrors to use. |
| `insecure-registries` | list | None | Insecure registries to allow. |
| `live-restore` | boolean | `false` | Enable live restore of docker when containers are still running. |
| `default-runtime` | string | `runc` | Default OCI runtime for containers. |
| `runtimes` | map | None | Register additional OCI runtimes. |
| `features` | map | None | Enable/disable specific features. |
| `builder` | map | None | BuildKit configuration. |
| `containerd` | string | None | Path to containerd socket. |
| `default-cgroupns-mode` | string | `private` | Default cgroup namespace mode. |
| `exec-opts` | list | None | Execution options. |
| `experimental` | boolean | `false` | Enable experimental features. |
| `fixed-cidr` | string | None | IPv4 subnet for fixed IPs. |
| `fixed-cidr-v6` | string | None | IPv6 subnet for fixed IPs. |
| `group` | string | `docker` | Group for the unix socket. |
| `hosts` | list | None | Daemon socket(s) to connect to. |
| `icc` | boolean | `true` | Enable inter-container communication. |
| `ip` | string | `0.0.0.0` | Default IP when binding container ports. |
| `ip-forward` | boolean | `true` | Enable net.ipv4.ip_forward. |
| `iptables` | boolean | `true` | Enable addition of iptables rules. |
| `ip-masq` | boolean | `true` | Enable IP masquerading. |
| `labels` | list | None | Daemon labels. |
| `max-concurrent-downloads` | integer | `3` | Max concurrent downloads. |
| `max-concurrent-uploads` | integer | `5` | Max concurrent uploads. |
| `max-download-attempts` | integer | `5` | Max download attempts. |
| `metrics-addr` | string | None | Address to serve metrics API. |
| `no-new-privileges` | boolean | `false` | Set no-new-privileges by default for new containers. |
| `oom-score-adjust` | integer | `-500` | Set the oom_score_adj for the daemon. |
| `pidfile` | string | `/var/run/docker.pid` | Path to use for daemon PID file. |
| `raw-logs` | boolean | `false` | Full timestamps without ANSI coloring. |
| `seccomp-profile` | string | None | Path to seccomp profile. |
| `selinux-enabled` | boolean | `false` | Enable selinux support. |
| `shutdown-timeout` | integer | `15` | Default timeout for stopping containers. |
| `tls` | boolean | `false` | Use TLS; implied by --tlsverify. |
| `tlscacert` | string | `~/.docker/ca.pem` | Trust certs signed only by this CA. |
| `tlscert` | string | `~/.docker/cert.pem` | Path to TLS certificate file. |
| `tlskey` | string | `~/.docker/key.pem` | Path to TLS key file. |
| `tlsverify` | boolean | `false` | Use TLS and verify the remote. |
| `userland-proxy` | boolean | `true` | Use userland proxy for loopback traffic. |
| `userns-remap` | string | None | User namespace remapping. |

## 4. `.dockerignore` Patterns

The `.dockerignore` file excludes files and directories from the build context.

### Syntax
- `#` for comments.
- `*` matches any sequence of non-separator characters.
- `?` matches any single non-separator character.
- `**` matches any number of directories.
- `!` negates a pattern.

### Common Patterns

**Node.js:**
```dockerignore
node_modules
npm-debug.log
Dockerfile
.dockerignore
.git
.gitignore
README.md
```

**Python:**
```dockerignore
__pycache__
*.pyc
*.pyo
*.pyd
.Python
env/
venv/
.env
Dockerfile
.dockerignore
.git
```

**Go:**
```dockerignore
bin/
obj/
*.exe
*.dll
*.so
*.dylib
Dockerfile
.dockerignore
.git
```

## 5. BuildKit Configuration (`buildkitd.toml`)

BuildKit can be configured via `buildkitd.toml`.

```toml
debug = true
# root is where all buildkit state is stored.
root = "/var/lib/buildkit"
# insecure-entitlements allows insecure entitlements, disabled by default.
insecure-entitlements = [ "network.host", "security.insecure" ]

[grpc]
  address = [ "tcp://0.0.0.0:1234" ]
  # debugAddress is address for attaching go pprof and expvar.
  debugAddress = "0.0.0.0:6060"
  uid = 0
  gid = 0
  [grpc.tls]
    cert = "/etc/buildkit/tls.crt"
    key = "/etc/buildkit/tls.key"
    ca = "/etc/buildkit/tlsca.crt"

[worker.oci]
  enabled = true
  # platforms is manually configure platforms, auto-detected by default.
  platforms = [ "linux/amd64", "linux/arm64" ]
  snapshotter = "auto" # overlayfs or native, default auto will try to use overlayfs
  rootless = false # see docs/rootless.md for more details on rootless mode.
  # Whether run subprocesses in main cgroup or create top-level cgroup.
  # Default is "cgroupfs" when not running rootless.
  cgroup-parent = "cgroupfs"
  # gc keeps/frees disk space.
  gc = true
  gckeepstorage = 9000
  [[worker.oci.gcpolicy]]
    keepBytes = 512000000
    keepDuration = 172800
    filters = [ "type==source.local", "type==exec.cachemount", "type==source.git.checkout"]
  [[worker.oci.gcpolicy]]
    all = true
    keepBytes = 1024000000

[worker.containerd]
  address = "/run/containerd/containerd.sock"
  enabled = true
  platforms = [ "linux/amd64", "linux/arm64" ]
  namespace = "buildkit"
  gc = true
  # gckeepstorage sets storage limit for default gc profile, in MB.
  gckeepstorage = 9000

[registry."docker.io"]
  mirrors = ["YOUR_REGISTRY_MIRROR"]
  http = true
  insecure = true
```

## 6. Docker Context Configuration

Docker contexts allow you to manage multiple Docker environments.

- Create a context: `docker context create my-context --docker "host=ssh://user@remote-host"`
- Use a context: `docker context use my-context`
- List contexts: `docker context ls`
- Inspect a context: `docker context inspect my-context`

## 7. Registry Configuration (`config.yml`)

Configuration for a private Docker registry.

```yaml
version: 0.1
log:
  fields:
    service: registry
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
```

## 8. Production `compose.yaml` Templates

### 8.1 Web App Stack

```yaml
services:
  webapp:
    image: YOUR_REGISTRY/webapp:v1.2.3
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    environment:
      - NODE_ENV=production
    secrets:
      - db_password
    networks:
      - frontend
      - backend
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

networks:
  frontend:
  backend:
    internal: true

secrets:
  db_password:
    external: true
```

### 8.2 Database Stack

```yaml
services:
  db:
    image: postgres:15-alpine
    restart: always
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
      POSTGRES_DB: myapp
    volumes:
      - db-data:/var/lib/postgresql/data
    secrets:
      - db_password
    networks:
      - backend
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 4G
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin -d myapp"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  db-data:

networks:
  backend:
    internal: true

secrets:
  db_password:
    external: true
```

### 8.3 Monitoring Stack (Prometheus/Grafana)

```yaml
services:
  prometheus:
    image: prom/prometheus:v2.45.0
    restart: unless-stopped
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
    ports:
      - "9090:9090"
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:10.0.3
    restart: unless-stopped
    volumes:
      - grafana-data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD__FILE=/run/secrets/grafana_password
    secrets:
      - grafana_password
    ports:
      - "3000:3000"
    networks:
      - monitoring
    depends_on:
      - prometheus

volumes:
  prometheus-data:
  grafana-data:

networks:
  monitoring:

secrets:
  grafana_password:
    external: true
```

### 8.4 ELK Stack

```yaml
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.9.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - es-data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
    networks:
      - elk

  logstash:
    image: docker.elastic.co/logstash/logstash:8.9.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf:ro
    ports:
      - "5044:5044"
    networks:
      - elk
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:8.9.0
    ports:
      - "5601:5601"
    networks:
      - elk
    depends_on:
      - elasticsearch

volumes:
  es-data:

networks:
  elk:
```

## 9. Troubleshooting and Optimization

### 9.1 Common Errors and Solutions

- **"port already in use"**: Use `lsof -i :PORT` to find the conflicting process, or change the port mapping in `compose.yaml`.
- **"no space left on device"**: Run `docker system prune -a --volumes` to clear unused data. Check the `overlay2` directory size.
- **"OOM killed"**: The container exceeded its memory limit. Increase the `mem_limit` in `compose.yaml` or optimize the application's memory usage.
- **"permission denied"**: Check the user/group permissions of the mounted volumes. Ensure the container user has access.
- **"network not found"**: Run `docker network create <network_name>` or ensure the network is defined in `compose.yaml`.
- **"image not found"**: Verify the registry URL, image tag, and `pull_policy`. Ensure you are logged in to the registry.
- **"container unhealthy"**: Check the `healthcheck` command and logs. Increase the `timeout` or `start_period` if the application takes longer to start.
- **"bind mount permission denied"**: On SELinux systems, append `:z` or `:Z` to the volume mount path (e.g., `./data:/data:z`).
- **"DNS resolution failed"**: Check the host's DNS settings or configure custom DNS servers in `daemon.json` or `compose.yaml`.
- **"cannot start service"**: Check `depends_on` conditions. Ensure required services are healthy before starting dependent services.
- **"exec format error"**: The image architecture does not match the host architecture (e.g., running an ARM image on an AMD64 host). Use `docker buildx` to build multi-platform images.
- **"context deadline exceeded"**: Increase the timeout for Docker commands or check network connectivity to the registry.

### 9.2 Cost and Time Optimization

- **Multi-stage builds**: Use multi-stage builds to create smaller final images. This reduces pull times and storage costs.
- **Layer caching**: Order Dockerfile instructions from least frequently changed to most frequently changed to maximize cache hits.
- **BuildKit cache mounts**: Use `--mount=type=cache` to cache package manager downloads (e.g., `apt`, `npm`, `pip`) between builds.
- **Parallel builds**: Use `docker compose build --parallel` to build multiple services concurrently.
- **Image pull policy**: Set `pull_policy: if-not-present` to avoid unnecessary image pulls.
- **Resource limits**: Set CPU and memory limits to prevent runaway containers from consuming all host resources.
- **Logging**: Configure log rotation (`max-size`, `max-file`) to prevent log files from filling up the disk.
- **Prune**: Regularly run `docker system prune` and `docker volume prune` to remove unused resources.
- **Compose profiles**: Use profiles to start only the services needed for a specific environment or task.
- **Compose watch**: Use `develop: watch` for fast iteration during development without rebuilding images.

### 9.3 Security Hardening

- **Non-root user**: Run containers as a non-root user (`USER 1000:1000`).
- **Read-only root filesystem**: Set `read_only: true` to prevent modifications to the container's root filesystem.
- **Drop capabilities**: Drop all Linux capabilities (`cap_drop: ["ALL"]`) and add only the necessary ones.
- **No new privileges**: Set `security_opt: ["no-new-privileges:true"]` to prevent processes from gaining additional privileges.
- **Seccomp profiles**: Use custom seccomp profiles to restrict system calls.
- **Resource limits**: Enforce CPU, memory, and PID limits to prevent denial-of-service attacks.
- **No privileged mode**: Avoid using `privileged: true` unless absolutely necessary.
- **Minimal base images**: Use minimal base images like `alpine`, `distroless`, or `scratch` to reduce the attack surface.
- **Secrets management**: Use Docker secrets instead of environment variables for sensitive data.
- **Network segmentation**: Use internal networks to isolate backend services from the public internet.

### 9.4 Upgrade Strategies

- **Blue-green deployment**: Run the new version alongside the old version and switch traffic when ready.
- **Rolling update**: Use `update_config` with `parallelism` and `delay` to update containers one by one.
- **Canary release**: Route a small percentage of traffic to the new version to test it before a full rollout.
- **Database migrations**: Run database migrations as an init container or a pre-start hook before starting the application.
- **Rollback**: Configure `rollback_config` to automatically roll back to the previous version if the update fails.
- **Zero-downtime**: Use `order: start-first` in `update_config` to start the new container before stopping the old one.
# Docker Super Specialist: Complete Configuration Reference

## 1. `compose.yaml` Complete Schema Reference

The `compose.yaml` file is the heart of Docker Compose. Below is the exhaustive reference for every top-level element and nested option.

### 1.1 Top-Level Elements

- `version`: (Deprecated) No longer required in Compose V2.
- `name`: Sets the project name. Overrides the directory name.
- `services`: Defines the containers to run.
- `networks`: Defines the networks to be created or used.
- `volumes`: Defines the persistent volumes.
- `configs`: Defines configuration files to be mounted.
- `secrets`: Defines sensitive data to be mounted securely.

### 1.2 `services` Attributes

Every service can have the following attributes:

| Attribute | Type | Default | Description | Example |
|---|---|---|---|---|
| `annotations` | map/list | None | Metadata for the container. | `annotations: { "com.example.foo": "bar" }` |
| `attach` | boolean | `true` | Whether to attach to the container's output. | `attach: false` |
| `build` | string/object | None | Configuration for building the image. | `build: ./dir` or `build: { context: ., dockerfile: Dockerfile.alt }` |
| `blkio_config` | object | None | Block IO configuration. | `blkio_config: { weight: 300 }` |
| `cpu_count` | integer | None | Number of usable CPUs. | `cpu_count: 2` |
| `cpu_percent` | integer | None | Usable percentage of available CPUs. | `cpu_percent: 50` |
| `cpu_shares` | integer | None | CPU shares (relative weight). | `cpu_shares: 73` |
| `cpu_period` | integer | None | CPU CFS (Completely Fair Scheduler) period. | `cpu_period: 100000` |
| `cpu_quota` | integer | None | CPU CFS quota. | `cpu_quota: 50000` |
| `cpu_rt_runtime` | integer | None | CPU real-time runtime. | `cpu_rt_runtime: 95000` |
| `cpu_rt_period` | integer | None | CPU real-time period. | `cpu_rt_period: 100000` |
| `cpus` | float | None | Number of CPUs. | `cpus: 1.5` |
| `cpuset` | string | None | CPUs in which to allow execution. | `cpuset: "0,1"` |
| `cap_add` | list | None | Add Linux capabilities. | `cap_add: ["SYS_ADMIN"]` |
| `cap_drop` | list | None | Drop Linux capabilities. | `cap_drop: ["ALL"]` |
| `cgroup` | string | None | Cgroup namespace mode. | `cgroup: "host"` |
| `cgroup_parent` | string | None | Optional parent cgroup. | `cgroup_parent: "m-executor-abcd"` |
| `command` | string/list | None | Override the default command. | `command: ["bundle", "exec", "thin", "-p", "3000"]` |
| `configs` | list | None | Grant access to configs. | `configs: ["my_config"]` |
| `container_name` | string | None | Custom container name. | `container_name: my-web-container` |
| `credential_spec` | object | None | Credential spec for managed service accounts (Windows). | `credential_spec: { file: "my-spec.json" }` |
| `depends_on` | list/object | None | Express dependency between services. | `depends_on: { db: { condition: service_healthy } }` |
| `deploy` | object | None | Configuration for deployment and resource limits. | `deploy: { replicas: 6 }` |
| `develop` | object | None | Configuration for development (Compose Watch). | `develop: { watch: [...] }` |
| `device_cgroup_rules` | list | None | Add rules to the cgroup allowed devices list. | `device_cgroup_rules: ["c 1:3 mr"]` |
| `devices` | list | None | Device mappings. | `devices: ["/dev/ttyUSB0:/dev/ttyUSB0"]` |
| `dns` | string/list | None | Custom DNS servers. | `dns: ["8.8.8.8", "9.9.9.9"]` |
| `dns_opt` | list | None | Custom DNS options. | `dns_opt: ["use-vc", "no-tld-query"]` |
| `dns_search` | string/list | None | Custom DNS search domains. | `dns_search: ["dc1.example.com"]` |
| `domainname` | string | None | Custom domain name. | `domainname: foo.com` |
| `entrypoint` | string/list | None | Override the default entrypoint. | `entrypoint: /code/entrypoint.sh` |
| `env_file` | string/list | None | Add environment variables from a file. | `env_file: .env` |
| `environment` | map/list | None | Add environment variables. | `environment: { RACK_ENV: development }` |
| `expose` | list | None | Expose ports without publishing them to the host. | `expose: ["3000"]` |
| `extends` | string/object | None | Extend another service. | `extends: { file: common.yml, service: webapp }` |
| `external_links` | list | None | Link to containers started outside this compose. | `external_links: ["redis_1", "project_db_1:mysql"]` |
| `extra_hosts` | list/map | None | Add hostname mappings. | `extra_hosts: ["somehost:162.242.195.82"]` |
| `group_add` | list | None | Add additional groups. | `group_add: ["mail"]` |
| `healthcheck` | object | None | Configure a check that's run to determine whether or not containers for this service are "healthy". | `healthcheck: { test: ["CMD", "curl", "-f", "http://localhost"] }` |
| `hostname` | string | None | Custom host name. | `hostname: foo` |
| `image` | string | None | Specify the image to start the container from. | `image: redis:alpine` |
| `init` | boolean | `false` | Run an init inside the container that forwards signals and reaps processes. | `init: true` |
| `ipc` | string | None | IPC namespace to use. | `ipc: host` |
| `isolation` | string | None | Specify a container's isolation technology. | `isolation: default` |
| `labels` | map/list | None | Add metadata to containers. | `labels: { com.example.description: "Accounting webapp" }` |
| `links` | list | None | Link to containers in another service. | `links: ["db", "db:database"]` |
| `logging` | object | None | Logging configuration for the service. | `logging: { driver: syslog, options: { syslog-address: "tcp://192.168.0.42:123" } }` |
| `mac_address` | string | None | MAC address. | `mac_address: 02:42:ac:11:65:43` |
| `mem_limit` | string | None | Memory limit. | `mem_limit: 1g` |
| `mem_reservation` | string | None | Memory soft limit. | `mem_reservation: 512m` |
| `mem_swappiness` | integer | None | Tune a container's memory swappiness behavior. | `mem_swappiness: 60` |
| `memswap_limit` | string | None | Swap limit equal to memory plus swap. | `memswap_limit: 2g` |
| `network_mode` | string | None | Network mode. | `network_mode: "host"` |
| `networks` | list/map | None | Networks to join. | `networks: ["frontend", "backend"]` |
| `oom_kill_disable` | boolean | `false` | Disable OOM Killer. | `oom_kill_disable: true` |
| `oom_score_adj` | integer | None | Tune the host's OOM preferences for containers. | `oom_score_adj: 500` |
| `pid` | string | None | PID namespace to use. | `pid: "host"` |
| `pids_limit` | integer | None | Tune a container's pids limit. | `pids_limit: 100` |
| `platform` | string | None | Target platform containers for this service will run on. | `platform: linux/amd64` |
| `ports` | list | None | Expose ports. | `ports: ["3000", "8000:8000", "9000:8080"]` |
| `privileged` | boolean | `false` | Give extended privileges to this container. | `privileged: true` |
| `profiles` | list | None | Define a list of named profiles for the service to be enabled under. | `profiles: ["frontend", "debug"]` |
| `pull_policy` | string | `always` | Define the decisions Compose makes when it starts to pull images. | `pull_policy: missing` |
| `read_only` | boolean | `false` | Mount the container's root filesystem as read only. | `read_only: true` |
| `restart` | string | `no` | Restart policy. | `restart: always` |
| `runtime` | string | None | Specify the runtime to use for the container. | `runtime: runc` |
| `scale` | integer | `1` | Specify the default number of containers to deploy for this service. | `scale: 3` |
| `secrets` | list | None | Grant access to secrets on a per-service basis. | `secrets: ["my_secret", "my_other_secret"]` |
| `security_opt` | list | None | Override the default labeling scheme for each container. | `security_opt: ["label:user:USER", "label:role:ROLE"]` |
| `shm_size` | string | None | Size of `/dev/shm`. | `shm_size: '2gb'` |
| `stdin_open` | boolean | `false` | Keep STDIN open even if not attached. | `stdin_open: true` |
| `stop_grace_period` | string | `10s` | Specify how long to wait when attempting to stop a container if it doesn't handle SIGTERM. | `stop_grace_period: 1m30s` |
| `stop_signal` | string | `SIGTERM` | Set an alternative signal to stop the container. | `stop_signal: SIGUSR1` |
| `storage_opt` | map | None | Storage driver options for this service. | `storage_opt: { size: '120G' }` |
| `sysctls` | map/list | None | Kernel parameters to set in the container. | `sysctls: { net.core.somaxconn: 1024 }` |
| `tmpfs` | string/list | None | Mount a temporary file system inside the container. | `tmpfs: /run` |
| `tty` | boolean | `false` | Allocate a pseudo-TTY. | `tty: true` |
| `ulimits` | map | None | Override the default ulimits for a container. | `ulimits: { nproc: 65535, nofile: { soft: 20000, hard: 40000 } }` |
| `user` | string | None | Override the user used to run the container process. | `user: "1000:1000"` |
| `userns_mode` | string | None | Disable the user namespace for this service, if Docker daemon is configured with user namespaces. | `userns_mode: "host"` |
| `uts` | string | None | UTS namespace to use. | `uts: "host"` |
| `volumes` | list | None | Mount host paths or named volumes, specified as sub-options to a service. | `volumes: ["/var/lib/mysql", "./cache:/tmp/cache", "datavolume:/var/lib/mysql"]` |
| `volumes_from` | list | None | Mount all of the volumes from another service or container. | `volumes_from: ["service_name", "container_name"]` |
| `working_dir` | string | None | Override the container's working directory. | `working_dir: /code` |

### 1.3 `networks` Attributes

| Attribute | Type | Default | Description | Example |
|---|---|---|---|---|
| `driver` | string | `bridge` | Specify which driver should be used for this network. | `driver: overlay` |
| `driver_opts` | map | None | Specify a list of options as key-value pairs to pass to the driver. | `driver_opts: { com.docker.network.bridge.name: br_1 }` |
| `attachable` | boolean | `false` | Only used when the driver is set to `overlay`. If set to `true`, then standalone containers can attach to this network. | `attachable: true` |
| `enable_ipv6` | boolean | `false` | Enable IPv6 networking. | `enable_ipv6: true` |
| `internal` | boolean | `false` | By default, Docker also connects a bridge network to it to provide external connectivity. If you want to create an externally isolated overlay network, you can set this option to `true`. | `internal: true` |
| `labels` | map/list | None | Add metadata to containers. | `labels: { com.example.description: "Financial transaction network" }` |
| `external` | boolean | `false` | If set to `true`, specifies that this network has been created outside of Compose. | `external: true` |
| `name` | string | None | Set a custom name for this network. | `name: my-app-net` |
| `ipam` | object | None | Specify custom IPAM config. | `ipam: { driver: default, config: [{ subnet: "172.28.0.0/16" }] }` |

### 1.4 `volumes` Attributes

| Attribute | Type | Default | Description | Example |
|---|---|---|---|---|
| `driver` | string | `local` | Specify which volume driver should be used for this volume. | `driver: foobar` |
| `driver_opts` | map | None | Specify a list of options as key-value pairs to pass to the driver for this volume. | `driver_opts: { type: "nfs", o: "addr=10.40.0.199,nolock,soft,rw", device: ":/docker/example" }` |
| `external` | boolean | `false` | If set to `true`, specifies that this volume has been created outside of Compose. | `external: true` |
| `labels` | map/list | None | Add metadata to containers. | `labels: { com.example.description: "Database volume" }` |
| `name` | string | None | Set a custom name for this volume. | `name: my-app-data` |

### 1.5 `configs` Attributes

| Attribute | Type | Default | Description | Example |
|---|---|---|---|---|
| `file` | string | None | The config is created with the contents of the file at the specified path. | `file: ./my_config.txt` |
| `external` | boolean | `false` | If set to `true`, specifies that this config has already been created. | `external: true` |
| `name` | string | None | The name of the config object in Docker. | `name: my_config` |
| `content` | string | None | The content of the config. | `content: | 
  server {
    listen 80;
  }` |

### 1.6 `secrets` Attributes

| Attribute | Type | Default | Description | Example |
|---|---|---|---|---|
| `file` | string | None | The secret is created with the contents of the file at the specified path. | `file: ./my_secret.txt` |
| `environment` | string | None | The secret is created with the value of an environment variable. | `environment: "MY_SECRET"` |
| `external` | boolean | `false` | If set to `true`, specifies that this secret has already been created. | `external: true` |
| `name` | string | None | The name of the secret object in Docker. | `name: my_secret` |

## 2. Dockerfile Instruction Reference

A complete reference for every Dockerfile instruction.

### `FROM`
Initializes a new build stage and sets the Base Image for subsequent instructions.
- Syntax: `FROM [--platform=<platform>] <image> [AS <name>]` or `FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]` or `FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]`
- Example: `FROM --platform=linux/amd64 ubuntu:22.04 AS builder`

### `RUN`
Executes any commands in a new layer on top of the current image and commits the results.
- Syntax: `RUN <command>` (shell form) or `RUN ["executable", "param1", "param2"]` (exec form)
- Flags: `--mount=type=cache|bind|secret|ssh`, `--network=default|none|host`
- Example: `RUN --mount=type=cache,target=/var/cache/apt apt-get update && apt-get install -y curl`

### `CMD`
Provides defaults for an executing container. There can only be one `CMD` instruction in a Dockerfile.
- Syntax: `CMD ["executable","param1","param2"]` (exec form, preferred) or `CMD ["param1","param2"]` (as default parameters to ENTRYPOINT) or `CMD command param1 param2` (shell form)
- Example: `CMD ["node", "server.js"]`

### `LABEL`
Adds metadata to an image.
- Syntax: `LABEL <key>=<value> <key>=<value> <key>=<value> ...`
- Example: `LABEL org.opencontainers.image.authors="team@example.com"`

### `EXPOSE`
Informs Docker that the container listens on the specified network ports at runtime.
- Syntax: `EXPOSE <port> [<port>/<protocol>...]`
- Example: `EXPOSE 80/tcp 80/udp`

### `ENV`
Sets the environment variable `<key>` to the value `<value>`.
- Syntax: `ENV <key>=<value> ...`
- Example: `ENV NODE_ENV=production PORT=3000`

### `ADD`
Copies new files, directories or remote file URLs from `<src>` and adds them to the filesystem of the image at the path `<dest>`.
- Syntax: `ADD [--chown=<user>:<group>] [--chmod=<perms>] [--checksum=<checksum>] <src>... <dest>`
- Example: `ADD https://example.com/big.tar.xz /usr/src/things/`

### `COPY`
Copies new files or directories from `<src>` and adds them to the filesystem of the container at the path `<dest>`.
- Syntax: `COPY [--chown=<user>:<group>] [--chmod=<perms>] <src>... <dest>`
- Example: `COPY --chown=node:node package*.json ./`

### `ENTRYPOINT`
Allows you to configure a container that will run as an executable.
- Syntax: `ENTRYPOINT ["executable", "param1", "param2"]` (exec form, preferred) or `ENTRYPOINT command param1 param2` (shell form)
- Example: `ENTRYPOINT ["docker-entrypoint.sh"]`

### `VOLUME`
Creates a mount point with the specified name and marks it as holding externally mounted volumes from native host or other containers.
- Syntax: `VOLUME ["/data"]`
- Example: `VOLUME /var/lib/mysql`

### `USER`
Sets the user name (or UID) and optionally the user group (or GID) to use when running the image and for any `RUN`, `CMD` and `ENTRYPOINT` instructions that follow it in the Dockerfile.
- Syntax: `USER <user>[:<group>]` or `USER <UID>[:<GID>]`
- Example: `USER 1000:1000`

### `WORKDIR`
Sets the working directory for any `RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD` instructions that follow it in the Dockerfile.
- Syntax: `WORKDIR /path/to/workdir`
- Example: `WORKDIR /app`

### `ARG`
Defines a variable that users can pass at build-time to the builder with the `docker build` command using the `--build-arg <varname>=<value>` flag.
- Syntax: `ARG <name>[=<default value>]`
- Example: `ARG VERSION=latest`

### `ONBUILD`
Adds to the image a trigger instruction to be executed at a later time, when the image is used as the base for another build.
- Syntax: `ONBUILD <INSTRUCTION>`
- Example: `ONBUILD COPY . /app/src`

### `STOPSIGNAL`
Sets the system call signal that will be sent to the container to exit.
- Syntax: `STOPSIGNAL signal`
- Example: `STOPSIGNAL SIGKILL`

### `HEALTHCHECK`
Tells Docker how to test a container to check that it is still working.
- Syntax: `HEALTHCHECK [OPTIONS] CMD command` or `HEALTHCHECK NONE`
- Options: `--interval=DURATION` (default: 30s), `--timeout=DURATION` (default: 30s), `--start-period=DURATION` (default: 0s), `--retries=N` (default: 3)
- Example: `HEALTHCHECK --interval=5m --timeout=3s CMD curl -f http://localhost/ || exit 1`

### `SHELL`
Allows the default shell used for the shell form of commands to be overridden.
- Syntax: `SHELL ["executable", "parameters"]`
- Example: `SHELL ["powershell", "-command"]`

## 3. `daemon.json` Complete Reference

The `daemon.json` file configures the Docker daemon.

| Field | Type | Default | Description |
|---|---|---|---|
| `storage-driver` | string | `overlay2` | The storage driver to use. |
| `log-driver` | string | `json-file` | The default logging driver. |
| `log-opts` | map | None | Options for the logging driver. |
| `default-address-pools` | list | None | Default address pools for node networks. |
| `dns` | list | None | DNS servers to use. |
| `registry-mirrors` | list | None | Registry mirrors to use. |
| `insecure-registries` | list | None | Insecure registries to allow. |
| `live-restore` | boolean | `false` | Enable live restore of docker when containers are still running. |
| `default-runtime` | string | `runc` | Default OCI runtime for containers. |
| `runtimes` | map | None | Register additional OCI runtimes. |
| `features` | map | None | Enable/disable specific features. |
| `builder` | map | None | BuildKit configuration. |
| `containerd` | string | None | Path to containerd socket. |
| `default-cgroupns-mode` | string | `private` | Default cgroup namespace mode. |
| `exec-opts` | list | None | Execution options. |
| `experimental` | boolean | `false` | Enable experimental features. |
| `fixed-cidr` | string | None | IPv4 subnet for fixed IPs. |
| `fixed-cidr-v6` | string | None | IPv6 subnet for fixed IPs. |
| `group` | string | `docker` | Group for the unix socket. |
| `hosts` | list | None | Daemon socket(s) to connect to. |
| `icc` | boolean | `true` | Enable inter-container communication. |
| `ip` | string | `0.0.0.0` | Default IP when binding container ports. |
| `ip-forward` | boolean | `true` | Enable net.ipv4.ip_forward. |
| `iptables` | boolean | `true` | Enable addition of iptables rules. |
| `ip-masq` | boolean | `true` | Enable IP masquerading. |
| `labels` | list | None | Daemon labels. |
| `max-concurrent-downloads` | integer | `3` | Max concurrent downloads. |
| `max-concurrent-uploads` | integer | `5` | Max concurrent uploads. |
| `max-download-attempts` | integer | `5` | Max download attempts. |
| `metrics-addr` | string | None | Address to serve metrics API. |
| `no-new-privileges` | boolean | `false` | Set no-new-privileges by default for new containers. |
| `oom-score-adjust` | integer | `-500` | Set the oom_score_adj for the daemon. |
| `pidfile` | string | `/var/run/docker.pid` | Path to use for daemon PID file. |
| `raw-logs` | boolean | `false` | Full timestamps without ANSI coloring. |
| `seccomp-profile` | string | None | Path to seccomp profile. |
| `selinux-enabled` | boolean | `false` | Enable selinux support. |
| `shutdown-timeout` | integer | `15` | Default timeout for stopping containers. |
| `tls` | boolean | `false` | Use TLS; implied by --tlsverify. |
| `tlscacert` | string | `~/.docker/ca.pem` | Trust certs signed only by this CA. |
| `tlscert` | string | `~/.docker/cert.pem` | Path to TLS certificate file. |
| `tlskey` | string | `~/.docker/key.pem` | Path to TLS key file. |
| `tlsverify` | boolean | `false` | Use TLS and verify the remote. |
| `userland-proxy` | boolean | `true` | Use userland proxy for loopback traffic. |
| `userns-remap` | string | None | User namespace remapping. |

## 4. `.dockerignore` Patterns

The `.dockerignore` file excludes files and directories from the build context.

### Syntax
- `#` for comments.
- `*` matches any sequence of non-separator characters.
- `?` matches any single non-separator character.
- `**` matches any number of directories.
- `!` negates a pattern.

### Common Patterns

**Node.js:**
```dockerignore
node_modules
npm-debug.log
Dockerfile
.dockerignore
.git
.gitignore
README.md
```

**Python:**
```dockerignore
__pycache__
*.pyc
*.pyo
*.pyd
.Python
env/
venv/
.env
Dockerfile
.dockerignore
.git
```

**Go:**
```dockerignore
bin/
obj/
*.exe
*.dll
*.so
*.dylib
Dockerfile
.dockerignore
.git
```

## 5. BuildKit Configuration (`buildkitd.toml`)

BuildKit can be configured via `buildkitd.toml`.

```toml
debug = true
# root is where all buildkit state is stored.
root = "/var/lib/buildkit"
# insecure-entitlements allows insecure entitlements, disabled by default.
insecure-entitlements = [ "network.host", "security.insecure" ]

[grpc]
  address = [ "tcp://0.0.0.0:1234" ]
  # debugAddress is address for attaching go pprof and expvar.
  debugAddress = "0.0.0.0:6060"
  uid = 0
  gid = 0
  [grpc.tls]
    cert = "/etc/buildkit/tls.crt"
    key = "/etc/buildkit/tls.key"
    ca = "/etc/buildkit/tlsca.crt"

[worker.oci]
  enabled = true
  # platforms is manually configure platforms, auto-detected by default.
  platforms = [ "linux/amd64", "linux/arm64" ]
  snapshotter = "auto" # overlayfs or native, default auto will try to use overlayfs
  rootless = false # see docs/rootless.md for more details on rootless mode.
  # Whether run subprocesses in main cgroup or create top-level cgroup.
  # Default is "cgroupfs" when not running rootless.
  cgroup-parent = "cgroupfs"
  # gc keeps/frees disk space.
  gc = true
  gckeepstorage = 9000
  [[worker.oci.gcpolicy]]
    keepBytes = 512000000
    keepDuration = 172800
    filters = [ "type==source.local", "type==exec.cachemount", "type==source.git.checkout"]
  [[worker.oci.gcpolicy]]
    all = true
    keepBytes = 1024000000

[worker.containerd]
  address = "/run/containerd/containerd.sock"
  enabled = true
  platforms = [ "linux/amd64", "linux/arm64" ]
  namespace = "buildkit"
  gc = true
  # gckeepstorage sets storage limit for default gc profile, in MB.
  gckeepstorage = 9000

[registry."docker.io"]
  mirrors = ["YOUR_REGISTRY_MIRROR"]
  http = true
  insecure = true
```

## 6. Docker Context Configuration

Docker contexts allow you to manage multiple Docker environments.

- Create a context: `docker context create my-context --docker "host=ssh://user@remote-host"`
- Use a context: `docker context use my-context`
- List contexts: `docker context ls`
- Inspect a context: `docker context inspect my-context`

## 7. Registry Configuration (`config.yml`)

Configuration for a private Docker registry.

```yaml
version: 0.1
log:
  fields:
    service: registry
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
```

## 8. Production `compose.yaml` Templates

### 8.1 Web App Stack

```yaml
services:
  webapp:
    image: YOUR_REGISTRY/webapp:v1.2.3
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    environment:
      - NODE_ENV=production
    secrets:
      - db_password
    networks:
      - frontend
      - backend
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

networks:
  frontend:
  backend:
    internal: true

secrets:
  db_password:
    external: true
```

### 8.2 Database Stack

```yaml
services:
  db:
    image: postgres:15-alpine
    restart: always
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
      POSTGRES_DB: myapp
    volumes:
      - db-data:/var/lib/postgresql/data
    secrets:
      - db_password
    networks:
      - backend
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 4G
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin -d myapp"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  db-data:

networks:
  backend:
    internal: true

secrets:
  db_password:
    external: true
```

### 8.3 Monitoring Stack (Prometheus/Grafana)

```yaml
services:
  prometheus:
    image: prom/prometheus:v2.45.0
    restart: unless-stopped
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
    ports:
      - "9090:9090"
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:10.0.3
    restart: unless-stopped
    volumes:
      - grafana-data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD__FILE=/run/secrets/grafana_password
    secrets:
      - grafana_password
    ports:
      - "3000:3000"
    networks:
      - monitoring
    depends_on:
      - prometheus

volumes:
  prometheus-data:
  grafana-data:

networks:
  monitoring:

secrets:
  grafana_password:
    external: true
```

### 8.4 ELK Stack

```yaml
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.9.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - es-data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
    networks:
      - elk

  logstash:
    image: docker.elastic.co/logstash/logstash:8.9.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf:ro
    ports:
      - "5044:5044"
    networks:
      - elk
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:8.9.0
    ports:
      - "5601:5601"
    networks:
      - elk
    depends_on:
      - elasticsearch

volumes:
  es-data:

networks:
  elk:
```

## 9. Troubleshooting and Optimization

### 9.1 Common Errors and Solutions

- **"port already in use"**: Use `lsof -i :PORT` to find the conflicting process, or change the port mapping in `compose.yaml`.
- **"no space left on device"**: Run `docker system prune -a --volumes` to clear unused data. Check the `overlay2` directory size.
- **"OOM killed"**: The container exceeded its memory limit. Increase the `mem_limit` in `compose.yaml` or optimize the application's memory usage.
- **"permission denied"**: Check the user/group permissions of the mounted volumes. Ensure the container user has access.
- **"network not found"**: Run `docker network create <network_name>` or ensure the network is defined in `compose.yaml`.
- **"image not found"**: Verify the registry URL, image tag, and `pull_policy`. Ensure you are logged in to the registry.
- **"container unhealthy"**: Check the `healthcheck` command and logs. Increase the `timeout` or `start_period` if the application takes longer to start.
- **"bind mount permission denied"**: On SELinux systems, append `:z` or `:Z` to the volume mount path (e.g., `./data:/data:z`).
- **"DNS resolution failed"**: Check the host's DNS settings or configure custom DNS servers in `daemon.json` or `compose.yaml`.
- **"cannot start service"**: Check `depends_on` conditions. Ensure required services are healthy before starting dependent services.
- **"exec format error"**: The image architecture does not match the host architecture (e.g., running an ARM image on an AMD64 host). Use `docker buildx` to build multi-platform images.
- **"context deadline exceeded"**: Increase the timeout for Docker commands or check network connectivity to the registry.

### 9.2 Cost and Time Optimization

- **Multi-stage builds**: Use multi-stage builds to create smaller final images. This reduces pull times and storage costs.
- **Layer caching**: Order Dockerfile instructions from least frequently changed to most frequently changed to maximize cache hits.
- **BuildKit cache mounts**: Use `--mount=type=cache` to cache package manager downloads (e.g., `apt`, `npm`, `pip`) between builds.
- **Parallel builds**: Use `docker compose build --parallel` to build multiple services concurrently.
- **Image pull policy**: Set `pull_policy: if-not-present` to avoid unnecessary image pulls.
- **Resource limits**: Set CPU and memory limits to prevent runaway containers from consuming all host resources.
- **Logging**: Configure log rotation (`max-size`, `max-file`) to prevent log files from filling up the disk.
- **Prune**: Regularly run `docker system prune` and `docker volume prune` to remove unused resources.
- **Compose profiles**: Use profiles to start only the services needed for a specific environment or task.
- **Compose watch**: Use `develop: watch` for fast iteration during development without rebuilding images.

### 9.3 Security Hardening

- **Non-root user**: Run containers as a non-root user (`USER 1000:1000`).
- **Read-only root filesystem**: Set `read_only: true` to prevent modifications to the container's root filesystem.
- **Drop capabilities**: Drop all Linux capabilities (`cap_drop: ["ALL"]`) and add only the necessary ones.
- **No new privileges**: Set `security_opt: ["no-new-privileges:true"]` to prevent processes from gaining additional privileges.
- **Seccomp profiles**: Use custom seccomp profiles to restrict system calls.
- **Resource limits**: Enforce CPU, memory, and PID limits to prevent denial-of-service attacks.
- **No privileged mode**: Avoid using `privileged: true` unless absolutely necessary.
- **Minimal base images**: Use minimal base images like `alpine`, `distroless`, or `scratch` to reduce the attack surface.
- **Secrets management**: Use Docker secrets instead of environment variables for sensitive data.
- **Network segmentation**: Use internal networks to isolate backend services from the public internet.

### 9.4 Upgrade Strategies

- **Blue-green deployment**: Run the new version alongside the old version and switch traffic when ready.
- **Rolling update**: Use `update_config` with `parallelism` and `delay` to update containers one by one.
- **Canary release**: Route a small percentage of traffic to the new version to test it before a full rollout.
- **Database migrations**: Run database migrations as an init container or a pre-start hook before starting the application.
- **Rollback**: Configure `rollback_config` to automatically roll back to the previous version if the update fails.
- **Zero-downtime**: Use `order: start-first` in `update_config` to start the new container before stopping the old one.
# Docker Super Specialist: Complete Configuration Reference

## 1. `compose.yaml` Complete Schema Reference

The `compose.yaml` file is the heart of Docker Compose. Below is the exhaustive reference for every top-level element and nested option.

### 1.1 Top-Level Elements

- `version`: (Deprecated) No longer required in Compose V2.
- `name`: Sets the project name. Overrides the directory name.
- `services`: Defines the containers to run.
- `networks`: Defines the networks to be created or used.
- `volumes`: Defines the persistent volumes.
- `configs`: Defines configuration files to be mounted.
- `secrets`: Defines sensitive data to be mounted securely.

### 1.2 `services` Attributes

Every service can have the following attributes:

| Attribute | Type | Default | Description | Example |
|---|---|---|---|---|
| `annotations` | map/list | None | Metadata for the container. | `annotations: { "com.example.foo": "bar" }` |
| `attach` | boolean | `true` | Whether to attach to the container's output. | `attach: false` |
| `build` | string/object | None | Configuration for building the image. | `build: ./dir` or `build: { context: ., dockerfile: Dockerfile.alt }` |
| `blkio_config` | object | None | Block IO configuration. | `blkio_config: { weight: 300 }` |
| `cpu_count` | integer | None | Number of usable CPUs. | `cpu_count: 2` |
| `cpu_percent` | integer | None | Usable percentage of available CPUs. | `cpu_percent: 50` |
| `cpu_shares` | integer | None | CPU shares (relative weight). | `cpu_shares: 73` |
| `cpu_period` | integer | None | CPU CFS (Completely Fair Scheduler) period. | `cpu_period: 100000` |
| `cpu_quota` | integer | None | CPU CFS quota. | `cpu_quota: 50000` |
| `cpu_rt_runtime` | integer | None | CPU real-time runtime. | `cpu_rt_runtime: 95000` |
| `cpu_rt_period` | integer | None | CPU real-time period. | `cpu_rt_period: 100000` |
| `cpus` | float | None | Number of CPUs. | `cpus: 1.5` |
| `cpuset` | string | None | CPUs in which to allow execution. | `cpuset: "0,1"` |
| `cap_add` | list | None | Add Linux capabilities. | `cap_add: ["SYS_ADMIN"]` |
| `cap_drop` | list | None | Drop Linux capabilities. | `cap_drop: ["ALL"]` |
| `cgroup` | string | None | Cgroup namespace mode. | `cgroup: "host"` |
| `cgroup_parent` | string | None | Optional parent cgroup. | `cgroup_parent: "m-executor-abcd"` |
| `command` | string/list | None | Override the default command. | `command: ["bundle", "exec", "thin", "-p", "3000"]` |
| `configs` | list | None | Grant access to configs. | `configs: ["my_config"]` |
| `container_name` | string | None | Custom container name. | `container_name: my-web-container` |
| `credential_spec` | object | None | Credential spec for managed service accounts (Windows). | `credential_spec: { file: "my-spec.json" }` |
| `depends_on` | list/object | None | Express dependency between services. | `depends_on: { db: { condition: service_healthy } }` |
| `deploy` | object | None | Configuration for deployment and resource limits. | `deploy: { replicas: 6 }` |
| `develop` | object | None | Configuration for development (Compose Watch). | `develop: { watch: [...] }` |
| `device_cgroup_rules` | list | None | Add rules to the cgroup allowed devices list. | `device_cgroup_rules: ["c 1:3 mr"]` |
| `devices` | list | None | Device mappings. | `devices: ["/dev/ttyUSB0:/dev/ttyUSB0"]` |
| `dns` | string/list | None | Custom DNS servers. | `dns: ["8.8.8.8", "9.9.9.9"]` |
| `dns_opt` | list | None | Custom DNS options. | `dns_opt: ["use-vc", "no-tld-query"]` |
| `dns_search` | string/list | None | Custom DNS search domains. | `dns_search: ["dc1.example.com"]` |
| `domainname` | string | None | Custom domain name. | `domainname: foo.com` |
| `entrypoint` | string/list | None | Override the default entrypoint. | `entrypoint: /code/entrypoint.sh` |
| `env_file` | string/list | None | Add environment variables from a file. | `env_file: .env` |
| `environment` | map/list | None | Add environment variables. | `environment: { RACK_ENV: development }` |
| `expose` | list | None | Expose ports without publishing them to the host. | `expose: ["3000"]` |
| `extends` | string/object | None | Extend another service. | `extends: { file: common.yml, service: webapp }` |
| `external_links` | list | None | Link to containers started outside this compose. | `external_links: ["redis_1", "project_db_1:mysql"]` |
| `extra_hosts` | list/map | None | Add hostname mappings. | `extra_hosts: ["somehost:162.242.195.82"]` |
| `group_add` | list | None | Add additional groups. | `group_add: ["mail"]` |
| `healthcheck` | object | None | Configure a check that's run to determine whether or not containers for this service are "healthy". | `healthcheck: { test: ["CMD", "curl", "-f", "http://localhost"] }` |
| `hostname` | string | None | Custom host name. | `hostname: foo` |
| `image` | string | None | Specify the image to start the container from. | `image: redis:alpine` |
| `init` | boolean | `false` | Run an init inside the container that forwards signals and reaps processes. | `init: true` |
| `ipc` | string | None | IPC namespace to use. | `ipc: host` |
| `isolation` | string | None | Specify a container's isolation technology. | `isolation: default` |
| `labels` | map/list | None | Add metadata to containers. | `labels: { com.example.description: "Accounting webapp" }` |
| `links` | list | None | Link to containers in another service. | `links: ["db", "db:database"]` |
| `logging` | object | None | Logging configuration for the service. | `logging: { driver: syslog, options: { syslog-address: "tcp://192.168.0.42:123" } }` |
| `mac_address` | string | None | MAC address. | `mac_address: 02:42:ac:11:65:43` |
| `mem_limit` | string | None | Memory limit. | `mem_limit: 1g` |
| `mem_reservation` | string | None | Memory soft limit. | `mem_reservation: 512m` |
| `mem_swappiness` | integer | None | Tune a container's memory swappiness behavior. | `mem_swappiness: 60` |
| `memswap_limit` | string | None | Swap limit equal to memory plus swap. | `memswap_limit: 2g` |
| `network_mode` | string | None | Network mode. | `network_mode: "host"` |
| `networks` | list/map | None | Networks to join. | `networks: ["frontend", "backend"]` |
| `oom_kill_disable` | boolean | `false` | Disable OOM Killer. | `oom_kill_disable: true` |
| `oom_score_adj` | integer | None | Tune the host's OOM preferences for containers. | `oom_score_adj: 500` |
| `pid` | string | None | PID namespace to use. | `pid: "host"` |
| `pids_limit` | integer | None | Tune a container's pids limit. | `pids_limit: 100` |
| `platform` | string | None | Target platform containers for this service will run on. | `platform: linux/amd64` |
| `ports` | list | None | Expose ports. | `ports: ["3000", "8000:8000", "9000:8080"]` |
| `privileged` | boolean | `false` | Give extended privileges to this container. | `privileged: true` |
| `profiles` | list | None | Define a list of named profiles for the service to be enabled under. | `profiles: ["frontend", "debug"]` |
| `pull_policy` | string | `always` | Define the decisions Compose makes when it starts to pull images. | `pull_policy: missing` |
| `read_only` | boolean | `false` | Mount the container's root filesystem as read only. | `read_only: true` |
| `restart` | string | `no` | Restart policy. | `restart: always` |
| `runtime` | string | None | Specify the runtime to use for the container. | `runtime: runc` |
| `scale` | integer | `1` | Specify the default number of containers to deploy for this service. | `scale: 3` |
| `secrets` | list | None | Grant access to secrets on a per-service basis. | `secrets: ["my_secret", "my_other_secret"]` |
| `security_opt` | list | None | Override the default labeling scheme for each container. | `security_opt: ["label:user:USER", "label:role:ROLE"]` |
| `shm_size` | string | None | Size of `/dev/shm`. | `shm_size: '2gb'` |
| `stdin_open` | boolean | `false` | Keep STDIN open even if not attached. | `stdin_open: true` |
| `stop_grace_period` | string | `10s` | Specify how long to wait when attempting to stop a container if it doesn't handle SIGTERM. | `stop_grace_period: 1m30s` |
| `stop_signal` | string | `SIGTERM` | Set an alternative signal to stop the container. | `stop_signal: SIGUSR1` |
| `storage_opt` | map | None | Storage driver options for this service. | `storage_opt: { size: '120G' }` |
| `sysctls` | map/list | None | Kernel parameters to set in the container. | `sysctls: { net.core.somaxconn: 1024 }` |
| `tmpfs` | string/list | None | Mount a temporary file system inside the container. | `tmpfs: /run` |
| `tty` | boolean | `false` | Allocate a pseudo-TTY. | `tty: true` |
| `ulimits` | map | None | Override the default ulimits for a container. | `ulimits: { nproc: 65535, nofile: { soft: 20000, hard: 40000 } }` |
| `user` | string | None | Override the user used to run the container process. | `user: "1000:1000"` |
| `userns_mode` | string | None | Disable the user namespace for this service, if Docker daemon is configured with user namespaces. | `userns_mode: "host"` |
| `uts` | string | None | UTS namespace to use. | `uts: "host"` |
| `volumes` | list | None | Mount host paths or named volumes, specified as sub-options to a service. | `volumes: ["/var/lib/mysql", "./cache:/tmp/cache", "datavolume:/var/lib/mysql"]` |
| `volumes_from` | list | None | Mount all of the volumes from another service or container. | `volumes_from: ["service_name", "container_name"]` |
| `working_dir` | string | None | Override the container's working directory. | `working_dir: /code` |

### 1.3 `networks` Attributes

| Attribute | Type | Default | Description | Example |
|---|---|---|---|---|
| `driver` | string | `bridge` | Specify which driver should be used for this network. | `driver: overlay` |
| `driver_opts` | map | None | Specify a list of options as key-value pairs to pass to the driver. | `driver_opts: { com.docker.network.bridge.name: br_1 }` |
| `attachable` | boolean | `false` | Only used when the driver is set to `overlay`. If set to `true`, then standalone containers can attach to this network. | `attachable: true` |
| `enable_ipv6` | boolean | `false` | Enable IPv6 networking. | `enable_ipv6: true` |
| `internal` | boolean | `false` | By default, Docker also connects a bridge network to it to provide external connectivity. If you want to create an externally isolated overlay network, you can set this option to `true`. | `internal: true` |
| `labels` | map/list | None | Add metadata to containers. | `labels: { com.example.description: "Financial transaction network" }` |
| `external` | boolean | `false` | If set to `true`, specifies that this network has been created outside of Compose. | `external: true` |
| `name` | string | None | Set a custom name for this network. | `name: my-app-net` |
| `ipam` | object | None | Specify custom IPAM config. | `ipam: { driver: default, config: [{ subnet: "172.28.0.0/16" }] }` |

### 1.4 `volumes` Attributes

| Attribute | Type | Default | Description | Example |
|---|---|---|---|---|
| `driver` | string | `local` | Specify which volume driver should be used for this volume. | `driver: foobar` |
| `driver_opts` | map | None | Specify a list of options as key-value pairs to pass to the driver for this volume. | `driver_opts: { type: "nfs", o: "addr=10.40.0.199,nolock,soft,rw", device: ":/docker/example" }` |
| `external` | boolean | `false` | If set to `true`, specifies that this volume has been created outside of Compose. | `external: true` |
| `labels` | map/list | None | Add metadata to containers. | `labels: { com.example.description: "Database volume" }` |
| `name` | string | None | Set a custom name for this volume. | `name: my-app-data` |

### 1.5 `configs` Attributes

| Attribute | Type | Default | Description | Example |
|---|---|---|---|---|
| `file` | string | None | The config is created with the contents of the file at the specified path. | `file: ./my_config.txt` |
| `external` | boolean | `false` | If set to `true`, specifies that this config has already been created. | `external: true` |
| `name` | string | None | The name of the config object in Docker. | `name: my_config` |
| `content` | string | None | The content of the config. | `content: | 
  server {
    listen 80;
  }` |

### 1.6 `secrets` Attributes

| Attribute | Type | Default | Description | Example |
|---|---|---|---|---|
| `file` | string | None | The secret is created with the contents of the file at the specified path. | `file: ./my_secret.txt` |
| `environment` | string | None | The secret is created with the value of an environment variable. | `environment: "MY_SECRET"` |
| `external` | boolean | `false` | If set to `true`, specifies that this secret has already been created. | `external: true` |
| `name` | string | None | The name of the secret object in Docker. | `name: my_secret` |

## 2. Dockerfile Instruction Reference

A complete reference for every Dockerfile instruction.

### `FROM`
Initializes a new build stage and sets the Base Image for subsequent instructions.
- Syntax: `FROM [--platform=<platform>] <image> [AS <name>]` or `FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]` or `FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]`
- Example: `FROM --platform=linux/amd64 ubuntu:22.04 AS builder`

### `RUN`
Executes any commands in a new layer on top of the current image and commits the results.
- Syntax: `RUN <command>` (shell form) or `RUN ["executable", "param1", "param2"]` (exec form)
- Flags: `--mount=type=cache|bind|secret|ssh`, `--network=default|none|host`
- Example: `RUN --mount=type=cache,target=/var/cache/apt apt-get update && apt-get install -y curl`

### `CMD`
Provides defaults for an executing container. There can only be one `CMD` instruction in a Dockerfile.
- Syntax: `CMD ["executable","param1","param2"]` (exec form, preferred) or `CMD ["param1","param2"]` (as default parameters to ENTRYPOINT) or `CMD command param1 param2` (shell form)
- Example: `CMD ["node", "server.js"]`

### `LABEL`
Adds metadata to an image.
- Syntax: `LABEL <key>=<value> <key>=<value> <key>=<value> ...`
- Example: `LABEL org.opencontainers.image.authors="team@example.com"`

### `EXPOSE`
Informs Docker that the container listens on the specified network ports at runtime.
- Syntax: `EXPOSE <port> [<port>/<protocol>...]`
- Example: `EXPOSE 80/tcp 80/udp`

### `ENV`
Sets the environment variable `<key>` to the value `<value>`.
- Syntax: `ENV <key>=<value> ...`
- Example: `ENV NODE_ENV=production PORT=3000`

### `ADD`
Copies new files, directories or remote file URLs from `<src>` and adds them to the filesystem of the image at the path `<dest>`.
- Syntax: `ADD [--chown=<user>:<group>] [--chmod=<perms>] [--checksum=<checksum>] <src>... <dest>`
- Example: `ADD https://example.com/big.tar.xz /usr/src/things/`

### `COPY`
Copies new files or directories from `<src>` and adds them to the filesystem of the container at the path `<dest>`.
- Syntax: `COPY [--chown=<user>:<group>] [--chmod=<perms>] <src>... <dest>`
- Example: `COPY --chown=node:node package*.json ./`

### `ENTRYPOINT`
Allows you to configure a container that will run as an executable.
- Syntax: `ENTRYPOINT ["executable", "param1", "param2"]` (exec form, preferred) or `ENTRYPOINT command param1 param2` (shell form)
- Example: `ENTRYPOINT ["docker-entrypoint.sh"]`

### `VOLUME`
Creates a mount point with the specified name and marks it as holding externally mounted volumes from native host or other containers.
- Syntax: `VOLUME ["/data"]`
- Example: `VOLUME /var/lib/mysql`

### `USER`
Sets the user name (or UID) and optionally the user group (or GID) to use when running the image and for any `RUN`, `CMD` and `ENTRYPOINT` instructions that follow it in the Dockerfile.
- Syntax: `USER <user>[:<group>]` or `USER <UID>[:<GID>]`
- Example: `USER 1000:1000`

### `WORKDIR`
Sets the working directory for any `RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD` instructions that follow it in the Dockerfile.
- Syntax: `WORKDIR /path/to/workdir`
- Example: `WORKDIR /app`

### `ARG`
Defines a variable that users can pass at build-time to the builder with the `docker build` command using the `--build-arg <varname>=<value>` flag.
- Syntax: `ARG <name>[=<default value>]`
- Example: `ARG VERSION=latest`

### `ONBUILD`
Adds to the image a trigger instruction to be executed at a later time, when the image is used as the base for another build.
- Syntax: `ONBUILD <INSTRUCTION>`
- Example: `ONBUILD COPY . /app/src`

### `STOPSIGNAL`
Sets the system call signal that will be sent to the container to exit.
- Syntax: `STOPSIGNAL signal`
- Example: `STOPSIGNAL SIGKILL`

### `HEALTHCHECK`
Tells Docker how to test a container to check that it is still working.
- Syntax: `HEALTHCHECK [OPTIONS] CMD command` or `HEALTHCHECK NONE`
- Options: `--interval=DURATION` (default: 30s), `--timeout=DURATION` (default: 30s), `--start-period=DURATION` (default: 0s), `--retries=N` (default: 3)
- Example: `HEALTHCHECK --interval=5m --timeout=3s CMD curl -f http://localhost/ || exit 1`

### `SHELL`
Allows the default shell used for the shell form of commands to be overridden.
- Syntax: `SHELL ["executable", "parameters"]`
- Example: `SHELL ["powershell", "-command"]`

## 3. `daemon.json` Complete Reference

The `daemon.json` file configures the Docker daemon.

| Field | Type | Default | Description |
|---|---|---|---|
| `storage-driver` | string | `overlay2` | The storage driver to use. |
| `log-driver` | string | `json-file` | The default logging driver. |
| `log-opts` | map | None | Options for the logging driver. |
| `default-address-pools` | list | None | Default address pools for node networks. |
| `dns` | list | None | DNS servers to use. |
| `registry-mirrors` | list | None | Registry mirrors to use. |
| `insecure-registries` | list | None | Insecure registries to allow. |
| `live-restore` | boolean | `false` | Enable live restore of docker when containers are still running. |
| `default-runtime` | string | `runc` | Default OCI runtime for containers. |
| `runtimes` | map | None | Register additional OCI runtimes. |
| `features` | map | None | Enable/disable specific features. |
| `builder` | map | None | BuildKit configuration. |
| `containerd` | string | None | Path to containerd socket. |
| `default-cgroupns-mode` | string | `private` | Default cgroup namespace mode. |
| `exec-opts` | list | None | Execution options. |
| `experimental` | boolean | `false` | Enable experimental features. |
| `fixed-cidr` | string | None | IPv4 subnet for fixed IPs. |
| `fixed-cidr-v6` | string | None | IPv6 subnet for fixed IPs. |
| `group` | string | `docker` | Group for the unix socket. |
| `hosts` | list | None | Daemon socket(s) to connect to. |
| `icc` | boolean | `true` | Enable inter-container communication. |
| `ip` | string | `0.0.0.0` | Default IP when binding container ports. |
| `ip-forward` | boolean | `true` | Enable net.ipv4.ip_forward. |
| `iptables` | boolean | `true` | Enable addition of iptables rules. |
| `ip-masq` | boolean | `true` | Enable IP masquerading. |
| `labels` | list | None | Daemon labels. |
| `max-concurrent-downloads` | integer | `3` | Max concurrent downloads. |
| `max-concurrent-uploads` | integer | `5` | Max concurrent uploads. |
| `max-download-attempts` | integer | `5` | Max download attempts. |
| `metrics-addr` | string | None | Address to serve metrics API. |
| `no-new-privileges` | boolean | `false` | Set no-new-privileges by default for new containers. |
| `oom-score-adjust` | integer | `-500` | Set the oom_score_adj for the daemon. |
| `pidfile` | string | `/var/run/docker.pid` | Path to use for daemon PID file. |
| `raw-logs` | boolean | `false` | Full timestamps without ANSI coloring. |
| `seccomp-profile` | string | None | Path to seccomp profile. |
| `selinux-enabled` | boolean | `false` | Enable selinux support. |
| `shutdown-timeout` | integer | `15` | Default timeout for stopping containers. |
| `tls` | boolean | `false` | Use TLS; implied by --tlsverify. |
| `tlscacert` | string | `~/.docker/ca.pem` | Trust certs signed only by this CA. |
| `tlscert` | string | `~/.docker/cert.pem` | Path to TLS certificate file. |
| `tlskey` | string | `~/.docker/key.pem` | Path to TLS key file. |
| `tlsverify` | boolean | `false` | Use TLS and verify the remote. |
| `userland-proxy` | boolean | `true` | Use userland proxy for loopback traffic. |
| `userns-remap` | string | None | User namespace remapping. |

## 4. `.dockerignore` Patterns

The `.dockerignore` file excludes files and directories from the build context.

### Syntax
- `#` for comments.
- `*` matches any sequence of non-separator characters.
- `?` matches any single non-separator character.
- `**` matches any number of directories.
- `!` negates a pattern.

### Common Patterns

**Node.js:**
```dockerignore
node_modules
npm-debug.log
Dockerfile
.dockerignore
.git
.gitignore
README.md
```

**Python:**
```dockerignore
__pycache__
*.pyc
*.pyo
*.pyd
.Python
env/
venv/
.env
Dockerfile
.dockerignore
.git
```

**Go:**
```dockerignore
bin/
obj/
*.exe
*.dll
*.so
*.dylib
Dockerfile
.dockerignore
.git
```

## 5. BuildKit Configuration (`buildkitd.toml`)

BuildKit can be configured via `buildkitd.toml`.

```toml
debug = true
# root is where all buildkit state is stored.
root = "/var/lib/buildkit"
# insecure-entitlements allows insecure entitlements, disabled by default.
insecure-entitlements = [ "network.host", "security.insecure" ]

[grpc]
  address = [ "tcp://0.0.0.0:1234" ]
  # debugAddress is address for attaching go pprof and expvar.
  debugAddress = "0.0.0.0:6060"
  uid = 0
  gid = 0
  [grpc.tls]
    cert = "/etc/buildkit/tls.crt"
    key = "/etc/buildkit/tls.key"
    ca = "/etc/buildkit/tlsca.crt"

[worker.oci]
  enabled = true
  # platforms is manually configure platforms, auto-detected by default.
  platforms = [ "linux/amd64", "linux/arm64" ]
  snapshotter = "auto" # overlayfs or native, default auto will try to use overlayfs
  rootless = false # see docs/rootless.md for more details on rootless mode.
  # Whether run subprocesses in main cgroup or create top-level cgroup.
  # Default is "cgroupfs" when not running rootless.
  cgroup-parent = "cgroupfs"
  # gc keeps/frees disk space.
  gc = true
  gckeepstorage = 9000
  [[worker.oci.gcpolicy]]
    keepBytes = 512000000
    keepDuration = 172800
    filters = [ "type==source.local", "type==exec.cachemount", "type==source.git.checkout"]
  [[worker.oci.gcpolicy]]
    all = true
    keepBytes = 1024000000

[worker.containerd]
  address = "/run/containerd/containerd.sock"
  enabled = true
  platforms = [ "linux/amd64", "linux/arm64" ]
  namespace = "buildkit"
  gc = true
  # gckeepstorage sets storage limit for default gc profile, in MB.
  gckeepstorage = 9000

[registry."docker.io"]
  mirrors = ["YOUR_REGISTRY_MIRROR"]
  http = true
  insecure = true
```

## 6. Docker Context Configuration

Docker contexts allow you to manage multiple Docker environments.

- Create a context: `docker context create my-context --docker "host=ssh://user@remote-host"`
- Use a context: `docker context use my-context`
- List contexts: `docker context ls`
- Inspect a context: `docker context inspect my-context`

## 7. Registry Configuration (`config.yml`)

Configuration for a private Docker registry.

```yaml
version: 0.1
log:
  fields:
    service: registry
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
```

## 8. Production `compose.yaml` Templates

### 8.1 Web App Stack

```yaml
services:
  webapp:
    image: YOUR_REGISTRY/webapp:v1.2.3
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    environment:
      - NODE_ENV=production
    secrets:
      - db_password
    networks:
      - frontend
      - backend
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

networks:
  frontend:
  backend:
    internal: true

secrets:
  db_password:
    external: true
```

### 8.2 Database Stack

```yaml
services:
  db:
    image: postgres:15-alpine
    restart: always
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
      POSTGRES_DB: myapp
    volumes:
      - db-data:/var/lib/postgresql/data
    secrets:
      - db_password
    networks:
      - backend
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 4G
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin -d myapp"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  db-data:

networks:
  backend:
    internal: true

secrets:
  db_password:
    external: true
```

### 8.3 Monitoring Stack (Prometheus/Grafana)

```yaml
services:
  prometheus:
    image: prom/prometheus:v2.45.0
    restart: unless-stopped
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
    ports:
      - "9090:9090"
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:10.0.3
    restart: unless-stopped
    volumes:
      - grafana-data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD__FILE=/run/secrets/grafana_password
    secrets:
      - grafana_password
    ports:
      - "3000:3000"
    networks:
      - monitoring
    depends_on:
      - prometheus

volumes:
  prometheus-data:
  grafana-data:

networks:
  monitoring:

secrets:
  grafana_password:
    external: true
```

### 8.4 ELK Stack

```yaml
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.9.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - es-data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
    networks:
      - elk

  logstash:
    image: docker.elastic.co/logstash/logstash:8.9.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf:ro
    ports:
      - "5044:5044"
    networks:
      - elk
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:8.9.0
    ports:
      - "5601:5601"
    networks:
      - elk
    depends_on:
      - elasticsearch

volumes:
  es-data:

networks:
  elk:
```

## 9. Troubleshooting and Optimization

### 9.1 Common Errors and Solutions

- **"port already in use"**: Use `lsof -i :PORT` to find the conflicting process, or change the port mapping in `compose.yaml`.
- **"no space left on device"**: Run `docker system prune -a --volumes` to clear unused data. Check the `overlay2` directory size.
- **"OOM killed"**: The container exceeded its memory limit. Increase the `mem_limit` in `compose.yaml` or optimize the application's memory usage.
- **"permission denied"**: Check the user/group permissions of the mounted volumes. Ensure the container user has access.
- **"network not found"**: Run `docker network create <network_name>` or ensure the network is defined in `compose.yaml`.
- **"image not found"**: Verify the registry URL, image tag, and `pull_policy`. Ensure you are logged in to the registry.
- **"container unhealthy"**: Check the `healthcheck` command and logs. Increase the `timeout` or `start_period` if the application takes longer to start.
- **"bind mount permission denied"**: On SELinux systems, append `:z` or `:Z` to the volume mount path (e.g., `./data:/data:z`).
- **"DNS resolution failed"**: Check the host's DNS settings or configure custom DNS servers in `daemon.json` or `compose.yaml`.
- **"cannot start service"**: Check `depends_on` conditions. Ensure required services are healthy before starting dependent services.
- **"exec format error"**: The image architecture does not match the host architecture (e.g., running an ARM image on an AMD64 host). Use `docker buildx` to build multi-platform images.
- **"context deadline exceeded"**: Increase the timeout for Docker commands or check network connectivity to the registry.

### 9.2 Cost and Time Optimization

- **Multi-stage builds**: Use multi-stage builds to create smaller final images. This reduces pull times and storage costs.
- **Layer caching**: Order Dockerfile instructions from least frequently changed to most frequently changed to maximize cache hits.
- **BuildKit cache mounts**: Use `--mount=type=cache` to cache package manager downloads (e.g., `apt`, `npm`, `pip`) between builds.
- **Parallel builds**: Use `docker compose build --parallel` to build multiple services concurrently.
- **Image pull policy**: Set `pull_policy: if-not-present` to avoid unnecessary image pulls.
- **Resource limits**: Set CPU and memory limits to prevent runaway containers from consuming all host resources.
- **Logging**: Configure log rotation (`max-size`, `max-file`) to prevent log files from filling up the disk.
- **Prune**: Regularly run `docker system prune` and `docker volume prune` to remove unused resources.
- **Compose profiles**: Use profiles to start only the services needed for a specific environment or task.
- **Compose watch**: Use `develop: watch` for fast iteration during development without rebuilding images.

### 9.3 Security Hardening

- **Non-root user**: Run containers as a non-root user (`USER 1000:1000`).
- **Read-only root filesystem**: Set `read_only: true` to prevent modifications to the container's root filesystem.
- **Drop capabilities**: Drop all Linux capabilities (`cap_drop: ["ALL"]`) and add only the necessary ones.
- **No new privileges**: Set `security_opt: ["no-new-privileges:true"]` to prevent processes from gaining additional privileges.
- **Seccomp profiles**: Use custom seccomp profiles to restrict system calls.
- **Resource limits**: Enforce CPU, memory, and PID limits to prevent denial-of-service attacks.
- **No privileged mode**: Avoid using `privileged: true` unless absolutely necessary.
- **Minimal base images**: Use minimal base images like `alpine`, `distroless`, or `scratch` to reduce the attack surface.
- **Secrets management**: Use Docker secrets instead of environment variables for sensitive data.
- **Network segmentation**: Use internal networks to isolate backend services from the public internet.

### 9.4 Upgrade Strategies

- **Blue-green deployment**: Run the new version alongside the old version and switch traffic when ready.
- **Rolling update**: Use `update_config` with `parallelism` and `delay` to update containers one by one.
- **Canary release**: Route a small percentage of traffic to the new version to test it before a full rollout.
- **Database migrations**: Run database migrations as an init container or a pre-start hook before starting the application.
- **Rollback**: Configure `rollback_config` to automatically roll back to the previous version if the update fails.
- **Zero-downtime**: Use `order: start-first` in `update_config` to start the new container before stopping the old one.
# Docker Super Specialist: Complete Configuration Reference

## 1. `compose.yaml` Complete Schema Reference

The `compose.yaml` file is the heart of Docker Compose. Below is the exhaustive reference for every top-level element and nested option.

### 1.1 Top-Level Elements

- `version`: (Deprecated) No longer required in Compose V2.
- `name`: Sets the project name. Overrides the directory name.
- `services`: Defines the containers to run.
- `networks`: Defines the networks to be created or used.
- `volumes`: Defines the persistent volumes.
- `configs`: Defines configuration files to be mounted.
- `secrets`: Defines sensitive data to be mounted securely.

### 1.2 `services` Attributes

Every service can have the following attributes:

| Attribute | Type | Default | Description | Example |
|---|---|---|---|---|
| `annotations` | map/list | None | Metadata for the container. | `annotations: { "com.example.foo": "bar" }` |
| `attach` | boolean | `true` | Whether to attach to the container's output. | `attach: false` |
| `build` | string/object | None | Configuration for building the image. | `build: ./dir` or `build: { context: ., dockerfile: Dockerfile.alt }` |
| `blkio_config` | object | None | Block IO configuration. | `blkio_config: { weight: 300 }` |
| `cpu_count` | integer | None | Number of usable CPUs. | `cpu_count: 2` |
| `cpu_percent` | integer | None | Usable percentage of available CPUs. | `cpu_percent: 50` |
| `cpu_shares` | integer | None | CPU shares (relative weight). | `cpu_shares: 73` |
| `cpu_period` | integer | None | CPU CFS (Completely Fair Scheduler) period. | `cpu_period: 100000` |
| `cpu_quota` | integer | None | CPU CFS quota. | `cpu_quota: 50000` |
| `cpu_rt_runtime` | integer | None | CPU real-time runtime. | `cpu_rt_runtime: 95000` |
| `cpu_rt_period` | integer | None | CPU real-time period. | `cpu_rt_period: 100000` |
| `cpus` | float | None | Number of CPUs. | `cpus: 1.5` |
| `cpuset` | string | None | CPUs in which to allow execution. | `cpuset: "0,1"` |
| `cap_add` | list | None | Add Linux capabilities. | `cap_add: ["SYS_ADMIN"]` |
| `cap_drop` | list | None | Drop Linux capabilities. | `cap_drop: ["ALL"]` |
| `cgroup` | string | None | Cgroup namespace mode. | `cgroup: "host"` |
| `cgroup_parent` | string | None | Optional parent cgroup. | `cgroup_parent: "m-executor-abcd"` |
| `command` | string/list | None | Override the default command. | `command: ["bundle", "exec", "thin", "-p", "3000"]` |
| `configs` | list | None | Grant access to configs. | `configs: ["my_config"]` |
| `container_name` | string | None | Custom container name. | `container_name: my-web-container` |
| `credential_spec` | object | None | Credential spec for managed service accounts (Windows). | `credential_spec: { file: "my-spec.json" }` |
| `depends_on` | list/object | None | Express dependency between services. | `depends_on: { db: { condition: service_healthy } }` |
| `deploy` | object | None | Configuration for deployment and resource limits. | `deploy: { replicas: 6 }` |
| `develop` | object | None | Configuration for development (Compose Watch). | `develop: { watch: [...] }` |
| `device_cgroup_rules` | list | None | Add rules to the cgroup allowed devices list. | `device_cgroup_rules: ["c 1:3 mr"]` |
| `devices` | list | None | Device mappings. | `devices: ["/dev/ttyUSB0:/dev/ttyUSB0"]` |
| `dns` | string/list | None | Custom DNS servers. | `dns: ["8.8.8.8", "9.9.9.9"]` |
| `dns_opt` | list | None | Custom DNS options. | `dns_opt: ["use-vc", "no-tld-query"]` |
| `dns_search` | string/list | None | Custom DNS search domains. | `dns_search: ["dc1.example.com"]` |
| `domainname` | string | None | Custom domain name. | `domainname: foo.com` |
| `entrypoint` | string/list | None | Override the default entrypoint. | `entrypoint: /code/entrypoint.sh` |
| `env_file` | string/list | None | Add environment variables from a file. | `env_file: .env` |
| `environment` | map/list | None | Add environment variables. | `environment: { RACK_ENV: development }` |
| `expose` | list | None | Expose ports without publishing them to the host. | `expose: ["3000"]` |
| `extends` | string/object | None | Extend another service. | `extends: { file: common.yml, service: webapp }` |
| `external_links` | list | None | Link to containers started outside this compose. | `external_links: ["redis_1", "project_db_1:mysql"]` |
| `extra_hosts` | list/map | None | Add hostname mappings. | `extra_hosts: ["somehost:162.242.195.82"]` |
| `group_add` | list | None | Add additional groups. | `group_add: ["mail"]` |
| `healthcheck` | object | None | Configure a check that's run to determine whether or not containers for this service are "healthy". | `healthcheck: { test: ["CMD", "curl", "-f", "http://localhost"] }` |
| `hostname` | string | None | Custom host name. | `hostname: foo` |
| `image` | string | None | Specify the image to start the container from. | `image: redis:alpine` |
| `init` | boolean | `false` | Run an init inside the container that forwards signals and reaps processes. | `init: true` |
| `ipc` | string | None | IPC namespace to use. | `ipc: host` |
| `isolation` | string | None | Specify a container's isolation technology. | `isolation: default` |
| `labels` | map/list | None | Add metadata to containers. | `labels: { com.example.description: "Accounting webapp" }` |
| `links` | list | None | Link to containers in another service. | `links: ["db", "db:database"]` |
| `logging` | object | None | Logging configuration for the service. | `logging: { driver: syslog, options: { syslog-address: "tcp://192.168.0.42:123" } }` |
| `mac_address` | string | None | MAC address. | `mac_address: 02:42:ac:11:65:43` |
| `mem_limit` | string | None | Memory limit. | `mem_limit: 1g` |
| `mem_reservation` | string | None | Memory soft limit. | `mem_reservation: 512m` |
| `mem_swappiness` | integer | None | Tune a container's memory swappiness behavior. | `mem_swappiness: 60` |
| `memswap_limit` | string | None | Swap limit equal to memory plus swap. | `memswap_limit: 2g` |
| `network_mode` | string | None | Network mode. | `network_mode: "host"` |
| `networks` | list/map | None | Networks to join. | `networks: ["frontend", "backend"]` |
| `oom_kill_disable` | boolean | `false` | Disable OOM Killer. | `oom_kill_disable: true` |
| `oom_score_adj` | integer | None | Tune the host's OOM preferences for containers. | `oom_score_adj: 500` |
| `pid` | string | None | PID namespace to use. | `pid: "host"` |
| `pids_limit` | integer | None | Tune a container's pids limit. | `pids_limit: 100` |
| `platform` | string | None | Target platform containers for this service will run on. | `platform: linux/amd64` |
| `ports` | list | None | Expose ports. | `ports: ["3000", "8000:8000", "9000:8080"]` |
| `privileged` | boolean | `false` | Give extended privileges to this container. | `privileged: true` |
| `profiles` | list | None | Define a list of named profiles for the service to be enabled under. | `profiles: ["frontend", "debug"]` |
| `pull_policy` | string | `always` | Define the decisions Compose makes when it starts to pull images. | `pull_policy: missing` |
| `read_only` | boolean | `false` | Mount the container's root filesystem as read only. | `read_only: true` |
| `restart` | string | `no` | Restart policy. | `restart: always` |
| `runtime` | string | None | Specify the runtime to use for the container. | `runtime: runc` |
| `scale` | integer | `1` | Specify the default number of containers to deploy for this service. | `scale: 3` |
| `secrets` | list | None | Grant access to secrets on a per-service basis. | `secrets: ["my_secret", "my_other_secret"]` |
| `security_opt` | list | None | Override the default labeling scheme for each container. | `security_opt: ["label:user:USER", "label:role:ROLE"]` |
| `shm_size` | string | None | Size of `/dev/shm`. | `shm_size: '2gb'` |
| `stdin_open` | boolean | `false` | Keep STDIN open even if not attached. | `stdin_open: true` |
| `stop_grace_period` | string | `10s` | Specify how long to wait when attempting to stop a container if it doesn't handle SIGTERM. | `stop_grace_period: 1m30s` |
| `stop_signal` | string | `SIGTERM` | Set an alternative signal to stop the container. | `stop_signal: SIGUSR1` |
| `storage_opt` | map | None | Storage driver options for this service. | `storage_opt: { size: '120G' }` |
| `sysctls` | map/list | None | Kernel parameters to set in the container. | `sysctls: { net.core.somaxconn: 1024 }` |
| `tmpfs` | string/list | None | Mount a temporary file system inside the container. | `tmpfs: /run` |
| `tty` | boolean | `false` | Allocate a pseudo-TTY. | `tty: true` |
| `ulimits` | map | None | Override the default ulimits for a container. | `ulimits: { nproc: 65535, nofile: { soft: 20000, hard: 40000 } }` |
| `user` | string | None | Override the user used to run the container process. | `user: "1000:1000"` |
| `userns_mode` | string | None | Disable the user namespace for this service, if Docker daemon is configured with user namespaces. | `userns_mode: "host"` |
| `uts` | string | None | UTS namespace to use. | `uts: "host"` |
| `volumes` | list | None | Mount host paths or named volumes, specified as sub-options to a service. | `volumes: ["/var/lib/mysql", "./cache:/tmp/cache", "datavolume:/var/lib/mysql"]` |
| `volumes_from` | list | None | Mount all of the volumes from another service or container. | `volumes_from: ["service_name", "container_name"]` |
| `working_dir` | string | None | Override the container's working directory. | `working_dir: /code` |

### 1.3 `networks` Attributes

| Attribute | Type | Default | Description | Example |
|---|---|---|---|---|
| `driver` | string | `bridge` | Specify which driver should be used for this network. | `driver: overlay` |
| `driver_opts` | map | None | Specify a list of options as key-value pairs to pass to the driver. | `driver_opts: { com.docker.network.bridge.name: br_1 }` |
| `attachable` | boolean | `false` | Only used when the driver is set to `overlay`. If set to `true`, then standalone containers can attach to this network. | `attachable: true` |
| `enable_ipv6` | boolean | `false` | Enable IPv6 networking. | `enable_ipv6: true` |
| `internal` | boolean | `false` | By default, Docker also connects a bridge network to it to provide external connectivity. If you want to create an externally isolated overlay network, you can set this option to `true`. | `internal: true` |
| `labels` | map/list | None | Add metadata to containers. | `labels: { com.example.description: "Financial transaction network" }` |
| `external` | boolean | `false` | If set to `true`, specifies that this network has been created outside of Compose. | `external: true` |
| `name` | string | None | Set a custom name for this network. | `name: my-app-net` |
| `ipam` | object | None | Specify custom IPAM config. | `ipam: { driver: default, config: [{ subnet: "172.28.0.0/16" }] }` |

### 1.4 `volumes` Attributes

| Attribute | Type | Default | Description | Example |
|---|---|---|---|---|
| `driver` | string | `local` | Specify which volume driver should be used for this volume. | `driver: foobar` |
| `driver_opts` | map | None | Specify a list of options as key-value pairs to pass to the driver for this volume. | `driver_opts: { type: "nfs", o: "addr=10.40.0.199,nolock,soft,rw", device: ":/docker/example" }` |
| `external` | boolean | `false` | If set to `true`, specifies that this volume has been created outside of Compose. | `external: true` |
| `labels` | map/list | None | Add metadata to containers. | `labels: { com.example.description: "Database volume" }` |
| `name` | string | None | Set a custom name for this volume. | `name: my-app-data` |

### 1.5 `configs` Attributes

| Attribute | Type | Default | Description | Example |
|---|---|---|---|---|
| `file` | string | None | The config is created with the contents of the file at the specified path. | `file: ./my_config.txt` |
| `external` | boolean | `false` | If set to `true`, specifies that this config has already been created. | `external: true` |
| `name` | string | None | The name of the config object in Docker. | `name: my_config` |
| `content` | string | None | The content of the config. | `content: | 
  server {
    listen 80;
  }` |

### 1.6 `secrets` Attributes

| Attribute | Type | Default | Description | Example |
|---|---|---|---|---|
| `file` | string | None | The secret is created with the contents of the file at the specified path. | `file: ./my_secret.txt` |
| `environment` | string | None | The secret is created with the value of an environment variable. | `environment: "MY_SECRET"` |
| `external` | boolean | `false` | If set to `true`, specifies that this secret has already been created. | `external: true` |
| `name` | string | None | The name of the secret object in Docker. | `name: my_secret` |

## 2. Dockerfile Instruction Reference

A complete reference for every Dockerfile instruction.

### `FROM`
Initializes a new build stage and sets the Base Image for subsequent instructions.
- Syntax: `FROM [--platform=<platform>] <image> [AS <name>]` or `FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]` or `FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]`
- Example: `FROM --platform=linux/amd64 ubuntu:22.04 AS builder`

### `RUN`
Executes any commands in a new layer on top of the current image and commits the results.
- Syntax: `RUN <command>` (shell form) or `RUN ["executable", "param1", "param2"]` (exec form)
- Flags: `--mount=type=cache|bind|secret|ssh`, `--network=default|none|host`
- Example: `RUN --mount=type=cache,target=/var/cache/apt apt-get update && apt-get install -y curl`

### `CMD`
Provides defaults for an executing container. There can only be one `CMD` instruction in a Dockerfile.
- Syntax: `CMD ["executable","param1","param2"]` (exec form, preferred) or `CMD ["param1","param2"]` (as default parameters to ENTRYPOINT) or `CMD command param1 param2` (shell form)
- Example: `CMD ["node", "server.js"]`

### `LABEL`
Adds metadata to an image.
- Syntax: `LABEL <key>=<value> <key>=<value> <key>=<value> ...`
- Example: `LABEL org.opencontainers.image.authors="team@example.com"`

### `EXPOSE`
Informs Docker that the container listens on the specified network ports at runtime.
- Syntax: `EXPOSE <port> [<port>/<protocol>...]`
- Example: `EXPOSE 80/tcp 80/udp`

### `ENV`
Sets the environment variable `<key>` to the value `<value>`.
- Syntax: `ENV <key>=<value> ...`
- Example: `ENV NODE_ENV=production PORT=3000`

### `ADD`
Copies new files, directories or remote file URLs from `<src>` and adds them to the filesystem of the image at the path `<dest>`.
- Syntax: `ADD [--chown=<user>:<group>] [--chmod=<perms>] [--checksum=<checksum>] <src>... <dest>`
- Example: `ADD https://example.com/big.tar.xz /usr/src/things/`

### `COPY`
Copies new files or directories from `<src>` and adds them to the filesystem of the container at the path `<dest>`.
- Syntax: `COPY [--chown=<user>:<group>] [--chmod=<perms>] <src>... <dest>`
- Example: `COPY --chown=node:node package*.json ./`

### `ENTRYPOINT`
Allows you to configure a container that will run as an executable.
- Syntax: `ENTRYPOINT ["executable", "param1", "param2"]` (exec form, preferred) or `ENTRYPOINT command param1 param2` (shell form)
- Example: `ENTRYPOINT ["docker-entrypoint.sh"]`

### `VOLUME`
Creates a mount point with the specified name and marks it as holding externally mounted volumes from native host or other containers.
- Syntax: `VOLUME ["/data"]`
- Example: `VOLUME /var/lib/mysql`

### `USER`
Sets the user name (or UID) and optionally the user group (or GID) to use when running the image and for any `RUN`, `CMD` and `ENTRYPOINT` instructions that follow it in the Dockerfile.
- Syntax: `USER <user>[:<group>]` or `USER <UID>[:<GID>]`
- Example: `USER 1000:1000`

### `WORKDIR`
Sets the working directory for any `RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD` instructions that follow it in the Dockerfile.
- Syntax: `WORKDIR /path/to/workdir`
- Example: `WORKDIR /app`

### `ARG`
Defines a variable that users can pass at build-time to the builder with the `docker build` command using the `--build-arg <varname>=<value>` flag.
- Syntax: `ARG <name>[=<default value>]`
- Example: `ARG VERSION=latest`

### `ONBUILD`
Adds to the image a trigger instruction to be executed at a later time, when the image is used as the base for another build.
- Syntax: `ONBUILD <INSTRUCTION>`
- Example: `ONBUILD COPY . /app/src`

### `STOPSIGNAL`
Sets the system call signal that will be sent to the container to exit.
- Syntax: `STOPSIGNAL signal`
- Example: `STOPSIGNAL SIGKILL`

### `HEALTHCHECK`
Tells Docker how to test a container to check that it is still working.
- Syntax: `HEALTHCHECK [OPTIONS] CMD command` or `HEALTHCHECK NONE`
- Options: `--interval=DURATION` (default: 30s), `--timeout=DURATION` (default: 30s), `--start-period=DURATION` (default: 0s), `--retries=N` (default: 3)
- Example: `HEALTHCHECK --interval=5m --timeout=3s CMD curl -f http://localhost/ || exit 1`

### `SHELL`
Allows the default shell used for the shell form of commands to be overridden.
- Syntax: `SHELL ["executable", "parameters"]`
- Example: `SHELL ["powershell", "-command"]`

## 3. `daemon.json` Complete Reference

The `daemon.json` file configures the Docker daemon.

| Field | Type | Default | Description |
|---|---|---|---|
| `storage-driver` | string | `overlay2` | The storage driver to use. |
| `log-driver` | string | `json-file` | The default logging driver. |
| `log-opts` | map | None | Options for the logging driver. |
| `default-address-pools` | list | None | Default address pools for node networks. |
| `dns` | list | None | DNS servers to use. |
| `registry-mirrors` | list | None | Registry mirrors to use. |
| `insecure-registries` | list | None | Insecure registries to allow. |
| `live-restore` | boolean | `false` | Enable live restore of docker when containers are still running. |
| `default-runtime` | string | `runc` | Default OCI runtime for containers. |
| `runtimes` | map | None | Register additional OCI runtimes. |
| `features` | map | None | Enable/disable specific features. |
| `builder` | map | None | BuildKit configuration. |
| `containerd` | string | None | Path to containerd socket. |
| `default-cgroupns-mode` | string | `private` | Default cgroup namespace mode. |
| `exec-opts` | list | None | Execution options. |
| `experimental` | boolean | `false` | Enable experimental features. |
| `fixed-cidr` | string | None | IPv4 subnet for fixed IPs. |
| `fixed-cidr-v6` | string | None | IPv6 subnet for fixed IPs. |
| `group` | string | `docker` | Group for the unix socket. |
| `hosts` | list | None | Daemon socket(s) to connect to. |
| `icc` | boolean | `true` | Enable inter-container communication. |
| `ip` | string | `0.0.0.0` | Default IP when binding container ports. |
| `ip-forward` | boolean | `true` | Enable net.ipv4.ip_forward. |
| `iptables` | boolean | `true` | Enable addition of iptables rules. |
| `ip-masq` | boolean | `true` | Enable IP masquerading. |
| `labels` | list | None | Daemon labels. |
| `max-concurrent-downloads` | integer | `3` | Max concurrent downloads. |
| `max-concurrent-uploads` | integer | `5` | Max concurrent uploads. |
| `max-download-attempts` | integer | `5` | Max download attempts. |
| `metrics-addr` | string | None | Address to serve metrics API. |
| `no-new-privileges` | boolean | `false` | Set no-new-privileges by default for new containers. |
| `oom-score-adjust` | integer | `-500` | Set the oom_score_adj for the daemon. |
| `pidfile` | string | `/var/run/docker.pid` | Path to use for daemon PID file. |
| `raw-logs` | boolean | `false` | Full timestamps without ANSI coloring. |
| `seccomp-profile` | string | None | Path to seccomp profile. |
| `selinux-enabled` | boolean | `false` | Enable selinux support. |
| `shutdown-timeout` | integer | `15` | Default timeout for stopping containers. |
| `tls` | boolean | `false` | Use TLS; implied by --tlsverify. |
| `tlscacert` | string | `~/.docker/ca.pem` | Trust certs signed only by this CA. |
| `tlscert` | string | `~/.docker/cert.pem` | Path to TLS certificate file. |
| `tlskey` | string | `~/.docker/key.pem` | Path to TLS key file. |
| `tlsverify` | boolean | `false` | Use TLS and verify the remote. |
| `userland-proxy` | boolean | `true` | Use userland proxy for loopback traffic. |
| `userns-remap` | string | None | User namespace remapping. |

## 4. `.dockerignore` Patterns

The `.dockerignore` file excludes files and directories from the build context.

### Syntax
- `#` for comments.
- `*` matches any sequence of non-separator characters.
- `?` matches any single non-separator character.
- `**` matches any number of directories.
- `!` negates a pattern.

### Common Patterns

**Node.js:**
```dockerignore
node_modules
npm-debug.log
Dockerfile
.dockerignore
.git
.gitignore
README.md
```

**Python:**
```dockerignore
__pycache__
*.pyc
*.pyo
*.pyd
.Python
env/
venv/
.env
Dockerfile
.dockerignore
.git
```

**Go:**
```dockerignore
bin/
obj/
*.exe
*.dll
*.so
*.dylib
Dockerfile
.dockerignore
.git
```

## 5. BuildKit Configuration (`buildkitd.toml`)

BuildKit can be configured via `buildkitd.toml`.

```toml
debug = true
# root is where all buildkit state is stored.
root = "/var/lib/buildkit"
# insecure-entitlements allows insecure entitlements, disabled by default.
insecure-entitlements = [ "network.host", "security.insecure" ]

[grpc]
  address = [ "tcp://0.0.0.0:1234" ]
  # debugAddress is address for attaching go pprof and expvar.
  debugAddress = "0.0.0.0:6060"
  uid = 0
  gid = 0
  [grpc.tls]
    cert = "/etc/buildkit/tls.crt"
    key = "/etc/buildkit/tls.key"
    ca = "/etc/buildkit/tlsca.crt"

[worker.oci]
  enabled = true
  # platforms is manually configure platforms, auto-detected by default.
  platforms = [ "linux/amd64", "linux/arm64" ]
  snapshotter = "auto" # overlayfs or native, default auto will try to use overlayfs
  rootless = false # see docs/rootless.md for more details on rootless mode.
  # Whether run subprocesses in main cgroup or create top-level cgroup.
  # Default is "cgroupfs" when not running rootless.
  cgroup-parent = "cgroupfs"
  # gc keeps/frees disk space.
  gc = true
  gckeepstorage = 9000
  [[worker.oci.gcpolicy]]
    keepBytes = 512000000
    keepDuration = 172800
    filters = [ "type==source.local", "type==exec.cachemount", "type==source.git.checkout"]
  [[worker.oci.gcpolicy]]
    all = true
    keepBytes = 1024000000

[worker.containerd]
  address = "/run/containerd/containerd.sock"
  enabled = true
  platforms = [ "linux/amd64", "linux/arm64" ]
  namespace = "buildkit"
  gc = true
  # gckeepstorage sets storage limit for default gc profile, in MB.
  gckeepstorage = 9000

[registry."docker.io"]
  mirrors = ["YOUR_REGISTRY_MIRROR"]
  http = true
  insecure = true
```

## 6. Docker Context Configuration

Docker contexts allow you to manage multiple Docker environments.

- Create a context: `docker context create my-context --docker "host=ssh://user@remote-host"`
- Use a context: `docker context use my-context`
- List contexts: `docker context ls`
- Inspect a context: `docker context inspect my-context`

## 7. Registry Configuration (`config.yml`)

Configuration for a private Docker registry.

```yaml
version: 0.1
log:
  fields:
    service: registry
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
```

## 8. Production `compose.yaml` Templates

### 8.1 Web App Stack

```yaml
services:
  webapp:
    image: YOUR_REGISTRY/webapp:v1.2.3
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    environment:
      - NODE_ENV=production
    secrets:
      - db_password
    networks:
      - frontend
      - backend
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

networks:
  frontend:
  backend:
    internal: true

secrets:
  db_password:
    external: true
```

### 8.2 Database Stack

```yaml
services:
  db:
    image: postgres:15-alpine
    restart: always
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
      POSTGRES_DB: myapp
    volumes:
      - db-data:/var/lib/postgresql/data
    secrets:
      - db_password
    networks:
      - backend
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 4G
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin -d myapp"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  db-data:

networks:
  backend:
    internal: true

secrets:
  db_password:
    external: true
```

### 8.3 Monitoring Stack (Prometheus/Grafana)

```yaml
services:
  prometheus:
    image: prom/prometheus:v2.45.0
    restart: unless-stopped
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
    ports:
      - "9090:9090"
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:10.0.3
    restart: unless-stopped
    volumes:
      - grafana-data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD__FILE=/run/secrets/grafana_password
    secrets:
      - grafana_password
    ports:
      - "3000:3000"
    networks:
      - monitoring
    depends_on:
      - prometheus

volumes:
  prometheus-data:
  grafana-data:

networks:
  monitoring:

secrets:
  grafana_password:
    external: true
```

### 8.4 ELK Stack

```yaml
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.9.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - es-data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
    networks:
      - elk

  logstash:
    image: docker.elastic.co/logstash/logstash:8.9.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf:ro
    ports:
      - "5044:5044"
    networks:
      - elk
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:8.9.0
    ports:
      - "5601:5601"
    networks:
      - elk
    depends_on:
      - elasticsearch

volumes:
  es-data:

networks:
  elk:
```

## 9. Troubleshooting and Optimization

### 9.1 Common Errors and Solutions

- **"port already in use"**: Use `lsof -i :PORT` to find the conflicting process, or change the port mapping in `compose.yaml`.
- **"no space left on device"**: Run `docker system prune -a --volumes` to clear unused data. Check the `overlay2` directory size.
- **"OOM killed"**: The container exceeded its memory limit. Increase the `mem_limit` in `compose.yaml` or optimize the application's memory usage.
- **"permission denied"**: Check the user/group permissions of the mounted volumes. Ensure the container user has access.
- **"network not found"**: Run `docker network create <network_name>` or ensure the network is defined in `compose.yaml`.
- **"image not found"**: Verify the registry URL, image tag, and `pull_policy`. Ensure you are logged in to the registry.
- **"container unhealthy"**: Check the `healthcheck` command and logs. Increase the `timeout` or `start_period` if the application takes longer to start.
- **"bind mount permission denied"**: On SELinux systems, append `:z` or `:Z` to the volume mount path (e.g., `./data:/data:z`).
- **"DNS resolution failed"**: Check the host's DNS settings or configure custom DNS servers in `daemon.json` or `compose.yaml`.
- **"cannot start service"**: Check `depends_on` conditions. Ensure required services are healthy before starting dependent services.
- **"exec format error"**: The image architecture does not match the host architecture (e.g., running an ARM image on an AMD64 host). Use `docker buildx` to build multi-platform images.
- **"context deadline exceeded"**: Increase the timeout for Docker commands or check network connectivity to the registry.

### 9.2 Cost and Time Optimization

- **Multi-stage builds**: Use multi-stage builds to create smaller final images. This reduces pull times and storage costs.
- **Layer caching**: Order Dockerfile instructions from least frequently changed to most frequently changed to maximize cache hits.
- **BuildKit cache mounts**: Use `--mount=type=cache` to cache package manager downloads (e.g., `apt`, `npm`, `pip`) between builds.
- **Parallel builds**: Use `docker compose build --parallel` to build multiple services concurrently.
- **Image pull policy**: Set `pull_policy: if-not-present` to avoid unnecessary image pulls.
- **Resource limits**: Set CPU and memory limits to prevent runaway containers from consuming all host resources.
- **Logging**: Configure log rotation (`max-size`, `max-file`) to prevent log files from filling up the disk.
- **Prune**: Regularly run `docker system prune` and `docker volume prune` to remove unused resources.
- **Compose profiles**: Use profiles to start only the services needed for a specific environment or task.
- **Compose watch**: Use `develop: watch` for fast iteration during development without rebuilding images.

### 9.3 Security Hardening

- **Non-root user**: Run containers as a non-root user (`USER 1000:1000`).
- **Read-only root filesystem**: Set `read_only: true` to prevent modifications to the container's root filesystem.
- **Drop capabilities**: Drop all Linux capabilities (`cap_drop: ["ALL"]`) and add only the necessary ones.
- **No new privileges**: Set `security_opt: ["no-new-privileges:true"]` to prevent processes from gaining additional privileges.
- **Seccomp profiles**: Use custom seccomp profiles to restrict system calls.
- **Resource limits**: Enforce CPU, memory, and PID limits to prevent denial-of-service attacks.
- **No privileged mode**: Avoid using `privileged: true` unless absolutely necessary.
- **Minimal base images**: Use minimal base images like `alpine`, `distroless`, or `scratch` to reduce the attack surface.
- **Secrets management**: Use Docker secrets instead of environment variables for sensitive data.
- **Network segmentation**: Use internal networks to isolate backend services from the public internet.

### 9.4 Upgrade Strategies

- **Blue-green deployment**: Run the new version alongside the old version and switch traffic when ready.
- **Rolling update**: Use `update_config` with `parallelism` and `delay` to update containers one by one.
- **Canary release**: Route a small percentage of traffic to the new version to test it before a full rollout.
- **Database migrations**: Run database migrations as an init container or a pre-start hook before starting the application.
- **Rollback**: Configure `rollback_config` to automatically roll back to the previous version if the update fails.
- **Zero-downtime**: Use `order: start-first` in `update_config` to start the new container before stopping the old one.
# Docker Super Specialist: Complete Configuration Reference

## 1. `compose.yaml` Complete Schema Reference

The `compose.yaml` file is the heart of Docker Compose. Below is the exhaustive reference for every top-level element and nested option.

### 1.1 Top-Level Elements

- `version`: (Deprecated) No longer required in Compose V2.
- `name`: Sets the project name. Overrides the directory name.
- `services`: Defines the containers to run.
- `networks`: Defines the networks to be created or used.
- `volumes`: Defines the persistent volumes.
- `configs`: Defines configuration files to be mounted.
- `secrets`: Defines sensitive data to be mounted securely.

### 1.2 `services` Attributes

Every service can have the following attributes:

| Attribute | Type | Default | Description | Example |
|---|---|---|---|---|
| `annotations` | map/list | None | Metadata for the container. | `annotations: { "com.example.foo": "bar" }` |
| `attach` | boolean | `true` | Whether to attach to the container's output. | `attach: false` |
| `build` | string/object | None | Configuration for building the image. | `build: ./dir` or `build: { context: ., dockerfile: Dockerfile.alt }` |
| `blkio_config` | object | None | Block IO configuration. | `blkio_config: { weight: 300 }` |
| `cpu_count` | integer | None | Number of usable CPUs. | `cpu_count: 2` |
| `cpu_percent` | integer | None | Usable percentage of available CPUs. | `cpu_percent: 50` |
| `cpu_shares` | integer | None | CPU shares (relative weight). | `cpu_shares: 73` |
| `cpu_period` | integer | None | CPU CFS (Completely Fair Scheduler) period. | `cpu_period: 100000` |
| `cpu_quota` | integer | None | CPU CFS quota. | `cpu_quota: 50000` |
| `cpu_rt_runtime` | integer | None | CPU real-time runtime. | `cpu_rt_runtime: 95000` |
| `cpu_rt_period` | integer | None | CPU real-time period. | `cpu_rt_period: 100000` |
| `cpus` | float | None | Number of CPUs. | `cpus: 1.5` |
| `cpuset` | string | None | CPUs in which to allow execution. | `cpuset: "0,1"` |
| `cap_add` | list | None | Add Linux capabilities. | `cap_add: ["SYS_ADMIN"]` |
| `cap_drop` | list | None | Drop Linux capabilities. | `cap_drop: ["ALL"]` |
| `cgroup` | string | None | Cgroup namespace mode. | `cgroup: "host"` |
| `cgroup_parent` | string | None | Optional parent cgroup. | `cgroup_parent: "m-executor-abcd"` |
| `command` | string/list | None | Override the default command. | `command: ["bundle", "exec", "thin", "-p", "3000"]` |
| `configs` | list | None | Grant access to configs. | `configs: ["my_config"]` |
| `container_name` | string | None | Custom container name. | `container_name: my-web-container` |
| `credential_spec` | object | None | Credential spec for managed service accounts (Windows). | `credential_spec: { file: "my-spec.json" }` |
| `depends_on` | list/object | None | Express dependency between services. | `depends_on: { db: { condition: service_healthy } }` |
| `deploy` | object | None | Configuration for deployment and resource limits. | `deploy: { replicas: 6 }` |
| `develop` | object | None | Configuration for development (Compose Watch). | `develop: { watch: [...] }` |
| `device_cgroup_rules` | list | None | Add rules to the cgroup allowed devices list. | `device_cgroup_rules: ["c 1:3 mr"]` |
| `devices` | list | None | Device mappings. | `devices: ["/dev/ttyUSB0:/dev/ttyUSB0"]` |
| `dns` | string/list | None | Custom DNS servers. | `dns: ["8.8.8.8", "9.9.9.9"]` |
| `dns_opt` | list | None | Custom DNS options. | `dns_opt: ["use-vc", "no-tld-query"]` |
| `dns_search` | string/list | None | Custom DNS search domains. | `dns_search: ["dc1.example.com"]` |
| `domainname` | string | None | Custom domain name. | `domainname: foo.com` |
| `entrypoint` | string/list | None | Override the default entrypoint. | `entrypoint: /code/entrypoint.sh` |
| `env_file` | string/list | None | Add environment variables from a file. | `env_file: .env` |
| `environment` | map/list | None | Add environment variables. | `environment: { RACK_ENV: development }` |
| `expose` | list | None | Expose ports without publishing them to the host. | `expose: ["3000"]` |
| `extends` | string/object | None | Extend another service. | `extends: { file: common.yml, service: webapp }` |
| `external_links` | list | None | Link to containers started outside this compose. | `external_links: ["redis_1", "project_db_1:mysql"]` |
| `extra_hosts` | list/map | None | Add hostname mappings. | `extra_hosts: ["somehost:162.242.195.82"]` |
| `group_add` | list | None | Add additional groups. | `group_add: ["mail"]` |
| `healthcheck` | object | None | Configure a check that's run to determine whether or not containers for this service are "healthy". | `healthcheck: { test: ["CMD", "curl", "-f", "http://localhost"] }` |
| `hostname` | string | None | Custom host name. | `hostname: foo` |
| `image` | string | None | Specify the image to start the container from. | `image: redis:alpine` |
| `init` | boolean | `false` | Run an init inside the container that forwards signals and reaps processes. | `init: true` |
| `ipc` | string | None | IPC namespace to use. | `ipc: host` |
| `isolation` | string | None | Specify a container's isolation technology. | `isolation: default` |
| `labels` | map/list | None | Add metadata to containers. | `labels: { com.example.description: "Accounting webapp" }` |
| `links` | list | None | Link to containers in another service. | `links: ["db", "db:database"]` |
| `logging` | object | None | Logging configuration for the service. | `logging: { driver: syslog, options: { syslog-address: "tcp://192.168.0.42:123" } }` |
| `mac_address` | string | None | MAC address. | `mac_address: 02:42:ac:11:65:43` |
| `mem_limit` | string | None | Memory limit. | `mem_limit: 1g` |
| `mem_reservation` | string | None | Memory soft limit. | `mem_reservation: 512m` |
| `mem_swappiness` | integer | None | Tune a container's memory swappiness behavior. | `mem_swappiness: 60` |
| `memswap_limit` | string | None | Swap limit equal to memory plus swap. | `memswap_limit: 2g` |
| `network_mode` | string | None | Network mode. | `network_mode: "host"` |
| `networks` | list/map | None | Networks to join. | `networks: ["frontend", "backend"]` |
| `oom_kill_disable` | boolean | `false` | Disable OOM Killer. | `oom_kill_disable: true` |
| `oom_score_adj` | integer | None | Tune the host's OOM preferences for containers. | `oom_score_adj: 500` |
| `pid` | string | None | PID namespace to use. | `pid: "host"` |
| `pids_limit` | integer | None | Tune a container's pids limit. | `pids_limit: 100` |
| `platform` | string | None | Target platform containers for this service will run on. | `platform: linux/amd64` |
| `ports` | list | None | Expose ports. | `ports: ["3000", "8000:8000", "9000:8080"]` |
| `privileged` | boolean | `false` | Give extended privileges to this container. | `privileged: true` |
| `profiles` | list | None | Define a list of named profiles for the service to be enabled under. | `profiles: ["frontend", "debug"]` |
| `pull_policy` | string | `always` | Define the decisions Compose makes when it starts to pull images. | `pull_policy: missing` |
| `read_only` | boolean | `false` | Mount the container's root filesystem as read only. | `read_only: true` |
| `restart` | string | `no` | Restart policy. | `restart: always` |
| `runtime` | string | None | Specify the runtime to use for the container. | `runtime: runc` |
| `scale` | integer | `1` | Specify the default number of containers to deploy for this service. | `scale: 3` |
| `secrets` | list | None | Grant access to secrets on a per-service basis. | `secrets: ["my_secret", "my_other_secret"]` |
| `security_opt` | list | None | Override the default labeling scheme for each container. | `security_opt: ["label:user:USER", "label:role:ROLE"]` |
| `shm_size` | string | None | Size of `/dev/shm`. | `shm_size: '2gb'` |
| `stdin_open` | boolean | `false` | Keep STDIN open even if not attached. | `stdin_open: true` |
| `stop_grace_period` | string | `10s` | Specify how long to wait when attempting to stop a container if it doesn't handle SIGTERM. | `stop_grace_period: 1m30s` |
| `stop_signal` | string | `SIGTERM` | Set an alternative signal to stop the container. | `stop_signal: SIGUSR1` |
| `storage_opt` | map | None | Storage driver options for this service. | `storage_opt: { size: '120G' }` |
| `sysctls` | map/list | None | Kernel parameters to set in the container. | `sysctls: { net.core.somaxconn: 1024 }` |
| `tmpfs` | string/list | None | Mount a temporary file system inside the container. | `tmpfs: /run` |
| `tty` | boolean | `false` | Allocate a pseudo-TTY. | `tty: true` |
| `ulimits` | map | None | Override the default ulimits for a container. | `ulimits: { nproc: 65535, nofile: { soft: 20000, hard: 40000 } }` |
| `user` | string | None | Override the user used to run the container process. | `user: "1000:1000"` |
| `userns_mode` | string | None | Disable the user namespace for this service, if Docker daemon is configured with user namespaces. | `userns_mode: "host"` |
| `uts` | string | None | UTS namespace to use. | `uts: "host"` |
| `volumes` | list | None | Mount host paths or named volumes, specified as sub-options to a service. | `volumes: ["/var/lib/mysql", "./cache:/tmp/cache", "datavolume:/var/lib/mysql"]` |
| `volumes_from` | list | None | Mount all of the volumes from another service or container. | `volumes_from: ["service_name", "container_name"]` |
| `working_dir` | string | None | Override the container's working directory. | `working_dir: /code` |

### 1.3 `networks` Attributes

| Attribute | Type | Default | Description | Example |
|---|---|---|---|---|
| `driver` | string | `bridge` | Specify which driver should be used for this network. | `driver: overlay` |
| `driver_opts` | map | None | Specify a list of options as key-value pairs to pass to the driver. | `driver_opts: { com.docker.network.bridge.name: br_1 }` |
| `attachable` | boolean | `false` | Only used when the driver is set to `overlay`. If set to `true`, then standalone containers can attach to this network. | `attachable: true` |
| `enable_ipv6` | boolean | `false` | Enable IPv6 networking. | `enable_ipv6: true` |
| `internal` | boolean | `false` | By default, Docker also connects a bridge network to it to provide external connectivity. If you want to create an externally isolated overlay network, you can set this option to `true`. | `internal: true` |
| `labels` | map/list | None | Add metadata to containers. | `labels: { com.example.description: "Financial transaction network" }` |
| `external` | boolean | `false` | If set to `true`, specifies that this network has been created outside of Compose. | `external: true` |
| `name` | string | None | Set a custom name for this network. | `name: my-app-net` |
| `ipam` | object | None | Specify custom IPAM config. | `ipam: { driver: default, config: [{ subnet: "172.28.0.0/16" }] }` |

### 1.4 `volumes` Attributes

| Attribute | Type | Default | Description | Example |
|---|---|---|---|---|
| `driver` | string | `local` | Specify which volume driver should be used for this volume. | `driver: foobar` |
| `driver_opts` | map | None | Specify a list of options as key-value pairs to pass to the driver for this volume. | `driver_opts: { type: "nfs", o: "addr=10.40.0.199,nolock,soft,rw", device: ":/docker/example" }` |
| `external` | boolean | `false` | If set to `true`, specifies that this volume has been created outside of Compose. | `external: true` |
| `labels` | map/list | None | Add metadata to containers. | `labels: { com.example.description: "Database volume" }` |
| `name` | string | None | Set a custom name for this volume. | `name: my-app-data` |

### 1.5 `configs` Attributes

| Attribute | Type | Default | Description | Example |
|---|---|---|---|---|
| `file` | string | None | The config is created with the contents of the file at the specified path. | `file: ./my_config.txt` |
| `external` | boolean | `false` | If set to `true`, specifies that this config has already been created. | `external: true` |
| `name` | string | None | The name of the config object in Docker. | `name: my_config` |
| `content` | string | None | The content of the config. | `content: | 
  server {
    listen 80;
  }` |

### 1.6 `secrets` Attributes

| Attribute | Type | Default | Description | Example |
|---|---|---|---|---|
| `file` | string | None | The secret is created with the contents of the file at the specified path. | `file: ./my_secret.txt` |
| `environment` | string | None | The secret is created with the value of an environment variable. | `environment: "MY_SECRET"` |
| `external` | boolean | `false` | If set to `true`, specifies that this secret has already been created. | `external: true` |
| `name` | string | None | The name of the secret object in Docker. | `name: my_secret` |

## 2. Dockerfile Instruction Reference

A complete reference for every Dockerfile instruction.

### `FROM`
Initializes a new build stage and sets the Base Image for subsequent instructions.
- Syntax: `FROM [--platform=<platform>] <image> [AS <name>]` or `FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]` or `FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]`
- Example: `FROM --platform=linux/amd64 ubuntu:22.04 AS builder`

### `RUN`
Executes any commands in a new layer on top of the current image and commits the results.
- Syntax: `RUN <command>` (shell form) or `RUN ["executable", "param1", "param2"]` (exec form)
- Flags: `--mount=type=cache|bind|secret|ssh`, `--network=default|none|host`
- Example: `RUN --mount=type=cache,target=/var/cache/apt apt-get update && apt-get install -y curl`

### `CMD`
Provides defaults for an executing container. There can only be one `CMD` instruction in a Dockerfile.
- Syntax: `CMD ["executable","param1","param2"]` (exec form, preferred) or `CMD ["param1","param2"]` (as default parameters to ENTRYPOINT) or `CMD command param1 param2` (shell form)
- Example: `CMD ["node", "server.js"]`

### `LABEL`
Adds metadata to an image.
- Syntax: `LABEL <key>=<value> <key>=<value> <key>=<value> ...`
- Example: `LABEL org.opencontainers.image.authors="team@example.com"`

### `EXPOSE`
Informs Docker that the container listens on the specified network ports at runtime.
- Syntax: `EXPOSE <port> [<port>/<protocol>...]`
- Example: `EXPOSE 80/tcp 80/udp`

### `ENV`
Sets the environment variable `<key>` to the value `<value>`.
- Syntax: `ENV <key>=<value> ...`
- Example: `ENV NODE_ENV=production PORT=3000`

### `ADD`
Copies new files, directories or remote file URLs from `<src>` and adds them to the filesystem of the image at the path `<dest>`.
- Syntax: `ADD [--chown=<user>:<group>] [--chmod=<perms>] [--checksum=<checksum>] <src>... <dest>`
- Example: `ADD https://example.com/big.tar.xz /usr/src/things/`

### `COPY`
Copies new files or directories from `<src>` and adds them to the filesystem of the container at the path `<dest>`.
- Syntax: `COPY [--chown=<user>:<group>] [--chmod=<perms>] <src>... <dest>`
- Example: `COPY --chown=node:node package*.json ./`

### `ENTRYPOINT`
Allows you to configure a container that will run as an executable.
- Syntax: `ENTRYPOINT ["executable", "param1", "param2"]` (exec form, preferred) or `ENTRYPOINT command param1 param2` (shell form)
- Example: `ENTRYPOINT ["docker-entrypoint.sh"]`

### `VOLUME`
Creates a mount point with the specified name and marks it as holding externally mounted volumes from native host or other containers.
- Syntax: `VOLUME ["/data"]`
- Example: `VOLUME /var/lib/mysql`

### `USER`
Sets the user name (or UID) and optionally the user group (or GID) to use when running the image and for any `RUN`, `CMD` and `ENTRYPOINT` instructions that follow it in the Dockerfile.
- Syntax: `USER <user>[:<group>]` or `USER <UID>[:<GID>]`
- Example: `USER 1000:1000`

### `WORKDIR`
Sets the working directory for any `RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD` instructions that follow it in the Dockerfile.
- Syntax: `WORKDIR /path/to/workdir`
- Example: `WORKDIR /app`

### `ARG`
Defines a variable that users can pass at build-time to the builder with the `docker build` command using the `--build-arg <varname>=<value>` flag.
- Syntax: `ARG <name>[=<default value>]`
- Example: `ARG VERSION=latest`

### `ONBUILD`
Adds to the image a trigger instruction to be executed at a later time, when the image is used as the base for another build.
- Syntax: `ONBUILD <INSTRUCTION>`
- Example: `ONBUILD COPY . /app/src`

### `STOPSIGNAL`
Sets the system call signal that will be sent to the container to exit.
- Syntax: `STOPSIGNAL signal`
- Example: `STOPSIGNAL SIGKILL`

### `HEALTHCHECK`
Tells Docker how to test a container to check that it is still working.
- Syntax: `HEALTHCHECK [OPTIONS] CMD command` or `HEALTHCHECK NONE`
- Options: `--interval=DURATION` (default: 30s), `--timeout=DURATION` (default: 30s), `--start-period=DURATION` (default: 0s), `--retries=N` (default: 3)
- Example: `HEALTHCHECK --interval=5m --timeout=3s CMD curl -f http://localhost/ || exit 1`

### `SHELL`
Allows the default shell used for the shell form of commands to be overridden.
- Syntax: `SHELL ["executable", "parameters"]`
- Example: `SHELL ["powershell", "-command"]`

## 3. `daemon.json` Complete Reference

The `daemon.json` file configures the Docker daemon.

| Field | Type | Default | Description |
|---|---|---|---|
| `storage-driver` | string | `overlay2` | The storage driver to use. |
| `log-driver` | string | `json-file` | The default logging driver. |
| `log-opts` | map | None | Options for the logging driver. |
| `default-address-pools` | list | None | Default address pools for node networks. |
| `dns` | list | None | DNS servers to use. |
| `registry-mirrors` | list | None | Registry mirrors to use. |
| `insecure-registries` | list | None | Insecure registries to allow. |
| `live-restore` | boolean | `false` | Enable live restore of docker when containers are still running. |
| `default-runtime` | string | `runc` | Default OCI runtime for containers. |
| `runtimes` | map | None | Register additional OCI runtimes. |
| `features` | map | None | Enable/disable specific features. |
| `builder` | map | None | BuildKit configuration. |
| `containerd` | string | None | Path to containerd socket. |
| `default-cgroupns-mode` | string | `private` | Default cgroup namespace mode. |
| `exec-opts` | list | None | Execution options. |
| `experimental` | boolean | `false` | Enable experimental features. |
| `fixed-cidr` | string | None | IPv4 subnet for fixed IPs. |
| `fixed-cidr-v6` | string | None | IPv6 subnet for fixed IPs. |
| `group` | string | `docker` | Group for the unix socket. |
| `hosts` | list | None | Daemon socket(s) to connect to. |
| `icc` | boolean | `true` | Enable inter-container communication. |
| `ip` | string | `0.0.0.0` | Default IP when binding container ports. |
| `ip-forward` | boolean | `true` | Enable net.ipv4.ip_forward. |
| `iptables` | boolean | `true` | Enable addition of iptables rules. |
| `ip-masq` | boolean | `true` | Enable IP masquerading. |
| `labels` | list | None | Daemon labels. |
| `max-concurrent-downloads` | integer | `3` | Max concurrent downloads. |
| `max-concurrent-uploads` | integer | `5` | Max concurrent uploads. |
| `max-download-attempts` | integer | `5` | Max download attempts. |
| `metrics-addr` | string | None | Address to serve metrics API. |
| `no-new-privileges` | boolean | `false` | Set no-new-privileges by default for new containers. |
| `oom-score-adjust` | integer | `-500` | Set the oom_score_adj for the daemon. |
| `pidfile` | string | `/var/run/docker.pid` | Path to use for daemon PID file. |
| `raw-logs` | boolean | `false` | Full timestamps without ANSI coloring. |
| `seccomp-profile` | string | None | Path to seccomp profile. |
| `selinux-enabled` | boolean | `false` | Enable selinux support. |
| `shutdown-timeout` | integer | `15` | Default timeout for stopping containers. |
| `tls` | boolean | `false` | Use TLS; implied by --tlsverify. |
| `tlscacert` | string | `~/.docker/ca.pem` | Trust certs signed only by this CA. |
| `tlscert` | string | `~/.docker/cert.pem` | Path to TLS certificate file. |
| `tlskey` | string | `~/.docker/key.pem` | Path to TLS key file. |
| `tlsverify` | boolean | `false` | Use TLS and verify the remote. |
| `userland-proxy` | boolean | `true` | Use userland proxy for loopback traffic. |
| `userns-remap` | string | None | User namespace remapping. |

## 4. `.dockerignore` Patterns

The `.dockerignore` file excludes files and directories from the build context.

### Syntax
- `#` for comments.
- `*` matches any sequence of non-separator characters.
- `?` matches any single non-separator character.
- `**` matches any number of directories.
- `!` negates a pattern.

### Common Patterns

**Node.js:**
```dockerignore
node_modules
npm-debug.log
Dockerfile
.dockerignore
.git
.gitignore
README.md
```

**Python:**
```dockerignore
__pycache__
*.pyc
*.pyo
*.pyd
.Python
env/
venv/
.env
Dockerfile
.dockerignore
.git
```

**Go:**
```dockerignore
bin/
obj/
*.exe
*.dll
*.so
*.dylib
Dockerfile
.dockerignore
.git
```

## 5. BuildKit Configuration (`buildkitd.toml`)

BuildKit can be configured via `buildkitd.toml`.

```toml
debug = true
# root is where all buildkit state is stored.
root = "/var/lib/buildkit"
# insecure-entitlements allows insecure entitlements, disabled by default.
insecure-entitlements = [ "network.host", "security.insecure" ]

[grpc]
  address = [ "tcp://0.0.0.0:1234" ]
  # debugAddress is address for attaching go pprof and expvar.
  debugAddress = "0.0.0.0:6060"
  uid = 0
  gid = 0
  [grpc.tls]
    cert = "/etc/buildkit/tls.crt"
    key = "/etc/buildkit/tls.key"
    ca = "/etc/buildkit/tlsca.crt"

[worker.oci]
  enabled = true
  # platforms is manually configure platforms, auto-detected by default.
  platforms = [ "linux/amd64", "linux/arm64" ]
  snapshotter = "auto" # overlayfs or native, default auto will try to use overlayfs
  rootless = false # see docs/rootless.md for more details on rootless mode.
  # Whether run subprocesses in main cgroup or create top-level cgroup.
  # Default is "cgroupfs" when not running rootless.
  cgroup-parent = "cgroupfs"
  # gc keeps/frees disk space.
  gc = true
  gckeepstorage = 9000
  [[worker.oci.gcpolicy]]
    keepBytes = 512000000
    keepDuration = 172800
    filters = [ "type==source.local", "type==exec.cachemount", "type==source.git.checkout"]
  [[worker.oci.gcpolicy]]
    all = true
    keepBytes = 1024000000

[worker.containerd]
  address = "/run/containerd/containerd.sock"
  enabled = true
  platforms = [ "linux/amd64", "linux/arm64" ]
  namespace = "buildkit"
  gc = true
  # gckeepstorage sets storage limit for default gc profile, in MB.
  gckeepstorage = 9000

[registry."docker.io"]
  mirrors = ["YOUR_REGISTRY_MIRROR"]
  http = true
  insecure = true
```

## 6. Docker Context Configuration

Docker contexts allow you to manage multiple Docker environments.

- Create a context: `docker context create my-context --docker "host=ssh://user@remote-host"`
- Use a context: `docker context use my-context`
- List contexts: `docker context ls`
- Inspect a context: `docker context inspect my-context`

## 7. Registry Configuration (`config.yml`)

Configuration for a private Docker registry.

```yaml
version: 0.1
log:
  fields:
    service: registry
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
```

## 8. Production `compose.yaml` Templates

### 8.1 Web App Stack

```yaml
services:
  webapp:
    image: YOUR_REGISTRY/webapp:v1.2.3
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    environment:
      - NODE_ENV=production
    secrets:
      - db_password
    networks:
      - frontend
      - backend
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

networks:
  frontend:
  backend:
    internal: true

secrets:
  db_password:
    external: true
```

### 8.2 Database Stack

```yaml
services:
  db:
    image: postgres:15-alpine
    restart: always
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
      POSTGRES_DB: myapp
    volumes:
      - db-data:/var/lib/postgresql/data
    secrets:
      - db_password
    networks:
      - backend
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 4G
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin -d myapp"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  db-data:

networks:
  backend:
    internal: true

secrets:
  db_password:
    external: true
```

### 8.3 Monitoring Stack (Prometheus/Grafana)

```yaml
services:
  prometheus:
    image: prom/prometheus:v2.45.0
    restart: unless-stopped
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
    ports:
      - "9090:9090"
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:10.0.3
    restart: unless-stopped
    volumes:
      - grafana-data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD__FILE=/run/secrets/grafana_password
    secrets:
      - grafana_password
    ports:
      - "3000:3000"
    networks:
      - monitoring
    depends_on:
      - prometheus

volumes:
  prometheus-data:
  grafana-data:

networks:
  monitoring:

secrets:
  grafana_password:
    external: true
```

### 8.4 ELK Stack

```yaml
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.9.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - es-data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
    networks:
      - elk

  logstash:
    image: docker.elastic.co/logstash/logstash:8.9.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf:ro
    ports:
      - "5044:5044"
    networks:
      - elk
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:8.9.0
    ports:
      - "5601:5601"
    networks:
      - elk
    depends_on:
      - elasticsearch

volumes:
  es-data:

networks:
  elk:
```

## 9. Troubleshooting and Optimization

### 9.1 Common Errors and Solutions

- **"port already in use"**: Use `lsof -i :PORT` to find the conflicting process, or change the port mapping in `compose.yaml`.
- **"no space left on device"**: Run `docker system prune -a --volumes` to clear unused data. Check the `overlay2` directory size.
- **"OOM killed"**: The container exceeded its memory limit. Increase the `mem_limit` in `compose.yaml` or optimize the application's memory usage.
- **"permission denied"**: Check the user/group permissions of the mounted volumes. Ensure the container user has access.
- **"network not found"**: Run `docker network create <network_name>` or ensure the network is defined in `compose.yaml`.
- **"image not found"**: Verify the registry URL, image tag, and `pull_policy`. Ensure you are logged in to the registry.
- **"container unhealthy"**: Check the `healthcheck` command and logs. Increase the `timeout` or `start_period` if the application takes longer to start.
- **"bind mount permission denied"**: On SELinux systems, append `:z` or `:Z` to the volume mount path (e.g., `./data:/data:z`).
- **"DNS resolution failed"**: Check the host's DNS settings or configure custom DNS servers in `daemon.json` or `compose.yaml`.
- **"cannot start service"**: Check `depends_on` conditions. Ensure required services are healthy before starting dependent services.
- **"exec format error"**: The image architecture does not match the host architecture (e.g., running an ARM image on an AMD64 host). Use `docker buildx` to build multi-platform images.
- **"context deadline exceeded"**: Increase the timeout for Docker commands or check network connectivity to the registry.

### 9.2 Cost and Time Optimization

- **Multi-stage builds**: Use multi-stage builds to create smaller final images. This reduces pull times and storage costs.
- **Layer caching**: Order Dockerfile instructions from least frequently changed to most frequently changed to maximize cache hits.
- **BuildKit cache mounts**: Use `--mount=type=cache` to cache package manager downloads (e.g., `apt`, `npm`, `pip`) between builds.
- **Parallel builds**: Use `docker compose build --parallel` to build multiple services concurrently.
- **Image pull policy**: Set `pull_policy: if-not-present` to avoid unnecessary image pulls.
- **Resource limits**: Set CPU and memory limits to prevent runaway containers from consuming all host resources.
- **Logging**: Configure log rotation (`max-size`, `max-file`) to prevent log files from filling up the disk.
- **Prune**: Regularly run `docker system prune` and `docker volume prune` to remove unused resources.
- **Compose profiles**: Use profiles to start only the services needed for a specific environment or task.
- **Compose watch**: Use `develop: watch` for fast iteration during development without rebuilding images.

### 9.3 Security Hardening

- **Non-root user**: Run containers as a non-root user (`USER 1000:1000`).
- **Read-only root filesystem**: Set `read_only: true` to prevent modifications to the container's root filesystem.
- **Drop capabilities**: Drop all Linux capabilities (`cap_drop: ["ALL"]`) and add only the necessary ones.
- **No new privileges**: Set `security_opt: ["no-new-privileges:true"]` to prevent processes from gaining additional privileges.
- **Seccomp profiles**: Use custom seccomp profiles to restrict system calls.
- **Resource limits**: Enforce CPU, memory, and PID limits to prevent denial-of-service attacks.
- **No privileged mode**: Avoid using `privileged: true` unless absolutely necessary.
- **Minimal base images**: Use minimal base images like `alpine`, `distroless`, or `scratch` to reduce the attack surface.
- **Secrets management**: Use Docker secrets instead of environment variables for sensitive data.
- **Network segmentation**: Use internal networks to isolate backend services from the public internet.

### 9.4 Upgrade Strategies

- **Blue-green deployment**: Run the new version alongside the old version and switch traffic when ready.
- **Rolling update**: Use `update_config` with `parallelism` and `delay` to update containers one by one.
- **Canary release**: Route a small percentage of traffic to the new version to test it before a full rollout.
- **Database migrations**: Run database migrations as an init container or a pre-start hook before starting the application.
- **Rollback**: Configure `rollback_config` to automatically roll back to the previous version if the update fails.
- **Zero-downtime**: Use `order: start-first` in `update_config` to start the new container before stopping the old one.
