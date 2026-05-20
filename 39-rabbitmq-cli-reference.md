# Comprehensive RabbitMQ CLI Reference for Tech Support Operations

## 1. Introduction

RabbitMQ is a robust, highly available message broker that serves as the backbone for many distributed systems. In production environments, managing RabbitMQ requires a deep understanding of its command-line interfaces (CLIs). This document serves as a comprehensive, advanced reference for the four primary RabbitMQ CLI tools: `rabbitmqctl`, `rabbitmq-diagnostics`, `rabbitmq-plugins`, and `rabbitmqadmin`. 

Designed specifically for tech support operations, site reliability engineers (SREs), and system administrators, this guide focuses on production operations, worst-case scenarios, massive message backlogs, and emergency mitigation. It goes beyond basic usage to provide advanced one-liners, troubleshooting workflows, and deep insights into the internal mechanics of RabbitMQ.

Whether you are dealing with network partitions, memory alarms, rogue consumers, or millions of backlogged messages, this reference provides the exact commands and strategies needed to restore service stability.

---

## 2. `rabbitmqctl` - Core Node and Cluster Management

The `rabbitmqctl` command is the primary tool for managing RabbitMQ nodes and clusters. It interacts directly with the Erlang VM and the RabbitMQ application, allowing administrators to perform core operational tasks such as cluster formation, user management, virtual host (vhost) configuration, and policy enforcement.

### 2.1 Node Management

Managing the lifecycle of a RabbitMQ node is the first step in any operational workflow. The following commands are essential for starting, stopping, and resetting nodes.

*   **Stop the RabbitMQ application (leaves the Erlang VM running):**
    ```bash
    rabbitmqctl stop_app
    ```
    *Use Case:* Required before resetting a node or changing cluster membership.

*   **Start the RabbitMQ application:**
    ```bash
    rabbitmqctl start_app
    ```
    *Use Case:* Brings the node back online after maintenance or reconfiguration.

*   **Stop the entire Erlang VM and RabbitMQ node:**
    ```bash
    rabbitmqctl stop
    ```
    *Use Case:* Complete shutdown of the node.

*   **Reset the node to its pristine state:**
    ```bash
    rabbitmqctl stop_app
    rabbitmqctl reset
    rabbitmqctl start_app
    ```
    *Use Case:* Completely wipes all data, users, vhosts, and cluster membership from the node. Use with extreme caution in production.

*   **Force reset the node:**
    ```bash
    rabbitmqctl force_reset
    ```
    *Use Case:* Similar to `reset`, but ignores cluster state. Used when a node is completely isolated and cannot communicate with the cluster.

### 2.2 Cluster Management

RabbitMQ clusters provide high availability and horizontal scaling. Managing cluster membership and state is critical during scaling events or disaster recovery.

*   **Join a cluster:**
    ```bash
    rabbitmqctl stop_app
    rabbitmqctl join_cluster rabbit@target-node
    rabbitmqctl start_app
    ```
    *Use Case:* Adds the current node to an existing cluster.

*   **Check cluster status:**
    ```bash
    rabbitmqctl cluster_status
    ```
    *Use Case:* Displays the current cluster members, running nodes, and partition status.

*   **Forget a cluster node:**
    ```bash
    rabbitmqctl forget_cluster_node rabbit@failed-node
    ```
    *Use Case:* Removes a permanently failed node from the cluster. The node must be offline.

*   **Update cluster nodes (after IP/hostname changes):**
    ```bash
    rabbitmqctl update_cluster_nodes rabbit@new-node
    ```
    *Use Case:* Updates the cluster membership list if the underlying infrastructure changes.

*   **Force boot a node (ignore cluster state):**
    ```bash
    rabbitmqctl force_boot
    ```
    *Use Case:* Forces a node to start even if it was not the last node to shut down. Useful in total cluster failure scenarios.

### 2.3 User and Access Control

Security and access control are paramount in production environments. `rabbitmqctl` provides granular control over users, passwords, and permissions.

*   **Add a new user:**
    ```bash
    rabbitmqctl add_user admin "SuperSecretPassword!"
    ```

*   **Change a user's password:**
    ```bash
    rabbitmqctl change_password admin "NewStrongPassword!"
    ```

*   **Set user tags (roles):**
    ```bash
    rabbitmqctl set_user_tags admin administrator
    ```
    *Available Tags:* `administrator`, `monitoring`, `policymaker`, `management`, `none`.

