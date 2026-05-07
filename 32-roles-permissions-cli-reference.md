# Roles and Permissions CLI Command Reference

## 1. Introduction

Welcome to the comprehensive Command Line Interface (CLI) reference for the Roles and Permissions management system. This document provides an exhaustive guide to all available commands, flags, arguments, and usage examples for managing roles, permissions, policies, and access control lists (ACLs) within your enterprise environment.

The Roles and Permissions CLI (`rp-cli`) is a powerful tool designed for system administrators, security engineers, and DevOps professionals to automate and manage access control at scale. It interacts directly with the Identity and Access Management (IAM) backend, ensuring that all changes are propagated securely and efficiently.

This guide is structured to cover everything from basic authentication to advanced policy management, troubleshooting, and best practices. Whether you are auditing current access levels or deploying new security models, this reference will serve as your definitive resource.

## 2. Global Flags and Configuration

Before diving into specific commands, it is essential to understand the global flags and configuration options available in `rp-cli`. These flags can be applied to almost any command to modify its behavior, output format, or execution context.

### 2.1 Global Flags

- `--config, -c <path>`: Specify a custom configuration file path. Default is `~/.rp-cli/config.yaml`.
- `--profile, -p <name>`: Use a specific profile from the configuration file. Useful for managing multiple environments (e.g., `dev`, `staging`, `prod`).
- `--region, -r <region>`: Specify the region for the API endpoint. Overrides the profile setting.
- `--output, -o <format>`: Define the output format. Supported formats: `json`, `yaml`, `table`, `text`. Default is `table`.
- `--verbose, -v`: Enable verbose logging. Useful for debugging.
- `--debug`: Enable debug-level logging, including raw API requests and responses.
- `--dry-run`: Simulate the command execution without making any actual changes to the backend.
- `--help, -h`: Display help information for the current command or subcommand.

### 2.2 Environment Variables

`rp-cli` also respects several environment variables, which can be used in CI/CD pipelines or automated scripts:

- `RP_CLI_TOKEN`: The authentication token for API access.
- `RP_CLI_PROFILE`: The default profile to use.
- `RP_CLI_REGION`: The default region.
- `RP_CLI_TIMEOUT`: The API request timeout in seconds (default: 30).

## 3. Authentication Commands

Authentication is the first step to using `rp-cli`. The CLI supports multiple authentication methods, including API tokens, OAuth2, and SSO integration.

### 3.1 `rp-cli auth login`

Authenticates the user and stores the session token locally.

**Usage:**
```bash
rp-cli auth login [flags]
```

**Flags:**
- `--method <method>`: Authentication method (`token`, `oauth2`, `sso`). Default is `oauth2`.
- `--token <token>`: Provide the token directly (useful for scripts).

**Examples:**
```bash
# Interactive OAuth2 login
rp-cli auth login

# Login using a specific token
rp-cli auth login --method token --token "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

### 3.2 `rp-cli auth logout`

Logs out the current user and clears the local session token.

**Usage:**
```bash
rp-cli auth logout [flags]
```

**Flags:**
- `--all`: Log out of all profiles.

**Examples:**
```bash
rp-cli auth logout
rp-cli auth logout --all
```

### 3.3 `rp-cli auth status`

Displays the current authentication status, including the active profile, user identity, and token expiration.

**Usage:**
```bash
rp-cli auth status
```

## 4. Role Management Commands

Roles are collections of permissions that can be assigned to users or groups. The following commands allow you to create, read, update, and delete roles.

### 4.1 `rp-cli role create`

Creates a new role in the system.

**Usage:**
```bash
rp-cli role create <role-name> [flags]
```

**Arguments:**
- `<role-name>`: The unique identifier for the role (e.g., `admin`, `developer`, `viewer`).

**Flags:**
- `--description, -d <text>`: A human-readable description of the role.
- `--permissions, -p <list>`: A comma-separated list of permission IDs to attach to the role.
- `--tags, -t <key=value>`: Key-value pairs for resource tagging.

**Examples:**
```bash
# Create a basic role
rp-cli role create developer --description "Standard developer access"

# Create a role with specific permissions and tags
rp-cli role create db-admin \
  --description "Database Administrator" \
  --permissions "db:read,db:write,db:delete" \
  --tags "department=engineering,env=prod"
```

### 4.2 `rp-cli role list`

Lists all available roles in the system.

**Usage:**
```bash
rp-cli role list [flags]
```

**Flags:**
- `--limit <number>`: Maximum number of roles to return (default: 50).
- `--offset <number>`: Pagination offset.
- `--filter <expression>`: Filter roles based on specific criteria (e.g., `name=admin*`).

**Examples:**
```bash
# List all roles
rp-cli role list

