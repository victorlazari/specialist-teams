# Dockerfile Instruction and CLI Command Reference

## 1. Introduction

The `Dockerfile` is the fundamental building block for creating container images in the Docker ecosystem. It is a text document that contains all the commands a user could call on the command line to assemble an image. This comprehensive reference guide provides an in-depth look at every Dockerfile instruction, along with the associated `docker build` CLI commands, flags, arguments, and detailed examples of usage.

Whether you are a novice containerizing your first application or a senior DevOps engineer optimizing multi-stage builds for enterprise deployments, this guide serves as the ultimate resource for mastering Dockerfile syntax and build execution.

## 2. Dockerfile Instructions Reference

A Dockerfile consists of a series of instructions, each creating a new layer in the resulting image. Understanding the nuances of each instruction is critical for building secure, efficient, and minimal container images.

### 2.1. FROM

The `FROM` instruction initializes a new build stage and sets the Base Image for subsequent instructions. As such, a valid `Dockerfile` must start with a `FROM` instruction.

**Syntax:**
```dockerfile
FROM [--platform=<platform>] <image> [AS <name>]
FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]
FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]
```

**Arguments:**
- `--platform`: Specifies the platform of the image in case `FROM` references a multi-platform image (e.g., `linux/amd64`, `linux/arm64`).
- `<image>`: The name of the base image.
- `<tag>`: (Optional) The specific version of the image. Defaults to `latest` if omitted.
- `<digest>`: (Optional) A cryptographic hash of the image for strict immutability.
- `AS <name>`: (Optional) Assigns a name to the build stage, which can be referenced later in multi-stage builds using `COPY --from=<name>`.

**Examples:**
```dockerfile
# Basic usage
FROM ubuntu:22.04

# Multi-stage build with naming
FROM golang:1.20-alpine AS builder

# Specifying a platform and digest for reproducible builds
FROM --platform=linux/amd64 node:18-alpine@sha256:1234567890abcdef
```

### 2.2. RUN

The `RUN` instruction executes any commands in a new layer on top of the current image and commits the results. The resulting committed image will be used for the next step in the `Dockerfile`.

**Syntax:**
```dockerfile
# Shell form (runs in /bin/sh -c on Linux or cmd /S /C on Windows)
RUN <command>

# Exec form (does not invoke a command shell)
RUN ["executable", "param1", "param2"]
```

**Flags (BuildKit only):**
- `--mount`: Allows mounting of files, directories, or caches without copying them into the image.
- `--network`: Controls the networking environment for the command.
- `--security`: Controls the security sandbox.

**Examples:**
```dockerfile
# Shell form
RUN apt-get update && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*

# Exec form
RUN ["/bin/bash", "-c", "echo Hello World"]

# Using BuildKit cache mounts to speed up dependency installation
RUN --mount=type=cache,target=/root/.npm \
    npm install
```

### 2.3. CMD

The `CMD` instruction provides defaults for an executing container. These defaults can include an executable, or they can omit the executable, in which case you must specify an `ENTRYPOINT` instruction as well. There can only be one `CMD` instruction in a `Dockerfile`. If you list more than one `CMD`, only the last `CMD` will take effect.

**Syntax:**
```dockerfile
# Exec form (preferred)
CMD ["executable","param1","param2"]

# Default parameters to ENTRYPOINT
CMD ["param1","param2"]

# Shell form
CMD command param1 param2
```

**Examples:**
```dockerfile
# Running a Node.js application
CMD ["node", "app.js"]

# Providing default arguments to an ENTRYPOINT
ENTRYPOINT ["ping"]
CMD ["localhost"]
```

### 2.4. LABEL

The `LABEL` instruction adds metadata to an image. A `LABEL` is a key-value pair. To include spaces within a `LABEL` value, use quotes and backslashes as you would in command-line parsing.

**Syntax:**
```dockerfile
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```

**Examples:**
```dockerfile
LABEL maintainer="engineering@example.com"
LABEL version="1.0.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
```

### 2.5. EXPOSE

