# Jira JSM On-Call CLI Reference

## Introduction

The `jira-jsm-oncall` Command Line Interface (CLI) is a powerful, comprehensive tool designed for system administrators, site reliability engineers (SREs), and DevOps professionals to manage Jira Service Management (JSM) on-call schedules, routing rules, escalation policies, and incident response workflows directly from the terminal. This document serves as the definitive reference guide for the `jira-jsm-oncall` CLI, detailing every command, flag, argument, and providing extensive examples of usage to facilitate seamless integration into automated scripts and daily operational routines.

By leveraging the `jira-jsm-oncall` CLI, teams can automate the provisioning of on-call schedules, dynamically adjust escalation policies during major incidents, and integrate JSM alerting capabilities into CI/CD pipelines and monitoring systems. The CLI interacts directly with the Jira Service Management REST API, ensuring that all operations are performed securely and efficiently.

This reference is structured logically, beginning with global configuration and authentication, moving through core scheduling and escalation management, and concluding with advanced scripting techniques and error handling. Whether you are a novice user looking to perform simple overrides or an advanced SRE building complex automation, this guide provides the necessary depth and clarity.

## Global Flags

The following global flags are available for all `jira-jsm-oncall` commands. They provide essential configuration options for authentication, output formatting, and operational behavior. Understanding and utilizing these flags is crucial for effective CLI usage, especially in automated environments.

| Flag | Short | Description | Default |
|------|-------|-------------|---------|
| `--api-token` | `-t` | The API token used for authenticating with the Jira Service Management instance. This token must have the appropriate scopes for the actions being performed. | `$JSM_API_TOKEN` |
| `--domain` | `-d` | The base URL of your Jira instance (e.g., `https://your-domain.atlassian.net`). | `$JSM_DOMAIN` |
| `--user` | `-u` | The email address associated with the API token. | `$JSM_USER` |
| `--output` | `-o` | Specifies the output format. Valid options are `json`, `yaml`, `table`, and `text`. `json` is highly recommended for scripting. | `table` |
| `--verbose` | `-v` | Enables verbose logging for debugging purposes. This will output raw HTTP requests and responses, which is invaluable for troubleshooting API issues. | `false` |
| `--quiet` | `-q` | Suppresses all non-essential output, returning only the final result or error code. Ideal for use in CI/CD pipelines where log noise should be minimized. | `false` |
| `--config` | `-c` | Path to a custom configuration file containing authentication and default settings. | `~/.jsm-cli.yaml` |
| `--dry-run` | | Simulates the command without making any actual changes to the Jira Service Management instance. Useful for validating complex configurations before deployment. | `false` |

## Authentication and Configuration

Before utilizing the `jira-jsm-oncall` CLI, it is imperative to configure authentication. This can be achieved via environment variables, a configuration file, or by passing global flags with each command. The CLI supports both interactive and non-interactive authentication methods to accommodate various use cases.

### `jira-jsm-oncall auth login`

Authenticates the CLI session and securely stores the credentials in the default configuration file (`~/.jsm-cli.yaml`). This is typically the first command run after installing the CLI.

**Usage:**
```bash
jira-jsm-oncall auth login [flags]
```

**Flags:**
- `--interactive`, `-i`: Prompts the user for the domain, username, and API token interactively. This is the recommended approach for local development environments.
- `--force`, `-f`: Overwrites any existing configuration without prompting for confirmation.

**Examples:**
```bash
# Interactive login
jira-jsm-oncall auth login --interactive

# Non-interactive login using environment variables (ideal for CI/CD)
export JSM_DOMAIN="https://acme.atlassian.net"
export JSM_USER="admin@acme.com"
export JSM_API_TOKEN="your_api_token_here"
jira-jsm-oncall auth login
```

### `jira-jsm-oncall auth status`

Verifies the current authentication status and displays the active configuration. This command is useful for confirming that the CLI is correctly configured and that the API token is valid and has not expired.

**Usage:**
```bash
jira-jsm-oncall auth status
```

**Output Example:**
```text
Authentication Status: Valid
Domain: https://acme.atlassian.net
User: admin@acme.com
Config File: /Users/admin/.jsm-cli.yaml
```

### `jira-jsm-oncall auth logout`

Removes the stored credentials from the configuration file, effectively logging the user out.

**Usage:**
```bash
jira-jsm-oncall auth logout
```

## Schedule Management