# List roles with a specific prefix
rp-cli role list --filter "name=dev-*"
```

### 4.3 `rp-cli role get`

Retrieves detailed information about a specific role.

**Usage:**
```bash
rp-cli role get <role-name> [flags]
```

**Examples:**
```bash
rp-cli role get developer --output json
```

### 4.4 `rp-cli role update`

Updates an existing role's properties.

**Usage:**
```bash
rp-cli role update <role-name> [flags]
```

**Flags:**
- `--description, -d <text>`: Update the description.
- `--add-permissions <list>`: Add permissions to the role.
- `--remove-permissions <list>`: Remove permissions from the role.

**Examples:**
```bash
rp-cli role update developer --add-permissions "repo:create,repo:delete"
```

### 4.5 `rp-cli role delete`

Deletes a role from the system.

**Usage:**
```bash
rp-cli role delete <role-name> [flags]
```

**Flags:**
- `--force, -f`: Bypass the confirmation prompt.

**Examples:**
```bash
rp-cli role delete old-role --force
```

## 5. Permission Management Commands

Permissions define specific actions that can be performed on resources. While permissions are usually predefined by the system, you can list and inspect them.

### 5.1 `rp-cli permission list`

Lists all available permissions.

**Usage:**
```bash
rp-cli permission list [flags]
```

**Flags:**
- `--resource <type>`: Filter permissions by resource type (e.g., `database`, `repository`).

**Examples:**
```bash
rp-cli permission list --resource database
```

### 5.2 `rp-cli permission get`

Retrieves details about a specific permission.

**Usage:**
```bash
rp-cli permission get <permission-id>
```

**Examples:**
```bash
rp-cli permission get db:write
```

## 6. Policy Management Commands

Policies are advanced constructs that define complex access rules, often involving conditions (e.g., time of day, IP address).

### 6.1 `rp-cli policy create`

Creates a new policy from a JSON or YAML file.

**Usage:**
```bash
rp-cli policy create [flags]
```

**Flags:**
- `--file, -f <path>`: Path to the policy definition file.

**Examples:**
```bash
rp-cli policy create --file ./strict-access-policy.json
```

### 6.2 `rp-cli policy attach`

Attaches a policy to a role, user, or group.

**Usage:**
```bash
rp-cli policy attach <policy-id> [flags]
```

**Flags:**
- `--role <role-name>`: Attach to a role.
- `--user <user-id>`: Attach to a user.
- `--group <group-id>`: Attach to a group.

**Examples:**
```bash
rp-cli policy attach strict-access --role contractor
```

### 6.3 `rp-cli policy detach`

Detaches a policy from a role, user, or group.

**Usage:**
```bash
rp-cli policy detach <policy-id> [flags]
```

**Examples:**
```bash
rp-cli policy detach strict-access --role contractor
```

## 7. Assignment Commands

Assignments link users or groups to roles.

### 7.1 `rp-cli assign user`

Assigns a role to a user.

**Usage:**
```bash
rp-cli assign user <user-id> <role-name>
```

**Examples:**
```bash
rp-cli assign user alice@example.com admin
```

### 7.2 `rp-cli assign group`

Assigns a role to a group.

**Usage:**
```bash
rp-cli assign group <group-id> <role-name>
```

**Examples:**
```bash
rp-cli assign group engineering-team developer
```

### 7.3 `rp-cli unassign user`

Removes a role assignment from a user.

**Usage:**
```bash
rp-cli unassign user <user-id> <role-name>
```

**Examples:**
```bash
rp-cli unassign user alice@example.com admin
```

## 8. Audit and Compliance Commands

Auditing is critical for maintaining security and compliance. `rp-cli` provides commands to review access logs and evaluate effective permissions.

### 8.1 `rp-cli audit logs`

Retrieves access and modification logs for roles and permissions.

**Usage:**
```bash
rp-cli audit logs [flags]
```

**Flags:**
- `--start-time <iso8601>`: Start time for the log query.
- `--end-time <iso8601>`: End time for the log query.
- `--actor <user-id>`: Filter logs by the user who performed the action.
- `--target <resource-id>`: Filter logs by the affected resource.

**Examples:**
```bash
rp-cli audit logs --start-time 2023-01-01T00:00:00Z --actor admin@example.com
```

### 8.2 `rp-cli audit evaluate`

Evaluates the effective permissions for a specific user on a specific resource. This is invaluable for troubleshooting "access denied" errors.

**Usage:**
```bash
rp-cli audit evaluate <user-id> <resource-id> <action>
```

**Examples:**
```bash
rp-cli audit evaluate bob@example.com db-prod-01 db:write
```

## 9. Advanced Usage and Scripting

`rp-cli` is designed to be easily integrated into shell scripts and automation pipelines.

### 9.1 JSON Output and `jq`

By using the `--output json` flag, you can pipe the output of `rp-cli` commands into tools like `jq` for advanced parsing and filtering.

**Example: Extracting all role names**
```bash
rp-cli role list --output json | jq -r '.[].name'
```

### 9.2 Bulk Operations

You can combine `rp-cli` with standard Unix tools like `xargs` to perform bulk operations.

**Example: Deleting multiple roles**
```bash
cat roles-to-delete.txt | xargs -I {} rp-cli role delete {} --force
```

## 10. Troubleshooting

If you encounter issues while using `rp-cli`, consider the following steps:

1. **Check Authentication:** Ensure your token is valid using `rp-cli auth status`.
2. **Enable Debug Logging:** Run your command with the `--debug` flag to see the raw API requests and responses. This often reveals underlying network or server errors.
3. **Verify Network Connectivity:** Ensure you can reach the API endpoint specified in your configuration.
4. **Review Permissions:** Ensure the user you are authenticated as has the necessary permissions to perform the action.

## 11. Conclusion

The `rp-cli` is a robust and versatile tool for managing roles and permissions. By mastering the commands and techniques outlined in this reference, you can ensure secure, efficient, and scalable access control across your organization. For further assistance, consult the official documentation or contact your support representative.

## Appendix: Additional Examples

### Extended Reference 1

## 1. Introduction

Welcome to the comprehensive Command Line Interface (CLI) reference for the Roles and Permissions management system. This document provides an exhaustive guide to all available commands, flags, arguments, and usage examples for managing roles, permissions, policies, and access control lists (ACLs) within your enterprise environment.

The Roles and Permissions CLI (`rp-cli`) is a powerful tool designed for system administrators, security engineers, and DevOps professionals to automate and manage access control at scale. It interacts directly with the Identity and Access Management (IAM) backend, ensuring that all changes are propagated securely and efficiently.

This guide is structured to cover everything from basic authentication to advanced policy management, troubleshooting, and best practices. Whether you are auditing current access levels or deploying new security models, this reference will serve as your definitive resource.

## 2. Global Flags and Configuration

Before diving into specific commands, it is essential to understand the global flags and configuration options available in `rp-cli`. These flags can be applied to almost any command to modify its behavior, output format, or execution context.

### 2.1 Global Flags

- `--config, -c <path>`: Specify a custom configuration file path. Default is `~/.rp-cli/config.yaml`.
- `--profile, -p <name>`: Use a specific profile from the configuration file. Useful for managing multiple environments (e.g., `dev`, `staging`, `prod`).
- `--region, -r <region>`: Specify the region for the API endpoint. Overrides the profile setting.
- `--output, -o <format>`: Define the output format. Supported formats: `json`, `yaml`, `table`, `text`. Default is `table`.
- `--verbose, -v`: Enable verbose logging. Useful for debugging.
- `--debug`: Enable debug-level logging, including raw API requests and responses.
- `--dry-run`: Simulate the command execution without making any actual changes to the backend.
- `--help, -h`: Display help information for the current command or subcommand.

### 2.2 Environment Variables

`rp-cli` also respects several environment variables, which can be used in CI/CD pipelines or automated scripts:

- `RP_CLI_TOKEN`: The authentication token for API access.
- `RP_CLI_PROFILE`: The default profile to use.
- `RP_CLI_REGION`: The default region.
- `RP_CLI_TIMEOUT`: The API request timeout in seconds (default: 30).

## 3. Authentication Commands

Authentication is the first step to using `rp-cli`. The CLI supports multiple authentication methods, including API tokens, OAuth2, and SSO integration.

### 3.1 `rp-cli auth login`

Authenticates the user and stores the session token locally.

**Usage:**
```bash
rp-cli auth login [flags]
```

**Flags:**
- `--method <method>`: Authentication method (`token`, `oauth2`, `sso`). Default is `oauth2`.
- `--token <token>`: Provide the token directly (useful for scripts).

**Examples:**
```bash
# Interactive OAuth2 login
rp-cli auth login

