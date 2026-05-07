# Troubleshooting & Diagnostics Guide: rabbitmq-documentdb

## 1. Introduction

Welcome to the comprehensive Troubleshooting and Diagnostics Guide for the `rabbitmq-documentdb` integration. This document is designed for senior software engineers, site reliability engineers (SREs), and system administrators who are responsible for maintaining, diagnosing, and resolving issues within architectures that leverage both RabbitMQ as a message broker and Amazon DocumentDB (with MongoDB compatibility) as the primary data store.

When integrating a high-throughput message broker like RabbitMQ with a scalable NoSQL database like DocumentDB, the complexity of the system increases significantly. Issues can arise from network partitions, misconfigurations, resource exhaustion, driver incompatibilities, or unhandled edge cases in the application logic. This guide provides a deep dive into error codes, recovery strategies, health checks, and common issues to ensure your system remains resilient and performant.

The `rabbitmq-documentdb` connector or integration pattern typically involves consuming messages from RabbitMQ queues, transforming or validating the payload, and persisting the resulting documents into DocumentDB. Conversely, it may involve tailing DocumentDB change streams (if supported or simulated) and publishing events to RabbitMQ. Both directions require robust error handling, retry mechanisms, and observability.

## 2. Architecture Overview and Failure Domains

Before diving into specific troubleshooting steps, it is crucial to understand the architecture and the potential failure domains. The integration generally consists of three main components:
1. **RabbitMQ Cluster**: The message broker responsible for receiving, routing, and storing messages until they are successfully processed.
2. **Integration Service / Consumer**: The application or worker service that consumes messages from RabbitMQ and interacts with DocumentDB.
3. **Amazon DocumentDB Cluster**: The managed NoSQL database service that stores the JSON-like documents.

### 2.1 Failure Domains
- **Network Layer**: VPC peering issues, security group misconfigurations, DNS resolution failures, or transient network partitions.
- **Compute Layer**: CPU/Memory exhaustion on the worker nodes, thread pool exhaustion, or application crashes.
- **Broker Layer (RabbitMQ)**: Queue buildup, memory alarms, disk alarms, split-brain scenarios, or connection limits reached.
- **Database Layer (DocumentDB)**: High CPU utilization, IOPS throttling, connection pool exhaustion, or replica lag.

## 3. Health Checks and Monitoring

Proactive monitoring and robust health checks are the first line of defense against system degradation.

### 3.1 RabbitMQ Health Checks
- **Alarms**: Monitor `rabbitmq_alarms` for memory and disk watermarks. If an alarm is triggered, RabbitMQ will block publishers, causing upstream backpressure.
- **Queue Depth**: Monitor `rabbitmq_queue_messages_ready` and `rabbitmq_queue_messages_unacknowledged`. A growing ready queue indicates consumers are too slow or offline. A growing unacknowledged queue indicates consumers are stuck processing or failing to ACK.
- **Connection Count**: Monitor `rabbitmq_connections`. A sudden drop indicates a network issue or application crash. A steady increase indicates connection leaks.

### 3.2 DocumentDB Health Checks
- **CPU Utilization**: Monitor `CPUUtilization`. DocumentDB is highly dependent on CPU for query processing.
- **Database Connections**: Monitor `DatabaseConnections`. Ensure it does not exceed the instance limit.
- **IOPS**: Monitor `VolumeReadIOPs` and `VolumeWriteIOPs`. Throttling here will severely impact write latency.
- **Replica Lag**: Monitor `ReplicaLag`. High lag can cause read-after-write consistency issues if reading from replicas.

### 3.3 Integration Service Health Checks
- **Liveness Probe**: Ensure the service process is running and not deadlocked.
- **Readiness Probe**: Ensure the service can successfully connect to both RabbitMQ and DocumentDB. Perform a lightweight ping to both services.
- **Processing Latency**: Track the time taken to process a single message from consumption to database persistence.

## 4. Common Error Codes and Diagnostics

### 4.1 RabbitMQ Error Codes
- **403 Access Refused**: The consumer or publisher does not have the correct permissions for the vhost, exchange, or queue.
  - *Diagnostic*: Check the RabbitMQ user permissions using `rabbitmqctl list_permissions`. Ensure the credentials in the application configuration are correct.
- **404 Not Found**: The specified exchange or queue does not exist.
  - *Diagnostic*: Verify the topology declaration logic in your application. Ensure queues and exchanges are declared before binding or publishing.
- **406 Precondition Failed**: The client attempted to declare a queue or exchange with parameters that conflict with an existing one.
  - *Diagnostic*: Check if the queue was previously declared with different arguments (e.g., `x-message-ttl`, `durable`). You may need to delete the queue and recreate it, or update the application to match the existing parameters.
