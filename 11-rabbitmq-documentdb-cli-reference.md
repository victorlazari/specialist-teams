# rabbitmq-documentdb CLI Command Reference

## Introduction

Welcome to the `rabbitmq-documentdb` CLI Command Reference. This document provides a comprehensive guide to using the command-line interface for RabbitMQ DocumentDB, a powerful and flexible document-oriented database built on top of RabbitMQ. Whether you are setting up your database, managing collections, or configuring security settings, this reference will provide the detailed information you need to effectively utilize the CLI.

### What is RabbitMQ DocumentDB?

RabbitMQ DocumentDB is a high-performance, scalable document database that leverages the messaging capabilities of RabbitMQ. It is designed to handle large volumes of data with ease, providing powerful indexing and querying capabilities along with robust security and replication features.

### Who Should Use This Document?

This document is intended for database administrators, developers, and IT professionals who are responsible for setting up and managing RabbitMQ DocumentDB environments. A basic understanding of document databases and command-line interfaces is assumed.

## Installation & Configuration

To begin using the `rabbitmq-documentdb` CLI, you must first install it on your system. The CLI is available for multiple platforms, including Windows, macOS, and Linux.

### Installation

#### Prerequisites

- **RabbitMQ**: Ensure that RabbitMQ is installed and running on your system.
- **Node.js**: The CLI tool is distributed via npm, so you need Node.js installed on your system. A minimum version of 12.x is recommended.

#### Steps

1. **Install via npm**:
   ```bash
   npm install -g rabbitmq-documentdb-cli
   ```

2. **Verify Installation**:
   ```bash
   rabbitmq-documentdb --version
   ```

   The command should output the installed version of the CLI, confirming a successful installation.

### Configuration

After installation, configure the CLI to connect to your RabbitMQ DocumentDB instance.

#### Configuration File

Create a configuration file at `~/.rabbitmq-documentdb/config.json`:

```json
{
  "host": "localhost",
  "port": 5672,
  "username": "admin",
  "password": "secret"
}
```

Alternatively, you can specify configuration options using environment variables:

- `RABBITMQ_DOCUMENTDB_HOST`
- `RABBITMQ_DOCUMENTDB_PORT`
- `RABBITMQ_DOCUMENTDB_USERNAME`
- `RABBITMQ_DOCUMENTDB_PASSWORD`

## Global Flags

Global flags are options that apply to all commands in the CLI. They can be used to modify the behavior of any command.

| Flag       | Description                                        |
|------------|----------------------------------------------------|
| `--help`   | Displays help information for the command          |
| `--version`| Outputs the version of the CLI                     |
| `--verbose`| Enables verbose output for debugging purposes      |

Example:

```bash
rabbitmq-documentdb --verbose create-db mydb
```

## Core Commands

### `init`

Initializes a new RabbitMQ DocumentDB environment. This command sets up the necessary infrastructure for the database to operate.

#### Syntax

```bash
rabbitmq-documentdb init
```

#### Options

- `--force`: Forces initialization even if the environment is already set up.

#### Example

```bash
rabbitmq-documentdb init --force
```

### `start`

Starts the RabbitMQ DocumentDB service.

#### Syntax

```bash
rabbitmq-documentdb start
```

#### Example

```bash
rabbitmq-documentdb start
```

### `stop`

Stops the RabbitMQ DocumentDB service.

#### Syntax

```bash
rabbitmq-documentdb stop
```

#### Example

```bash
rabbitmq-documentdb stop
```

### `status`

Displays the current status of the RabbitMQ DocumentDB service.

#### Syntax

```bash
rabbitmq-documentdb status
```

#### Example

```bash
rabbitmq-documentdb status
```

## Database Management Commands

### `create-db`

Creates a new database within the RabbitMQ DocumentDB instance.

#### Syntax

```bash
rabbitmq-documentdb create-db <database-name>
```

#### Options

- `--if-not-exists`: Only create the database if it does not already exist.

#### Example

```bash
rabbitmq-documentdb create-db mydb --if-not-exists
```

### `drop-db`

Drops an existing database, deleting all its data.

#### Syntax

```bash
rabbitmq-documentdb drop-db <database-name>
```

#### Options

- `--force`: Forces deletion without confirmation.

#### Example

```bash
rabbitmq-documentdb drop-db mydb --force
```

### `list-dbs`

Lists all databases within the RabbitMQ DocumentDB instance.

#### Syntax

```bash
rabbitmq-documentdb list-dbs
```

#### Example

```bash
rabbitmq-documentdb list-dbs
```

## Collection Management Commands

### `create-collection`

Creates a new collection within a specified database.

#### Syntax

```bash
rabbitmq-documentdb create-collection <database-name> <collection-name>
```

#### Example

```bash
rabbitmq-documentdb create-collection mydb mycollection
```

### `drop-collection`

Drops a collection from a specified database.

#### Syntax

```bash
rabbitmq-documentdb drop-collection <database-name> <collection-name>
```

#### Options

- `--force`: Forces deletion without confirmation.