*   **List all users:**
    ```bash
    rabbitmqctl list_users
    ```

*   **Delete a user:**
    ```bash
    rabbitmqctl delete_user rogue_user
    ```

### 2.4 Virtual Host (Vhost) Management

Vhosts provide logical grouping and separation of resources (queues, exchanges, bindings) within a single RabbitMQ instance.

*   **Add a vhost:**
    ```bash
    rabbitmqctl add_vhost /production
    ```

*   **List vhosts:**
    ```bash
    rabbitmqctl list_vhosts name tracing
    ```

*   **Set permissions for a user on a vhost:**
    ```bash
    rabbitmqctl set_permissions -p /production admin ".*" ".*" ".*"
    ```
    *Format:* `set_permissions [-p vhost] user conf write read`

*   **Clear permissions:**
    ```bash
    rabbitmqctl clear_permissions -p /production admin
    ```

*   **Delete a vhost:**
    ```bash
    rabbitmqctl delete_vhost /test_environment
    ```
    *Warning:* This deletes all queues, exchanges, and messages within the vhost.

### 2.5 Policy Management

Policies are used to control advanced features like High Availability (HA) mirroring, queue length limits, and dead-lettering across multiple queues and exchanges.

*   **Set an HA policy (mirror to all nodes):**
    ```bash
    rabbitmqctl set_policy ha-all "^ha\." '{"ha-mode":"all"}' --priority 0 --apply-to queues
    ```

*   **Set a queue length limit policy:**
    ```bash
    rabbitmqctl set_policy limit-length "^limited\." '{"max-length":10000, "overflow":"reject-publish"}' --apply-to queues
    ```

*   **Set a dead-letter exchange (DLX) policy:**
    ```bash
    rabbitmqctl set_policy dlx-policy "^.*" '{"dead-letter-exchange":"my-dlx"}' --apply-to queues
    ```

*   **List policies:**
    ```bash
    rabbitmqctl list_policies -p /production
    ```

*   **Clear a policy:**
    ```bash
    rabbitmqctl clear_policy ha-all
    ```

### 2.6 Advanced One-Liners for `rabbitmqctl`

For tech support operations, speed is critical. Here are advanced one-liners for rapid diagnostics and mitigation.

*   **Find the top 10 queues with the most messages:**
    ```bash
    rabbitmqctl list_queues name messages | sort -k2 -nr | head -n 10
    ```

*   **Find queues with no consumers:**
    ```bash
    rabbitmqctl list_queues name consumers | awk '$2 == 0 {print $1}'
    ```

*   **Purge all queues in a specific vhost (Use with extreme caution):**
    ```bash
    rabbitmqctl list_queues -p /my_vhost name | tail -n +2 | xargs -I {} rabbitmqctl purge_queue -p /my_vhost {}
    ```

*   **List all connections from a specific IP address:**
    ```bash
    rabbitmqctl list_connections peer_host peer_port state | grep "192.168.1.100"
    ```

*   **Close all connections from a specific user:**
    ```bash
    rabbitmqctl list_connections user pid | grep "bad_user" | awk '{print $2}' | xargs -I {} rabbitmqctl close_connection {} "Closed by admin"
    ```

---

## 3. `rabbitmq-diagnostics` - Monitoring and Troubleshooting

The `rabbitmq-diagnostics` tool is specifically designed for health checks, monitoring, and deep introspection of the Erlang VM and RabbitMQ application. It is the go-to tool for diagnosing performance issues, memory leaks, and network problems.

### 3.1 Node Status and Health Checks

*   **Comprehensive node status:**
    ```bash
    rabbitmq-diagnostics status
    ```
    *Use Case:* Displays Erlang VM stats, memory usage, file descriptors, and running applications.

*   **Check if the node is running and fully booted:**
    ```bash
    rabbitmq-diagnostics check_running
    ```

*   **Check local alarms (memory/disk):**
    ```bash
    rabbitmq-diagnostics check_local_alarms
    ```

*   **Check cluster alarms:**
    ```bash
    rabbitmq-diagnostics check_alarms
    ```

*   **Ping the node to check responsiveness:**
    ```bash
    rabbitmq-diagnostics ping
    ```

### 3.2 Memory and Disk Alarms

RabbitMQ uses memory and disk alarms to protect itself from crashing. When an alarm is triggered, publishers are blocked.

