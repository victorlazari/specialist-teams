# Ticket-Supreme CLI Command Reference

## 1. Introduction

Welcome to the comprehensive Command Line Interface (CLI) reference for **Ticket-Supreme**, the ultimate enterprise-grade ticketing and issue-tracking system. This document provides an exhaustive, deep-dive guide into every command, flag, argument, and configuration option available in the `ticket-supreme` CLI tool. Whether you are a system administrator, a DevOps engineer, or a power user, this guide will equip you with the knowledge required to master the Ticket-Supreme ecosystem.

The `ticket-supreme` CLI is designed to be fast, reliable, and highly scriptable. It interacts directly with the Ticket-Supreme REST API, allowing you to automate workflows, manage users, handle complex ticket routing, and perform bulk operations with ease.

### 1.1. Global Flags

The following global flags can be applied to almost any command within the `ticket-supreme` CLI:

- `--config, -c <path>`: Specify a custom path to the configuration file. Default is `~/.ticket-supreme/config.yaml`.
- `--verbose, -v`: Enable verbose logging. Useful for debugging.
- `--debug, -d`: Enable debug-level logging, which includes raw API requests and responses.
- `--output, -o <format>`: Specify the output format. Supported formats: `json`, `yaml`, `table`, `csv`. Default is `table`.
- `--profile, -p <name>`: Use a specific configuration profile.
- `--dry-run`: Simulate the command without making any actual changes to the system.
- `--help, -h`: Display help information for the current command.

---

## 2. Authentication and Configuration

Before you can interact with the Ticket-Supreme server, you must authenticate and configure your CLI environment.

### 2.1. `ticket-supreme login`

Authenticates the user with the Ticket-Supreme server and stores the session token locally.

**Usage:**
```bash
ticket-supreme login [flags]
```

**Flags:**
- `--username, -u <string>`: Your Ticket-Supreme username.
- `--password, -p <string>`: Your Ticket-Supreme password. If omitted, you will be prompted securely.
- `--token, -t <string>`: Authenticate using a Personal Access Token (PAT) instead of credentials.
- `--server, -s <url>`: The URL of the Ticket-Supreme server (e.g., `https://tickets.example.com`).

**Examples:**
```bash
# Interactive login
ticket-supreme login --server https://tickets.example.com

# Login using a Personal Access Token
ticket-supreme login --server https://tickets.example.com --token abc123xyz
```

### 2.2. `ticket-supreme logout`

Logs out the current user and securely deletes the local session token.

**Usage:**
```bash
ticket-supreme logout
```

### 2.3. `ticket-supreme config`

Manages CLI configuration settings.

**Commands:**
- `ticket-supreme config set <key> <value>`: Set a configuration value.
- `ticket-supreme config get <key>`: Retrieve a configuration value.
- `ticket-supreme config list`: List all configuration values.

**Examples:**
```bash
# Set default output format to JSON
ticket-supreme config set output json

# View current server URL
ticket-supreme config get server
```

---

## 3. Ticket Management

The core functionality of Ticket-Supreme revolves around managing tickets. The `ticket` command group provides extensive capabilities for creating, updating, querying, and deleting tickets.

### 3.1. `ticket-supreme ticket create`

Creates a new ticket in the system.

**Usage:**
```bash
ticket-supreme ticket create [flags]
```

**Flags:**
- `--title, -t <string>`: (Required) The title or summary of the ticket.
- `--description, -d <string>`: The detailed description of the issue.
- `--priority, -p <level>`: The priority level (`low`, `medium`, `high`, `critical`). Default is `medium`.
- `--assignee, -a <username>`: Assign the ticket to a specific user.
- `--labels, -l <list>`: Comma-separated list of labels to apply.
- `--project <id>`: (Required) The ID of the project this ticket belongs to.
- `--attachment <path>`: Path to a file to attach to the ticket. Can be specified multiple times.

**Examples:**
```bash
# Create a simple ticket
ticket-supreme ticket create --project PRJ-1 --title "Database connection timeout" --priority high

# Create a detailed ticket with labels and an assignee
ticket-supreme ticket create --project PRJ-1 \
  --title "UI misalignment on dashboard" \
  --description "The main dashboard widgets overlap on mobile screens." \
  --assignee jdoe \
  --labels "bug,ui,frontend"
```

### 3.2. `ticket-supreme ticket view`