The `EXPOSE` instruction informs Docker that the container listens on the specified network ports at runtime. You can specify whether the port listens on TCP or UDP, and the default is TCP if the protocol is not specified. Note that `EXPOSE` does not actually publish the port; it functions as a type of documentation between the person who builds the image and the person who runs the container.

**Syntax:**
```dockerfile
EXPOSE <port> [<port>/<protocol>...]
```

**Examples:**
```dockerfile
EXPOSE 80
EXPOSE 80/tcp
EXPOSE 80/udp
```

### 2.6. ENV

The `ENV` instruction sets the environment variable `<key>` to the value `<value>`. This value will be in the environment for all subsequent instructions in the build stage and will persist when a container is run from the resulting image.

**Syntax:**
```dockerfile
ENV <key>=<value> ...
```

**Examples:**
```dockerfile
ENV APP_HOME=/usr/src/app \
    NODE_ENV=production \
    PORT=8080
```

### 2.7. ADD

The `ADD` instruction copies new files, directories, or remote file URLs from `<src>` and adds them to the filesystem of the image at the path `<dest>`.

**Syntax:**
```dockerfile
ADD [--chown=<user>:<group>] [--chmod=<perms>] [--checksum=<checksum>] <src>... <dest>
ADD [--chown=<user>:<group>] [--chmod=<perms>] ["<src>",... "<dest>"]
```

**Features:**
- Automatic extraction of local tar archives.
- Downloading of remote URLs.

**Examples:**
```dockerfile
# Copying a local file
ADD hom* /mydir/

# Downloading a remote file
ADD https://example.com/big.tar.xz /usr/src/things/

# Extracting a local archive
ADD rootfs.tar.xz /
```

### 2.8. COPY

The `COPY` instruction copies new files or directories from `<src>` and adds them to the filesystem of the container at the path `<dest>`. Unlike `ADD`, `COPY` does not support remote URLs or automatic archive extraction, making it the preferred instruction for simply copying local files.

**Syntax:**
```dockerfile
COPY [--chown=<user>:<group>] [--chmod=<perms>] <src>... <dest>
COPY [--chown=<user>:<group>] [--chmod=<perms>] ["<src>",... "<dest>"]
```

**Flags:**
- `--from=<name>`: Copies files from a previous build stage or another image instead of the build context.
- `--chown`: Changes the ownership of the copied files.
- `--chmod`: Changes the permissions of the copied files.

**Examples:**
```dockerfile
# Basic copy
COPY requirements.txt /app/

# Copying with ownership change
COPY --chown=node:node . /app

# Copying from a previous build stage
COPY --from=builder /go/src/app/bin/server /usr/local/bin/
```

### 2.9. ENTRYPOINT

An `ENTRYPOINT` allows you to configure a container that will run as an executable. Command line arguments to `docker run <image>` will be appended after all elements in an exec form `ENTRYPOINT`, and will override all elements specified using `CMD`.

**Syntax:**
```dockerfile
# Exec form (preferred)
ENTRYPOINT ["executable", "param1", "param2"]

# Shell form
ENTRYPOINT command param1 param2
```

**Examples:**
```dockerfile
ENTRYPOINT ["nginx", "-g", "daemon off;"]
```

### 2.10. VOLUME

The `VOLUME` instruction creates a mount point with the specified name and marks it as holding externally mounted volumes from native host or other containers.

**Syntax:**
```dockerfile
VOLUME ["/data"]
VOLUME /var/log /var/db
```

**Examples:**
```dockerfile
VOLUME ["/var/lib/mysql"]
```

### 2.11. USER

The `USER` instruction sets the user name (or UID) and optionally the user group (or GID) to use as the default user and group for the remainder of the current stage.

**Syntax:**
```dockerfile
USER <user>[:<group>]
USER <UID>[:<GID>]
```

**Examples:**
```dockerfile
RUN groupadd -r postgres && useradd --no-log-init -r -g postgres postgres
USER postgres
```

### 2.12. WORKDIR