#### Example

```bash
rabbitmq-documentdb drop-collection mydb mycollection --force
```

## Document Operations

### `insert`

Inserts a new document into a specified collection.

#### Syntax

```bash
rabbitmq-documentdb insert <database-name> <collection-name> <document-json>
```

#### Example

```bash
rabbitmq-documentdb insert mydb mycollection '{"name": "John Doe", "age": 30}'
```

### `find`

Finds documents in a collection based on a query.

#### Syntax

```bash
rabbitmq-documentdb find <database-name> <collection-name> <query-json>
```

#### Options

- `--limit`: Limits the number of documents returned.

#### Example

```bash
rabbitmq-documentdb find mydb mycollection '{"age": {"$gt": 25}}' --limit 10
```

### `update`

Updates documents in a collection based on a query.

#### Syntax

```bash
rabbitmq-documentdb update <database-name> <collection-name> <query-json> <update-json>
```

#### Example

```bash
rabbitmq-documentdb update mydb mycollection '{"name": "John Doe"}' '{"$set": {"age": 31}}'
```

### `delete`

Deletes documents from a collection based on a query.

#### Syntax

```bash
rabbitmq-documentdb delete <database-name> <collection-name> <query-json>
```

#### Options

- `--limit`: Limits the number of documents deleted.

#### Example

```bash
rabbitmq-documentdb delete mydb mycollection '{"age": {"$lt": 20}}' --limit 5
```

## Indexing Commands

### `create-index`

Creates an index on a collection to improve query performance.

#### Syntax

```bash
rabbitmq-documentdb create-index <database-name> <collection-name> <index-json>
```

#### Example

```bash
rabbitmq-documentdb create-index mydb mycollection '{"age": 1}'
```

### `drop-index`

Drops an index from a collection.

#### Syntax

```bash
rabbitmq-documentdb drop-index <database-name> <collection-name> <index-name>
```

#### Example

```bash
rabbitmq-documentdb drop-index mydb mycollection age_index
```

## Cluster & Replication Commands

### `add-node`

Adds a new node to the RabbitMQ DocumentDB cluster.

#### Syntax

```bash
rabbitmq-documentdb add-node <node-address>
```

#### Example

```bash
rabbitmq-documentdb add-node 192.168.1.10
```

### `remove-node`

Removes a node from the RabbitMQ DocumentDB cluster.

#### Syntax

```bash
rabbitmq-documentdb remove-node <node-address>
```

#### Example

```bash
rabbitmq-documentdb remove-node 192.168.1.10
```

## Security & Access Control

### `create-user`

Creates a new user for accessing the database.

#### Syntax

```bash
rabbitmq-documentdb create-user <username> <password>
```

#### Options

- `--role`: Specifies the role for the user (e.g., `admin`, `read-only`).

#### Example

```bash
rabbitmq-documentdb create-user johndoe secretpass --role admin
```

### `delete-user`

Deletes an existing user.

#### Syntax

```bash
rabbitmq-documentdb delete-user <username>
```

#### Example

```bash
rabbitmq-documentdb delete-user johndoe
```

## Troubleshooting & Diagnostics

### `logs`

Displays the logs for the RabbitMQ DocumentDB service.

#### Syntax

```bash
rabbitmq-documentdb logs
```

#### Options

- `--tail`: Displays the last N lines of the logs.
- `--follow`: Continuously outputs new log entries.

#### Example

```bash
rabbitmq-documentdb logs --tail 100 --follow
```

### `diagnostics`

Runs diagnostic checks on the RabbitMQ DocumentDB service.

#### Syntax

```bash
rabbitmq-documentdb diagnostics
```

#### Example

```bash
rabbitmq-documentdb diagnostics
```

## Advanced Usage Examples

### Creating a Database and Collection

```bash
rabbitmq-documentdb create-db mydb
rabbitmq-documentdb create-collection mydb users
```

### Inserting Documents and Creating an Index

```bash
rabbitmq-documentdb insert mydb users '{"name": "Alice", "email": "alice@example.com"}'
rabbitmq-documentdb create-index mydb users '{"email": 1}'
```

### Querying and Updating Documents

```bash
rabbitmq-documentdb find mydb users '{"email": "alice@example.com"}'
rabbitmq-documentdb update mydb users '{"name": "Alice"}' '{"$set": {"email": "alice@newdomain.com"}}'
```

### Managing Cluster Nodes

```bash
rabbitmq-documentdb add-node 192.168.1.11
rabbitmq-documentdb remove-node 192.168.1.11
```

### Setting Up User Access

```bash
rabbitmq-documentdb create-user admin strongpassword --role admin
rabbitmq-documentdb delete-user guest
```

## Conclusion

This comprehensive CLI reference provides detailed information on using the `rabbitmq-documentdb` command-line interface. From basic setup and configuration to advanced operations and diagnostics, this guide is designed to help you effectively manage your RabbitMQ DocumentDB environment. For further information, refer to the official RabbitMQ DocumentDB documentation or contact support.