Retrieves and displays the details of a specific ticket.

**Usage:**
```bash
ticket-supreme ticket view <ticket-id> [flags]
```

**Flags:**
- `--comments`: Include ticket comments in the output.
- `--history`: Include the audit history of the ticket.

**Examples:**
```bash
# View basic ticket details
ticket-supreme ticket view TS-1042

# View ticket details including comments in JSON format
ticket-supreme ticket view TS-1042 --comments --output json
```

### 3.3. `ticket-supreme ticket update`

Updates an existing ticket.

**Usage:**
```bash
ticket-supreme ticket update <ticket-id> [flags]
```

**Flags:**
- `--status, -s <status>`: Change the ticket status (`open`, `in-progress`, `resolved`, `closed`).
- `--priority, -p <level>`: Update the priority level.
- `--assignee, -a <username>`: Reassign the ticket.
- `--add-labels <list>`: Comma-separated list of labels to add.
- `--remove-labels <list>`: Comma-separated list of labels to remove.

**Examples:**
```bash
# Mark a ticket as resolved
ticket-supreme ticket update TS-1042 --status resolved

# Reassign a ticket and escalate priority
ticket-supreme ticket update TS-1042 --assignee msmith --priority critical
```

### 3.4. `ticket-supreme ticket comment`

Adds a comment to a ticket.

**Usage:**
```bash
ticket-supreme ticket comment <ticket-id> [flags]
```

**Flags:**
- `--body, -b <string>`: (Required) The content of the comment.
- `--internal`: Mark the comment as internal (visible only to staff).

**Examples:**
```bash
ticket-supreme ticket comment TS-1042 --body "I have deployed the hotfix to staging."
```

### 3.5. `ticket-supreme ticket list`

Lists and filters tickets based on various criteria.

**Usage:**
```bash
ticket-supreme ticket list [flags]
```

**Flags:**
- `--project <id>`: Filter by project ID.
- `--assignee <username>`: Filter by assignee. Use `me` for the current user.
- `--status <status>`: Filter by status.
- `--priority <level>`: Filter by priority.
- `--created-after <date>`: Filter tickets created after a specific date (ISO 8601 format).
- `--limit <number>`: Maximum number of tickets to return. Default is 50.

**Examples:**
```bash
# List all open tickets assigned to me
ticket-supreme ticket list --assignee me --status open

# List high priority tickets in a specific project
ticket-supreme ticket list --project PRJ-1 --priority high --limit 100
```

---

## 4. Project Management

Projects are the top-level organizational units in Ticket-Supreme. The `project` command group allows administrators to manage these entities.

### 4.1. `ticket-supreme project create`

Creates a new project.

**Usage:**
```bash
ticket-supreme project create [flags]
```

**Flags:**
- `--name, -n <string>`: (Required) The name of the project.
- `--key, -k <string>`: (Required) A unique, short identifier for the project (e.g., `ENG`).
- `--description, -d <string>`: A description of the project.
- `--lead <username>`: The project lead or manager.

**Examples:**
```bash
ticket-supreme project create --name "Engineering" --key ENG --lead jdoe
```

### 4.2. `ticket-supreme project list`

Lists all projects accessible to the user.

**Usage:**
```bash
ticket-supreme project list [flags]
```

**Examples:**
```bash
ticket-supreme project list --output table
```

---

## 5. User and Team Management

Managing access and organizational structures is handled via the `user` and `team` command groups.

### 5.1. `ticket-supreme user invite`

Invites a new user to the Ticket-Supreme instance.

**Usage:**
```bash
ticket-supreme user invite <email> [flags]
```

**Flags:**
- `--role <role>`: The role to assign (`admin`, `agent`, `user`). Default is `user`.
- `--team <team-name>`: Automatically add the user to a specific team.

**Examples:**
```bash
ticket-supreme user invite new.hire@example.com --role agent --team "Support Tier 1"
```

### 5.2. `ticket-supreme team create`

Creates a new team.

**Usage:**
```bash
ticket-supreme team create <team-name> [flags]
```

**Flags:**
- `--description <string>`: A description of the team's purpose.

**Examples:**
```bash
ticket-supreme team create "DevOps" --description "Infrastructure and deployment team"
```

---

## 6. Advanced Operations

For power users and automated scripts, Ticket-Supreme provides advanced capabilities.

### 6.1. `ticket-supreme bulk-update`