Managing on-call schedules is a core capability of the `jira-jsm-oncall` CLI. These commands allow you to create, update, list, and delete schedules, as well as manage overrides and rotations. Effective schedule management ensures that the right personnel are notified at the right time, minimizing mean time to resolution (MTTR).

### `jira-jsm-oncall schedule list`

Retrieves a list of all on-call schedules within the specified Jira Service Management team or project. This command supports extensive filtering and pagination to handle large environments.

**Usage:**
```bash
jira-jsm-oncall schedule list [flags]
```

**Flags:**
- `--team-id`: Filter schedules by a specific Team ID.
- `--project-key`: Filter schedules by a specific Project Key.
- `--name-contains`: Filter schedules whose name contains the specified string.
- `--limit`: Maximum number of schedules to return (default: 50, max: 100).
- `--offset`: Pagination offset.

**Examples:**
```bash
# List all schedules for a specific team in JSON format
jira-jsm-oncall schedule list --team-id "team-123" --output json

# List schedules for a project with a limit of 10
jira-jsm-oncall schedule list --project-key "ITSM" --limit 10

# Search for schedules containing "Database" in the name
jira-jsm-oncall schedule list --name-contains "Database"
```

### `jira-jsm-oncall schedule get`

Retrieves detailed information about a specific on-call schedule, including its rotations, layers, and current on-call responders. This command is essential for auditing schedule configurations and understanding the current on-call state.

**Usage:**
```bash
jira-jsm-oncall schedule get <schedule-id> [flags]
```

**Arguments:**
- `<schedule-id>`: The unique identifier of the schedule.

**Flags:**
- `--expand`: Expand specific fields such as `rotations`, `overrides`, or `finalTimeline`. Multiple fields can be comma-separated.
- `--date`: Retrieve the schedule state for a specific date and time (ISO 8601 format).

**Examples:**
```bash
# Get detailed schedule information including the final timeline
jira-jsm-oncall schedule get "sch-456" --expand finalTimeline

# Check who was on-call last Friday
jira-jsm-oncall schedule get "sch-456" --date "2023-10-20T12:00:00Z" --expand finalTimeline
```

### `jira-jsm-oncall schedule create`

Creates a new on-call schedule. This command typically requires a JSON or YAML payload defining the schedule's structure, including timezone, rotations, and participants. Creating schedules via the CLI ensures consistency and allows for version-controlled infrastructure-as-code practices.

**Usage:**
```bash
jira-jsm-oncall schedule create [flags]
```

**Flags:**
- `--name`, `-n`: The name of the new schedule.
- `--team-id`: The ID of the team that owns the schedule.
- `--timezone`: The timezone for the schedule (e.g., `America/New_York`).
- `--description`: A brief description of the schedule's purpose.
- `--file`, `-f`: Path to a JSON or YAML file containing the complete schedule definition.

**Examples:**
```bash
# Create a schedule using a comprehensive definition file
jira-jsm-oncall schedule create --file ./new-schedule.yaml

# Create a basic schedule via flags (rotations must be added separately)
jira-jsm-oncall schedule create \
  --name "Database On-Call" \
  --team-id "team-789" \
  --timezone "UTC" \
  --description "Primary on-call rotation for database administrators."
```

### `jira-jsm-oncall schedule update`

Updates an existing on-call schedule. Partial updates are supported, allowing you to modify specific attributes without providing the entire schedule definition.

**Usage:**
```bash
jira-jsm-oncall schedule update <schedule-id> [flags]
```

**Arguments:**
- `<schedule-id>`: The unique identifier of the schedule to update.

**Flags:**
- `--name`, `-n`: The new name for the schedule.
- `--timezone`: The new timezone.
- `--description`: The new description.
- `--file`, `-f`: Path to a JSON or YAML file containing the updated schedule definition.

**Examples:**
```bash
# Update the timezone and description of an existing schedule
jira-jsm-oncall schedule update "sch-456" \
  --timezone "Europe/London" \
  --description "Updated timezone to align with the London office."

# Apply a comprehensive update from a file
jira-jsm-oncall schedule update "sch-456" --file ./updated-schedule.yaml
```

### `jira-jsm-oncall schedule delete`

Permanently deletes an on-call schedule. This action cannot be undone and should be used with extreme caution. It is recommended to verify the schedule ID before execution.

**Usage:**
```bash
jira-jsm-oncall schedule delete <schedule-id> [flags]
```

