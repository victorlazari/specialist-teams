# Database CLI Command Reference

## Overview
The Database Command Line Interface (CLI) provides a comprehensive suite of tools for managing, querying, and administering database clusters. This reference guide details every command, flag, argument, and provides extensive examples for enterprise-grade database operations.

## Global Flags
These flags can be applied to any command within the Database CLI.

| Flag | Short | Description | Default |
|------|-------|-------------|---------|
| `--config` | `-c` | Path to the configuration file | `~/.dbcli/config.yaml` |
| `--output` | `-o` | Output format (json, table, yaml) | `table` |
| `--verbose` | `-v` | Enable verbose logging | `false` |
| `--cluster` | `-C` | Target cluster identifier | `default` |

## 1. Connection Management

### `dbcli connect`
Establishes a connection to the target database cluster.

**Usage:**
`dbcli connect [OPTIONS] <connection-string>`

**Flags:**
- `--timeout` (`-t`): Connection timeout in seconds (default: 30)
- `--ssl-mode`: SSL connection mode (disable, require, verify-ca, verify-full)
- `--cert`: Path to client certificate

**Examples:**
```bash
# Connect using a standard connection string
dbcli connect postgresql://user:pass@localhost:5432/mydb

# Connect with strict SSL verification
dbcli connect --ssl-mode=verify-full --cert=/path/to/cert.pem postgresql://prod-db.internal:5432/main
```

### `dbcli disconnect`
Terminates active connections to the cluster.

**Usage:**
`dbcli disconnect [OPTIONS]`

**Flags:**
- `--force` (`-f`): Forcefully terminate connections without waiting for active transactions
- `--session-id`: Disconnect a specific session ID

## 2. Cluster Administration

### `dbcli cluster status`
Retrieves the current health and status of the database cluster.

**Usage:**
`dbcli cluster status [OPTIONS]`

**Flags:**
- `--detailed` (`-d`): Include node-level metrics and replication lag

**Examples:**
```bash
# Get a quick overview of cluster health
dbcli cluster status

# Get detailed metrics in JSON format
dbcli cluster status --detailed --output=json
```

### `dbcli cluster scale`
Scales the database cluster by adding or removing nodes.

**Usage:**
`dbcli cluster scale [OPTIONS]`

**Flags:**
- `--replicas` (`-r`): Target number of replica nodes
- `--instance-type`: Cloud instance type for new nodes
- `--region`: Target region for scaling

**Examples:**
```bash
# Scale read replicas to 5
dbcli cluster scale --replicas=5

# Add a new node in a specific region
dbcli cluster scale --replicas=3 --region=us-east-1 --instance-type=db.r5.large
```

## 3. Data Operations

### `dbcli query`
Executes a SQL query against the database.

**Usage:**
`dbcli query [OPTIONS] <sql-statement>`

**Flags:**
- `--file` (`-f`): Read SQL statement from a file
- `--dry-run`: Parse and validate the query without executing
- `--timeout`: Query execution timeout in seconds

**Examples:**
```bash
# Execute a simple SELECT query
dbcli query "SELECT * FROM users LIMIT 10"

# Execute a complex script from a file
dbcli query --file=migrations/001_init.sql
```

### `dbcli backup create`
Initiates a full or incremental backup of the database.

**Usage:**
`dbcli backup create [OPTIONS]`

**Flags:**
- `--type`: Backup type (full, incremental)
- `--destination`: Storage destination (s3://bucket/path, local path)
- `--compress`: Compression algorithm (gzip, zstd, none)

**Examples:**
```bash
# Create a full backup to S3
dbcli backup create --type=full --destination=s3://my-backups/db/ --compress=zstd
```

### `dbcli backup restore`
Restores the database from a previous backup.

**Usage:**
`dbcli backup restore [OPTIONS] <backup-id>`

**Flags:**
- `--point-in-time` (`-p`): Target timestamp for point-in-time recovery
- `--target-cluster`: Restore to a different cluster

**Examples:**
```bash
# Restore from a specific backup ID
dbcli backup restore bck-123456789

# Perform point-in-time recovery
dbcli backup restore bck-123456789 --point-in-time="2023-10-25T14:30:00Z"
```

## 4. User and Role Management

### `dbcli user create`
Creates a new database user.

**Usage:**
`dbcli user create [OPTIONS] <username>`

**Flags:**
- `--password`: User password (will prompt if not provided)
- `--roles`: Comma-separated list of roles to assign
- `--valid-until`: Expiration date for the user account

**Examples:**
```bash
# Create a read-only user
dbcli user create analytics_user --roles=readonly

# Create an admin user with an expiration date
dbcli user create temp_admin --roles=admin --valid-until="2024-01-01"
```

### `dbcli role grant`
Grants specific permissions to a role.

**Usage:**
`dbcli role grant [OPTIONS] <role-name>`

**Flags:**
- `--privileges`: Comma-separated list of privileges (SELECT, INSERT, UPDATE, DELETE, ALL)
- `--tables`: Target tables (or '*' for all)
- `--schema`: Target schema

**Examples:**
```bash
# Grant SELECT on all tables in public schema
dbcli role grant readonly --privileges=SELECT --schema=public --tables="*"
```

## 5. Performance and Diagnostics

### `dbcli analyze`
Analyzes query performance and provides optimization recommendations.

**Usage:**
`dbcli analyze [OPTIONS] <query-id>`

**Flags:**
- `--explain`: Generate execution plan
- `--format`: Output format for the execution plan (text, json, xml)

**Examples:**
```bash
# Analyze a specific slow query
dbcli analyze qry-987654321 --explain --format=json
```

### `dbcli logs tail`
Streams real-time database logs.

**Usage:**
`dbcli logs tail [OPTIONS]`

**Flags:**
- `--level`: Minimum log level (info, warn, error, fatal)
- `--grep`: Filter logs by a specific pattern
- `--lines` (`-n`): Number of lines to show initially

**Examples:**
```bash
# Tail error logs
dbcli logs tail --level=error

# Search for specific transaction IDs in logs
dbcli logs tail --grep="tx-555"
```

## Conclusion
The Database CLI is a powerful tool designed to streamline database administration. For further assistance, use the `dbcli help` command or refer to the official documentation.