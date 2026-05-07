# SeaweedFS Troubleshooting & Diagnostics Guide

SeaweedFS is a highly scalable distributed storage system designed for object storage, file systems, and data lakes. While it is known for its simplicity and O(1) disk read performance, operating a distributed system at scale inevitably leads to scenarios requiring troubleshooting and diagnostics. This comprehensive guide covers error codes, recovery strategies, health checks, and common issues encountered in SeaweedFS deployments.

## 1. Architecture Overview and Failure Domains

Before diving into specific troubleshooting steps, it is crucial to understand the core components of SeaweedFS and their respective failure domains:

*   **Master Server (`weed master`):** Manages the cluster topology, volume locations, and assigns file IDs (FIDs). It uses Raft for consensus. Failures here affect new file writes and volume lookups but do not impact existing read operations if clients have cached volume locations.
*   **Volume Server (`weed volume`):** Stores the actual data in large (typically 30GB) pre-allocated files called volumes. Failures here directly impact data availability.
*   **Filer (`weed filer`):** Provides a POSIX-like file system interface and S3 API on top of the volume servers. It stores metadata (directory structures, file names) in a separate database (e.g., LevelDB, Redis, Cassandra). Failures here affect directory listings and S3 operations.

## 2. Common Issues and Error Codes

### 2.1. "No Free Volumes Left"

This is one of the most common errors in SeaweedFS. It occurs when the master server cannot allocate a new volume for incoming data.

**Symptoms:**
*   Uploads fail with errors like `Cannot grow volume group! No free space left!` or `context canceled`.
*   The `weed shell` command `volume.list` shows all volumes are full.

**Causes:**
*   **Physical Disk Space Exhaustion:** The underlying disks on the volume servers are full.
*   **Replication Constraints:** If you require replication (e.g., `010` for rack-level replication), but there are no available volume servers in different racks with free space, allocation will fail even if some individual servers have space.
*   **Data Hoarding/Deleted Files Not Vacuumed:** SeaweedFS uses an append-only structure. Deleting a file only marks it as deleted in the metadata. The actual disk space is not freed until a "vacuum" operation runs.

**Diagnostics & Recovery:**
1.  **Check Disk Space:** Run `df -h` on all volume servers.
2.  **Check Volume Status:** Use `weed shell` and run `volume.list` to see the distribution of volumes and their free space.
3.  **Force Vacuuming:** If there is a high percentage of deleted data (garbage), force a vacuum operation.
    ```bash
    weed shell
    > volume.vacuum -garbageThreshold 0.1
    ```
    *Note: If the disks are 100% full, vacuuming might fail because it requires some temporary space to compact the volumes. In this case, you may need to temporarily disable replication to free up space, or add a new volume server.*
4.  **Temporary Replication Reduction (Emergency):** If you are completely out of space and cannot add hardware immediately, you can temporarily reduce replication to free up volumes.
    ```bash
    weed shell
    > lock
    > volume.configure.replication -replication 000 -volumeId <id>
    > volume.fix.replication
    > volume.balance -force
    > unlock
    ```
    *Warning: This reduces your fault tolerance. Restore replication as soon as space is available.*

### 2.2. Filer Metadata Inconsistencies

The Filer maintains the mapping between file paths and the FIDs stored on the volume servers. Inconsistencies can occur due to crashes or network partitions.

**Symptoms:**
*   Files appear in directory listings but cannot be downloaded (404 Not Found).
*   `fs.verify` reports missing chunks or orphaned files.

**Diagnostics & Recovery:**
1.  **Run `fs.verify`:** This command checks the consistency between the Filer metadata and the actual data on the volume servers.
    ```bash
    weed shell
    > fs.verify
    ```
2.  **Fix Missing Chunks:** If `fs.verify` finds issues, you may need to restore from backups or, if using Erasure Coding (EC), attempt to rebuild the missing shards.
3.  **Check Filer Database:** Ensure the underlying database used by the Filer (e.g., Redis, Cassandra) is healthy and not experiencing performance bottlenecks or data corruption.

### 2.3. Master Node Election Failures

If the master nodes cannot elect a leader, the cluster becomes read-only for new allocations.

**Symptoms:**
*   Logs show repeated Raft election timeouts.
*   `weed shell` cannot connect to the master.