# Login using a specific token
rp-cli auth login --method token --token "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

### 3.2 `rp-cli auth logout`

Logs out the current user and clears the local session token.

**Usage:**
```bash
rp-cli auth logout [flags]
```

**Flags:**
- `--all`: Log out of all profiles.

**Examples:**
```bash
rp-cli auth logout
rp-cli auth logout --all
```

### 3.3 `rp-cli auth status`

Displays the current authentication status, including the active profile, user identity, and token expiration.

**Usage:**
```bash
rp-cli auth status
```

## 4. Role Management Commands

Roles are collections of permissions that can be assigned to users or groups. The following commands allow you to create, read, update, and delete roles.

### 4.1 `rp-cli role create`

Creates a new role in the system.

**Usage:**
```bash
rp-cli role create <role-name> [flags]
```

**Arguments:**
- `<role-name>`: The unique identifier for the role (e.g., `admin`, `developer`, `viewer`).

**Flags:**
- `--description, -d <text>`: A human-readable description of the role.
- `--permissions, -p <list>`: A comma-separated list of permission IDs to attach to the role.
- `--tags, -t <key=value>`: Key-value pairs for resource tagging.

**Examples:**
```bash
# Create a basic role
rp-cli role create developer --description "Standard developer access"

# Create a role with specific permissions and tags
rp-cli role create db-admin \
  --description "Database Administrator" \
  --permissions "db:read,db:write,db:delete" \
  --tags "department=engineering,env=prod"
```

### 4.2 `rp-cli role list`

Lists all available roles in the system.

**Usage:**
```bash
rp-cli role list [flags]
```

**Flags:**
- `--limit <number>`: Maximum number of roles to return (default: 50).
- `--offset <number>`: Pagination offset.
- `--filter <expression>`: Filter roles based on specific criteria (e.g., `name=admin*`).

**Examples:**
```bash
# List all roles
rp-cli role list

# List roles with a specific prefix
rp-cli role list --filter "name=dev-*"
```

### 4.3 `rp-cli role get`

Retrieves detailed information about a specific role.

**Usage:**
```bash
rp-cli role get <role-name> [flags]
```

**Examples:**
```bash
rp-cli role get developer --output json
```

### 4.4 `rp-cli role update`

Updates an existing role's properties.

**Usage:**
```bash
rp-cli role update <role-name> [flags]
```

**Flags:**
- `--description, -d <text>`: Update the description.
- `--add-permissions <list>`: Add permissions to the role.
- `--remove-permissions <list>`: Remove permissions from the role.

