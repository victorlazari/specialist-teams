# Oncall-Master-Supreme CLI Command Reference

## 1. Introduction

Welcome to the comprehensive Command Line Interface (CLI) reference for **oncall-master-supreme**. This document serves as the definitive guide for operators, system administrators, and site reliability engineers (SREs) who utilize the `oncall-master-supreme` toolset to manage on-call rotations, incident response, alerting pipelines, and system health diagnostics. 

The `oncall-master-supreme` CLI is designed to be a robust, high-performance, and highly extensible utility. It interfaces directly with the core on-call management API, providing real-time access to schedules, escalations, and incident metadata. Whether you are acknowledging a critical page at 3 AM or configuring complex multi-tier escalation policies during business hours, this CLI provides the necessary commands to execute your tasks efficiently.

This guide covers every command, flag, argument, and provides detailed examples of usage. It is structured logically, starting from basic authentication and configuration, moving through daily operational commands, and concluding with advanced administrative and diagnostic utilities.

---

## 2. Global Flags and Configuration

Before diving into specific commands, it is essential to understand the global flags that can be applied to any `oncall-master-supreme` command. These flags control authentication, output formatting, verbosity, and network behavior.

### 2.1 Global Flags

- `--config, -c <path>`: Specifies the path to a custom configuration file. By default, the CLI looks for `~/.oncall-master-supreme/config.yaml`.
- `--profile, -p <name>`: Selects a specific configuration profile from the config file. Useful for managing multiple environments (e.g., `production`, `staging`, `eu-west`).
- `--output, -o <format>`: Determines the output format. Supported formats are `text` (default), `json`, `yaml`, and `table`.
- `--verbose, -v`: Enables verbose logging. Can be stacked (e.g., `-vv` for debug, `-vvv` for trace) to increase log detail.
- `--quiet, -q`: Suppresses all non-essential output. Only fatal errors and explicitly requested data are printed.
- `--timeout, -t <seconds>`: Sets the maximum time to wait for an API response. Default is 30 seconds.
- `--dry-run`: Simulates the command without making any actual changes to the system. Highly recommended for destructive operations.

### 2.2 Environment Variables

The CLI also respects several environment variables, which can be used to override default behaviors or avoid passing sensitive information via command-line arguments:

- `ONCALL_API_KEY`: The authentication token used to communicate with the backend API.
- `ONCALL_API_URL`: The base URL of the on-call management server.
- `ONCALL_DEFAULT_OUTPUT`: Sets the default output format.
- `ONCALL_HTTP_PROXY`: Specifies an HTTP proxy for outbound requests.

---

## 3. Authentication and Setup

### 3.1 `oncall-master-supreme login`

Authenticates the CLI with the backend server and securely stores the session token.

**Usage:**
`oncall-master-supreme login [flags]`

**Flags:**
- `--method <type>`: The authentication method to use. Options: `password`, `sso`, `api-key`. Default is `sso`.
- `--username, -u <string>`: The username for password authentication.
- `--password <string>`: The password. If omitted when using password auth, the CLI will prompt securely.

**Examples:**
```bash
# Login using SSO (opens a browser window)
oncall-master-supreme login --method sso

# Login using an API key
oncall-master-supreme login --method api-key --password "sk_live_123456789"
```

### 3.2 `oncall-master-supreme configure`

Interactive wizard to set up the default configuration file.

**Usage:**
`oncall-master-supreme configure [flags]`

**Flags:**
- `--set <key=value>`: Non-interactively set a specific configuration key.

**Examples:**
```bash
# Run the interactive setup
oncall-master-supreme configure

# Set the default output format to JSON
oncall-master-supreme configure --set output=json
```

---

## 4. Incident Management

The core functionality of `oncall-master-supreme` revolves around managing incidents. These commands allow you to create, acknowledge, resolve, and analyze incidents.

### 4.1 `oncall-master-supreme incident create`

Triggers a new incident manually.

**Usage:**
`oncall-master-supreme incident create [flags]`