**Causes:**
*   Network partitions between master nodes.
*   Insufficient number of master nodes available to form a quorum (e.g., 2 out of 3 nodes are down).
*   Clock skew between master nodes.

**Diagnostics & Recovery:**
1.  **Check Network Connectivity:** Ensure all master nodes can communicate on the Raft port (default 9333).
2.  **Verify Quorum:** Ensure a majority of master nodes are running. If you have 3 masters, at least 2 must be up.
3.  **Check NTP:** Ensure all nodes have synchronized clocks using NTP or Chrony.

## 3. Health Checks and Monitoring

Proactive monitoring is essential for maintaining a healthy SeaweedFS cluster.

### 3.1. Built-in Endpoints

SeaweedFS provides several HTTP endpoints for health checking:

*   **Master Status:** `http://<master-ip>:9333/cluster/status` - Returns JSON detailing the cluster topology, volume servers, and available space.
*   **Volume Server Status:** `http://<volume-ip>:8080/status` - Returns JSON detailing the volumes hosted on that specific server.
*   **Healthz Endpoint:** Many components support a `/healthz` endpoint that returns a 200 OK if the service is running. (Note: There have been historical issues with the S3 API `/healthz` endpoint returning 404, which may require checking specific component logs).

### 3.2. Prometheus Metrics

SeaweedFS natively exports Prometheus metrics. You should configure Prometheus to scrape these metrics and set up alerts in Grafana.

**Key Metrics to Monitor:**
*   `weed_master_volume_free_count`: Alert when the number of free volumes drops below a critical threshold.
*   `weed_volume_deleted_bytes` vs `weed_volume_size_bytes`: Monitor the garbage ratio to ensure vacuuming is running effectively.
*   `weed_filer_request_duration_seconds`: Monitor API latency.
*   `weed_s3_request_error_count`: Track S3 API errors.

## 4. Advanced Diagnostics: Erasure Coding (EC)

SeaweedFS supports Erasure Coding to reduce storage overhead for cold data. Troubleshooting EC volumes requires specific commands.

**Symptoms:**
*   Errors reading older, cold data.
*   `fs.verify` reports corrupted EC shards.

**Diagnostics & Recovery:**
1.  **Identify Corrupted Shards:** Use `weed shell` to inspect the EC volumes.
2.  **Rebuild Shards:** If a volume server holding EC shards fails, you can rebuild the missing shards on a new server.
    ```bash
    weed shell
    > ec.rebuild -volumeId <id>
    ```
3.  **Hardware Issues:** Corrupted EC shards are often a symptom of underlying hardware issues, such as bad RAM (especially non-ECC RAM) or failing disks. Always run hardware diagnostics (e.g., `memtest86`, `smartctl`) if you encounter silent data corruption.

## 5. Disaster Recovery Strategies

### 5.1. Metadata Backup

The Filer metadata is the most critical component to back up. If you lose the volume servers, you lose the data, but if you lose the Filer metadata, you lose the directory structure and file names, making the raw data on the volume servers nearly useless.

*   **Strategy:** Regularly back up the underlying database used by the Filer (e.g., dump the LevelDB directory, snapshot the Cassandra keyspace).

### 5.2. Volume Backup

While SeaweedFS handles replication internally, you may still want off-site backups.

*   **Strategy:** You can use tools like `rclone` to sync the Filer mount to an external S3 provider (e.g., AWS, Backblaze).
*   **Volume-Level Sync:** For large clusters, you can periodically sync the raw `.dat` and `.idx` files from the volume servers to cold storage, though restoring from this requires careful metadata synchronization.

### 5.3. Recovering from Total Disk Failure

If a volume server's disk fails completely and you have replication enabled (e.g., `010`):

1.  Remove the failed volume server from the cluster configuration.
2.  Add a new volume server with a fresh disk.
3.  Use `weed shell` to fix replication. The master will instruct the remaining volume servers to copy the missing volumes to the new server.
    ```bash
    weed shell
    > volume.fix.replication
    ```

## 6. Conclusion

Troubleshooting SeaweedFS requires a solid understanding of its distributed architecture. By monitoring disk space, ensuring proper vacuuming, and maintaining the health of the Filer metadata database, most common issues can be avoided. In the event of hardware failure, SeaweedFS's replication and Erasure Coding features provide robust mechanisms for data recovery.