**Arguments:**
- `<schedule-id>`: The unique identifier of the schedule to delete.

**Flags:**
- `--force`, `-f`: Bypasses the confirmation prompt.

**Examples:**
```bash
# Delete a schedule (will prompt for confirmation)
jira-jsm-oncall schedule delete "sch-456"

# Delete a schedule without prompting for confirmation (use in scripts)
jira-jsm-oncall schedule delete "sch-456" --force
```

## Override Management

Overrides allow temporary modifications to an on-call schedule, typically used when a scheduled responder is unavailable due to illness, personal leave, or shift swapping. Overrides ensure that coverage is maintained without permanently altering the underlying schedule rotations.

### `jira-jsm-oncall override create`

Creates a new override for a specific schedule.

**Usage:**
```bash
jira-jsm-oncall override create [flags]
```

**Flags:**
- `--schedule-id`: The ID of the schedule to override.
- `--user-id`: The ID of the user who will take over the shift.
- `--start`: The start time of the override in ISO 8601 format.
- `--end`: The end time of the override in ISO 8601 format.
- `--rotation-id`: (Optional) The specific rotation to override. If omitted, the override applies to the entire schedule.

**Examples:**
```bash
# Create an override for the weekend
jira-jsm-oncall override create \
  --schedule-id "sch-456" \
  --user-id "usr-999" \
  --start "2023-10-28T00:00:00Z" \
  --end "2023-10-30T00:00:00Z"
```

### `jira-jsm-oncall override list`

Lists all active and upcoming overrides for a specific schedule.

**Usage:**
```bash
jira-jsm-oncall override list --schedule-id <schedule-id> [flags]
```

**Flags:**
- `--schedule-id`: The ID of the schedule.
- `--start`: Filter overrides starting after this time.
- `--end`: Filter overrides ending before this time.
- `--user-id`: Filter overrides assigned to a specific user.

**Examples:**
```bash
# List all overrides for a schedule in the current month
jira-jsm-oncall override list \
  --schedule-id "sch-456" \
  --start "2023-10-01T00:00:00Z" \
  --end "2023-10-31T23:59:59Z"
```

### `jira-jsm-oncall override delete`

Removes an existing override. This is useful if an override was created in error or if the original responder becomes available again.

**Usage:**
```bash
jira-jsm-oncall override delete <override-id> --schedule-id <schedule-id>
```

**Arguments:**
- `<override-id>`: The unique identifier of the override.

**Flags:**
- `--schedule-id`: The ID of the schedule associated with the override.

**Examples:**
```bash
# Delete a specific override
jira-jsm-oncall override delete "ovr-123" --schedule-id "sch-456"
```

## Escalation Policy Management

Escalation policies define the rules for notifying responders when an incident occurs. They ensure that if the primary on-call person does not acknowledge an alert within a specified timeframe, the alert is escalated to secondary responders, team leads, or management.

### `jira-jsm-oncall escalation list`

Lists all escalation policies within a team or project.

**Usage:**
```bash
jira-jsm-oncall escalation list [flags]
```

**Flags:**
- `--team-id`: Filter by Team ID.
- `--project-key`: Filter by Project Key.
- `--name-contains`: Filter by name.

**Examples:**
```bash
# List escalation policies for a specific team
jira-jsm-oncall escalation list --team-id "team-123"
```

### `jira-jsm-oncall escalation get`

Retrieves detailed information about a specific escalation policy, including its rules, delays, and targets.

**Usage:**
```bash
jira-jsm-oncall escalation get <escalation-id>
```

**Arguments:**
- `<escalation-id>`: The unique identifier of the escalation policy.

**Examples:**
```bash
# Get details of an escalation policy
jira-jsm-oncall escalation get "esc-789"
```

### `jira-jsm-oncall escalation create`

Creates a new escalation policy. Due to the complexity of escalation rules, this command typically relies on a configuration file.

**Usage:**
```bash
jira-jsm-oncall escalation create [flags]
```

**Flags:**
- `--name`, `-n`: The name of the escalation policy.
- `--team-id`: The ID of the team that owns the policy.
- `--description`: A description of the policy.
- `--file`, `-f`: Path to a JSON or YAML file containing the policy definition.

**Examples:**
```bash
# Create an escalation policy from a definition file
jira-jsm-oncall escalation create --file ./high-sev-escalation.yaml
```

### `jira-jsm-oncall escalation update`

Updates an existing escalation policy.