- **504 Channel Error**: Often caused by attempting to acknowledge a message on a different channel than it was received on, or double-acknowledging.
  - *Diagnostic*: Review the application's concurrency model. Ensure channels are not shared across threads concurrently and that ACKs are sent exactly once per delivery tag.

### 4.2 DocumentDB Error Codes
- **Authentication Failed (Code 18)**: Invalid username, password, or authentication database.
  - *Diagnostic*: Verify the credentials. Ensure the connection string specifies `?authSource=admin` if required.
- **Cursor Not Found (Code 43)**: The cursor timed out on the server or was closed.
  - *Diagnostic*: If processing large batches, the cursor may time out before the batch is fully processed. Increase the cursor timeout or reduce the batch size.
- **Duplicate Key Error (Code 11000)**: Attempted to insert a document with an `_id` or unique index value that already exists.
  - *Diagnostic*: Check the application logic for idempotency. If consuming from RabbitMQ, messages may be delivered more than once (at-least-once delivery). Ensure the database operations use `updateOne` with `upsert: true` instead of `insertOne` to handle duplicates gracefully.
- **Connection Pool Exhausted**: The application cannot acquire a connection from the pool within the timeout period.
  - *Diagnostic*: Check for connection leaks in the application (e.g., not closing cursors or sessions). Increase the connection pool size if the workload genuinely requires more concurrent connections, but monitor DocumentDB's `DatabaseConnections` metric.

## 5. Network and Connectivity Troubleshooting

### 5.1 RabbitMQ Connection Drops
- **Symptom**: Consumers frequently disconnect and reconnect.
- **Diagnostic**:
  - Check the RabbitMQ server logs for `missed heartbeats`. If heartbeats are missed, the network might be congested, or the consumer process might be blocked (e.g., long garbage collection pauses).
  - Increase the heartbeat timeout in the client connection settings (e.g., from 60s to 120s).
  - Verify that intermediate load balancers or firewalls are not terminating idle TCP connections prematurely. Enable TCP keepalives.

### 5.2 DocumentDB Connection Timeouts
- **Symptom**: The application throws `MongoTimeoutException` when attempting to connect or execute queries.
- **Diagnostic**:
  - Verify VPC peering and Security Groups. DocumentDB is VPC-only. The integration service must be in the same VPC or a peered VPC, and the Security Group attached to the DocumentDB cluster must allow inbound traffic on port 27017 from the service's IP range.
  - Check DNS resolution. DocumentDB cluster endpoints must resolve to private IP addresses. Ensure DNS hostnames and DNS resolution are enabled in the VPC.
  - Test connectivity using `mongo shell` or `mongosh` from the worker node: `mongosh --host <cluster-endpoint> --port 27017 --username <user> --password <pass> --tls`.

## 6. Performance and Resource Exhaustion

### 6.1 RabbitMQ Memory Alarms
- **Symptom**: RabbitMQ stops accepting new messages. The management UI shows the memory watermark is breached.
- **Diagnostic**:
  - Identify the queues consuming the most memory. This is often due to a massive backlog of unacknowledged or ready messages.
  - Check if consumers are offline or processing too slowly.
  - Consider implementing lazy queues (`x-queue-mode: lazy`) to page messages to disk immediately, reducing RAM usage at the cost of disk I/O.

### 6.2 DocumentDB High CPU Utilization
- **Symptom**: DocumentDB CPU is consistently above 80%, leading to increased query latency and consumer backlog.
- **Diagnostic**:
  - Identify slow queries using the DocumentDB profiler. Enable profiling for operations taking longer than a specific threshold.
  - Ensure appropriate indexes exist for the queries being executed. Missing indexes cause full collection scans, which are highly CPU-intensive.
  - Optimize the integration service to use bulk write operations (`insertMany`, `bulkWrite`) instead of individual inserts to reduce network round trips and database overhead.

### 6.3 Consumer Thread Pool Exhaustion
- **Symptom**: The integration service is running, but message consumption rate drops to zero. CPU utilization on the worker node is low.
- **Diagnostic**:
  - Take a thread dump of the application process.
  - Look for threads blocked on database I/O or waiting for a connection from the pool.
  - Ensure asynchronous operations are properly handled and not blocking the main consumer threads.
  - Tune the RabbitMQ consumer prefetch count (`basic.qos`). A high prefetch count with slow database operations can lead to memory exhaustion in the application and thread starvation.

## 7. Data Consistency and Message Loss

### 7.1 Message Loss
- **Symptom**: Messages published to RabbitMQ are not found in DocumentDB, and there are no errors in the logs.
- **Diagnostic**:
  - **Publisher Confirms**: Ensure the publisher uses publisher confirms to guarantee messages reach the broker.
  - **Queue Durability**: Ensure queues are declared as `durable` and messages are published with `delivery_mode=2` (persistent).
  - **Consumer Acknowledgements**: Ensure the consumer uses manual acknowledgements (`autoAck=false`). The consumer MUST only ACK the message *after* it has been successfully persisted to DocumentDB. If `autoAck=true`, the message is removed from the queue as soon as it is delivered, risking loss if the consumer crashes before processing.