The `WORKDIR` instruction sets the working directory for any `RUN`, `CMD`, `ENTRYPOINT`, `COPY`, and `ADD` instructions that follow it in the `Dockerfile`. If the `WORKDIR` doesn't exist, it will be created even if it's not used in any subsequent `Dockerfile` instruction.

**Syntax:**
```dockerfile
WORKDIR /path/to/workdir
```

**Examples:**
```dockerfile
WORKDIR /usr/src/app
COPY . .
RUN npm install
```

### 2.13. ARG

The `ARG` instruction defines a variable that users can pass at build-time to the builder with the `docker build` command using the `--build-arg <varname>=<value>` flag.

**Syntax:**
```dockerfile
ARG <name>[=<default value>]
```

**Examples:**
```dockerfile
ARG VERSION=latest
FROM alpine:$VERSION

ARG BUILD_DATE
LABEL build_date=$BUILD_DATE
```

### 2.14. ONBUILD

The `ONBUILD` instruction adds to the image a trigger instruction to be executed at a later time, when the image is used as the base for another build.

**Syntax:**
```dockerfile
ONBUILD <INSTRUCTION>
```

**Examples:**
```dockerfile
ONBUILD ADD . /app/src
ONBUILD RUN /usr/local/bin/python-build --dir /app/src
```

### 2.15. STOPSIGNAL

The `STOPSIGNAL` instruction sets the system call signal that will be sent to the container to exit.

**Syntax:**
```dockerfile
STOPSIGNAL signal
```

**Examples:**
```dockerfile
STOPSIGNAL SIGKILL
```

### 2.16. HEALTHCHECK

The `HEALTHCHECK` instruction tells Docker how to test a container to check that it is still working.

**Syntax:**
```dockerfile
HEALTHCHECK [OPTIONS] CMD command
HEALTHCHECK NONE
```

**Options:**
- `--interval=DURATION` (default: 30s)
- `--timeout=DURATION` (default: 30s)
- `--start-period=DURATION` (default: 0s)
- `--retries=N` (default: 3)

**Examples:**
```dockerfile
HEALTHCHECK --interval=5m --timeout=3s \
  CMD curl -f http://localhost/ || exit 1
```

### 2.17. SHELL

The `SHELL` instruction allows the default shell used for the shell form of commands to be overridden.

**Syntax:**
```dockerfile
SHELL ["executable", "parameters"]
```

**Examples:**
```dockerfile
SHELL ["powershell", "-command"]
```

---

## 3. Docker Build CLI Command Reference

The `docker build` command builds Docker images from a Dockerfile and a "context". A build's context is the set of files at a specified location `PATH` or `URL`.

### 3.1. Basic Usage

**Syntax:**
```bash
docker build [OPTIONS] PATH | URL | -
```

**Examples:**
```bash
# Build an image using the current directory as context
docker build .

# Build an image and tag it
docker build -t myapp:1.0 .

# Build an image using a specific Dockerfile
docker build -f /path/to/Dockerfile .
```

### 3.2. Core Flags and Arguments

#### `--tag` / `-t`
Name and optionally a tag in the `name:tag` format.

**Example:**
```bash
docker build -t my-registry.com/my-app:v2.1.0 .
```

#### `--file` / `-f`
Name of the Dockerfile (Default is `PATH/Dockerfile`).

**Example:**
```bash
docker build -f dockerfiles/Dockerfile.prod .
```

#### `--build-arg`
Set build-time variables.

**Example:**
```bash
docker build --build-arg HTTP_PROXY=http://10.20.30.2:1234 .
```

#### `--no-cache`
Do not use cache when building the image.

**Example:**
```bash
docker build --no-cache -t myapp:latest .
```

#### `--target`
Set the target build stage to build.

**Example:**
```bash
docker build --target builder -t myapp-builder .
```

#### `--platform`
Set platform if server is multi-platform capable.

**Example:**
```bash
docker build --platform linux/arm64 -t myapp:arm64 .
```

#### `--progress`
Set type of progress output (`auto`, `plain`, `tty`). Use `plain` to show container output.

**Example:**
```bash
docker build --progress=plain .
```

#### `--secret`
Secret file to expose to the build (only if BuildKit enabled).