*   **View detailed memory breakdown:**
    ```bash
    rabbitmq-diagnostics memory_breakdown
    ```
    *Use Case:* Identifies exactly what is consuming memory (e.g., queues, binaries, connections, management database).

*   **Check free disk space:**
    ```bash
    rabbitmq-diagnostics status | grep -i disk
    ```

*   **Force garbage collection on the Erlang VM:**
    ```bash
    rabbitmq-diagnostics force_gc
    ```
    *Use Case:* Temporarily frees up memory if the Erlang garbage collector is lagging behind.

### 3.3 Connection and Channel Inspection

Rogue clients can open thousands of connections or channels, exhausting file descriptors and memory.

*   **List all channels with unacknowledged messages:**
    ```bash
    rabbitmq-diagnostics list_channels connection messages_unacknowledged | awk '$2 > 0'
    ```

*   **Find connections with the highest data transfer rates:**
    ```bash
    rabbitmq-diagnostics list_connections peer_host recv_oct send_oct | sort -k2 -nr | head -n 10
    ```

*   **List file descriptor usage:**
    ```bash
    rabbitmq-diagnostics status | grep -A 5 "File Descriptors"
    ```

### 3.4 Advanced One-Liners for `rabbitmq-diagnostics`

*   **Monitor memory usage in real-time (every 5 seconds):**
    ```bash
    watch -n 5 "rabbitmq-diagnostics memory_breakdown | head -n 15"
    ```

*   **Identify the Erlang processes consuming the most memory:**
    ```bash
    rabbitmq-diagnostics observer_cli
    ```
    *(Note: Requires the `observer_cli` plugin or Erlang observer, but provides a top-like interface for the Erlang VM).*

*   **Check for network partitions across the cluster:**
    ```bash
    rabbitmq-diagnostics cluster_status | grep -i partition
    ```

---

## 4. `rabbitmq-plugins` - Plugin Management

RabbitMQ's functionality is heavily extended through plugins. The `rabbitmq-plugins` command manages the enabling and disabling of these extensions.

### 4.1 Enabling and Disabling Plugins

*   **List all available and enabled plugins:**
    ```bash
    rabbitmq-plugins list
    ```

*   **Enable the management UI and HTTP API:**
    ```bash
    rabbitmq-plugins enable rabbitmq_management
    ```

*   **Enable the Prometheus metrics exporter:**
    ```bash
    rabbitmq-plugins enable rabbitmq_prometheus
    ```

*   **Enable the Shovel plugin (for cross-cluster replication):**
    ```bash
    rabbitmq-plugins enable rabbitmq_shovel rabbitmq_shovel_management
    ```

*   **Disable a plugin:**
    ```bash
    rabbitmq-plugins disable rabbitmq_mqtt
    ```

### 4.2 Offline Plugin Management

Sometimes plugins need to be managed while the RabbitMQ node is offline.

*   **Enable a plugin offline:**
    ```bash
    rabbitmq-plugins enable --offline rabbitmq_management
    ```

*   **Disable a plugin offline:**
    ```bash
    rabbitmq-plugins disable --offline rabbitmq_management
    ```

### 4.3 Advanced One-Liners for `rabbitmq-plugins`

*   **Enable all officially supported plugins (Not recommended for production, but useful for testing):**
    ```bash
    rabbitmq-plugins list -E -m | xargs rabbitmq-plugins enable
    ```

*   **Find implicitly enabled plugins (dependencies of explicitly enabled plugins):**
    ```bash
    rabbitmq-plugins list -i
    ```

---

## 5. `rabbitmqadmin` - HTTP API CLI Wrapper

`rabbitmqadmin` is a Python script that wraps the RabbitMQ HTTP API. It is incredibly powerful for scripting, automation, and interacting with RabbitMQ without needing Erlang CLI access. It must be downloaded from the management UI (`http://node-ip:15672/cli/rabbitmqadmin`).

### 5.1 Configuration and Setup

*   **Basic usage with credentials:**
    ```bash
    rabbitmqadmin -H localhost -u admin -p password -V /production list queues
    ```

*   **Using a configuration file (`~/.rabbitmqadmin.conf`):**
    ```ini
    [default]
    hostname = localhost
    port = 15672
    username = admin
    password = password
    vhost = /production
    ```

### 5.2 Queue and Exchange Management