**Examples:**
```bash
rp-cli role update developer --add-permissions "repo:create,repo:delete"
```

### 4.5 `rp-cli role delete`

Deletes a role from the system.

**Usage:**
```bash
rp-cli role delete <role-name> [flags]
```

**Flags:**
- `--force, -f`: Bypass the confirmation prompt.

**Examples:**
```bash
rp-cli role delete old-role --force
```

## 5. Permission Management Commands

Permissions define specific actions that can be performed on resources. While permissions are usually predefined by the system, you can list and inspect them.

### 5.1 `rp-cli permission list`

Lists all available permissions.

**Usage:**
```bash
rp-cli permission list [flags]
```

**Flags:**
- `--resource <type>`: Filter permissions by resource type (e.g., `database`, `repository`).

**Examples:**
```bash
rp-cli permission list --resource database
```

### 5.2 `rp-cli permission get`

Retrieves details about a specific permission.

**Usage:**
```bash
rp-cli permission get <permission-id>
```

**Examples:**
```bash
rp-cli permission get db:write
```

## 6. Policy Management Commands

Policies are advanced constructs that define complex access rules, often involving conditions (e.g., time of day, IP address).

### 6.1 `rp-cli policy create`

Creates a new policy from a JSON or YAML file.

**Usage:**
```bash
rp-cli policy create [flags]
```

**Flags:**
- `--file, -f <path>`: Path to the policy definition file.

**Examples:**
```bash
rp-cli policy create --file ./strict-access-policy.json
```

### 6.2 `rp-cli policy attach`

Attaches a policy to a role, user, or group.

**Usage:**
```bash
rp-cli policy attach <policy-id> [flags]
```

**Flags:**
- `--role <role-name>`: Attach to a role.
- `--user <user-id>`: Attach to a user.
- `--group <group-id>`: Attach to a group.

**Examples:**
```bash
rp-cli policy attach strict-access --role contractor
```

### 6.3 `rp-cli policy detach`

Detaches a policy from a role, user, or group.

**Usage:**
```bash
rp-cli policy detach <policy-id> [flags]
```

**Examples:**
```bash
rp-cli policy detach strict-access --role contractor
```

## 7. Assignment Commands

Assignments link users or groups to roles.

### 7.1 `rp-cli assign user`

Assigns a role to a user.

**Usage:**
```bash
rp-cli assign user <user-id> <role-name>
```

**Examples:**
```bash
rp-cli assign user alice@example.com admin
```

### 7.2 `rp-cli assign group`

Assigns a role to a group.

**Usage:**
```bash
rp-cli assign group <group-id> <role-name>
```

**Examples:**
```bash
rp-cli assign group engineering-team developer
```

### 7.3 `rp-cli unassign user`

Removes a role assignment from a user.

**Usage:**
```bash
rp-cli unassign user <user-id> <role-name>
```

**Examples:**
```bash
rp-cli unassign user alice@example.com admin
```

## 8. Audit and Compliance Commands

Auditing is critical for maintaining security and compliance. `rp-cli` provides commands to review access logs and evaluate effective permissions.

### 8.1 `rp-cli audit logs`

Retrieves access and modification logs for roles and permissions.

**Usage:**
```bash
rp-cli audit logs [flags]
```

**Flags:**
- `--start-time <iso8601>`: Start time for the log query.
- `--end-time <iso8601>`: End time for the log query.
- `--actor <user-id>`: Filter logs by the user who performed the action.
- `--target <resource-id>`: Filter logs by the affected resource.

**Examples:**
```bash
rp-cli audit logs --start-time 2023-01-01T00:00:00Z --actor admin@example.com
```

### 8.2 `rp-cli audit evaluate`

Evaluates the effective permissions for a specific user on a specific resource. This is invaluable for troubleshooting "access denied" errors.

**Usage:**
```bash
rp-cli audit evaluate <user-id> <resource-id> <action>
```

**Examples:**
```bash
rp-cli audit evaluate bob@example.com db-prod-01 db:write
```

## 9. Advanced Usage and Scripting

`rp-cli` is designed to be easily integrated into shell scripts and automation pipelines.

### 9.1 JSON Output and `jq`

By using the `--output json` flag, you can pipe the output of `rp-cli` commands into tools like `jq` for advanced parsing and filtering.

**Example: Extracting all role names**
```bash
rp-cli role list --output json | jq -r '.[].name'
```

### 9.2 Bulk Operations

You can combine `rp-cli` with standard Unix tools like `xargs` to perform bulk operations.

**Example: Deleting multiple roles**
```bash
cat roles-to-delete.txt | xargs -I {} rp-cli role delete {} --force
```

## 10. Troubleshooting

If you encounter issues while using `rp-cli`, consider the following steps:

1. **Check Authentication:** Ensure your token is valid using `rp-cli auth status`.
2. **Enable Debug Logging:** Run your command with the `--debug` flag to see the raw API requests and responses. This often reveals underlying network or server errors.
3. **Verify Network Connectivity:** Ensure you can reach the API endpoint specified in your configuration.
4. **Review Permissions:** Ensure the user you are authenticated as has the necessary permissions to perform the action.

## 11. Conclusion

The `rp-cli` is a robust and versatile tool for managing roles and permissions. By mastering the commands and techniques outlined in this reference, you can ensure secure, efficient, and scalable access control across your organization. For further assistance, consult the official documentation or contact your support representative.