Performs a bulk update on multiple tickets using a JSON payload or a query.

**Usage:**
```bash
ticket-supreme bulk-update [flags]
```

**Flags:**
- `--query <jql>`: A Ticket-Supreme Query Language (TSQL) string to select tickets.
- `--set-status <status>`: The new status to apply.
- `--set-assignee <username>`: The new assignee.

**Examples:**
```bash
# Close all resolved tickets older than 30 days
ticket-supreme bulk-update --query "status = resolved AND updated < -30d" --set-status closed
```

### 6.2. `ticket-supreme export`

Exports ticket data for reporting or backup purposes.

**Usage:**
```bash
ticket-supreme export [flags]
```

**Flags:**
- `--query <tsql>`: Filter tickets to export.
- `--format <format>`: Export format (`csv`, `json`). Default is `csv`.
- `--file <path>`: Output file path.

**Examples:**
```bash
ticket-supreme export --query "project = ENG" --format csv --file eng_tickets.csv
```

---

## 7. Troubleshooting and Diagnostics

When things go wrong, the CLI provides built-in diagnostic tools.

### 7.1. `ticket-supreme ping`

Checks the connectivity and latency to the Ticket-Supreme server.

**Usage:**
```bash
ticket-supreme ping
```

### 7.2. `ticket-supreme doctor`

Runs a comprehensive suite of checks on your local configuration, network connectivity, and authentication status.

**Usage:**
```bash
ticket-supreme doctor
```

**Output Example:**
```text
[OK] Configuration file found at ~/.ticket-supreme/config.yaml
[OK] Server URL is valid (https://tickets.example.com)
[OK] Network connectivity established (Latency: 45ms)
[OK] Authentication token is valid (Expires in 14 days)
[WARN] CLI version is outdated. Current: v1.2.0, Latest: v1.3.1
```

---

## 8. Webhooks and Integrations

Manage external integrations directly from the CLI.

### 8.1. `ticket-supreme webhook create`

Registers a new webhook endpoint.

**Usage:**
```bash
ticket-supreme webhook create [flags]
```

**Flags:**
- `--url <url>`: (Required) The endpoint URL.
- `--events <list>`: Comma-separated list of events to subscribe to (e.g., `ticket.created`, `ticket.updated`).
- `--secret <string>`: A secret token for payload signature verification.

**Examples:**
```bash
ticket-supreme webhook create --url https://api.mycompany.com/webhook \
  --events "ticket.created,ticket.updated" \
  --secret "super_secret_string"
```

---

## 9. Conclusion

The `ticket-supreme` CLI is a powerful tool that brings the full capabilities of the Ticket-Supreme platform to your terminal. By mastering these commands, you can significantly enhance your productivity, automate tedious tasks, and integrate ticketing workflows seamlessly into your CI/CD pipelines and daily operations. For further assistance, always remember that `ticket-supreme --help` is your best friend.


## Appendix 1: Extended Reference

# Ticket-Supreme CLI Command Reference

## 1. Introduction

Welcome to the comprehensive Command Line Interface (CLI) reference for **Ticket-Supreme**, the ultimate enterprise-grade ticketing and issue-tracking system. This document provides an exhaustive, deep-dive guide into every command, flag, argument, and configuration option available in the `ticket-supreme` CLI tool. Whether you are a system administrator, a DevOps engineer, or a power user, this guide will equip you with the knowledge required to master the Ticket-Supreme ecosystem.

The `ticket-supreme` CLI is designed to be fast, reliable, and highly scriptable. It interacts directly with the Ticket-Supreme REST API, allowing you to automate workflows, manage users, handle complex ticket routing, and perform bulk operations with ease.

### 1.1. Global Flags

The following global flags can be applied to almost any command within the `ticket-supreme` CLI:

- `--config, -c <path>`: Specify a custom path to the configuration file. Default is `~/.ticket-supreme/config.yaml`.
- `--verbose, -v`: Enable verbose logging. Useful for debugging.
- `--debug, -d`: Enable debug-level logging, which includes raw API requests and responses.
- `--output, -o <format>`: Specify the output format. Supported formats: `json`, `yaml`, `table`, `csv`. Default is `table`.
- `--profile, -p <name>`: Use a specific configuration profile.
- `--dry-run`: Simulate the command without making any actual changes to the system.
- `--help, -h`: Display help information for the current command.