**Flags:**
- `--title, -t <string>`: (Required) A brief summary of the incident.
- `--description, -d <string>`: A detailed description of the problem, including any relevant logs or metrics.
- `--severity, -s <level>`: The severity level. Options: `SEV-1` (Critical), `SEV-2` (High), `SEV-3` (Moderate), `SEV-4` (Low). Default is `SEV-3`.
- `--service <name>`: The name of the service affected by the incident.
- `--assignee <user_id>`: Directly assign the incident to a specific user, bypassing the normal escalation policy.

**Examples:**
```bash
# Create a critical incident for the payment gateway
oncall-master-supreme incident create --title "Payment Gateway Latency Spike" --severity SEV-1 --service payment-api

# Create an incident with a detailed description from a file
oncall-master-supreme incident create --title "Database Connection Pool Exhausted" --description "$(cat error.log)"
```

### 4.2 `oncall-master-supreme incident list`

Retrieves a list of incidents based on specified filters.

**Usage:**
`oncall-master-supreme incident list [flags]`

**Flags:**
- `--status <state>`: Filter by status. Options: `triggered`, `acknowledged`, `resolved`, `all`. Default is `triggered,acknowledged`.
- `--service <name>`: Filter by affected service.
- `--since <duration>`: Show incidents created within the specified time frame (e.g., `1h`, `24h`, `7d`).
- `--limit, -l <int>`: Maximum number of results to return. Default is 50.

**Examples:**
```bash
# List all currently active incidents
oncall-master-supreme incident list

# List resolved incidents from the last 24 hours in JSON format
oncall-master-supreme incident list --status resolved --since 24h -o json
```

### 4.3 `oncall-master-supreme incident ack`

Acknowledges one or more incidents, indicating that someone is actively investigating.

**Usage:**
`oncall-master-supreme incident ack <incident_id>... [flags]`

**Flags:**
- `--message, -m <string>`: An optional message to append to the incident timeline.

**Examples:**
```bash
# Acknowledge a single incident
oncall-master-supreme incident ack INC-1024

# Acknowledge multiple incidents with a message
oncall-master-supreme incident ack INC-1024 INC-1025 -m "Investigating network partition"
```

### 4.4 `oncall-master-supreme incident resolve`

Marks an incident as resolved.

**Usage:**
`oncall-master-supreme incident resolve <incident_id>... [flags]`

**Flags:**
- `--resolution, -r <string>`: (Required) A summary of how the incident was resolved.
- `--root-cause <string>`: The identified root cause, if known.

**Examples:**
```bash
# Resolve an incident
oncall-master-supreme incident resolve INC-1024 -r "Restarted the primary database node" --root-cause "OOM Killer terminated postgres process"
```

---

## 5. Schedule and Rotation Management

Managing who is on call and when is critical. These commands interface with the scheduling engine.

### 5.1 `oncall-master-supreme schedule view`

Displays the on-call schedule for a given team or service.

**Usage:**
`oncall-master-supreme schedule view <schedule_id> [flags]`

**Flags:**
- `--start <date>`: The start date for the view (format: YYYY-MM-DD). Default is today.
- `--end <date>`: The end date for the view. Default is 7 days from the start date.
- `--user <user_id>`: Highlight shifts for a specific user.

**Examples:**
```bash
# View the schedule for the next week
oncall-master-supreme schedule view SCHED-DBA

# View the schedule for a specific month
oncall-master-supreme schedule view SCHED-DBA --start 2023-10-01 --end 2023-10-31
```

### 5.2 `oncall-master-supreme schedule override`

Creates a temporary override in the schedule, replacing one user with another.

**Usage:**
`oncall-master-supreme schedule override <schedule_id> [flags]`

**Flags:**
- `--user <user_id>`: (Required) The user who will take the shift.
- `--start <datetime>`: (Required) The start time of the override (ISO 8601 format).
- `--end <datetime>`: (Required) The end time of the override.

**Examples:**
```bash
# Override a shift for the weekend
oncall-master-supreme schedule override SCHED-DBA --user alice.smith --start "2023-10-14T00:00:00Z" --end "2023-10-16T00:00:00Z"
```

---

## 6. Escalation Policies

