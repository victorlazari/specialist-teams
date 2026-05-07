# Jira Field Schemas CLI Reference

## Introduction

Jira Field Schemas are an integral part of the Jira ecosystem, allowing administrators to manage and configure how custom fields behave across various projects within Jira. The `jira-field-schemas` command-line interface (CLI) serves as a powerful tool for Jira administrators and developers to interact with Jira field schemas programmatically. This CLI allows for efficient management of field schemas, enabling automation, integration with CI/CD pipelines, and batch processing of schema configurations.

The `jira-field-schemas` CLI provides a comprehensive set of commands for initializing, listing, retrieving, updating, and managing field schemas. By using this CLI, users can perform complex operations with ease, ensuring consistency and accuracy in schema management across their Jira instance.

## Architecture

The `jira-field-schemas` CLI is built using a modular architecture that leverages the Jira REST API to perform operations on field schemas. The CLI is written in a language that supports cross-platform compatibility, ensuring that it can run on Windows, macOS, and Linux systems without modification.

### Key Components:

1. **Command Parser**: The command parser interprets user input, mapping commands and options to their respective functionalities. It ensures that the correct syntax is used and provides helpful error messages and suggestions.

2. **API Client**: The API client is responsible for interacting with the Jira REST API. It handles authentication, request formation, response parsing, and error handling. This component ensures that all interactions with Jira are secure and efficient.

3. **Configuration Manager**: This component manages user-specific configurations, such as authentication credentials, default server URLs, and other preferences. It ensures that users can seamlessly switch between different Jira instances and maintain consistent settings.

4. **Output Formatter**: The output formatter presents command results in a human-readable format, supporting various output formats such as JSON, XML, and plain text. This flexibility allows users to integrate command outputs into scripts and other tools easily.

## Installation

The `jira-field-schemas` CLI can be installed using package managers or by downloading the standalone binary from the official repository.

### Installation via Package Manager

#### On macOS:

```bash
brew install jira-field-schemas
```

#### On Linux (Debian-based):

```bash
sudo apt-get update
sudo apt-get install jira-field-schemas
```

#### On Windows:

The CLI can be installed using the Windows Package Manager (winget):

```powershell
winget install jira-field-schemas
```

### Installation via Standalone Binary