---

## 2. Authentication and Configuration

Before you can interact with the Ticket-Supreme server, you must authenticate and configure your CLI environment.

### 2.1. `ticket-supreme login`

Authenticates the user with the Ticket-Supreme server and stores the session token locally.

**Usage:**
```bash
ticket-supreme login [flags]
```

**Flags:**
- `--username, -u <string>`: Your Ticket-Supreme username.
- `--password, -p <string>`: Your Ticket-Supreme password. If omitted, you will be prompted securely.
- `--token, -t <string>`: Authenticate using a Personal Access Token (PAT) instead of credentials.
- `--server, -s <url>`: The URL of the Ticket-Supreme server (e.g., `https://tickets.example.com`).

**Examples:**
```bash
# Interactive login
ticket-supreme login --server https://tickets.example.com

# Login using a Personal Access Token
ticket-supreme login --server https://tickets.example.com --token abc123xyz
```

### 2.2. `ticket-supreme logout`

Logs out the current user and securely deletes the local session token.

**Usage:**
```bash
ticket-supreme logout
```

### 2.3. `ticket-supreme config`

Manages CLI configuration settings.

**Commands:**
- `ticket-supreme config set <key> <value>`: Set a configuration value.
- `ticket-supreme config get <key>`: Retrieve a configuration value.
- `ticket-supreme config list`: List all configuration values.

**Examples:**
```bash
# Set default output format to JSON
ticket-supreme config set output json

# View current server URL
ticket-supreme config get server
```

---

## 3. Ticket Management

The core functionality of Ticket-Supreme revolves around managing tickets. The `ticket` command group provides extensive capabilities for creating, updating, querying, and deleting tickets.

### 3.1. `ticket-supreme ticket create`

Creates a new ticket in the system.

**Usage:**
```bash
ticket-supreme ticket create [flags]
```

**Flags:**
- `--title, -t <string>`: (Required) The title or summary of the ticket.
- `--description, -d <string>`: The detailed description of the issue.
- `--priority, -p <level>`: The priority level (`low`, `medium`, `high`, `critical`). Default is `medium`.
- `--assignee, -a <username>`: Assign the ticket to a specific user.
- `--labels, -l <list>`: Comma-separated list of labels to apply.
- `--project <id>`: (Required) The ID of the project this ticket belongs to.
- `--attachment <path>`: Path to a file to attach to the ticket. Can be specified multiple times.

**Examples:**
```bash
# Create a simple ticket
ticket-supreme ticket create --project PRJ-1 --title "Database connection timeout" --priority high

# Create a detailed ticket with labels and an assignee
ticket-supreme ticket create --project PRJ-1 \
  --title "UI misalignment on dashboard" \
  --description "The main dashboard widgets overlap on mobile screens." \
  --assignee jdoe \
  --labels "bug,ui,frontend"
```

### 3.2. `ticket-supreme ticket view`

Retrieves and displays the details of a specific ticket.

**Usage:**
```bash
ticket-supreme ticket view <ticket-id> [flags]
```

**Flags:**
- `--comments`: Include ticket comments in the output.
- `--history`: Include the audit history of the ticket.

**Examples:**
```bash
# View basic ticket details
ticket-supreme ticket view TS-1042

# View ticket details including comments in JSON format
ticket-supreme ticket view TS-1042 --comments --output json
```

### 3.3. `ticket-supreme ticket update`

Updates an existing ticket.

**Usage:**
```bash
ticket-supreme ticket update <ticket-id> [flags]
```

**Flags:**
- `--status, -s <status>`: Change the ticket status (`open`, `in-progress`, `resolved`, `closed`).
- `--priority, -p <level>`: Update the priority level.
- `--assignee, -a <username>`: Reassign the ticket.
- `--add-labels <list>`: Comma-separated list of labels to add.
- `--remove-labels <list>`: Comma-separated list of labels to remove.

**Examples:**
```bash
# Mark a ticket as resolved
ticket-supreme ticket update TS-1042 --status resolved

# Reassign a ticket and escalate priority
ticket-supreme ticket update TS-1042 --assignee msmith --priority critical
```

### 3.4. `ticket-supreme ticket comment`

Adds a comment to a ticket.

**Usage:**
```bash
ticket-supreme ticket comment <ticket-id> [flags]
```