Escalation policies define the path an alert takes if it is not acknowledged promptly.

### 6.1 `oncall-master-supreme policy list`

Lists all configured escalation policies.

**Usage:**
`oncall-master-supreme policy list [flags]`

**Flags:**
- `--search <string>`: Filter policies by name.

**Examples:**
```bash
oncall-master-supreme policy list --search "Tier 1"
```

### 6.2 `oncall-master-supreme policy update`

Modifies an existing escalation policy. This is a complex command that often requires passing a JSON or YAML definition.

**Usage:**
`oncall-master-supreme policy update <policy_id> [flags]`

**Flags:**
- `--file, -f <path>`: Path to a YAML or JSON file containing the new policy definition.

**Examples:**
```bash
# Update a policy using a local file
oncall-master-supreme policy update POL-998 --file new_policy.yaml
```

---

## 7. Service and Integration Management

Services represent the logical components of your infrastructure, and integrations are how external tools (like Prometheus, Datadog, or New Relic) send alerts to `oncall-master-supreme`.

### 7.1 `oncall-master-supreme service create`

Registers a new service in the system.

**Usage:**
`oncall-master-supreme service create [flags]`

**Flags:**
- `--name, -n <string>`: (Required) The name of the service.
- `--description <string>`: A description of what the service does.
- `--policy <policy_id>`: (Required) The default escalation policy for this service.

**Examples:**
```bash
oncall-master-supreme service create --name "User Authentication API" --policy POL-123
```

### 7.2 `oncall-master-supreme integration add`

Adds a new inbound integration to a service.

**Usage:**
`oncall-master-supreme integration add <service_id> [flags]`

**Flags:**
- `--type <string>`: (Required) The type of integration (e.g., `prometheus`, `datadog`, `generic-webhook`).
- `--name <string>`: A human-readable name for the integration.

**Examples:**
```bash
# Add a Datadog integration
oncall-master-supreme integration add SVC-456 --type datadog --name "Datadog Primary"
```

---

## 8. User and Team Administration

Commands for managing the personnel who use the system.

### 8.1 `oncall-master-supreme user invite`

Invites a new user to the platform.

**Usage:**
`oncall-master-supreme user invite [flags]`

**Flags:**
- `--email <string>`: (Required) The email address of the user.
- `--role <string>`: The role to assign (e.g., `admin`, `responder`, `observer`). Default is `responder`.
- `--team <team_id>`: Automatically add the user to a specific team.

**Examples:**
```bash
oncall-master-supreme user invite --email "new.hire@company.com" --role responder --team TEAM-ENG
```

### 8.2 `oncall-master-supreme team create`

Creates a new team.

**Usage:**
`oncall-master-supreme team create [flags]`

**Flags:**
- `--name <string>`: (Required) The name of the team.
- `--description <string>`: A description of the team's responsibilities.

**Examples:**
```bash
oncall-master-supreme team create --name "Platform Engineering"
```

---

## 9. Advanced Diagnostics and Maintenance

These commands are typically used by administrators to troubleshoot the CLI itself or perform bulk operations.

### 9.1 `oncall-master-supreme ping`

Tests connectivity to the backend API and verifies authentication.

**Usage:**
`oncall-master-supreme ping`

**Examples:**
```bash
$ oncall-master-supreme ping
OK: Connected to API at https://api.oncall-master.com (Latency: 45ms)
Authentication: Valid (User: admin@company.com)
```

### 9.2 `oncall-master-supreme export`

Exports system configuration (services, policies, schedules) for backup or migration purposes.

**Usage:**
`oncall-master-supreme export [flags]`

**Flags:**
- `--resource <type>`: The type of resource to export (e.g., `services`, `policies`, `all`). Default is `all`.
- `--dir <path>`: The directory to save the exported files. Default is the current directory.

**Examples:**
```bash
# Export all configurations to a backup directory
oncall-master-supreme export --resource all --dir ./oncall_backup_2023
```

### 9.3 `oncall-master-supreme logs`

Fetches audit logs for the system.

**Usage:**
`oncall-master-supreme logs [flags]`