**Example:**
```bash
docker build --secret id=mysecret,src=mysecret.txt .
```

#### `--ssh`
SSH agent socket or keys to expose to the build (only if BuildKit enabled).

**Example:**
```bash
docker build --ssh default .
```

### 3.3. Advanced BuildKit Features

Docker BuildKit is the modern build engine for Docker, offering improved performance, caching, and security features. It is enabled by default in recent Docker versions.

#### Cache Exporters and Importers
BuildKit allows exporting the build cache to an external location (like a registry) and importing it later to speed up builds across different machines.

**Exporting Cache:**
```bash
docker buildx build --push -t myapp:latest \
  --cache-to type=registry,ref=myapp:buildcache .
```

**Importing Cache:**
```bash
docker buildx build --push -t myapp:latest \
  --cache-from type=registry,ref=myapp:buildcache .
```

#### Multi-Platform Builds
BuildKit makes it easy to build images for multiple architectures simultaneously.

**Example:**
```bash
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest --push .
```

### 3.4. Build Context Management

The build context is sent to the Docker daemon before the build starts. Managing the context size is crucial for build performance.

#### `.dockerignore` File
Before the docker CLI sends the context to the docker daemon, it looks for a file named `.dockerignore` in the root directory of the context. If this file exists, the CLI modifies the context to exclude files and directories that match patterns in it.

**Example `.dockerignore`:**
```text
# Ignore .git directory
.git

# Ignore node_modules
node_modules/

# Ignore sensitive files
.env
secrets/

# Ignore markdown files except README.md
*.md
!README.md
```

#### Remote Contexts
You can build images directly from a Git repository or a tarball URL.

**Git Repository:**
```bash
docker build https://github.com/docker/rootfs.git#container:docker
```

**Tarball URL:**
```bash
docker build http://server/context.tar.gz
```

## 4. Best Practices for Dockerfile and Builds

To maximize the efficiency, security, and maintainability of your Docker images, adhere to the following best practices:

1.  **Use Multi-Stage Builds:** Multi-stage builds allow you to use multiple `FROM` statements in your Dockerfile. Each `FROM` instruction can use a different base, and each of them begins a new stage of the build. You can selectively copy artifacts from one stage to another, leaving behind everything you don't want in the final image. This drastically reduces the final image size.
2.  **Minimize the Number of Layers:** Only the instructions `RUN`, `COPY`, `ADD` create layers. Other instructions create temporary intermediate images, and do not increase the size of the build. Combine `RUN` commands using `&&` to reduce the number of layers.
3.  **Leverage Build Cache:** Docker steps through the instructions in your Dockerfile in order. As each instruction is examined, Docker looks for an existing image in its cache that it can reuse, rather than creating a new (duplicate) image. Order your instructions from least likely to change to most likely to change to maximize cache hits.
4.  **Don't Install Unnecessary Packages:** Avoid installing extra or "nice to have" packages just because they might be useful. This reduces complexity, dependencies, file sizes, and build times.
5.  **Decouple Applications:** Each container should have only one concern. Decoupling applications into multiple containers makes it much easier to scale horizontally and reuse containers.
6.  **Sort Multi-Line Arguments:** Whenever possible, ease later changes by sorting multi-line arguments alphanumerically. This helps to avoid duplication of packages and makes the list much easier to update.
7.  **Use `.dockerignore`:** Always use a `.dockerignore` file to exclude unnecessary files and directories from the build context. This speeds up the build process and prevents sensitive data from accidentally being included in the image.
8.  **Run as Non-Root User:** By default, Docker containers run as the root user. This poses a significant security risk. Always create a dedicated user and group in your Dockerfile and use the `USER` instruction to switch to that user before running your application.

## 5. Conclusion

Mastering the `Dockerfile` syntax and the `docker build` CLI is essential for any modern software engineering workflow. By understanding the intricacies of each instruction and leveraging advanced features like BuildKit and multi-stage builds, you can create highly optimized, secure, and scalable container images. This comprehensive reference serves as a foundational guide to navigating the complexities of containerization and achieving operational excellence in your deployments.