**Flags:**
- `--body, -b <string>`: (Required) The content of the comment.
- `--internal`: Mark the comment as internal (visible only to staff).

**Examples:**
```bash
ticket-supreme ticket comment TS-1042 --body "I have deployed the hotfix to staging."
```

### 3.5. `ticket-supreme ticket list`

Lists and filters tickets based on various criteria.

**Usage:**
```bash
ticket-supreme ticket list [flags]
```

**Flags:**
- `--project <id>`: Filter by project ID.
- `--assignee <username>`: Filter by assignee. Use `me` for the current user.
- `--status <status>`: Filter by status.
- `--priority <level>`: Filter by priority.
- `--created-after <date>`: Filter tickets created after a specific date (ISO 8601 format).
- `--limit <number>`: Maximum number of tickets to return. Default is 50.

**Examples:**
```bash
# List all open tickets assigned to me
ticket-supreme ticket list --assignee me --status open

# List high priority tickets in a specific project
ticket-supreme ticket list --project PRJ-1 --priority high --limit 100
```

---

## 4. Project Management

Projects are the top-level organizational units in Ticket-Supreme. The `project` command group allows administrators to manage these entities.

### 4.1. `ticket-supreme project create`

Creates a new project.

**Usage:**
```bash
ticket-supreme project create [flags]
```

**Flags:**
- `--name, -n <string>`: (Required) The name of the project.
- `--key, -k <string>`: (Required) A unique, short identifier for the project (e.g., `ENG`).
- `--description, -d <string>`: A description of the project.
- `--lead <username>`: The project lead or manager.

**Examples:**
```bash
ticket-supreme project create --name "Engineering" --key ENG --lead jdoe
```

### 4.2. `ticket-supreme project list`

Lists all projects accessible to the user.

**Usage:**
```bash
ticket-supreme project list [flags]
```

**Examples:**
```bash
ticket-supreme project list --output table
```

---

## 5. User and Team Management

Managing access and organizational structures is handled via the `user` and `team` command groups.

### 5.1. `ticket-supreme user invite`

Invites a new user to the Ticket-Supreme instance.

**Usage:**
```bash
ticket-supreme user invite <email> [flags]
```

**Flags:**
- `--role <role>`: The role to assign (`admin`, `agent`, `user`). Default is `user`.
- `--team <team-name>`: Automatically add the user to a specific team.

**Examples:**
```bash
ticket-supreme user invite new.hire@example.com --role agent --team "Support Tier 1"
```

### 5.2. `ticket-supreme team create`

Creates a new team.

**Usage:**
```bash
ticket-supreme team create <team-name> [flags]
```

**Flags:**
- `--description <string>`: A description of the team's purpose.

**Examples:**
```bash
ticket-supreme team create "DevOps" --description "Infrastructure and deployment team"
```

---

## 6. Advanced Operations

For power users and automated scripts, Ticket-Supreme provides advanced capabilities.

### 6.1. `ticket-supreme bulk-update`

Performs a bulk update on multiple tickets using a JSON payload or a query.

**Usage:**
```bash
ticket-supreme bulk-update [flags]
```

**Flags:**
- `--query <jql>`: A Ticket-Supreme Query Language (TSQL) string to select tickets.
- `--set-status <status>`: The new status to apply.
- `--set-assignee <username>`: The new assignee.

**Examples:**
```bash
# Close all resolved tickets older than 30 days
ticket-supreme bulk-update --query "status = resolved AND updated < -30d" --set-status closed
```

### 6.2. `ticket-supreme export`

Exports ticket data for reporting or backup purposes.

**Usage:**
```bash
ticket-supreme export [flags]
```

**Flags:**
- `--query <tsql>`: Filter tickets to export.
- `--format <format>`: Export format (`csv`, `json`). Default is `csv`.
- `--file <path>`: Output file path.

**Examples:**
```bash
ticket-supreme export --query "project = ENG" --format csv --file eng_tickets.csv
```

---

## 7. Troubleshooting and Diagnostics

When things go wrong, the CLI provides built-in diagnostic tools.

### 7.1. `ticket-supreme ping`

Checks the connectivity and latency to the Ticket-Supreme server.

**Usage:**
```bash
ticket-supreme ping
```

### 7.2. `ticket-supreme doctor`

Runs a comprehensive suite of checks on your local configuration, network connectivity, and authentication status.

**Usage:**
```bash
ticket-supreme doctor
```