**Usage:**
```bash
jira-jsm-oncall escalation update <escalation-id> [flags]
```

**Arguments:**
- `<escalation-id>`: The unique identifier of the escalation policy.

**Flags:**
- `--file`, `-f`: Path to a JSON or YAML file containing the updated policy definition.

**Examples:**
```bash
# Update an escalation policy
jira-jsm-oncall escalation update "esc-789" --file ./updated-escalation.yaml
```

## Routing Rule Management

Routing rules determine how incoming alerts are directed to specific integrations, teams, or escalation policies based on alert content and metadata. They are the intelligent routing layer of JSM.

### `jira-jsm-oncall routing list`

Lists all routing rules for a specific integration or team.

**Usage:**
```bash
jira-jsm-oncall routing list [flags]
```

**Flags:**
- `--integration-id`: Filter by Integration ID.
- `--team-id`: Filter by Team ID.

**Examples:**
```bash
# List routing rules for a specific integration
jira-jsm-oncall routing list --integration-id "int-456"
```

### `jira-jsm-oncall routing create`

Creates a new routing rule.

**Usage:**
```bash
jira-jsm-oncall routing create [flags]
```

**Flags:**
- `--integration-id`: The ID of the integration.
- `--name`: The name of the routing rule.
- `--condition`: The condition expression for the rule (e.g., `priority == P1`).
- `--target-type`: The type of target (`escalation`, `schedule`, `user`).
- `--target-id`: The ID of the target.
- `--order`: The evaluation order of the rule.

**Examples:**
```bash
# Create a routing rule for P1 alerts
jira-jsm-oncall routing create \
  --integration-id "int-456" \
  --name "Route P1 to High Sev Escalation" \
  --condition "priority == P1" \
  --target-type "escalation" \
  --target-id "esc-789" \
  --order 1
```

## Alert and Incident Interaction

While primarily focused on configuration, the CLI also provides commands for interacting with active alerts and incidents, allowing SREs to acknowledge, close, or escalate alerts directly from the terminal without needing to access the web interface.

### `jira-jsm-oncall alert list`

Lists active alerts based on specified criteria.

**Usage:**
```bash
jira-jsm-oncall alert list [flags]
```

**Flags:**
- `--status`: Filter by alert status (`open`, `acknowledged`, `closed`).
- `--priority`: Filter by priority (`P1`, `P2`, `P3`, `P4`, `P5`).
- `--team-id`: Filter by Team ID.
- `--limit`: Maximum number of alerts to return.
- `--query`: A custom JQL-like query string for advanced filtering.

**Examples:**
```bash
# List all open P1 alerts for a specific team
jira-jsm-oncall alert list --status open --priority P1 --team-id "team-123"

# Use a custom query to find alerts containing specific text
jira-jsm-oncall alert list --query "message ~ 'database connection failed'"
```

### `jira-jsm-oncall alert ack`

Acknowledges one or more alerts, indicating that a responder is actively investigating the issue.

**Usage:**
```bash
jira-jsm-oncall alert ack <alert-id>... [flags]
```

**Arguments:**
- `<alert-id>`: One or more alert IDs to acknowledge.

**Flags:**
- `--note`: An optional note to add to the alert upon acknowledgment.
- `--user-id`: The ID of the user acknowledging the alert (if different from the authenticated user, requires admin privileges).

**Examples:**
```bash
# Acknowledge an alert with a note
jira-jsm-oncall alert ack "alt-123" --note "Investigating the database latency issue."
```

### `jira-jsm-oncall alert close`

Closes one or more alerts, indicating that the underlying issue has been resolved.

**Usage:**
```bash
jira-jsm-oncall alert close <alert-id>... [flags]
```

**Arguments:**
- `<alert-id>`: One or more alert IDs to close.

**Flags:**
- `--note`: An optional note to add to the alert upon closure.

**Examples:**
```bash
# Close multiple alerts
jira-jsm-oncall alert close "alt-123" "alt-124" --note "Resolved via automated rollback."
```

### `jira-jsm-oncall alert add-note`

Adds a note or comment to an existing alert without changing its status.

**Usage:**
```bash
jira-jsm-oncall alert add-note <alert-id> --note <text>
```

**Arguments:**
- `<alert-id>`: The ID of the alert.

**Flags:**
- `--note`: The text of the note to add.

**Examples:**
```bash
# Add an investigative note to an alert
jira-jsm-oncall alert add-note "alt-123" --note "CPU utilization spiked to 99% on db-node-01."
```