*   **Declare a queue:**
    ```bash
    rabbitmqadmin declare queue name=my_queue durable=true
    ```

*   **Declare an exchange:**
    ```bash
    rabbitmqadmin declare exchange name=my_exchange type=direct
    ```

*   **Create a binding:**
    ```bash
    rabbitmqadmin declare binding source=my_exchange destination=my_queue routing_key=my_routing_key
    ```

*   **Delete a queue:**
    ```bash
    rabbitmqadmin delete queue name=my_queue
    ```

### 5.3 Message Publishing and Consumption

*   **Publish a message:**
    ```bash
    rabbitmqadmin publish exchange=my_exchange routing_key=my_routing_key payload="Hello World" properties="{\"delivery_mode\": 2}"
    ```

*   **Get (consume) a message from a queue (without acknowledging):**
    ```bash
    rabbitmqadmin get queue=my_queue requeue=true count=1
    ```

*   **Get and acknowledge a message (removes it from the queue):**
    ```bash
    rabbitmqadmin get queue=my_queue requeue=false count=10
    ```

### 5.4 Advanced One-Liners for `rabbitmqadmin`

*   **Export the entire RabbitMQ configuration (definitions) to a JSON file:**
    ```bash
    rabbitmqadmin export rabbitmq_config.json
    ```

*   **Import configuration from a JSON file:**
    ```bash
    rabbitmqadmin import rabbitmq_config.json
    ```

*   **Find all queues with more than 10,000 messages and output in JSON format for parsing:**
    ```bash
    rabbitmqadmin -f json list queues name messages | jq '.[] | select(.messages > 10000) | .name'
    ```

*   **Publish 1000 test messages rapidly using a bash loop:**
    ```bash
    for i in {1..1000}; do rabbitmqadmin publish exchange=amq.default routing_key=test_queue payload="Message $i"; done
    ```

---

## 6. Worst-Case Scenarios and Massive Message Backlogs

In tech support operations, you are often called in when things have gone catastrophically wrong. This section covers the most severe RabbitMQ incidents and how to resolve them using the CLI tools.

### 6.1 Handling Network Partitions

A network partition (split-brain) occurs when cluster nodes lose communication. RabbitMQ's behavior depends on the `cluster_partition_handling` strategy (e.g., `ignore`, `pause_minority`, `autoheal`).

**Symptoms:**
*   `rabbitmq-diagnostics cluster_status` shows partitions.
*   Management UI shows nodes in red.
*   Inconsistent queue states across nodes.

**Mitigation Workflow:**
1.  **Identify the partition:**
    ```bash
    rabbitmq-diagnostics cluster_status
    ```
2.  **Determine the trusted node:** Identify which node has the most up-to-date data or the most connected clients.
3.  **Stop the application on the minority/untrusted nodes:**
    ```bash
    rabbitmqctl stop_app
    ```
4.  **Start the application on the untrusted nodes:**
    ```bash
    rabbitmqctl start_app
    ```
    *(If `autoheal` or `pause_minority` is configured, this may resolve automatically. If `ignore` is used, manual intervention is required).*
5.  **If the partition persists, force a reset and rejoin:**
    ```bash
    rabbitmqctl stop_app
    rabbitmqctl reset
    rabbitmqctl join_cluster rabbit@trusted-node
    rabbitmqctl start_app
    ```

### 6.2 Clearing Massive Backlogs

When consumers fail, queues can accumulate millions of messages, leading to memory and disk alarms.

**Symptoms:**
*   Memory alarms triggered.
*   Publishers blocked.
*   High disk I/O.

**Mitigation Workflow:**
1.  **Identify the offending queues:**
    ```bash
    rabbitmqctl list_queues name messages memory | sort -k2 -nr | head -n 5
    ```
2.  **Option A: Purge the queue (Data Loss Acceptable):**
    ```bash
    rabbitmqctl purge_queue offending_queue_name
    ```
3.  **Option B: Delete and Recreate the queue (Faster than purging for massive queues):**
    ```bash
    rabbitmqadmin delete queue name=offending_queue_name
    rabbitmqadmin declare queue name=offending_queue_name durable=true
    ```
4.  **Option C: Apply a Length Limit Policy (Prevent future backlogs):**
    ```bash
    rabbitmqctl set_policy limit-backlog "^offending_queue_name$" '{"max-length":500000, "overflow":"drop-head"}' --apply-to queues
    ```