**Flags:**
- `--since <duration>`: Timeframe for logs.
- `--action <string>`: Filter by specific actions (e.g., `incident.resolve`, `user.login`).

**Examples:**
```bash
# View all incident resolution logs from the last 24 hours
oncall-master-supreme logs --action incident.resolve --since 24h
```

---

## 10. Best Practices and Workflows

To maximize the effectiveness of the `oncall-master-supreme` CLI, consider the following best practices:

1.  **Use Profiles:** If you manage multiple environments (e.g., staging and production), use the `--profile` flag to switch contexts easily without constantly changing API keys.
2.  **Scripting:** The `-o json` flag is invaluable when writing bash or python scripts that wrap the CLI. You can pipe the output to `jq` for complex parsing.
3.  **Aliases:** Create shell aliases for common commands. For example, `alias ack='oncall-master-supreme incident ack'` can save precious seconds during a critical outage.
4.  **Dry Runs:** Always use `--dry-run` when updating complex escalation policies or schedules via file uploads to catch syntax errors before they affect production routing.

## 11. Conclusion

The `oncall-master-supreme` CLI is a powerful tool that brings the full capabilities of the on-call management platform directly to your terminal. By mastering these commands, SREs and operators can significantly reduce mean time to acknowledgment (MTTA) and mean time to resolution (MTTR), ensuring higher reliability and smoother operations for all managed services.

---

## 12. Detailed Command Reference (Extended)

This section provides an even deeper dive into edge cases, advanced flag combinations, and troubleshooting for specific commands.

### 12.1 Advanced Incident Filtering

When dealing with a massive outage, the `incident list` command can be overwhelming. Utilizing advanced filtering is crucial.

**Combining Filters:**
You can combine multiple filters to pinpoint specific issues. For example, to find all critical incidents assigned to the database team that have been acknowledged but not resolved for over 2 hours:

```bash
oncall-master-supreme incident list \
  --severity SEV-1 \
  --team TEAM-DBA \
  --status acknowledged \
  --older-than 2h
```

**Output Formatting with jq:**
When integrating with other tools, JSON output is essential. Here is an example of extracting just the incident IDs and titles using `jq`:

```bash
oncall-master-supreme incident list -o json | jq '.[] | {id: .id, title: .title}'
```

### 12.2 Complex Schedule Overrides

Overrides can sometimes overlap or conflict. The CLI handles these gracefully, but understanding the precedence is important.

**Override Precedence:**
1.  Manual overrides created via the CLI or UI always take precedence over the base schedule.
2.  If two manual overrides overlap, the most recently created override takes precedence for the overlapping duration.

**Removing an Override:**
If an override was created in error, it can be removed using the `schedule override delete` command.

```bash
oncall-master-supreme schedule override delete <override_id>
```

### 12.3 Webhook Integration Management

Generic webhooks are the most flexible way to integrate custom internal tools with `oncall-master-supreme`.

**Creating a Webhook Integration:**
```bash
oncall-master-supreme integration add SVC-123 --type generic-webhook --name "Internal CI/CD Monitor"
```
This command will output a unique Webhook URL. You must configure your internal tool to send POST requests to this URL with a specific JSON payload structure.

**Testing a Webhook:**
You can simulate an incoming webhook payload using the `integration test` command:

```bash
oncall-master-supreme integration test <integration_id> --payload ./test_payload.json
```

### 12.4 Audit and Compliance

For enterprise environments, auditing who did what and when is a strict requirement. The `logs` command provides access to this immutable audit trail.

**Exporting Audit Logs for Compliance:**
To export logs for a compliance review (e.g., SOC2), you can output the logs to a CSV format (if supported by your configuration) or JSON for ingestion into a SIEM like Splunk.

```bash
oncall-master-supreme logs --since 30d --format json > audit_logs_last_30_days.json
```

## 13. Troubleshooting the CLI

Even the best tools encounter issues. Here are common problems and how to resolve them.

**Error: "Authentication Failed"**
- **Cause:** Your API key has expired, or your SSO session has timed out.
- **Solution:** Run `oncall-master-supreme login` again. If using an API key, verify it is still active in the web UI.

