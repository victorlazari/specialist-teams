# Manus Workflows CLI Reference

## Introduction

The Manus Workflows CLI is a command-line interface tool designed to streamline and automate workflow management within the Manus ecosystem. It offers a comprehensive suite of commands for initializing, running, managing, and monitoring workflows. This document serves as a comprehensive command reference, detailing every command, flag, and argument available in the Manus Workflows CLI. Whether you are a new user or an experienced engineer, this guide provides the in-depth knowledge necessary to effectively use the CLI in diverse scenarios.

## Installation & Setup

Before using the Manus Workflows CLI, ensure that you have installed the tool correctly and configured your environment to interact with the Manus platform.

### Prerequisites

- **Operating System**: Compatible with Windows, macOS, and Linux.
- **Node.js**: Version 14.0 or higher.
- **Network Access**: Ensure access to the Manus API endpoint.

### Installation

To install the Manus Workflows CLI, use npm, the Node.js package manager:

```bash
npm install -g @manus/workflows-cli
```

### Setup

Configuration involves setting up authentication and default parameters:

1. **Authenticate**: Use your Manus API token to authenticate.

   ```bash
   manus-workflows login --token <YOUR_API_TOKEN>
   ```

2. **Set Default Parameters**:

   Configure default values for frequent parameters:

   ```bash
   manus-workflows config set default-project my-project
   manus-workflows config set default-region us-central1
   ```

## Global Flags

Global flags can be used with any command to modify their behavior. These flags are optional unless specified otherwise.

- `-h, --help`: Display help information about commands and flags.
- `-v, --verbose`: Enable verbose output for debugging.
- `--version`: Display the CLI version.
- `-p, --project`: Override the default project for the current command.
- `-r, --region`: Override the default region for the current command.

## Core Commands

### 1. `init`

Initialize a new workflow in your current directory.

#### Usage

```bash
manus-workflows init [options]
```

#### Options

- `--template <template-name>`: Specify a template for initializing the workflow.
- `-n, --name <workflow-name>`: Name for the new workflow.

#### Examples

- Initialize a workflow with a specific template:

  ```bash
  manus-workflows init --template basic
  ```

- Initialize with a custom name:

  ```bash
  manus-workflows init -n my-custom-workflow
  ```

### 2. `run`

Execute a workflow.

#### Usage

```bash
manus-workflows run <workflow-id> [options]
```

#### Options

- `--async`: Run the workflow asynchronously.
- `--input <file-path>`: Provide input parameters from a JSON file.

#### Examples

- Run a workflow synchronously:

  ```bash
  manus-workflows run 12345
  ```

- Run asynchronously with input parameters:

  ```bash
  manus-workflows run 12345 --async --input params.json
  ```

### 3. `list`

List all workflows in the current project.

#### Usage

```bash
manus-workflows list [options]
```

#### Options

- `--status <status>`: Filter workflows by status (e.g., running, completed).
- `--limit <number>`: Limit the number of workflows displayed.

#### Examples

- List all running workflows:

  ```bash
  manus-workflows list --status running
  ```

- List the latest 10 workflows:

  ```bash
  manus-workflows list --limit 10
  ```

### 4. `describe`

Get detailed information about a specific workflow.

#### Usage

```bash
manus-workflows describe <workflow-id>
```

#### Examples

- Describe a workflow by ID:

  ```bash
  manus-workflows describe 12345
  ```

### 5. `stop`

Stop a running workflow.

#### Usage

```bash
manus-workflows stop <workflow-id>
```

#### Examples

- Stop a workflow:

  ```bash
  manus-workflows stop 12345
  ```

### 6. `delete`

Delete a workflow.

#### Usage

```bash
manus-workflows delete <workflow-id>
```

#### Examples

- Delete a workflow:

  ```bash
  manus-workflows delete 12345
  ```

### 7. `logs`

Retrieve logs for a specific workflow.

#### Usage

```bash
manus-workflows logs <workflow-id>
```

#### Options

- `--tail`: Continuously stream logs.

#### Examples

- Retrieve logs for a workflow:

  ```bash
  manus-workflows logs 12345
  ```

- Stream logs in real-time:

  ```bash
  manus-workflows logs 12345 --tail
  ```