## Appendix: Additional Examples

### Extended Reference 2

## 1. Introduction

Welcome to the comprehensive Command Line Interface (CLI) reference for the Roles and Permissions management system. This document provides an exhaustive guide to all available commands, flags, arguments, and usage examples for managing roles, permissions, policies, and access control lists (ACLs) within your enterprise environment.

The Roles and Permissions CLI (`rp-cli`) is a powerful tool designed for system administrators, security engineers, and DevOps professionals to automate and manage access control at scale. It interacts directly with the Identity and Access Management (IAM) backend, ensuring that all changes are propagated securely and efficiently.

This guide is structured to cover everything from basic authentication to advanced policy management, troubleshooting, and best practices. Whether you are auditing current access levels or deploying new security models, this reference will serve as your definitive resource.

## 2. Global Flags and Configuration

Before diving into specific commands, it is essential to understand the global flags and configuration options available in `rp-cli`. These flags can be applied to almost any command to modify its behavior, output format, or execution context.

### 2.1 Global Flags

- `--config, -c <path>`: Specify a custom configuration file path. Default is `~/.rp-cli/config.yaml`.
- `--profile, -p <name>`: Use a specific profile from the configuration file. Useful for managing multiple environments (e.g., `dev`, `staging`, `prod`).
- `--region, -r <region>`: Specify the region for the API endpoint. Overrides the profile setting.
- `--output, -o <format>`: Define the output format. Supported formats: `json`, `yaml`, `table`, `text`. Default is `table`.
- `--verbose, -v`: Enable verbose logging. Useful for debugging.
- `--debug`: Enable debug-level logging, including raw API requests and responses.
- `--dry-run`: Simulate the command execution without making any actual changes to the backend.
- `--help, -h`: Display help information for the current command or subcommand.

### 2.2 Environment Variables

`rp-cli` also respects several environment variables, which can be used in CI/CD pipelines or automated scripts:

- `RP_CLI_TOKEN`: The authentication token for API access.
- `RP_CLI_PROFILE`: The default profile to use.
- `RP_CLI_REGION`: The default region.
- `RP_CLI_TIMEOUT`: The API request timeout in seconds (default: 30).

## 3. Authentication Commands

Authentication is the first step to using `rp-cli`. The CLI supports multiple authentication methods, including API tokens, OAuth2, and SSO integration.

### 3.1 `rp-cli auth login`

Authenticates the user and stores the session token locally.

**Usage:**
```bash
rp-cli auth login [flags]
```

**Flags:**
- `--method <method>`: Authentication method (`token`, `oauth2`, `sso`). Default is `oauth2`.
- `--token <token>`: Provide the token directly (useful for scripts).

**Examples:**
```bash
# Interactive OAuth2 login
rp-cli auth login

# Login using a specific token
rp-cli auth login --method token --token "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

### 3.2 `rp-cli auth logout`

Logs out the current user and clears the local session token.

**Usage:**
```bash
rp-cli auth logout [flags]
```

**Flags:**
- `--all`: Log out of all profiles.

**Examples:**
```bash
rp-cli auth logout
rp-cli auth logout --all
```

### 3.3 `rp-cli auth status`

Displays the current authentication status, including the active profile, user identity, and token expiration.

**Usage:**
```bash
rp-cli auth status
```

## 4. Role Management Commands

Roles are collections of permissions that can be assigned to users or groups. The following commands allow you to create, read, update, and delete roles.

### 4.1 `rp-cli role create`

Creates a new role in the system.

**Usage:**
```bash
rp-cli role create <role-name> [flags]
```

**Arguments:**
- `<role-name>`: The unique identifier for the role (e.g., `admin`, `developer`, `viewer`).

**Flags:**
- `--description, -d <text>`: A human-readable description of the role.
- `--permissions, -p <list>`: A comma-separated list of permission IDs to attach to the role.
- `--tags, -t <key=value>`: Key-value pairs for resource tagging.

**Examples:**
```bash
# Create a basic role
rp-cli role create developer --description "Standard developer access"

# Create a role with specific permissions and tags
rp-cli role create db-admin \
  --description "Database Administrator" \
  --permissions "db:read,db:write,db:delete" \
  --tags "department=engineering,env=prod"
```

### 4.2 `rp-cli role list`

Lists all available roles in the system.

**Usage:**
```bash
rp-cli role list [flags]
```

**Flags:**
- `--limit <number>`: Maximum number of roles to return (default: 50).
- `--offset <number>`: Pagination offset.
- `--filter <expression>`: Filter roles based on specific criteria (e.g., `name=admin*`).

**Examples:**
```bash
# List all roles
rp-cli role list