## Advanced Configuration and Scripting

The `jira-jsm-oncall` CLI is designed to be highly scriptable. By utilizing the `--output json` flag in conjunction with tools like `jq`, complex workflows can be automated, enabling infrastructure-as-code and GitOps methodologies.

### Example: Exporting and Importing Schedules

You can export a schedule to a file, modify it, and create a new schedule based on the modified definition. This is useful for migrating schedules between environments or creating templates.

```bash
# Export the schedule
jira-jsm-oncall schedule get "sch-456" --output json > schedule.json

# Modify the schedule using jq (e.g., change the name)
jq '.name = "Backup Database On-Call"' schedule.json > new-schedule.json

# Create the new schedule
jira-jsm-oncall schedule create --file new-schedule.json
```

### Example: Bulk Acknowledgment of Alerts

During a major incident or a known maintenance window, you may need to acknowledge a large number of alerts simultaneously to prevent alert fatigue.

```bash
# Get all open P3 alerts and acknowledge them
alerts=$(jira-jsm-oncall alert list --status open --priority P3 --output json | jq -r '.[].id')
for alert in $alerts; do
  jira-jsm-oncall alert ack "$alert" --note "Bulk acknowledgment during incident INC-101"
done
```

### Example: Dynamic Override Creation

You can create a script that automatically creates an override when a team member updates their status in a separate HR or calendar system.

```bash
#!/bin/bash
# Script to create an override based on input parameters
SCHEDULE_ID=$1
USER_ID=$2
START_TIME=$3
END_TIME=$4

echo "Creating override for user $USER_ID on schedule $SCHEDULE_ID..."
jira-jsm-oncall override create \
  --schedule-id "$SCHEDULE_ID" \
  --user-id "$USER_ID" \
  --start "$START_TIME" \
  --end "$END_TIME" \
  --output json | jq '.id'
```

## Error Handling and Diagnostics

The CLI uses standard HTTP status codes and provides detailed error messages to assist with troubleshooting. Understanding these errors is critical for building robust automation scripts.

- **401 Unauthorized**: Verify your API token and username. Ensure the token has not expired and is correctly formatted in your configuration or environment variables.
- **403 Forbidden**: The authenticated user lacks the necessary permissions to perform the action. Check your Jira Service Management role assignments and ensure the API token has the required scopes.
- **404 Not Found**: The specified resource (e.g., schedule ID, alert ID) does not exist. Verify the ID and ensure it belongs to the correct team or project.
- **422 Unprocessable Entity**: The provided payload (e.g., JSON file for schedule creation) is invalid. Check the syntax and ensure all required fields are present.
- **429 Too Many Requests**: You have exceeded the API rate limits. Implement exponential backoff and retry logic in your scripts to handle these gracefully.
- **500 Internal Server Error**: An unexpected error occurred on the Jira Service Management servers. Check the Atlassian status page for ongoing incidents.

To diagnose issues, append the `--verbose` flag to any command to view the raw HTTP requests and responses. This output can be redirected to a file for further analysis or sharing with Atlassian support.

```bash
jira-jsm-oncall schedule list --verbose > debug.log 2>&1
```

## Best Practices

1. **Use Service Accounts**: For automated scripts and CI/CD pipelines, use a dedicated service account rather than a personal user account. This ensures that automation continues to function even if a user leaves the organization.
2. **Secure API Tokens**: Never hardcode API tokens in scripts or commit them to version control. Use environment variables or secure secret management systems (e.g., HashiCorp Vault, AWS Secrets Manager).
3. **Validate with Dry Runs**: When making complex changes to schedules or escalation policies, use the `--dry-run` flag (if available) or test the changes in a sandbox environment first.
4. **Leverage JSON Output**: Always use `--output json` when writing scripts. Parsing JSON with tools like `jq` is significantly more robust than attempting to parse tabular or text output.
5. **Implement Retry Logic**: Network instability and API rate limits are inevitable. Ensure your scripts include retry logic with exponential backoff to handle transient errors gracefully.

## Conclusion

The `jira-jsm-oncall` CLI is an indispensable tool for modern operations teams, providing the flexibility and power needed to manage complex on-call configurations and respond to incidents rapidly. By integrating these commands into your operational playbooks and automation pipelines, you can significantly reduce manual overhead, ensure configuration consistency, and improve your organization's overall incident response capabilities.