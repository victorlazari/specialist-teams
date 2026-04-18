# SeaweedFS: Architecture, Core Components, and Production Deployment

---

## Introduction

SeaweedFS is a high-performance, distributed object storage system designed to provide efficient, scalable, and fault-tolerant storage for unstructured data. Its architecture is heavily inspired by the principles outlined in the [Haystack](https://www.cs.cornell.edu/projects/haystack/) and [F4](https://research.google/pubs/pub38149/) papers, which focus on optimizing disk access for large-scale photo storage and Facebook's cold storage system, respectively. One of the core innovations SeaweedFS brings is **O(1) disk access**, allowing near-constant time retrieval of files regardless of dataset size, a significant improvement over traditional file systems that often suffer from increasing latency as data volume grows.

The system is designed to serve multiple storage paradigms, including blob storage, file storage, object storage via S3 API compatibility, and integration with data warehouses. Additionally, SeaweedFS supports a FUSE-based filesystem mount, enabling seamless POSIX-like file interactions.

This document provides an in-depth exploration of SeaweedFS’s architecture, its three core components, storage paradigms, replication strategies, volume management, filer metadata solutions, S3 compatibility, and practical deployment considerations for production environments.

---

## SeaweedFS Architecture Overview

SeaweedFS is fundamentally structured around three core components: the **Master Server**, **Volume Server**, and **Filer**. Each serves a distinct role in managing metadata, storing data blobs, and presenting a unified file system interface to clients.

At its core, SeaweedFS employs a **flat namespace** model with a unique file ID (fid) system that enables O(1) access to any file. This is accomplished by dividing stored files into small, fixed-size volumes (e.g., 32GB or 4GB volumes), which are then distributed across volume servers. The master server maintains a global directory of all volumes and their locations, enabling clients to locate data efficiently.

### Key Architectural Principles

- **O(1) Disk Access**: SeaweedFS uses a combination of volume files and index files, storing data in append-only volumes and maintaining in-memory indexes of file locations. This allows direct seek operations to retrieve file data without scanning large directories or index trees.

- **Inspired by Haystack and F4**: The Haystack system introduced the concept of separating the metadata from the bulk data, storing large volumes of small files efficiently by appending data and maintaining indexes in memory for fast access. F4 added further optimizations for fault tolerance and erasure coding. SeaweedFS integrates these ideas with modern distributed storage capabilities.

- **Lightweight Metadata Management**: Unlike traditional distributed file systems that maintain heavy metadata services, SeaweedFS’s master server keeps only essential metadata, offloading actual file metadata and directory structure management to the filer component or external metadata stores.

- **Flexible Storage Paradigms**: SeaweedFS supports multiple paradigms including raw blob storage, hierarchical file storage, object storage with S3 API compatibility, and integration with data warehouse solutions.

---

## Core Components of SeaweedFS

### 1. Master Server

The Master Server acts as the central coordinator of the SeaweedFS cluster. It manages the global namespace of volumes, keeps track of volume server registrations, and maintains the mapping between file IDs and their physical locations.

The master server is designed to be lightweight and stateless regarding actual file data, focusing on:

- **Volume Management**: Tracking volumes, their sizes, and their states (read-only, writable, compacting).
- **Cluster Coordination**: Managing volume server heartbeats and health status.
- **Garbage Collection Scheduling**: Coordinating volume compaction and deletion of expired files.
- **Replication Control**: Defining replication strategies and ensuring data durability.

Multiple master servers can be deployed in a highly available configuration, using [Raft](https://raft.github.io/) consensus protocol to elect a leader and maintain consistency.

---

### 2. Volume Server

Volume Servers are the data storage nodes responsible for persisting the actual file data. Each volume server manages a set of volumes, which are append-only data files accompanied by index files that map file IDs to file offsets.

Key functionalities include:

- **Data Storage**: Storing file data chunks (objects or blobs) efficiently on disk.
- **Index Maintenance**: Maintaining indexes in memory and on disk for fast retrieval.
- **Volume Lifecycle Management**: Creating, compacting, and deleting volumes based on policies and master server instructions.
- **Replication**: Serving data to replicas and synchronizing with other volume servers based on replication strategies.

Volume servers expose HTTP APIs for clients and other components to store and retrieve files by file ID.

---

### 3. Filer

The Filer component provides a hierarchical namespace and metadata management layer, effectively acting as a distributed file system interface on top of the flat SeaweedFS object storage.

Unlike the master server, which only knows about volumes and file IDs, the filer manages:

- **Directory Structures**: Maintaining traditional file system directory trees.
- **File Metadata**: Storing file attributes like names, permissions, timestamps, and extended attributes.
- **File to Fid Mapping**: Mapping human-readable filenames to internal file IDs managed by volume servers.
- **Metadata Storage Backend Support**: Supporting pluggable metadata stores such as Cassandra, Redis, MySQL, LevelDB, or local disk.

The filer exposes APIs compatible with standard file system operations and supports mounting via FUSE, enabling clients to interact with SeaweedFS as if it were a local file system.

---

## Storage Paradigms: Blob Storage, File Storage, Object Storage, Data Warehouse, and FUSE

SeaweedFS is versatile, supporting multiple storage paradigms to meet various application demands.

### Blob Storage

SeaweedFS excels as a **blob storage** system, storing large numbers of small to medium-sized files as opaque blobs. Each file is assigned a unique file ID (fid), which encodes the volume and key location, enabling O(1) lookup. Blob storage in SeaweedFS is optimized for:

- **High Throughput**: Efficient sequential writes and random reads.
- **Low Latency**: Direct index-based file retrieval.
- **Scalability**: Distributing blobs across volume servers without bottlenecks.

This mode is ideal for storing media files, backups, logs, and other unstructured data.

### File Storage

Through the **Filer** component, SeaweedFS supports hierarchical **file storage** with directory trees and metadata, simulating a traditional file system. This enables applications that require:

- **POSIX-like semantics**: Directory navigation, file renaming, permissions.
- **Metadata rich files**: Extended attributes and timestamps.
- **File-level operations**: Directory listing, file manipulation.

The filer can be accessed via REST APIs or mounted locally via FUSE, providing seamless integration with existing file-based workflows.

### Object Storage (S3 API Compatibility)

SeaweedFS offers an **S3-compatible object storage interface**, enabling users to interact with their data using widely adopted Amazon S3 APIs. This supports:

- **Buckets and Objects**: Logical grouping of objects into buckets.
- **Bucket Policies**: Access controls and permissions.
- **Server-Side Encryption (SSE-S3)**: Transparent encryption of data at rest.
- **API Compatibility**: Integrations with cloud-native tools and applications expecting S3 interfaces.

The S3 API server typically listens on port 8333 by default and can coexist with the filer and volume servers.

### Data Warehouse Integration

While SeaweedFS does not natively provide data warehouse capabilities, its efficient storage and indexing of large datasets make it a suitable backend for data lakes and warehouse solutions. Files stored in SeaweedFS can be consumed by data processing frameworks (e.g., Apache Spark, Presto) for analytics.

The system’s design supports:

- **Massive Parallel Access**: Distributed volume servers enable concurrent data retrieval.
- **TTL and Compaction**: Efficient lifecycle management of stored data.
- **Metadata Querying**: Leveraging filer metadata backends to enable fast lookups.

### FUSE Mounting

SeaweedFS supports **FUSE (Filesystem in Userspace)**, allowing users to mount the filer namespace locally and access SeaweedFS as a native file system on Linux and macOS. This offers:

- **Transparent File Access**: Applications can read/write files without modifications.
- **POSIX Semantics Support**: Including file permissions, directory traversal, and symbolic links.
- **Caching**: Read and write caching strategies for performance optimization.

FUSE mounting is particularly useful for legacy applications or environments where native file system access is required.

---

## Replication Strategies and Rack/Data Center Awareness

Replication is critical for ensuring data durability, availability, and fault tolerance in distributed storage systems. SeaweedFS supports configurable replication strategies defined by three-digit codes representing replication modes and replica placement, combined with rack and datacenter awareness to minimize correlated failures.

### Replication Code Format

Replication codes are three-digit numeric strings such as `000`, `001`, `010`, `100`, `200`, `110`. Each digit represents a replication dimension:

- The **first digit** indicates the number of data replicas.
- The **second digit** indicates the number of rack replicas.
- The **third digit** indicates the number of data center replicas.

For example, the code `110` signifies:

- 1 data replica (the original copy)
- 1 replica in another rack
- 0 replicas in another data center

### Common Replication Strategies

- **000**: No replication, single copy of data.
- **001**: Replication across data centers only.
- **010**: Replication across racks only.
- **100**: Replication across nodes within the same rack.
- **200**: Two copies on the same node or rack (high redundancy).
- **110**: One replica in another rack, no cross-datacenter replication.

These strategies allow users to balance between durability, availability, and performance, depending on their infrastructure and failure model assumptions.

### Rack and Data Center Awareness

SeaweedFS supports tagging volume servers with rack and data center identifiers. The replication manager uses this information to ensure replicas are placed on different racks or data centers to:

- **Avoid single points of failure**: Prevent data loss due to rack or data center outages.
- **Improve availability**: Serve data from replicas on different failure domains.
- **Optimize network usage**: Minimize cross-datacenter traffic while maintaining durability.

This awareness is configured via master server settings and volume server registration metadata.

---

## Volume Management

Volume management in SeaweedFS is a critical aspect that balances performance, storage utilization, and data lifecycle policies.

### Volume Size Limits

Volumes in SeaweedFS are fixed-size data files that store objects. Common volume sizes are configurable, typically ranging from 4GB to 32GB per volume, with trade-offs:

- **Smaller volumes (e.g., 4GB)**: Easier to compact and move, but more volumes to manage.
- **Larger volumes (e.g., 32GB)**: Better sequential write throughput, fewer files, but longer compaction times.

Volume size limits are set via master server configuration and influence how data is distributed and balanced.

### Volume Compaction

Since SeaweedFS uses append-only volumes, deleted or expired files leave "holes" that waste space. Volume compaction reclaims this space by:

- **Copying live files**: Reading active files and writing them to a new volume.
- **Discarding deleted data**: Removing dead file data from the volume.
- **Updating indexes**: Rebuilding index files to reflect the compacted volume.

Compaction is scheduled by the master server based on volume fragmentation metrics and TTL expiration.

### Volume Balancing

To ensure even storage utilization and performance, the master server balances volumes across volume servers by:

- **Monitoring volume sizes and loads**.
- **Migrating volumes**: Moving volumes from overloaded servers to less loaded ones.
- **Respecting replication policies**: Ensuring replicas are balanced across failure domains.

Balancing helps prevent hotspots and improves cluster stability.

### Time-To-Live (TTL) Policies

SeaweedFS supports TTL for volumes and files, allowing automatic expiration and cleanup of aged data. TTL policies can be set per volume or file, triggering:

- **Garbage collection**: Marking files as deleted.
- **Volume compaction**: Reclaiming space from expired files.

TTL is useful for workloads with ephemeral data, such as logs or caches.

---

## Filer Architecture and Supported Metadata Stores

The Filer component provides the hierarchical namespace and file metadata management layer essential for file system semantics.

### Filer Architecture

The filer operates as a stateless service that:

- **Receives file system requests**: From clients via APIs, FUSE, or S3 gateway.
- **Manages directory and file metadata**: Maps file paths to SeaweedFS file IDs.
- **Handles metadata operations**: Create, rename, delete, attribute updates.
- **Coordinates with volume servers**: To allocate file IDs and manage blobs.

The filer maintains consistency by relying on external metadata stores and a write-ahead log for durability.

### Supported Metadata Stores

SeaweedFS filer supports multiple pluggable metadata backends, allowing users to choose the store that best fits their scalability, consistency, and operational requirements:

| Metadata Store | Description | Use Cases | Consistency Model | Notes |
|----------------|-------------|-----------|-------------------|-------|
| **LevelDB**    | Embedded key-value store | Small deployments, local metadata storage | Strong consistency | Default for simple setups |
| **MySQL**      | Relational database | Medium to large deployments with SQL support | Strong consistency | Requires separate DB server |
| **PostgreSQL** | Advanced relational DB | Large-scale, complex metadata queries | Strong consistency | Supports JSONB and full-text search |
| **Cassandra**  | Distributed wide-column store | Very large scale, high write throughput | Eventual consistency (tunable) | Suitable for geo-distributed metadata |
| **Redis**      | In-memory key-value store | Low latency metadata, caching layer | Strong consistency (single node) | Typically used as cache or for small metadata |
| **Etcd**       | Distributed KV store | Cluster configuration and coordination | Strong consistency | Used in Kubernetes environments |

The choice of metadata store impacts performance, durability, and operational complexity. For highly available and scalable environments, distributed stores like Cassandra or PostgreSQL clusters are recommended.

---

## S3 API Compatibility and Setup

SeaweedFS provides an S3-compatible API gateway, enabling users to interact with the system using Amazon S3 protocols, which is essential for integrating with cloud-native applications and ecosystems.

### S3 Gateway Features

- **Bucket and Object Management**: Create, list, and delete buckets and objects.
- **Bucket Policies**: Fine-grained access control using policy documents.
- **Server-Side Encryption (SSE-S3)**: Automatic encryption and decryption of objects at rest.
- **Multipart Uploads**: Support for large object uploads in parts.
- **Versioning**: Object version management (partial support).
- **Access Logging**: Audit trails for bucket and object access.

### Default Ports and Configuration

The S3 gateway typically listens on **port 8333** by default, configurable via command-line flags or configuration files. It can coexist with volume servers and filer instances on the same host or cluster.

Configuration options include:

- **Access Key and Secret Key**: For authenticating clients.
- **Bucket Location Constraints**: To simulate AWS region semantics.
- **SSL/TLS Setup**: For secure communication.
- **Quota and Rate Limits**: Enforcing storage limits and protecting against abuse.

### Bucket Policies

SeaweedFS supports AWS-style bucket policies expressed in JSON. These policies control:

- **Principal**: Who can access the bucket.
- **Action**: Permitted operations (GetObject, PutObject, DeleteObject).
- **Resource**: Buckets or objects the policy applies to.
- **Conditions**: IP restrictions, date/time, etc.

Policies enable multi-tenant environments and secure access management.

### SSE-S3 Encryption

SeaweedFS supports SSE-S3, where the server encrypts data before writing to disk and decrypts it on read, using internally managed encryption keys. This:

- Ensures data confidentiality at rest.
- Requires no client-side encryption management.
- Is transparent to S3 API clients.

---

## Quick Start and Production Setup Guide

This section details practical steps to deploy SeaweedFS in both development and production environments, focusing on default ports, high availability, and best practices.

### Quick Start

A minimal SeaweedFS cluster consists of one master, one volume server, and one filer.

1. **Start Master Server**  
   Default port: 9333  
   ```bash
   weed master
   ```
   This starts the master server with default settings, serving cluster metadata.

2. **Start Volume Server**  
   Default port: 8080  
   ```bash
   weed volume -dir=/data -max=5 -mserver=localhost:9333
   ```
   The volume server stores data volumes under `/data`, registers with the master at `localhost:9333`, and maintains up to 5 volumes.

3. **Start Filer**  
   Default port: 8888  
   ```bash
   weed filer -master=localhost:9333 -port=8888
   ```
   The filer provides the file system interface and metadata management.

4. **Mount via FUSE (optional)**  
   ```bash
   weed mount -filer=localhost:8888 -dir=/mnt/seaweedfs
   ```
   This mounts SeaweedFS at `/mnt/seaweedfs`.

5. **Access S3 API (optional)**  
   ```bash
   weed s3 -master=localhost:9333 -port=8333
   ```
   Starts the S3 gateway on port 8333.

### Production Deployment Considerations

#### High Availability Masters

For fault tolerance and consistency, deploy multiple master servers in a Raft cluster. For example, start three masters:

```bash
weed master -mdir=/mnt/master1 -defaultReplicaPlacement=110
weed master -mdir=/mnt/master2 -defaultReplicaPlacement=110 -peers=<list_of_peers>
weed master -mdir=/mnt/master3 -defaultReplicaPlacement=110 -peers=<list_of_peers>
```

This setup ensures one leader and two followers, providing strong consistency and failover.

#### Volume Server Clustering

Deploy multiple volume servers across different physical nodes, racks, or data centers, tagged accordingly to leverage replication and rack awareness.

Configure each volume server with:

- Data directories with sufficient disk space.
- Registration to all master nodes for redundancy.
- Volume size and replication settings aligned with workload.

#### Filer Metadata Backend

For production, use an external metadata store such as MySQL or Cassandra for filer metadata:

```bash
weed filer -master=localhost:9333 -metadataStore=mysql -metadataSource="user:password@tcp(host:3306)/seaweedfs"
```

This ensures metadata persistence, scalability, and failover.

#### Network and Security

- Use secure communication channels (TLS) between components.
- Configure firewall rules to restrict access to sensitive ports.
- Enable bucket policies and authentication for S3 API access.
- Monitor components with built-in metrics and logging.

#### Monitoring and Maintenance

- Monitor master and volume server health and latency.
- Schedule regular volume compaction and garbage collection.
- Adjust replication policies based on observed failure domains.
- Backup metadata stores regularly.

---

## Summary Table of SeaweedFS Components and Features

| Component      | Role                               | Default Port | Key Features                              | Notes                         |
|----------------|----------------------------------|--------------|------------------------------------------|-------------------------------|
| Master Server  | Cluster coordination, volume metadata | 9333         | Volume management, replication control, leader election (Raft) | Stateless with persistent metadata storage |
| Volume Server  | Data storage and retrieval       | 8080         | Append-only volumes, in-memory index, compaction, replication | Manages physical data files       |
| Filer          | File system interface, metadata  | 8888         | Directory hierarchy, metadata management, pluggable backends, FUSE mount | Supports various metadata stores  |
| S3 Gateway     | S3-compatible object storage API | 8333         | Bucket policies, SSE-S3 encryption, multipart uploads | Enables cloud-native integrations |

---

## References

1. **SeaweedFS Official Documentation**  
   https://seaweedfs.com/

2. **Haystack: A Scalable Indexing System for Objects at Facebook**  
   Ghemawat, S., Gobioff, H., Leung, S.-T. (2006)  
   https://research.facebook.com/publications/haystack-a-scalable-indexing-system-for-objects-at-facebook/

3. **F4: Facebook's Warm BLOB Storage System**  
   Li, W., et al. (2014)  
   https://research.google/pubs/pub38149/

4. **Raft Consensus Algorithm**  
   Ongaro, D., Ousterhout, J. (2014)  
   https://raft.github.io/

5. **S3 API Specification**  
   Amazon Web Services  
   https://docs.aws.amazon.com/AmazonS3/latest/API/Welcome.html

6. **SeaweedFS GitHub Repository**  
   https://github.com/chrislusf/seaweedfs

7. **SeaweedFS FUSE Mounting Guide**  
   https://github.com/chrislusf/seaweedfs/wiki/Fuse-Mount

---

This comprehensive document should provide storage architects, system engineers, and developers with a deep understanding of SeaweedFS’s architecture, core components, and how to deploy it effectively in production environments. The system’s design balances simplicity, performance, and scalability, making it a compelling choice for modern distributed storage needs.