# List roles with a specific prefix
rp-cli role list --filter "name=dev-*"
```

### 4.3 `rp-cli role get`

Retrieves detailed information about a specific role.

**Usage:**
```bash
rp-cli role get <role-name> [flags]
```

**Examples:**
```bash
rp-cli role get developer --output json
```

### 4.4 `rp-cli role update`

Updates an existing role's properties.

**Usage:**
```bash
rp-cli role update <role-name> [flags]
```

**Flags:**
- `--description, -d <text>`: Update the description.
- `--add-permissions <list>`: Add permissions to the role.
- `--remove-permissions <list>`: Remove permissions from the role.

**Examples:**
```bash
rp-cli role update developer --add-permissions "repo:create,repo:delete"
```

### 4.5 `rp-cli role delete`

Deletes a role from the system.

**Usage:**
```bash
rp-cli role delete <role-name> [flags]
```

**Flags:**
- `--force, -f`: Bypass the confirmation prompt.

**Examples:**
```bash
rp-cli role delete old-role --force
```

## 5. Permission Management Commands

Permissions define specific actions that can be performed on resources. While permissions are usually predefined by the system, you can list and inspect them.

### 5.1 `rp-cli permission list`

Lists all available permissions.

**Usage:**
```bash
rp-cli permission list [flags]
```

**Flags:**
- `--resource <type>`: Filter permissions by resource type (e.g., `database`, `repository`).

**Examples:**
```bash
rp-cli permission list --resource database
```

### 5.2 `rp-cli permission get`

Retrieves details about a specific permission.

**Usage:**
```bash
rp-cli permission get <permission-id>
```

**Examples:**
```bash
rp-cli permission get db:write
```

## 6. Policy Management Commands

Policies are advanced constructs that define complex access rules, often involving conditions (e.g., time of day, IP address).

### 6.1 `rp-cli policy create`

Creates a new policy from a JSON or YAML file.

**Usage:**
```bash
rp-cli policy create [flags]
```

**Flags:**
- `--file, -f <path>`: Path to the policy definition file.

**Examples:**
```bash
rp-cli policy create --file ./strict-access-policy.json
```

### 6.2 `rp-cli policy attach`

Attaches a policy to a role, user, or group.

**Usage:**
```bash
rp-cli policy attach <policy-id> [flags]
```

**Flags:**
- `--role <role-name>`: Attach to a role.
- `--user <user-id>`: Attach to a user.
- `--group <group-id>`: Attach to a group.

**Examples:**
```bash
rp-cli policy attach strict-access --role contractor
```

### 6.3 `rp-cli policy detach`

Detaches a policy from a role, user, or group.

**Usage:**
```bash
rp-cli policy detach <policy-id> [flags]
```

**Examples:**
```bash
rp-cli policy detach strict-access --role contractor
```

## 7. Assignment Commands

Assignments link users or groups to roles.

### 7.1 `rp-cli assign user`

Assigns a role to a user.

**Usage:**
```bash
rp-cli assign user <user-id> <role-name>
```

**Examples:**
```bash
rp-cli assign user alice@example.com admin
```

### 7.2 `rp-cli assign group`

Assigns a role to a group.

**Usage:**
```bash
rp-cli assign group <group-id> <role-name>
```

**Examples:**
```bash
rp-cli assign group engineering-team developer
```

### 7.3 `rp-cli unassign user`

Removes a role assignment from a user.

**Usage:**
```bash
rp-cli unassign user <user-id> <role-name>
```

**Examples:**
```bash
rp-cli unassign user alice@example.com admin
```

## 8. Audit and Compliance Commands

Auditing is critical for maintaining security and compliance. `rp-cli` provides commands to review access logs and evaluate effective permissions.

### 8.1 `rp-cli audit logs`

Retrieves access and modification logs for roles and permissions.

**Usage:**
```bash
rp-cli audit logs [flags]
```

**Flags:**
- `--start-time <iso8601>`: Start time for the log query.
- `--end-time <iso8601>`: End time for the log query.
- `--actor <user-id>`: Filter logs by the user who performed the action.
- `--target <resource-id>`: Filter logs by the affected resource.

**Examples:**
```bash
rp-cli audit logs --start-time 2023-01-01T00:00:00Z --actor admin@example.com
```

### 8.2 `rp-cli audit evaluate`

Evaluates the effective permissions for a specific user on a specific resource. This is invaluable for troubleshooting "access denied" errors.

**Usage:**
```bash
rp-cli audit evaluate <user-id> <resource-id> <action>
```

**Examples:**
```bash
rp-cli audit evaluate bob@example.com db-prod-01 db:write
```

## 9. Advanced Usage and Scripting

`rp-cli` is designed to be easily integrated into shell scripts and automation pipelines.

### 9.1 JSON Output and `jq`

By using the `--output json` flag, you can pipe the output of `rp-cli` commands into tools like `jq` for advanced parsing and filtering.

**Example: Extracting all role names**
```bash
rp-cli role list --output json | jq -r '.[].name'
```

### 9.2 Bulk Operations

You can combine `rp-cli` with standard Unix tools like `xargs` to perform bulk operations.

**Example: Deleting multiple roles**
```bash
cat roles-to-delete.txt | xargs -I {} rp-cli role delete {} --force
```

## 10. Troubleshooting

If you encounter issues while using `rp-cli`, consider the following steps:

1. **Check Authentication:** Ensure your token is valid using `rp-cli auth status`.
2. **Enable Debug Logging:** Run your command with the `--debug` flag to see the raw API requests and responses. This often reveals underlying network or server errors.
3. **Verify Network Connectivity:** Ensure you can reach the API endpoint specified in your configuration.
4. **Review Permissions:** Ensure the user you are authenticated as has the necessary permissions to perform the action.

## 11. Conclusion

The `rp-cli` is a robust and versatile tool for managing roles and permissions. By mastering the commands and techniques outlined in this reference, you can ensure secure, efficient, and scalable access control across your organization. For further assistance, consult the official documentation or contact your support representative.

## Appendix: Additional Examples

### Extended Reference 3

## 1. Introduction

Welcome to the comprehensive Command Line Interface (CLI) reference for the Roles and Permissions management system. This document provides an exhaustive guide to all available commands, flags, arguments, and usage examples for managing roles, permissions, policies, and access control lists (ACLs) within your enterprise environment.

The Roles and Permissions CLI (`rp-cli`) is a powerful tool designed for system administrators, security engineers, and DevOps professionals to automate and manage access control at scale. It interacts directly with the Identity and Access Management (IAM) backend, ensuring that all changes are propagated securely and efficiently.

This guide is structured to cover everything from basic authentication to advanced policy management, troubleshooting, and best practices. Whether you are auditing current access levels or deploying new security models, this reference will serve as your definitive resource.

## 2. Global Flags and Configuration

Before diving into specific commands, it is essential to understand the global flags and configuration options available in `rp-cli`. These flags can be applied to almost any command to modify its behavior, output format, or execution context.

### 2.1 Global Flags

- `--config, -c <path>`: Specify a custom configuration file path. Default is `~/.rp-cli/config.yaml`.
- `--profile, -p <name>`: Use a specific profile from the configuration file. Useful for managing multiple environments (e.g., `dev`, `staging`, `prod`).
- `--region, -r <region>`: Specify the region for the API endpoint. Overrides the profile setting.
- `--output, -o <format>`: Define the output format. Supported formats: `json`, `yaml`, `table`, `text`. Default is `table`.
- `--verbose, -v`: Enable verbose logging. Useful for debugging.
- `--debug`: Enable debug-level logging, including raw API requests and responses.
- `--dry-run`: Simulate the command execution without making any actual changes to the backend.
- `--help, -h`: Display help information for the current command or subcommand.

### 2.2 Environment Variables

`rp-cli` also respects several environment variables, which can be used in CI/CD pipelines or automated scripts:

- `RP_CLI_TOKEN`: The authentication token for API access.
- `RP_CLI_PROFILE`: The default profile to use.
- `RP_CLI_REGION`: The default region.
- `RP_CLI_TIMEOUT`: The API request timeout in seconds (default: 30).

## 3. Authentication Commands

Authentication is the first step to using `rp-cli`. The CLI supports multiple authentication methods, including API tokens, OAuth2, and SSO integration.

### 3.1 `rp-cli auth login`

Authenticates the user and stores the session token locally.

**Usage:**
```bash
rp-cli auth login [flags]
```

**Flags:**
- `--method <method>`: Authentication method (`token`, `oauth2`, `sso`). Default is `oauth2`.
- `--token <token>`: Provide the token directly (useful for scripts).

**Examples:**
```bash
# Interactive OAuth2 login
rp-cli auth login