**Output Example:**
```text
[OK] Configuration file found at ~/.ticket-supreme/config.yaml
[OK] Server URL is valid (https://tickets.example.com)
[OK] Network connectivity established (Latency: 45ms)
[OK] Authentication token is valid (Expires in 14 days)
[WARN] CLI version is outdated. Current: v1.2.0, Latest: v1.3.1
```

---

## 8. Webhooks and Integrations

Manage external integrations directly from the CLI.

### 8.1. `ticket-supreme webhook create`

Registers a new webhook endpoint.

**Usage:**
```bash
ticket-supreme webhook create [flags]
```

**Flags:**
- `--url <url>`: (Required) The endpoint URL.
- `--events <list>`: Comma-separated list of events to subscribe to (e.g., `ticket.created`, `ticket.updated`).
- `--secret <string>`: A secret token for payload signature verification.

**Examples:**
```bash
ticket-supreme webhook create --url https://api.mycompany.com/webhook \
  --events "ticket.created,ticket.updated" \
  --secret "super_secret_string"
```

---

## 9. Conclusion

The `ticket-supreme` CLI is a powerful tool that brings the full capabilities of the Ticket-Supreme platform to your terminal. By mastering these commands, you can significantly enhance your productivity, automate tedious tasks, and integrate ticketing workflows seamlessly into your CI/CD pipelines and daily operations. For further assistance, always remember that `ticket-supreme --help` is your best friend.


## Appendix 2: Extended Reference

# Ticket-Supreme CLI Command Reference

## 1. Introduction

Welcome to the comprehensive Command Line Interface (CLI) reference for **Ticket-Supreme**, the ultimate enterprise-grade ticketing and issue-tracking system. This document provides an exhaustive, deep-dive guide into every command, flag, argument, and configuration option available in the `ticket-supreme` CLI tool. Whether you are a system administrator, a DevOps engineer, or a power user, this guide will equip you with the knowledge required to master the Ticket-Supreme ecosystem.

The `ticket-supreme` CLI is designed to be fast, reliable, and highly scriptable. It interacts directly with the Ticket-Supreme REST API, allowing you to automate workflows, manage users, handle complex ticket routing, and perform bulk operations with ease.

### 1.1. Global Flags

The following global flags can be applied to almost any command within the `ticket-supreme` CLI:

- `--config, -c <path>`: Specify a custom path to the configuration file. Default is `~/.ticket-supreme/config.yaml`.
- `--verbose, -v`: Enable verbose logging. Useful for debugging.
- `--debug, -d`: Enable debug-level logging, which includes raw API requests and responses.
- `--output, -o <format>`: Specify the output format. Supported formats: `json`, `yaml`, `table`, `csv`. Default is `table`.
- `--profile, -p <name>`: Use a specific configuration profile.
- `--dry-run`: Simulate the command without making any actual changes to the system.
- `--help, -h`: Display help information for the current command.

---

## 2. Authentication and Configuration

Before you can interact with the Ticket-Supreme server, you must authenticate and configure your CLI environment.

### 2.1. `ticket-supreme login`

Authenticates the user with the Ticket-Supreme server and stores the session token locally.

**Usage:**
```bash
ticket-supreme login [flags]
```

**Flags:**
- `--username, -u <string>`: Your Ticket-Supreme username.
- `--password, -p <string>`: Your Ticket-Supreme password. If omitted, you will be prompted securely.
- `--token, -t <string>`: Authenticate using a Personal Access Token (PAT) instead of credentials.
- `--server, -s <url>`: The URL of the Ticket-Supreme server (e.g., `https://tickets.example.com`).

**Examples:**
```bash
# Interactive login
ticket-supreme login --server https://tickets.example.com

# Login using a Personal Access Token
ticket-supreme login --server https://tickets.example.com --token abc123xyz
```

### 2.2. `ticket-supreme logout`

Logs out the current user and securely deletes the local session token.

**Usage:**
```bash
ticket-supreme logout
```

### 2.3. `ticket-supreme config`

Manages CLI configuration settings.

**Commands:**
- `ticket-supreme config set <key> <value>`: Set a configuration value.
- `ticket-supreme config get <key>`: Retrieve a configuration value.
- `ticket-supreme config list`: List all configuration values.

**Examples:**
```bash
# Set default output format to JSON
ticket-supreme config set output json

# View current server URL
ticket-supreme config get server
```

