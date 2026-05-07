# Jira Status Workflows CLI Reference

## Overview
The `jira-status-workflows` CLI is a powerful command-line interface designed to manage, audit, and automate Jira status transitions and workflow configurations. This comprehensive reference details every command, flag, argument, and provides extensive examples for enterprise usage.

## Global Flags
These flags can be used with any command in the CLI:
- `--config, -c <path>`: Path to a custom configuration file (default: `~/.jira-status-workflows.yaml`)
- `--profile, -p <name>`: Use a specific authentication profile
- `--verbose, -v`: Enable verbose logging for debugging
- `--output, -o <format>`: Output format (`json`, `yaml`, `table`, `csv`)
- `--dry-run`: Simulate the command without making actual changes

## Commands

### 1. `auth`
Manage authentication profiles for different Jira instances.

#### `auth login`
Authenticate with a Jira instance.
**Arguments:**
- `<url>`: The base URL of the Jira instance.
**Flags:**
- `--token <token>`: API token for authentication.
- `--user <email>`: User email address.
**Example:**
```bash
jira-status-workflows auth login https://company.atlassian.net --user admin@company.com --token $JIRA_TOKEN
```

#### `auth status`
Check the current authentication status.
**Example:**
```bash
jira-status-workflows auth status --profile production
```

### 2. `workflow`
Manage and inspect Jira workflows.

#### `workflow list`
List all available workflows in the instance.
**Flags:**
- `--project <key>`: Filter workflows by project key.
- `--active`: Only show active workflows.
**Example:**
```bash
jira-status-workflows workflow list --project ENG --active --output table
```

#### `workflow export`
Export a workflow configuration to a file.
**Arguments:**
- `<workflow-name>`: The exact name of the workflow.
**Flags:**
- `--file <path>`: Output file path.
- `--format <format>`: Export format (`xml`, `json`).
**Example:**
```bash
jira-status-workflows workflow export "Software Simplified Workflow" --file ./workflows/software.json --format json
```

#### `workflow import`
Import a workflow configuration from a file.
**Arguments:**
- `<file-path>`: Path to the workflow configuration file.
**Flags:**
- `--overwrite`: Overwrite existing workflow with the same name.
**Example:**
```bash
jira-status-workflows workflow import ./workflows/software.json --overwrite --dry-run
```

### 3. `status`
Manage individual statuses within workflows.

#### `status list`
List all statuses available in the Jira instance.
**Flags:**
- `--category <category>`: Filter by status category (`To Do`, `In Progress`, `Done`).
**Example:**
```bash
jira-status-workflows status list --category "In Progress"
```

#### `status create`
Create a new status.
**Arguments:**
- `<name>`: Name of the new status.
**Flags:**
- `--description <text>`: Description of the status.
- `--category <category>`: Status category.
**Example:**
```bash
jira-status-workflows status create "Awaiting QA" --description "Ready for testing" --category "In Progress"
```

### 4. `transition`
Manage and execute workflow transitions.

#### `transition list`
List available transitions for a specific issue.
**Arguments:**
- `<issue-key>`: The Jira issue key (e.g., ENG-123).
**Example:**
```bash
jira-status-workflows transition list ENG-123
```

#### `transition execute`
Execute a transition on an issue.
**Arguments:**
- `<issue-key>`: The Jira issue key.
- `<transition-id>`: The ID of the transition to execute.
**Flags:**
- `--comment <text>`: Add a comment during the transition.
- `--resolution <name>`: Set a resolution (if required by the transition).
**Example:**
```bash
jira-status-workflows transition execute ENG-123 41 --comment "Code review passed, moving to QA."
```

### 5. `audit`
Audit workflow configurations for best practices and errors.

#### `audit run`
Run an audit on all workflows or a specific project.
**Flags:**
- `--project <key>`: Project to audit.
- `--rules <file>`: Custom ruleset file for the audit.
**Example:**
```bash
jira-status-workflows audit run --project ENG --rules ./audit-rules.yaml --output json > audit-report.json
```

## Environment Variables
- `JIRA_API_TOKEN`: Default API token for authentication.
- `JIRA_BASE_URL`: Default Jira instance URL.
- `JIRA_USER_EMAIL`: Default user email.

## Advanced Usage Examples

### Bulk Transitioning Issues
You can combine commands with standard Unix tools to perform bulk operations.
```bash
# Find all issues in "Code Review" and transition them to "QA"
jira-status-workflows search "project = ENG AND status = 'Code Review'" --output json | jq -r '.[].key' | xargs -I {} jira-status-workflows transition execute {} 51 --comment "Bulk transition to QA"
```

### Automated Workflow Backup
```bash
# Backup all active workflows to a directory
jira-status-workflows workflow list --active --output json | jq -r '.[].name' | while read workflow; do
  jira-status-workflows workflow export "$workflow" --file "./backups/${workflow// /_}.json"
done
```

## Troubleshooting
If you encounter issues:
1. Run the command with `--verbose` to see detailed API requests and responses.
2. Ensure your API token has the necessary permissions (Jira Administrator rights are required for workflow modifications).
3. Check the `~/.jira-status-workflows/logs/` directory for detailed error logs.