### 6.3 Recovering from Memory/Disk Alarms

When RabbitMQ hits its memory watermark (default 40% of RAM) or disk watermark (default 50MB free), it blocks all publishing connections.

**Symptoms:**
*   `rabbitmq-diagnostics check_alarms` shows `true`.
*   Clients receive `connection.blocked` notifications.

**Mitigation Workflow:**
1.  **Check what is consuming memory:**
    ```bash
    rabbitmq-diagnostics memory_breakdown
    ```
2.  **If queues are consuming memory, purge or delete them (see 6.2).**
3.  **If connections are consuming memory, identify and close rogue connections:**
    ```bash
    rabbitmqctl list_connections pid channels | awk '$2 > 100 {print $1}' | xargs -I {} rabbitmqctl close_connection {} "Too many channels"
    ```
4.  **Force garbage collection:**
    ```bash
    rabbitmq-diagnostics force_gc
    ```
5.  **As a last resort, temporarily increase the memory watermark (Not recommended for long-term stability):**
    ```bash
    rabbitmqctl set_vm_memory_high_watermark 0.6
    ```

---

## 7. Tech Support Operations Playbook

When responding to a P1/Sev1 incident involving RabbitMQ, follow this structured playbook using the CLI tools.

### Phase 1: Information Gathering (First 5 Minutes)

Do not make any changes. Gather state.

1.  **Check Alarms:** `rabbitmq-diagnostics check_alarms`
2.  **Check Cluster Status:** `rabbitmq-diagnostics cluster_status`
3.  **Check Memory:** `rabbitmq-diagnostics memory_breakdown`
4.  **Identify Top Queues:** `rabbitmqctl list_queues name messages consumers | sort -k2 -nr | head -n 10`
5.  **Check Connections:** `rabbitmqctl list_connections peer_host state | wc -l`

### Phase 2: Emergency Mitigation (Minutes 5-15)

Take action to restore service, even if it means data loss (depending on business SLAs).

1.  **Unblock Publishers:** If memory alarms are active, purge non-critical queues or apply a `max-length` policy to drop old messages.
2.  **Kill Rogue Clients:** If a specific IP is opening thousands of connections, block it at the firewall or use `rabbitmqctl close_connection`.
3.  **Resolve Partitions:** Restart the RabbitMQ application on minority nodes.

### Phase 3: Root Cause Analysis and Remediation (Post-Incident)

1.  **Review Logs:** Check `/var/log/rabbitmq/rabbit@node.log`.
2.  **Export Definitions:** `rabbitmqadmin export config.json` to review policies and topology.
3.  **Implement Limits:** Apply `max-length` and `max-length-bytes` policies to all queues to prevent future memory exhaustion.
4.  **Optimize Consumers:** Work with the development team to ensure consumers are acknowledging messages correctly and using appropriate `prefetch` counts.

---

## 8. Relation to the Specialist Team Framework

This document, `39-rabbitmq-cli-reference.md`, is a critical component of the broader specialist-teams repository. It serves as the definitive operational guide for the RabbitMQ tech support specialist. 

**How it relates to the other 6 files:**

1.  **`specialist.md` (The Core Persona):** This CLI reference provides the technical foundation and "muscle memory" for the RabbitMQ specialist persona defined in the main file. The specialist relies on these exact commands to execute their duties.
2.  **Architecture and Topology Guides:** While other files may define *how* queues and exchanges should be structured, this file provides the tools to *enforce* and *inspect* that topology in real-time.
3.  **Monitoring and Alerting Runbooks:** When an alert fires (e.g., High Memory Usage), the runbooks will reference the diagnostic commands detailed in Section 3 (`rabbitmq-diagnostics`) of this document.
4.  **Disaster Recovery Plans:** The cluster management and network partition mitigation strategies outlined here are the exact steps executed during a disaster recovery scenario.
5.  **Security and Compliance Policies:** The user, vhost, and permission management commands in Section 2 are used to implement the security baselines defined in the compliance documentation.
6.  **Developer Best Practices:** While developers focus on code, this CLI reference allows the tech support specialist to debug developer issues (e.g., unacknowledged messages, channel leaks) and provide actionable feedback to the engineering teams.

By mastering the commands and workflows in this reference, the tech support specialist can confidently manage, troubleshoot, and recover RabbitMQ clusters in the most demanding production environments.