## Advanced Commands

### 1. `schedule`

Schedule a workflow to run at a specified time.

#### Usage

```bash
manus-workflows schedule <workflow-id> --cron <cron-expression>
```

#### Examples

- Schedule a workflow to run every day at midnight:

  ```bash
  manus-workflows schedule 12345 --cron "0 0 * * *"
  ```

### 2. `trigger`

Manually trigger a scheduled workflow.

#### Usage

```bash
manus-workflows trigger <workflow-id>
```

#### Examples

- Trigger a scheduled workflow:

  ```bash
  manus-workflows trigger 12345
  ```

### 3. `export`

Export a workflow definition to a file.

#### Usage

```bash
manus-workflows export <workflow-id> --output <file-path>
```

#### Examples

- Export a workflow to a JSON file:

  ```bash
  manus-workflows export 12345 --output workflow.json
  ```

### 4. `import`

Import a workflow definition from a file.

#### Usage

```bash
manus-workflows import --input <file-path>
```

#### Examples

- Import a workflow from a JSON file:

  ```bash
  manus-workflows import --input workflow.json
  ```

### 5. `validate`

Validate a workflow definition without executing it.

#### Usage

```bash
manus-workflows validate --input <file-path>
```

#### Examples

- Validate a workflow from a JSON file:

  ```bash
  manus-workflows validate --input workflow.json
  ```

## Configuration Management

The Manus Workflows CLI allows for comprehensive configuration management, enabling users to customize their workflow environment and settings.

### Configuration Files

Configuration files are stored in the user’s home directory under `.manus/config.json`. This file contains settings such as default project, region, and authentication tokens.

#### Managing Configuration

- **Set Configuration**:

  ```bash
  manus-workflows config set <key> <value>
  ```

  Example:

  ```bash
  manus-workflows config set default-region us-west1
  ```

- **Get Configuration**:

  ```bash
  manus-workflows config get <key>
  ```

  Example:

  ```bash
  manus-workflows config get default-project
  ```

- **List All Configurations**:

  ```bash
  manus-workflows config list
  ```

## Environment Variables

Environment variables can override default configurations and are beneficial for CI/CD pipelines and other automated environments.

### Supported Environment Variables

- `MANUS_API_TOKEN`: Overrides the API token used for authentication.
- `MANUS_DEFAULT_PROJECT`: Overrides the default project setting.
- `MANUS_DEFAULT_REGION`: Overrides the default region setting.

### Example Usage

To run a command with a specific API token:

```bash
export MANUS_API_TOKEN=your-token-here
manus-workflows list
```

## Detailed Examples and Use Cases

### Use Case 1: Automating Daily Reports

1. **Initialize a Workflow**:

   ```bash
   manus-workflows init --template daily-report -n daily-report-workflow
   ```

2. **Schedule the Workflow**:

   ```bash
   manus-workflows schedule 12345 --cron "0 6 * * *"
   ```

3. **Trigger the Workflow for Testing**:

   ```bash
   manus-workflows trigger 12345
   ```

### Use Case 2: Migrating Workflows Between Environments

1. **Export the Workflow Definition**:

   ```bash
   manus-workflows export 67890 --output prod-workflow.json
   ```

2. **Import the Workflow into a New Environment**:

   ```bash
   manus-workflows import --input prod-workflow.json
   ```

3. **Validate the Imported Workflow**:

   ```bash
   manus-workflows validate --input prod-workflow.json
   ```

## Troubleshooting Common CLI Errors

### Error: "Authentication Failed"

- **Cause**: Incorrect or missing API token.
- **Solution**: Re-authenticate using:

  ```bash
  manus-workflows login --token <YOUR_API_TOKEN>
  ```

### Error: "Workflow Not Found"

- **Cause**: Incorrect workflow ID.
- **Solution**: Verify the workflow ID and ensure it exists in the specified project and region.

### Error: "Insufficient Permissions"

- **Cause**: Lack of necessary permissions to execute a command.
- **Solution**: Ensure that your API token has the appropriate permissions for the operation.

### Error: "Invalid Cron Expression"

- **Cause**: Incorrectly formatted cron expression.
- **Solution**: Validate the cron expression using online tools or refer to cron documentation for correct syntax.