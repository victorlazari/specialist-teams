# Valkey-Redis CLI Command Reference

## Introduction

Valkey-Redis is a robust and efficient command-line interface (CLI) tool designed to interact with Redis, a popular open-source, in-memory data structure store used as a database, cache, and message broker. The CLI is an essential tool for developers and system administrators, providing a powerful means to manage and interact with Redis instances, perform operations, and automate tasks.

This document serves as an exhaustive CLI command reference for Valkey-Redis, detailing every command, flag, and argument. We will also provide comprehensive examples to illustrate the practical use of each command.

---

## Table of Contents

1. [Getting Started](#getting-started)
2. [Basic Commands](#basic-commands)
3. [Key Management Commands](#key-management-commands)
4. [Data Operations](#data-operations)
5. [Server Management](#server-management)
6. [Security and Authentication](#security-and-authentication)
7. [Scripting with Lua](#scripting-with-lua)
8. [Troubleshooting and Debugging](#troubleshooting-and-debugging)
9. [Advanced Usage and Tips](#advanced-usage-and-tips)
10. [Conclusion](#conclusion)

---

## Getting Started

### Installation

To install Valkey-Redis CLI, download the latest binary from the official repository or use a package manager:

```bash
# Using a package manager
sudo apt-get install valkey-redis-cli
```

### Connecting to a Redis Instance

To connect to a local Redis instance, use:

```bash
valkey-redis-cli
```

For a remote server, specify the host and port:

```bash
valkey-redis-cli -h <hostname> -p <port>
```

### Authentication

If the Redis server requires authentication, use the `-a` flag:

```bash
valkey-redis-cli -h <hostname> -p <port> -a <password>
```

---

## Basic Commands

### PING

The `PING` command is used to test the connection to the Redis server.

```bash
valkey-redis-cli PING
```

**Expected Output:**

```
PONG
```

### ECHO

The `ECHO` command is used to return a message back from the server.

```bash
valkey-redis-cli ECHO "Hello, Redis!"
```

**Expected Output:**

```
"Hello, Redis!"
```

---

## Key Management Commands

### SET

The `SET` command assigns a value to a key.

```bash
valkey-redis-cli SET key "value"
```

**Options:**

- `EX <seconds>`: Set the expiry time in seconds.
- `PX <milliseconds>`: Set the expiry time in milliseconds.
- `NX`: Set the key only if it does not already exist.
- `XX`: Set the key only if it already exists.

**Example:**

```bash
valkey-redis-cli SET key "value" EX 10 NX
```

### GET

The `GET` command retrieves the value of a key.

```bash
valkey-redis-cli GET key
```

### DEL

The `DEL` command removes one or more keys.

```bash
valkey-redis-cli DEL key1 key2
```

---

## Data Operations

### INCR/DECR

Increment or decrement the integer value of a key.

```bash
valkey-redis-cli INCR counter
valkey-redis-cli DECR counter
```

### APPEND

Append a value to a key.

```bash
valkey-redis-cli APPEND key "more data"
```

### STRLEN

Get the length of the value stored in a key.

```bash
valkey-redis-cli STRLEN key
```

---

## Server Management

### INFO

The `INFO` command provides information and statistics about the server.

```bash
valkey-redis-cli INFO
```

### CONFIG

Manage server configuration settings.

- **Get Configuration:**

  ```bash
  valkey-redis-cli CONFIG GET "*"
  ```

- **Set Configuration:**

  ```bash
  valkey-redis-cli CONFIG SET <parameter> <value>
  ```

### MONITOR

Stream every request received by the Redis server.

```bash
valkey-redis-cli MONITOR
```

---

## Security and Authentication

### AUTH

Authenticate to the server.

```bash
valkey-redis-cli AUTH <password>
```

### ACL

Manage Access Control Lists.

- **List ACLs:**

  ```bash
  valkey-redis-cli ACL LIST
  ```

- **Add a User:**

  ```bash
  valkey-redis-cli ACL SETUSER <username> on ><password>
  ```

---

## Scripting with Lua

### EVAL

Execute a Lua script server-side.

```bash
valkey-redis-cli EVAL "<script>" <numkeys> <key> [<arg> ...]
```

### SCRIPT

Manage the scripting subsystem.

- **Load a Script:**

  ```bash
  valkey-redis-cli SCRIPT LOAD <script>
  ```

- **Flush Scripts:**

  ```bash
  valkey-redis-cli SCRIPT FLUSH
  ```

---

## Troubleshooting and Debugging

### DEBUG

Debugging commands for troubleshooting.

- **Object Debugging:**

  ```bash
  valkey-redis-cli DEBUG OBJECT <key>
  ```

- **Segfault:**

  ```bash
  valkey-redis-cli DEBUG SEGFAULT
  ```

### LATENCY

Analyze latency problems.

```bash
valkey-redis-cli LATENCY DOCTOR
```

---

## Advanced Usage and Tips

### Pipelining

Improve performance by batching commands.

```bash
valkey-redis-cli --pipe
```

### Backup and Restore

Create snapshots of your data.

- **Create a Snapshot:**

  ```bash
  valkey-redis-cli SAVE
  ```

- **Restore from a Snapshot:**

  Copy the dump file to the target server and restart it.

### Cluster Management

Manage Redis clusters with commands like `CLUSTER INFO`, `CLUSTER NODES`, etc.

```bash
valkey-redis-cli CLUSTER INFO
```

---

## Conclusion

The Valkey-Redis CLI provides a comprehensive suite of commands to effectively manage and interact with Redis instances. By understanding these commands and their applications, you can efficiently perform a wide array of tasks, from basic data manipulation to advanced server management. This document serves as a detailed guide to maximizing the potential of the Valkey-Redis CLI, ensuring that you have the tools necessary to handle any Redis-related task with confidence.