# Login using a specific token
rp-cli auth login --method token --token "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

### 3.2 `rp-cli auth logout`

Logs out the current user and clears the local session token.

**Usage:**
```bash
rp-cli auth logout [flags]
```

**Flags:**
- `--all`: Log out of all profiles.

**Examples:**
```bash
rp-cli auth logout
rp-cli auth logout --all
```

### 3.3 `rp-cli auth status`

Displays the current authentication status, including the active profile, user identity, and token expiration.

**Usage:**
```bash
rp-cli auth status
```

## 4. Role Management Commands

Roles are collections of permissions that can be assigned to users or groups. The following commands allow you to create, read, update, and delete roles.

### 4.1 `rp-cli role create`

Creates a new role in the system.

**Usage:**
```bash
rp-cli role create <role-name> [flags]
```

**Arguments:**
- `<role-name>`: The unique identifier for the role (e.g., `admin`, `developer`, `viewer`).

**Flags:**
- `--description, -d <text>`: A human-readable description of the role.
- `--permissions, -p <list>`: A comma-separated list of permission IDs to attach to the role.
- `--tags, -t <key=value>`: Key-value pairs for resource tagging.

**Examples:**
```bash
# Create a basic role
rp-cli role create developer --description "Standard developer access"

# Create a role with specific permissions and tags
rp-cli role create db-admin \
  --description "Database Administrator" \
  --permissions "db:read,db:write,db:delete" \
  --tags "department=engineering,env=prod"
```

### 4.2 `rp-cli role list`

Lists all available roles in the system.

**Usage:**
```bash
rp-cli role list [flags]
```

**Flags:**
- `--limit <number>`: Maximum number of roles to return (default: 50).
- `--offset <number>`: Pagination offset.
- `--filter <expression>`: Filter roles based on specific criteria (e.g., `name=admin*`).

**Examples:**
```bash
# List all roles
rp-cli role list

# List roles with a specific prefix
rp-cli role list --filter "name=dev-*"
```

### 4.3 `rp-cli role get`

Retrieves detailed information about a specific role.

**Usage:**
```bash
rp-cli role get <role-name> [flags]
```

**Examples:**
```bash
rp-cli role get developer --output json
```

### 4.4 `rp-cli role update`

Updates an existing role's properties.

**Usage:**
```bash
rp-cli role update <role-name> [flags]
```

**Flags:**
- `--description, -d <text>`: Update the description.
- `--add-permissions <list>`: Add permissions to the role.
- `--remove-permissions <list>`: Remove permissions from the role.

**Examples:**
```bash
rp-cli role update developer --add-permissions "repo:create,repo:delete"
```