---

## 3. Ticket Management

The core functionality of Ticket-Supreme revolves around managing tickets. The `ticket` command group provides extensive capabilities for creating, updating, querying, and deleting tickets.

### 3.1. `ticket-supreme ticket create`

Creates a new ticket in the system.

**Usage:**
```bash
ticket-supreme ticket create [flags]
```

**Flags:**
- `--title, -t <string>`: (Required) The title or summary of the ticket.
- `--description, -d <string>`: The detailed description of the issue.
- `--priority, -p <level>`: The priority level (`low`, `medium`, `high`, `critical`). Default is `medium`.
- `--assignee, -a <username>`: Assign the ticket to a specific user.
- `--labels, -l <list>`: Comma-separated list of labels to apply.
- `--project <id>`: (Required) The ID of the project this ticket belongs to.
- `--attachment <path>`: Path to a file to attach to the ticket. Can be specified multiple times.

**Examples:**
```bash
# Create a simple ticket
ticket-supreme ticket create --project PRJ-1 --title "Database connection timeout" --priority high

# Create a detailed ticket with labels and an assignee
ticket-supreme ticket create --project PRJ-1 \
  --title "UI misalignment on dashboard" \
  --description "The main dashboard widgets overlap on mobile screens." \
  --assignee jdoe \
  --labels "bug,ui,frontend"
```

### 3.2. `ticket-supreme ticket view`

Retrieves and displays the details of a specific ticket.

**Usage:**
```bash
ticket-supreme ticket view <ticket-id> [flags]
```

**Flags:**
- `--comments`: Include ticket comments in the output.
- `--history`: Include the audit history of the ticket.

**Examples:**
```bash
# View basic ticket details
ticket-supreme ticket view TS-1042

# View ticket details including comments in JSON format
ticket-supreme ticket view TS-1042 --comments --output json
```

### 3.3. `ticket-supreme ticket update`

Updates an existing ticket.

**Usage:**
```bash
ticket-supreme ticket update <ticket-id> [flags]
```

**Flags:**
- `--status, -s <status>`: Change the ticket status (`open`, `in-progress`, `resolved`, `closed`).
- `--priority, -p <level>`: Update the priority level.
- `--assignee, -a <username>`: Reassign the ticket.
- `--add-labels <list>`: Comma-separated list of labels to add.
- `--remove-labels <list>`: Comma-separated list of labels to remove.

**Examples:**
```bash
# Mark a ticket as resolved
ticket-supreme ticket update TS-1042 --status resolved

# Reassign a ticket and escalate priority
ticket-supreme ticket update TS-1042 --assignee msmith --priority critical
```

### 3.4. `ticket-supreme ticket comment`

Adds a comment to a ticket.

**Usage:**
```bash
ticket-supreme ticket comment <ticket-id> [flags]
```

**Flags:**
- `--body, -b <string>`: (Required) The content of the comment.
- `--internal`: Mark the comment as internal (visible only to staff).

**Examples:**
```bash
ticket-supreme ticket comment TS-1042 --body "I have deployed the hotfix to staging."
```

### 3.5. `ticket-supreme ticket list`

Lists and filters tickets based on various criteria.

**Usage:**
```bash
ticket-supreme ticket list [flags]
```

**Flags:**
- `--project <id>`: Filter by project ID.
- `--assignee <username>`: Filter by assignee. Use `me` for the current user.
- `--status <status>`: Filter by status.
- `--priority <level>`: Filter by priority.
- `--created-after <date>`: Filter tickets created after a specific date (ISO 8601 format).
- `--limit <number>`: Maximum number of tickets to return. Default is 50.

**Examples:**
```bash
# List all open tickets assigned to me
ticket-supreme ticket list --assignee me --status open

# List high priority tickets in a specific project
ticket-supreme ticket list --project PRJ-1 --priority high --limit 100
```

---

## 4. Project Management

Projects are the top-level organizational units in Ticket-Supreme. The `project` command group allows administrators to manage these entities.

### 4.1. `ticket-supreme project create`

Creates a new project.

**Usage:**
```bash
ticket-supreme project create [flags]
```

**Flags:**
- `--name, -n <string>`: (Required) The name of the project.
- `--key, -k <string>`: (Required) A unique, short identifier for the project (e.g., `ENG`).
- `--description, -d <string>`: A description of the project.
- `--lead <username>`: The project lead or manager.