**Error: "Resource Not Found"**
- **Cause:** You are referencing an ID (like a Service ID or Policy ID) that does not exist or you do not have permission to view.
- **Solution:** Double-check the ID. Use the corresponding `list` command (e.g., `service list`) to verify the ID.

**Error: "Rate Limit Exceeded"**
- **Cause:** You are making too many API requests in a short period. This often happens when running poorly optimized scripts.
- **Solution:** Implement exponential backoff in your scripts. The CLI will automatically retry on 429 errors if the `--retry` flag is enabled (default is true), but sustained high volume will still fail.

## 14. API Rate Limits and Quotas

The `oncall-master-supreme` backend enforces rate limits to ensure stability. The CLI respects these limits and provides headers in verbose mode to help you track your usage.

- **Standard Tier:** 100 requests per minute.
- **Enterprise Tier:** 1000 requests per minute.

If you consistently hit rate limits, consider using the bulk operations API endpoints (currently only available via direct HTTP requests, not the CLI) or contacting support to increase your quota.

## 15. Contributing and Extending

The `oncall-master-supreme` CLI is open-source (internal to the company). Contributions are welcome.

**Building from Source:**
1. Clone the repository.
2. Run `make build`.
3. The binary will be located in the `bin/` directory.

**Adding a New Command:**
Commands are structured using the Cobra framework in Go. To add a new command, create a new file in the `cmd/` directory and register it with the root command. Ensure you add comprehensive unit tests and update this documentation.

---

## Appendix A: Full Command Index

This appendix provides a quick alphabetical reference to all commands available in the CLI.

- `ack`: Acknowledge an incident.
- `configure`: Set up the CLI configuration.
- `export`: Export system configuration.
- `incident ack`: Acknowledge an incident.
- `incident create`: Trigger a new incident.
- `incident list`: List incidents.
- `incident resolve`: Resolve an incident.
- `integration add`: Add an integration to a service.
- `integration test`: Test an integration payload.
- `login`: Authenticate with the server.
- `logs`: View audit logs.
- `ping`: Test connectivity.
- `policy list`: List escalation policies.
- `policy update`: Update an escalation policy.
- `schedule override`: Create a schedule override.
- `schedule override delete`: Remove a schedule override.
- `schedule view`: View a schedule.
- `service create`: Create a new service.
- `team create`: Create a new team.
- `user invite`: Invite a new user.

## Appendix B: Glossary of Terms

- **Incident:** An event that requires attention from an on-call responder.
- **Service:** A logical component of your infrastructure (e.g., "Database", "Payment API").
- **Escalation Policy:** A set of rules defining who should be notified when an incident occurs and how the notification should escalate if not acknowledged.
- **Schedule:** A calendar defining who is on call at any given time.
- **Override:** A temporary change to a schedule.
- **Integration:** A connection between an external tool and `oncall-master-supreme`.
- **MTTA:** Mean Time To Acknowledgment.
- **MTTR:** Mean Time To Resolution.

## Appendix C: Example Workflows

### Workflow 1: Onboarding a New Service

1. Create the team: `oncall-master-supreme team create --name "Search Team"`
2. Invite users: `oncall-master-supreme user invite --email "bob@example.com" --team TEAM-SEARCH`
3. Create an escalation policy (via UI or API, then get ID).
4. Create the service: `oncall-master-supreme service create --name "Elasticsearch Cluster" --policy POL-SEARCH`
5. Add an integration: `oncall-master-supreme integration add SVC-ES --type datadog --name "DD Monitors"`

### Workflow 2: Handling a Major Outage

1. Acknowledge the page: `oncall-master-supreme incident ack INC-999`
2. Check related incidents: `oncall-master-supreme incident list --service SVC-ES --status triggered`
3. Escalate if necessary (using UI or specific API call).
4. Resolve when fixed: `oncall-master-supreme incident resolve INC-999 -r "Increased heap size and restarted nodes"`

By following these workflows and utilizing the full power of the CLI, teams can manage their on-call responsibilities with unprecedented efficiency and control.