### 4.5 `rp-cli role delete`

Deletes a role from the system.

**Usage:**
```bash
rp-cli role delete <role-name> [flags]
```

**Flags:**
- `--force, -f`: Bypass the confirmation prompt.

**Examples:**
```bash
rp-cli role delete old-role --force
```

## 5. Permission Management Commands

Permissions define specific actions that can be performed on resources. While permissions are usually predefined by the system, you can list and inspect them.

### 5.1 `rp-cli permission list`

Lists all available permissions.

**Usage:**
```bash
rp-cli permission list [flags]
```

**Flags:**
- `--resource <type>`: Filter permissions by resource type (e.g., `database`, `repository`).

**Examples:**
```bash
rp-cli permission list --resource database
```

### 5.2 `rp-cli permission get`

Retrieves details about a specific permission.

**Usage:**
```bash
rp-cli permission get <permission-id>
```

**Examples:**
```bash
rp-cli permission get db:write
```

## 6. Policy Management Commands

Policies are advanced constructs that define complex access rules, often involving conditions (e.g., time of day, IP address).

### 6.1 `rp-cli policy create`

Creates a new policy from a JSON or YAML file.

**Usage:**
```bash
rp-cli policy create [flags]
```

**Flags:**
- `--file, -f <path>`: Path to the policy definition file.

**Examples:**
```bash
rp-cli policy create --file ./strict-access-policy.json
```

### 6.2 `rp-cli policy attach`

Attaches a policy to a role, user, or group.

**Usage:**
```bash
rp-cli policy attach <policy-id> [flags]
```

**Flags:**
- `--role <role-name>`: Attach to a role.
- `--user <user-id>`: Attach to a user.
- `--group <group-id>`: Attach to a group.

**Examples:**
```bash
rp-cli policy attach strict-access --role contractor
```

### 6.3 `rp-cli policy detach`

Detaches a policy from a role, user, or group.

**Usage:**
```bash
rp-cli policy detach <policy-id> [flags]
```

**Examples:**
```bash
rp-cli policy detach strict-access --role contractor
```

## 7. Assignment Commands

Assignments link users or groups to roles.

### 7.1 `rp-cli assign user`

Assigns a role to a user.

**Usage:**
```bash
rp-cli assign user <user-id> <role-name>
```

**Examples:**
```bash
rp-cli assign user alice@example.com admin
```

### 7.2 `rp-cli assign group`

Assigns a role to a group.

**Usage:**
```bash
rp-cli assign group <group-id> <role-name>
```

**Examples:**
```bash
rp-cli assign group engineering-team developer
```

### 7.3 `rp-cli unassign user`

Removes a role assignment from a user.

**Usage:**
```bash
rp-cli unassign user <user-id> <role-name>
```

**Examples:**
```bash
rp-cli unassign user alice@example.com admin
```

## 8. Audit and Compliance Commands

Auditing is critical for maintaining security and compliance. `rp-cli` provides commands to review access logs and evaluate effective permissions.

### 8.1 `rp-cli audit logs`

Retrieves access and modification logs for roles and permissions.

**Usage:**
```bash
rp-cli audit logs [flags]
```

**Flags:**
- `--start-time <iso8601>`: Start time for the log query.
- `--end-time <iso8601>`: End time for the log query.
- `--actor <user-id>`: Filter logs by the user who performed the action.
- `--target <resource-id>`: Filter logs by the affected resource.

**Examples:**
```bash
rp-cli audit logs --start-time 2023-01-01T00:00:00Z --actor admin@example.com
```

### 8.2 `rp-cli audit evaluate`

Evaluates the effective permissions for a specific user on a specific resource. This is invaluable for troubleshooting "access denied" errors.

**Usage:**
```bash
rp-cli audit evaluate <user-id> <resource-id> <action>
```

**Examples:**
```bash
rp-cli audit evaluate bob@example.com db-prod-01 db:write
```

## 9. Advanced Usage and Scripting

`rp-cli` is designed to be easily integrated into shell scripts and automation pipelines.

### 9.1 JSON Output and `jq`

By using the `--output json` flag, you can pipe the output of `rp-cli` commands into tools like `jq` for advanced parsing and filtering.

**Example: Extracting all role names**
```bash
rp-cli role list --output json | jq -r '.[].name'
```

### 9.2 Bulk Operations

You can combine `rp-cli` with standard Unix tools like `xargs` to perform bulk operations.

**Example: Deleting multiple roles**
```bash
cat roles-to-delete.txt | xargs -I {} rp-cli role delete {} --force
```

## 10. Troubleshooting

If you encounter issues while using `rp-cli`, consider the following steps:

1. **Check Authentication:** Ensure your token is valid using `rp-cli auth status`.
2. **Enable Debug Logging:** Run your command with the `--debug` flag to see the raw API requests and responses. This often reveals underlying network or server errors.
3. **Verify Network Connectivity:** Ensure you can reach the API endpoint specified in your configuration.
4. **Review Permissions:** Ensure the user you are authenticated as has the necessary permissions to perform the action.

## 11. Conclusion

The `rp-cli` is a robust and versatile tool for managing roles and permissions. By mastering the commands and techniques outlined in this reference, you can ensure secure, efficient, and scalable access control across your organization. For further assistance, consult the official documentation or contact your support representative.