**Examples:**
```bash
ticket-supreme project create --name "Engineering" --key ENG --lead jdoe
```

### 4.2. `ticket-supreme project list`

Lists all projects accessible to the user.

**Usage:**
```bash
ticket-supreme project list [flags]
```

**Examples:**
```bash
ticket-supreme project list --output table
```

---

## 5. User and Team Management

Managing access and organizational structures is handled via the `user` and `team` command groups.

### 5.1. `ticket-supreme user invite`

Invites a new user to the Ticket-Supreme instance.

**Usage:**
```bash
ticket-supreme user invite <email> [flags]
```

**Flags:**
- `--role <role>`: The role to assign (`admin`, `agent`, `user`). Default is `user`.
- `--team <team-name>`: Automatically add the user to a specific team.

**Examples:**
```bash
ticket-supreme user invite new.hire@example.com --role agent --team "Support Tier 1"
```

### 5.2. `ticket-supreme team create`

Creates a new team.

**Usage:**
```bash
ticket-supreme team create <team-name> [flags]
```

**Flags:**
- `--description <string>`: A description of the team's purpose.

**Examples:**
```bash
ticket-supreme team create "DevOps" --description "Infrastructure and deployment team"
```

---

## 6. Advanced Operations

For power users and automated scripts, Ticket-Supreme provides advanced capabilities.

### 6.1. `ticket-supreme bulk-update`

Performs a bulk update on multiple tickets using a JSON payload or a query.

**Usage:**
```bash
ticket-supreme bulk-update [flags]
```

**Flags:**
- `--query <jql>`: A Ticket-Supreme Query Language (TSQL) string to select tickets.
- `--set-status <status>`: The new status to apply.
- `--set-assignee <username>`: The new assignee.

**Examples:**
```bash
# Close all resolved tickets older than 30 days
ticket-supreme bulk-update --query "status = resolved AND updated < -30d" --set-status closed
```

### 6.2. `ticket-supreme export`

Exports ticket data for reporting or backup purposes.

**Usage:**
```bash
ticket-supreme export [flags]
```

**Flags:**
- `--query <tsql>`: Filter tickets to export.
- `--format <format>`: Export format (`csv`, `json`). Default is `csv`.
- `--file <path>`: Output file path.

**Examples:**
```bash
ticket-supreme export --query "project = ENG" --format csv --file eng_tickets.csv
```

---

## 7. Troubleshooting and Diagnostics

When things go wrong, the CLI provides built-in diagnostic tools.

### 7.1. `ticket-supreme ping`

Checks the connectivity and latency to the Ticket-Supreme server.

**Usage:**
```bash
ticket-supreme ping
```

### 7.2. `ticket-supreme doctor`

Runs a comprehensive suite of checks on your local configuration, network connectivity, and authentication status.

**Usage:**
```bash
ticket-supreme doctor
```

**Output Example:**
```text
[OK] Configuration file found at ~/.ticket-supreme/config.yaml
[OK] Server URL is valid (https://tickets.example.com)
[OK] Network connectivity established (Latency: 45ms)
[OK] Authentication token is valid (Expires in 14 days)
[WARN] CLI version is outdated. Current: v1.2.0, Latest: v1.3.1
```

---

## 8. Webhooks and Integrations

Manage external integrations directly from the CLI.

### 8.1. `ticket-supreme webhook create`

Registers a new webhook endpoint.

**Usage:**
```bash
ticket-supreme webhook create [flags]
```

**Flags:**
- `--url <url>`: (Required) The endpoint URL.
- `--events <list>`: Comma-separated list of events to subscribe to (e.g., `ticket.created`, `ticket.updated`).
- `--secret <string>`: A secret token for payload signature verification.

**Examples:**
```bash
ticket-supreme webhook create --url https://api.mycompany.com/webhook \
  --events "ticket.created,ticket.updated" \
  --secret "super_secret_string"
```

---

## 9. Conclusion

The `ticket-supreme` CLI is a powerful tool that brings the full capabilities of the Ticket-Supreme platform to your terminal. By mastering these commands, you can significantly enhance your productivity, automate tedious tasks, and integrate ticketing workflows seamlessly into your CI/CD pipelines and daily operations. For further assistance, always remember that `ticket-supreme --help` is your best friend.