### 7.2 Data Duplication
- **Symptom**: The same logical record appears multiple times in DocumentDB.
- **Diagnostic**:
  - RabbitMQ guarantees at-least-once delivery. Network issues or consumer crashes can cause messages to be redelivered.
  - The integration logic MUST be idempotent. Use a unique identifier from the message payload as the DocumentDB `_id` or as part of a unique index.
  - Use `upsert` operations to overwrite existing data or ignore duplicates gracefully.

## 8. Recovery Strategies

### 8.1 Handling Poison Messages
- **Scenario**: A message payload is malformed or causes a deterministic error in the application logic or database (e.g., schema validation failure).
- **Strategy**:
  - Do not endlessly requeue the message (which causes a tight loop and high CPU).
  - Implement a Dead Letter Exchange (DLX). Configure the main queue to route rejected messages to the DLX.
  - The consumer should catch non-transient exceptions, log the error, and `basic.reject` or `basic.nack` the message with `requeue=false`.
  - Monitor the Dead Letter Queue (DLQ) and set up alerts. Operators can manually inspect, fix, and replay messages from the DLQ.

### 8.2 Handling Transient Database Outages
- **Scenario**: DocumentDB is temporarily unavailable (e.g., during a primary instance failover).
- **Strategy**:
  - The consumer should catch transient exceptions (e.g., connection timeouts, socket errors).
  - Implement an exponential backoff retry mechanism within the consumer before rejecting the message.
  - If retries are exhausted, `basic.nack` the message with `requeue=true` to put it back in the queue, or route it to a retry queue with a TTL that eventually routes back to the main queue (delayed retry pattern).

### 8.3 Recovering from Split-Brain (RabbitMQ)
- **Scenario**: A network partition causes the RabbitMQ cluster to split into multiple independent clusters.
- **Strategy**:
  - Configure the `cluster_partition_handling` strategy appropriately (e.g., `pause_minority` or `autoheal`).
  - If manual intervention is required, identify the partition with the most up-to-date data, stop the nodes in the other partition, and force them to rejoin the cluster. Note that messages published to the losing partition may be lost.

## 9. Security and Authentication Issues

### 9.1 TLS/SSL Handshake Failures
- **Symptom**: The application fails to connect to DocumentDB with SSL handshake exceptions.
- **Diagnostic**:
  - DocumentDB requires TLS by default. Ensure the application is configured to use TLS.
  - The application must trust the Amazon RDS Certificate Authority (CA). Download the `rds-combined-ca-bundle.pem` and configure the database driver to use it.
  - Verify that the driver version supports the TLS version required by DocumentDB (TLS 1.2+).

### 9.2 RabbitMQ Credential Rotation
- **Scenario**: Passwords for RabbitMQ users need to be rotated without downtime.
- **Strategy**:
  - Create a new user with the same permissions as the old user.
  - Update the application configuration to use the new credentials and perform a rolling restart of the integration services.
  - Once all services are using the new credentials, delete or disable the old user.

## 10. Advanced Debugging Techniques

### 10.1 Distributed Tracing
- Implement distributed tracing (e.g., OpenTelemetry, Jaeger) to track the lifecycle of a message.
- Inject a trace ID into the RabbitMQ message headers when publishing.
- The consumer extracts the trace ID and includes it in the database operation context and application logs.
- This allows operators to visualize the end-to-end latency and pinpoint exactly where a message is delayed or failing.

### 10.2 Network Packet Capture
- If network issues are suspected but not clearly visible in logs, use `tcpdump` or Wireshark to capture traffic between the worker node and RabbitMQ/DocumentDB.
- Analyze the PCAP file for TCP retransmissions, connection resets (RST), or TLS handshake failures.

### 10.3 DocumentDB Profiler and Explain Plans
- Use the `db.collection.explain("executionStats").find(...)` command to analyze how DocumentDB executes a query.
- Look for `COLLSCAN` (collection scan) which indicates a missing index.
- Check the `executionTimeMillis` and `totalDocsExamined` to identify inefficient queries.

## 11. Conclusion

Troubleshooting the `rabbitmq-documentdb` integration requires a holistic understanding of both systems and the network that connects them. By implementing robust health checks, comprehensive monitoring, idempotent consumer logic, and proper error handling strategies like Dead Letter Queues, you can build a highly resilient and scalable architecture. When issues do arise, follow the systematic diagnostic steps outlined in this guide to quickly identify the root cause and apply the appropriate recovery strategy. Always prioritize data consistency and ensure that your monitoring alerts you to degradation before it impacts the end-user experience.