1. Download the latest release from the [official release page](https://example.com/jira-field-schemas/releases).
2. Extract the binary and place it in a directory included in your system's PATH.

### Verifying Installation

After installation, verify that the CLI is correctly installed by running:

```bash
jira-field-schemas --version
```

This command should return the version number of the installed CLI.

## Configuration

Before using the `jira-field-schemas` CLI, you need to configure it to connect to your Jira instance. The configuration involves setting up authentication credentials and specifying the Jira server URL.

### Setting Up Authentication

The CLI supports multiple authentication methods, including basic authentication and OAuth.

#### Basic Authentication

```bash
jira-field-schemas configure --username <your-username> --password <your-password>
```

#### OAuth

For OAuth, you need to provide consumer key, private key, and other OAuth-specific parameters:

```bash
jira-field-schemas configure --oauth --consumer-key <consumer-key> --private-key <path-to-private-key>
```

### Specifying Jira Server URL

Specify the URL of your Jira instance:

```bash
jira-field-schemas configure --server-url https://your-jira-instance.atlassian.net
```

### Verifying Configuration

To check your current configuration settings, use:

```bash
jira-field-schemas config view
```

This will display all current configuration settings, including authentication details and server URL.

## Commands

The `jira-field-schemas` CLI provides a range of commands to interact with Jira field schemas. Below are detailed explanations of the initial commands: `init`, `list`, and `get`.

### Command: init

The `init` command initializes a new field schema in your Jira instance.

#### Syntax:

```bash
jira-field-schemas init --name <schema-name> [--description <description>]
```

#### Options:

- `--name`: The name of the field schema you want to create. This is a required parameter.
- `--description`: A brief description of the field schema. This is an optional parameter.

#### Example:

```bash
jira-field-schemas init --name "Custom Issue Fields" --description "Schema for custom issue fields across projects."
```

This command initializes a new field schema named "Custom Issue Fields" with the given description.

### Command: list

The `list` command retrieves a list of all field schemas available in your Jira instance.

#### Syntax:

```bash
jira-field-schemas list [--filter <filter>] [--format <format>]
```

#### Options:

- `--filter`: Apply a filter to the list of field schemas, specifying criteria like name or project.
- `--format`: Specify the output format for the list. Options include `json`, `xml`, and `table`. The default is `table`.

#### Example:

```bash
jira-field-schemas list --filter "Custom" --format json
```

This command lists all field schemas containing "Custom" in their name, outputting the results in JSON format.

### Command: get

The `get` command retrieves detailed information about a specific field schema.

#### Syntax:

```bash
jira-field-schemas get --id <schema-id> [--format <format>]
```

#### Options:

- `--id`: The unique identifier of the field schema. This is a required parameter.
- `--format`: Specify the output format. Options include `json`, `xml`, and `table`. The default is `json`.

#### Example:

```bash
jira-field-schemas get --id 10101 --format xml
```

This command retrieves detailed information about the field schema with ID `10101`, outputting the results in XML format.

Each command is designed to provide flexibility and control over Jira field schemas, enabling administrators to manage their configurations effectively and efficiently.

### Command: `create`

The `create` command allows you to create a new field schema in JIRA. This command is useful for defining a new set of fields that can be associated with your JIRA projects or issue types.

```shell
jira-field-schemas create --name <schema-name> --description <schema-description> [options]
```

#### Flags and Arguments:

- `--name <schema-name>`: (Required) The name of the new field schema. Must be unique within your JIRA instance.
- `--description <schema-description>`: (Optional) A brief description of the field schema. Helps in identifying the purpose of the schema.
- `--copy-from <existing-schema-id>`: (Optional) Create a new schema by copying fields from an existing schema. The `<existing-schema-id>` must be a valid ID of an existing field schema.
- `--fields <field-id-list>`: (Optional) A comma-separated list of field IDs to include in the new schema. If not provided, the schema will be created without any fields.

#### Edge Cases:

- **Unique Name Constraint**: The command will fail if you try to create a schema with a name that already exists. Make sure the `<schema-name>` is unique.
- **Field ID Validation**: The command will check if each `field-id` in `<field-id-list>` exists. If any field ID is invalid, the command will not complete.
- **Schema Copy Validity**: If using `--copy-from`, the specified schema must exist and be accessible. Otherwise, the command will return an error.

#### Examples:

1. **Create a Basic Schema**:
   ```shell
   jira-field-schemas create --name "Bug Tracking Schema" --description "Schema for tracking bugs"
   ```

2. **Create a Schema by Copying an Existing One**:
   ```shell
   jira-field-schemas create --name "New Schema" --copy-from "12345"
   ```

3. **Create a Schema with Specific Fields**:
   ```shell
   jira-field-schemas create --name "Custom Schema" --fields "101,102,103"
   ```

### Command: `update`

The `update` command allows you to modify an existing field schema. You can change its name, description, or the fields it includes.

```shell
jira-field-schemas update --id <schema-id> [options]
```

#### Flags and Arguments:

- `--id <schema-id>`: (Required) The ID of the field schema to update.
- `--name <new-name>`: (Optional) New name for the field schema. Must be unique.
- `--description <new-description>`: (Optional) New description for the field schema.
- `--add-fields <field-id-list>`: (Optional) Add fields to the schema. Provide a comma-separated list of field IDs.
- `--remove-fields <field-id-list>`: (Optional) Remove fields from the schema. Provide a comma-separated list of field IDs.

#### Edge Cases:

- **Non-Existent Schema**: The command will fail if the `<schema-id>` does not correspond to an existing schema.
- **Unique Name Constraint**: If changing the schema name, ensure the new name does not clash with existing schema names.
- **Field ID Validation**: The command checks all field IDs in `--add-fields` and `--remove-fields`. Invalid field IDs will cause the command to fail.

#### Examples:

1. **Update Schema Name and Description**:
   ```shell
   jira-field-schemas update --id "12345" --name "Updated Schema Name" --description "Updated description"
   ```

2. **Add Fields to a Schema**:
   ```shell
   jira-field-schemas update --id "12345" --add-fields "104,105"
   ```

3. **Remove Fields from a Schema**:
   ```shell
   jira-field-schemas update --id "12345" --remove-fields "101,102"
   ```

### Command: `delete`

The `delete` command allows you to remove an existing field schema. Use this command with caution, as deleting a schema may affect projects associated with it.

```shell
jira-field-schemas delete --id <schema-id> [options]
```

#### Flags and Arguments:

- `--id <schema-id>`: (Required) The ID of the field schema to delete.
- `--force`: (Optional) Force deletion without confirmation prompt.

#### Edge Cases:

- **Non-Existent Schema**: If the `<schema-id>` does not exist, the command will fail.
- **Dependency Check**: The command will check if the schema is associated with any projects or issue types. If so, it may prevent deletion unless `--force` is used.

#### Examples:

1. **Delete a Schema with Confirmation**:
   ```shell
   jira-field-schemas delete --id "12345"
   ```

2. **Force Delete a Schema**:
   ```shell
   jira-field-schemas delete --id "12345" --force
   ```

### Command: `assign`

The `assign` command allows you to associate a field schema with a project or issue type.

```shell
jira-field-schemas assign --schema-id <schema-id> --project-id <project-id> [--issue-type-id <issue-type-id>]
```

#### Flags and Arguments:

- `--schema-id <schema-id>`: (Required) The ID of the field schema to assign.
- `--project-id <project-id>`: (Required) The ID of the project to which the schema will be assigned.
- `--issue-type-id <issue-type-id>`: (Optional) The ID of the issue type to which the schema will be assigned. If omitted, the schema is assigned to all issue types in the project.

#### Edge Cases:

- **Non-Existent Schema or Project**: The command will fail if either `<schema-id>` or `<project-id>` does not exist.
- **Issue Type Validation**: If `--issue-type-id` is provided, it must be valid for the specified project. Invalid issue type IDs will cause the command to fail.

#### Examples:

1. **Assign a Schema to a Project**:
   ```shell
   jira-field-schemas assign --schema-id "12345" --project-id "54321"
   ```

2. **Assign a Schema to a Specific Issue Type**:
   ```shell
   jira-field-schemas assign --schema-id "12345" --project-id "54321" --issue-type-id "98765"
   ```

### Command: `unassign`

The `unassign` command removes the association of a field schema from a project or issue type.

```shell
jira-field-schemas unassign --schema-id <schema-id> --project-id <project-id> [--issue-type-id <issue-type-id>]
```

#### Flags and Arguments:

- `--schema-id <schema-id>`: (Required) The ID of the field schema to unassign.
- `--project-id <project-id>`: (Required) The ID of the project from which the schema will be unassigned.
- `--issue-type-id <issue-type-id>`: (Optional) The ID of the issue type from which the schema will be unassigned. If omitted, the schema is unassigned from all issue types in the project.

#### Edge Cases:

- **Non-Existent Schema or Project**: The command will fail if either `<schema-id>` or `<project-id>` does not exist.
- **Issue Type Validation**: If `--issue-type-id` is provided, it must be valid and currently associated with the specified schema and project.

#### Examples:

1. **Unassign a Schema from a Project**:
   ```shell
   jira-field-schemas unassign --schema-id "12345" --project-id "54321"
   ```

2. **Unassign a Schema from a Specific Issue Type**:
   ```shell
   jira-field-schemas unassign --schema-id "12345" --project-id "54321" --issue-type-id "98765"
   ```

## Advanced Usage

### Customizing Field Schemas

When working with `jira-field-schemas`, advanced users can customize field schemas to tailor them to specific project needs. This involves modifying existing schemas or creating new ones to accommodate unique field configurations. For example, if you require a custom field for tracking additional data specific to your project, you can add this field to a new or existing schema.

#### Steps to Customize Field Schemas:

1. **Identify Requirements:** Determine what additional fields or modifications are necessary for your project. This might include custom fields, changing field types, or altering field behaviors.

2. **Clone an Existing Schema:** Use the command `jira-field-schemas clone <existing-schema-id> <new-schema-name>` to create a copy of an existing schema. This is useful for using an existing configuration as a base.

3. **Modify the Schema:**
   - Use the command `jira-field-schemas add-field <schema-id> <field-id>` to add new fields.
   - To change the configuration of a field, use `jira-field-schemas update-field <schema-id> <field-id> --type <new-type> --options <options>`.
   - To remove a field, use `jira-field-schemas remove-field <schema-id> <field-id>`.

4. **Assign the Schema:** Use `jira-field-schemas assign <schema-id> <project-id>` to apply your newly customized schema to the desired project.

### Automating Field Schema Management

For organizations managing multiple projects with varying requirements, automating schema management can save significant time. Use scripts to automate repetitive tasks such as creating, modifying, and assigning field schemas.

#### Example Automation Script:

```bash
#!/bin/bash

# Variables
PROJECT_ID="PROJ123"
BASE_SCHEMA_ID="10001"
NEW_SCHEMA_NAME="Custom Project Schema"

# Clone the existing schema
NEW_SCHEMA_ID=$(jira-field-schemas clone $BASE_SCHEMA_ID $NEW_SCHEMA_NAME)

# Add custom fields
jira-field-schemas add-field $NEW_SCHEMA_ID "customfield_101"
jira-field-schemas add-field $NEW_SCHEMA_ID "customfield_102"

# Update field configurations
jira-field-schemas update-field $NEW_SCHEMA_ID "customfield_101" --type "Text" --options "multiline"

# Assign the new schema to the project
jira-field-schemas assign $NEW_SCHEMA_ID $PROJECT_ID

echo "Schema $NEW_SCHEMA_NAME has been configured and assigned to project $PROJECT_ID."
```

## Scripting

Scripting with `jira-field-schemas` involves using shell scripts or other scripting languages to automate the execution of CLI commands. This is particularly useful for batch operations or integrating schema management into larger automated workflows.

### Key Considerations

- **Environment Setup:** Ensure that your scripting environment can execute CLI commands. Install necessary dependencies and ensure access permissions are correctly configured.
- **Error Handling:** Implement robust error handling to manage command execution failures. Use conditional checks and logging to capture and address issues.
- **Parameterization:** Use variables to make your scripts flexible and reusable across different projects or environments.

### Example Script for Bulk Schema Assignment

```bash
#!/bin/bash

# Assign a field schema to multiple projects

SCHEMA_ID="20001"
PROJECT_IDS=("PROJ1" "PROJ2" "PROJ3")

for PROJECT_ID in "${PROJECT_IDS[@]}"; do
  echo "Assigning schema $SCHEMA_ID to project $PROJECT_ID..."
  if jira-field-schemas assign $SCHEMA_ID $PROJECT_ID; then
    echo "Successfully assigned schema to $PROJECT_ID."
  else
    echo "Failed to assign schema to $PROJECT_ID. Check logs for details." >&2
  fi
done
```

## Troubleshooting

When using `jira-field-schemas`, you may encounter issues related to command execution, schema configurations, or permission restrictions. Below are common troubleshooting steps and solutions.

### Common Issues

1. **Permission Denied:**
   - **Cause:** Lack of sufficient permissions to execute commands or modify schemas.
   - **Solution:** Ensure your user account has the necessary permissions in Jira. Consult your Jira administrator if needed.

2. **Invalid Schema ID or Field ID:**
   - **Cause:** Typographical errors or non-existent IDs.
   - **Solution:** Double-check the IDs for accuracy. Use `jira-field-schemas list` to verify existing IDs.

3. **Command Not Found:**
   - **Cause:** CLI tool is not installed or not in the PATH.
   - **Solution:** Ensure the CLI tool is correctly installed and the binary is in your system's PATH.

4. **API Rate Limits:**
   - **Cause:** Excessive API requests in a short period.
   - **Solution:** Implement rate limiting in your scripts or contact your Jira administrator to adjust API limits.

### Debugging Tips

- **Verbose Logging:** Use the `--verbose` flag with your CLI commands to get detailed output, which can help identify where the error occurs.
- **Logs:** Check Jira server logs for any server-side issues that might be affecting the CLI operations.
- **Test Environment:** Reproduce issues in a test environment to avoid affecting production data while troubleshooting.

## Best Practices

To maximize efficiency and maintainability when using `jira-field-schemas`, adhere to the following best practices:

### Version Control

- **Track Changes:** Use a version control system like Git to track changes in your schema scripts. This allows you to revert to previous versions if needed.
- **Document Changes:** Maintain a changelog documenting schema changes, reasons, and the expected impact on projects.

### Consistent Naming Conventions

- **Schema Naming:** Use descriptive and consistent naming conventions for schemas to easily identify their purpose and associated projects.
- **Field Naming:** Ensure custom fields have clear, descriptive names that reflect their usage or content.

### Regular Audits

- **Schema Review:** Periodically review field schemas to ensure they are still meeting project needs and remove any deprecated fields.
- **Field Usage:** Monitor field usage to identify fields that are rarely used and consider removing them to streamline schemas.

### Security and Permissions

- **Access Control:** Limit schema modification permissions to a select group of administrators to prevent unauthorized changes.
- **Audit Logs:** Regularly review audit logs to monitor changes made to schemas and ensure compliance with organizational policies.

By following these best practices and utilizing advanced usage techniques, scripting, and troubleshooting strategies, you can effectively manage Jira field schemas to suit your organization's specific needs.