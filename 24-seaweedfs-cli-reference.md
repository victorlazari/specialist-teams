# SeaweedFS CLI Command Reference

SeaweedFS is a distributed file system designed to handle large amounts of data efficiently. It is particularly well-suited for storing billions of files, each of which can be relatively small. SeaweedFS is known for its simplicity, high throughput, and ease of use. This document serves as a comprehensive CLI Command Reference for SeaweedFS, detailing each command, flag, and argument.

## Table of Contents

1. [Introduction to SeaweedFS Architecture](#introduction-to-seaweedfs-architecture)
2. [SeaweedFS Command Line Interface (CLI)](#seaweedfs-command-line-interface-cli)
3. [Master Command](#master-command)
   - [Start Master Server](#start-master-server)
   - [Master Server Options](#master-server-options)
4. [Volume Command](#volume-command)
   - [Start Volume Server](#start-volume-server)
   - [Volume Server Options](#volume-server-options)
5. [Filer Command](#filer-command)
   - [Start Filer Server](#start-filer-server)
   - [Filer Server Options](#filer-server-options)
6. [Weed Command](#weed-command)
   - [General Usage](#general-usage)
   - [Subcommands and Options](#subcommands-and-options)
7. [Troubleshooting](#troubleshooting)
8. [Examples of Usage](#examples-of-usage)
9. [Conclusion](#conclusion)

## Introduction to SeaweedFS Architecture

SeaweedFS is designed with a master-volume architecture. The master server keeps track of the file system's topology and file metadata, while the volume servers store the actual file data. This separation allows for efficient scaling and high availability.

- **Master Server**: Manages topology, file metadata, and volume server health.
- **Volume Server**: Stores file data and interacts directly with clients for file operations.
- **Filer Server**: Provides a filesystem-like interface for applications to interact with SeaweedFS.

## SeaweedFS Command Line Interface (CLI)

The `weed` command is the main command-line interface for SeaweedFS. It controls and interacts with the various components of the SeaweedFS architecture. The CLI provides flexibility and control over the system, allowing administrators to manage, configure, and monitor SeaweedFS effectively.

## Master Command

### Start Master Server

The Master Server is the brain of the SeaweedFS system. It is responsible for file metadata and managing volume servers.

```bash
weed master
```

### Master Server Options

- `-mdir`: Specifies the directory for metadata storage. Default is `/tmp/seaweedfs`.

  ```bash
  weed master -mdir=/data/seaweedfs/master
  ```

- `-ip`: Sets the IP address of the master server. Defaults to the local machine IP.

  ```bash
  weed master -ip=192.168.1.100
  ```

- `-port`: Sets the port for the master server. Default is `9333`.

  ```bash
  weed master -port=9333
  ```

- `-peers`: Specifies other master server addresses in a multi-master setup.

  ```bash
  weed master -peers=192.168.1.101:9333,192.168.1.102:9333
  ```

#### Example

Starting a master server on a custom directory and port:

```bash
weed master -mdir=/data/seaweedfs/master -port=9334
```

### Troubleshooting Master Server

- **Issue**: Master server fails to start.
  - **Solution**: Ensure the directory specified in `-mdir` is writable. Check port availability.

## Volume Command

### Start Volume Server

Volume servers store the actual file data and communicate with the master server for metadata and volume management.

```bash
weed volume
```

### Volume Server Options

- `-dir`: Specifies the data directory for storing volumes. Default is `/tmp/seaweedfs`.

  ```bash
  weed volume -dir=/data/seaweedfs/volume
  ```

- `-max`: Sets the maximum number of volumes. Default is `7`.

  ```bash
  weed volume -max=10
  ```

- `-ip`: Sets the IP address of the volume server.

  ```bash
  weed volume -ip=192.168.1.101
  ```

- `-port`: Sets the port for the volume server. Default is `8080`.

  ```bash
  weed volume -port=8081
  ```

#### Example

Starting a volume server with custom directory and maximum volume count:

```bash
weed volume -dir=/data/seaweedfs/volume -max=10
```

### Troubleshooting Volume Server

- **Issue**: Volume server cannot connect to master.
  - **Solution**: Verify the master server's IP and port. Check network connectivity.

## Filer Command

### Start Filer Server

The Filer Server provides a traditional filesystem interface for SeaweedFS, enabling applications to interact with it as if it were a regular file system.

```bash
weed filer
```

### Filer Server Options

- `-port`: Sets the port for the filer server. Default is `8888`.

  ```bash
  weed filer -port=8889
  ```

- `-master`: Specifies the master server's address.

  ```bash
  weed filer -master=192.168.1.100:9333
  ```

- `-defaultReplicaPlacement`: Sets default replica placement policy.

  ```bash
  weed filer -defaultReplicaPlacement=001
  ```

#### Example

Starting a filer server with a specified master server:

```bash
weed filer -master=192.168.1.100:9333 -port=8889
```

### Troubleshooting Filer Server

- **Issue**: Unable to access files via the filer.
  - **Solution**: Ensure the filer server is running and accessible. Verify the master server's address.

## Weed Command

### General Usage

The `weed` command is versatile, supporting various subcommands to interact with different components of SeaweedFS.

### Subcommands and Options

#### `weed backup`

Creates a backup of SeaweedFS metadata.

- `-master`: Address of the master server.
- `-dir`: Directory to store backup files.

```bash
weed backup -master=192.168.1.100:9333 -dir=/backup/seaweedfs
```

#### `weed export`

Exports files from SeaweedFS to a local directory.

- `-dir`: Local directory to store exported files.
- `-volumeId`: Specific volume ID to export.

```bash
weed export -dir=/local/export -volumeId=3
```

#### `weed shell`

Starts a shell interface for interacting with SeaweedFS.

- `-master`: Address of the master server.

```bash
weed shell -master=192.168.1.100:9333
```

#### `weed upload`

Uploads files to SeaweedFS.

- `-master`: Address of the master server.
- `-file`: File to upload.

```bash
weed upload -master=192.168.1.100:9333 -file=/path/to/file.txt
```

#### Example

Using `weed shell` to interact with SeaweedFS:

```bash
weed shell -master=192.168.1.100:9333
# Inside shell
> ls /
> put /local/path/to/file.txt /remote/path/in/seaweedfs/
```

### Troubleshooting CLI

- **Issue**: Command not found error.
  - **Solution**: Ensure the SeaweedFS binary is in your system's PATH. Check installation.

## Examples of Usage

### Scenario: Setting up a SeaweedFS Cluster

1. **Start the Master Server**:

   ```bash
   weed master -mdir=/data/seaweedfs/master -port=9333
   ```

2. **Start Volume Servers** on different nodes:

   ```bash
   weed volume -dir=/data/seaweedfs/volume1 -max=5 -port=8080
   weed volume -dir=/data/seaweedfs/volume2 -max=5 -port=8081
   ```

3. **Start Filer Server**:

   ```bash
   weed filer -master=192.168.1.100:9333 -port=8888
   ```

4. **Upload Files**:

   ```bash
   weed upload -master=192.168.1.100:9333 -file=/path/to/local/file.txt
   ```

5. **Access Files via Filer**:

   Use a web browser or REST API to interact with the filer server.

### Scenario: Backing Up Metadata

1. **Create a Backup**:

   ```bash
   weed backup -master=192.168.1.100:9333 -dir=/backup/seaweedfs
   ```

2. **Verify Backup**: 

   Check the backup directory for saved metadata files.

## Conclusion

SeaweedFS provides a robust and scalable solution for distributed file storage. Its command-line interface allows for flexible management and configuration of the system. By understanding the commands and options available, administrators can effectively deploy, manage, and troubleshoot SeaweedFS clusters. This reference guide should serve as a valuable resource for anyone working with SeaweedFS, providing detailed explanations and examples to facilitate efficient system operation.