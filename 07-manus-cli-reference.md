# Manus CLI Command Reference

## Table of Contents
1. [Architecture](#architecture)
2. [Configuration Files](#configuration-files)
3. [Environment Variables](#environment-variables)
4. [Error Codes](#error-codes)
5. [Best Practices](#best-practices)
6. [Commands](#commands)
   - [deploy](#deploy)
   - [test](#test)
   - [lint](#lint)
   - [serve](#serve)
   - [plugin](#plugin)
   - [auth](#auth)
   - [logs](#logs)
   - [metrics](#metrics)
   - [trace](#trace)
   - [debug](#debug)
   - [doctor](#doctor)
   - [upgrade](#upgrade)
   - [rollback](#rollback)
   - [snapshot](#snapshot)
   - [restore](#restore)
   - [export](#export)
   - [import](#import)
   - [sync](#sync)
   - [watch](#watch)
   - [clean](#clean)
   - [cache](#cache)
   - [env](#env)
   - [secret](#secret)
   - [alias](#alias)
   - [completion](#completion)
   - [help](#help)

## Architecture

The `manus` CLI is a command-line interface designed for managing and deploying applications. It is built with a modular architecture to facilitate extensibility and easy integration with various services and platforms. The core of the `manus` CLI is written in Go, ensuring high performance and reliability. Each command is a separate module that can be independently maintained and updated, allowing for agile development and rapid feature deployment.

### Core Components
- **Command Processor**: Handles parsing and execution of commands.
- **Plugin System**: Supports extending functionality through plugins.
- **Configuration Manager**: Manages configuration files and environment variables.
- **Error Handler**: Provides robust error handling and logging.

## Configuration Files

Configuration files for `manus` are typically written in YAML or JSON format. They define settings and preferences for the CLI and can be placed in the user's home directory or the project directory.

### Default Locations
- User Configuration: `~/.manus/config.yaml`
- Project Configuration: `./manus.yaml`

### Configuration Options
- **Default Environment**: Specifies the default environment (e.g., development, production).
- **API Endpoints**: Defines API endpoints for remote services.
- **Authentication**: Stores authentication credentials or tokens.
- **Logging**: Configures logging levels and formats.

### Example Configuration (YAML)
```yaml
default_environment: production
api_endpoints:
  deploy_service: https://api.example.com/deploy
authentication:
  token: YOUR_ACCESS_TOKEN
logging:
  level: info
  format: json
```

## Environment Variables

`manus` CLI uses environment variables to override configuration file settings or provide additional runtime parameters.

### Common Environment Variables
- `MANUS_CONFIG_PATH`: Path to a custom configuration file.
- `MANUS_LOG_LEVEL`: Overrides the logging level specified in the configuration.
- `MANUS_API_TOKEN`: Sets the API token for authentication.

### Example Usage
```bash
export MANUS_LOG_LEVEL=debug
manus deploy --force
```

## Error Codes

The `manus` CLI provides detailed error codes to help diagnose issues. These error codes are standardized to ensure consistency across different commands.

### Common Error Codes
- `0`: Success
- `1`: General Error
- `2`: Invalid Argument
- `3`: Network Error
- `4`: Authentication Failed
- `5`: Permission Denied

## Best Practices

- **Use Configuration Files**: Centralize configuration in files to simplify management.
- **Environment-Specific Settings**: Use different configuration files for different environments.
- **Version Control**: Keep your configuration files under version control.
- **Secure Secrets**: Use environment variables for sensitive information such as API tokens.
- **Regular Updates**: Regularly update the CLI and plugins to benefit from the latest features and security patches.

## Commands

### `deploy`

Deploys an application to a specified environment.

#### Usage
```bash
manus deploy [OPTIONS] <TARGET>
```

#### Options
- `-e, --environment <env>`: Specifies the target environment. Default is `production`.
- `-f, --force`: Forces the deployment even if there are pending changes.
- `-t, --tag <version>`: Deploys a specific version or tag.
- `-c, --config <file>`: Uses a specific configuration file.

#### Examples
1. Deploy to the default environment:
   ```bash
   manus deploy my-app
   ```
2. Force deploy to staging environment:
   ```bash
   manus deploy -e staging --force my-app
   ```
3. Deploy a specific version using a custom config file:
   ```bash
   manus deploy -t v1.2.3 -c ~/custom-manus.yaml my-app
   ```

### `test`

Runs tests for the application.

#### Usage
```bash
manus test [OPTIONS] <TARGET>
```

#### Options
- `-u, --unit`: Runs only unit tests.
- `-i, --integration`: Runs integration tests.
- `-a, --all`: Runs all available tests.
- `-r, --report <format>`: Generates a test report in the specified format (e.g., `json`, `xml`).

#### Examples
1. Run all tests for a target:
   ```bash
   manus test -a my-app
   ```
2. Run only unit tests:
   ```bash
   manus test --unit my-app
   ```
3. Generate a test report in JSON format:
   ```bash
   manus test -a -r json my-app
   ```

### `lint`

Analyzes the code for potential errors and style issues.

#### Usage
```bash
manus lint [OPTIONS] <TARGET>
```

#### Options
- `-f, --fix`: Automatically fix issues where possible.
- `-c, --config <file>`: Use a specific linter configuration file.
- `-o, --output <file>`: Output the lint results to a file.

#### Examples
1. Lint the entire codebase:
   ```bash
   manus lint my-app
   ```
2. Lint and automatically fix issues:
   ```bash
   manus lint --fix my-app
   ```
3. Output lint results to a file:
   ```bash
   manus lint -o lint-results.txt my-app
   ```

### `serve`

Starts a local development server.

#### Usage
```bash
manus serve [OPTIONS] <TARGET>
```

#### Options
- `-p, --port <port>`: Specifies the port number. Default is `8080`.
- `-h, --host <host>`: Specifies the host. Default is `localhost`.
- `-w, --watch`: Watches for file changes and reloads automatically.

#### Examples
1. Start a server on the default port:
   ```bash
   manus serve my-app
   ```
2. Start a server on a specific port and host:
   ```bash
   manus serve -p 3000 -h 0.0.0.0 my-app
   ```
3. Start a server with file watching enabled:
   ```bash
   manus serve --watch my-app
   ```

### `plugin`

Manages plugins for the `manus` CLI.

#### Usage
```bash
manus plugin [COMMAND] [OPTIONS]
```

#### Subcommands
- `install <name>`: Installs a plugin by name.
- `remove <name>`: Removes a plugin by name.
- `list`: Lists all installed plugins.

#### Examples
1. Install a plugin:
   ```bash
   manus plugin install example-plugin
   ```
2. Remove a plugin:
   ```bash
   manus plugin remove example-plugin
   ```
3. List installed plugins:
   ```bash
   manus plugin list
   ```

### `auth`

Manages authentication and credentials.

#### Usage
```bash
manus auth [COMMAND] [OPTIONS]
```

#### Subcommands
- `login`: Logs in to a service.
- `logout`: Logs out from a service.
- `status`: Displays the current authentication status.

#### Examples
1. Log in to a service:
   ```bash
   manus auth login
   ```
2. Log out from a service:
   ```bash
   manus auth logout
   ```
3. Check authentication status:
   ```bash
   manus auth status
   ```

### `logs`

Fetches and displays logs for a specified service.

#### Usage
```bash
manus logs [OPTIONS] <SERVICE>
```

#### Options
- `-f, --follow`: Continuously streams logs.
- `-n, --number <count>`: Displays the last `n` lines of logs.
- `-l, --level <level>`: Filters logs by level (e.g., `info`, `error`).

#### Examples
1. Display the last 100 lines of logs:
   ```bash
   manus logs -n 100 my-service
   ```
2. Stream logs in real-time:
   ```bash
   manus logs --follow my-service
   ```
3. Display only error logs:
   ```bash
   manus logs -l error my-service
   ```

### `metrics`

Retrieves and displays metrics for a specified application.

#### Usage
```bash
manus metrics [OPTIONS] <TARGET>
```

#### Options
- `-t, --time <range>`: Specifies the time range for metrics (e.g., `1h`, `24h`).
- `-f, --format <format>`: Specifies the output format (e.g., `table`, `json`).
- `-m, --metric <name>`: Retrieves a specific metric.

#### Examples
1. Retrieve metrics for the last hour:
   ```bash
   manus metrics -t 1h my-app
   ```
2. Output metrics in JSON format:
   ```bash
   manus metrics -f json my-app
   ```
3. Retrieve a specific metric:
   ```bash
   manus metrics -m cpu_usage my-app
   ```

### `trace`

Generates a trace report for debugging purposes.

#### Usage
```bash
manus trace [OPTIONS] <TARGET>
```

#### Options
- `-d, --duration <seconds>`: Specifies the duration for tracing.
- `-o, --output <file>`: Outputs the trace report to a file.
- `-s, --summary`: Generates a summary of the trace.

#### Examples
1. Generate a trace for 30 seconds:
   ```bash
   manus trace -d 30 my-app
   ```
2. Output the trace report to a file:
   ```bash
   manus trace -o trace-report.txt my-app
   ```
3. Generate a summary of the trace:
   ```bash
   manus trace --summary my-app
   ```

### `debug`

Enables debugging mode for the application.

#### Usage
```bash
manus debug [OPTIONS] <TARGET>
```

#### Options
- `-p, --port <port>`: Specifies the debug port. Default is `9229`.
- `-b, --break`: Breaks at the start of the application.
- `-c, --config <file>`: Uses a specific debug configuration file.

#### Examples
1. Start debugging on the default port:
   ```bash
   manus debug my-app
   ```
2. Start debugging and break at the start:
   ```bash
   manus debug --break my-app
   ```
3. Use a specific debug configuration file:
   ```bash
   manus debug -c debug-config.yaml my-app
   ```

### `doctor`

Checks the environment and configuration for potential issues.

#### Usage
```bash
manus doctor [OPTIONS]
```

#### Options
- `-f, --fix`: Attempts to fix detected issues.
- `-v, --verbose`: Outputs detailed information during the check.

#### Examples
1. Run a basic environment check:
   ```bash
   manus doctor
   ```
2. Run a check and attempt to fix issues:
   ```bash
   manus doctor --fix
   ```
3. Run a detailed environment check:
   ```bash
   manus doctor --verbose
   ```

### `upgrade`

Upgrades the `manus` CLI and plugins to the latest version.

#### Usage
```bash
manus upgrade [OPTIONS]
```

#### Options
- `-c, --check`: Checks for available updates without upgrading.
- `-p, --plugin <name>`: Upgrades a specific plugin.
- `-f, --force`: Forces the upgrade even if the current version is up-to-date.

#### Examples
1. Upgrade the CLI and all plugins:
   ```bash
   manus upgrade
   ```
2. Check for available updates:
   ```bash
   manus upgrade --check
   ```
3. Upgrade a specific plugin:
   ```bash
   manus upgrade --plugin example-plugin
   ```

### `rollback`

Rolls back the application to a previous version.

#### Usage
```bash
manus rollback [OPTIONS] <TARGET>
```

#### Options
- `-t, --tag <version>`: Specifies the version to roll back to.
- `-f, --force`: Forces the rollback even if there are pending changes.
- `-c, --confirm`: Confirms the rollback without prompting.

#### Examples
1. Roll back to the previous version:
   ```bash
   manus rollback my-app
   ```
2. Roll back to a specific version:
   ```bash
   manus rollback -t v1.1.0 my-app
   ```
3. Forcefully roll back to a specific version:
   ```bash
   manus rollback --force -t v1.1.0 my-app
   ```

### `snapshot`

Creates a snapshot of the current application state.

#### Usage
```bash
manus snapshot [OPTIONS] <TARGET>
```

#### Options
- `-n, --name <snapshot-name>`: Names the snapshot.
- `-d, --description <text>`: Adds a description to the snapshot.
- `-l, --label <label>`: Adds a label to the snapshot for filtering.

#### Examples
1. Create a snapshot with a name:
   ```bash
   manus snapshot -n "pre-deploy" my-app
   ```
2. Add a description to the snapshot:
   ```bash
   manus snapshot -d "Snapshot before major upgrade" my-app
   ```
3. Add a label to the snapshot:
   ```bash
   manus snapshot -l "v1.2" my-app
   ```

### `restore`

Restores an application from a snapshot.

#### Usage
```bash
manus restore [OPTIONS] <TARGET>
```

#### Options
- `-n, --name <snapshot-name>`: Specifies the snapshot to restore.
- `-f, --force`: Forces the restoration even if there are pending changes.
- `-c, --confirm`: Confirms the restoration without prompting.

#### Examples
1. Restore from a named snapshot:
   ```bash
   manus restore -n "pre-deploy" my-app
   ```
2. Forcefully restore from a snapshot:
   ```bash
   manus restore --force -n "pre-deploy" my-app
   ```
3. Confirm restoration from a snapshot:
   ```bash
   manus restore --confirm -n "pre-deploy" my-app
   ```

### `export`

Exports application data to a specified format.

#### Usage
```bash
manus export [OPTIONS] <TARGET>
```

#### Options
- `-f, --format <format>`: Specifies the export format (e.g., `json`, `csv`).
- `-o, --output <file>`: Specifies the output file for the export.
- `-c, --compress`: Compresses the exported data.

#### Examples
1. Export data to JSON format:
   ```bash
   manus export -f json my-app
   ```
2. Export data to a CSV file and compress:
   ```bash
   manus export -f csv -o data.csv --compress my-app
   ```
3. Export data to a specific file:
   ```bash
   manus export -o export.json my-app
   ```

### `import`

Imports data into the application.

#### Usage
```bash
manus import [OPTIONS] <TARGET>
```

#### Options
- `-f, --file <file>`: Specifies the file to import.
- `-d, --dry-run`: Performs a dry run without making changes.
- `-m, --map <mapping>`: Applies a custom mapping to the import process.

#### Examples
1. Import data from a file:
   ```bash
   manus import -f data.json my-app
   ```
2. Perform a dry run of the import:
   ```bash
   manus import --dry-run -f data.json my-app
   ```
3. Import data with a custom mapping:
   ```bash
   manus import -m "id:ID,name:NAME" -f data.csv my-app
   ```

### `sync`

Synchronizes the application state with a remote source.

#### Usage
```bash
manus sync [OPTIONS] <TARGET>
```

#### Options
- `-r, --remote <url>`: Specifies the remote source URL.
- `-d, --dry-run`: Performs a dry run without making changes.
- `-f, --force`: Forces synchronization even if the local state is ahead.

#### Examples
1. Sync with a remote source:
   ```bash
   manus sync -r https://remote.example.com my-app
   ```
2. Perform a dry run of the sync:
   ```bash
   manus sync --dry-run -r https://remote.example.com my-app
   ```
3. Force synchronization:
   ```bash
   manus sync --force -r https://remote.example.com my-app
   ```

### `watch`

Watches files for changes and triggers actions.

#### Usage
```bash
manus watch [OPTIONS] <TARGET>
```

#### Options
- `-p, --pattern <glob>`: Specifies the file pattern to watch.
- `-c, --command <cmd>`: Specifies the command to run on change.
- `-d, --debounce <milliseconds>`: Sets the debounce interval for change detection.

#### Examples
1. Watch all JavaScript files for changes:
   ```bash
   manus watch -p "*.js" my-app
   ```
2. Run a command on file change:
   ```bash
   manus watch -c "npm test" my-app
   ```
3. Set a debounce interval for change detection:
   ```bash
   manus watch -d 500 -p "*.css" my-app
   ```

### `clean`

Cleans up temporary files and caches.

#### Usage
```bash
manus clean [OPTIONS] <TARGET>
```

#### Options
- `-c, --cache`: Cleans the cache.
- `-t, --temp`: Cleans temporary files.
- `-f, --force`: Forces the clean operation.

#### Examples
1. Clean the cache:
   ```bash
   manus clean --cache my-app
   ```
2. Clean temporary files:
   ```bash
   manus clean --temp my-app
   ```
3. Forcefully clean all:
   ```bash
   manus clean --force --cache --temp my-app
   ```

### `cache`

Manages the application cache.

#### Usage
```bash
manus cache [COMMAND] [OPTIONS]
```

#### Subcommands
- `clear`: Clears the cache.
- `list`: Lists cached items.
- `status`: Displays cache status.

#### Examples
1. Clear the cache:
   ```bash
   manus cache clear
   ```
2. List cached items:
   ```bash
   manus cache list
   ```
3. Display cache status:
   ```bash
   manus cache status
   ```

### `env`

Manages environment variables for the application.

#### Usage
```bash
manus env [COMMAND] [OPTIONS]
```

#### Subcommands
- `set <key> <value>`: Sets an environment variable.
- `get <key>`: Gets the value of an environment variable.
- `list`: Lists all environment variables.

#### Examples
1. Set an environment variable:
   ```bash
   manus env set NODE_ENV production
   ```
2. Get the value of an environment variable:
   ```bash
   manus env get NODE_ENV
   ```
3. List all environment variables:
   ```bash
   manus env list
   ```

### `secret`

Manages secrets for the application.

#### Usage
```bash
manus secret [COMMAND] [OPTIONS]
```

#### Subcommands
- `add <key> <value>`: Adds a secret.
- `remove <key>`: Removes a secret.
- `list`: Lists all secrets.

#### Examples
1. Add a secret:
   ```bash
   manus secret add API_KEY 12345
   ```
2. Remove a secret:
   ```bash
   manus secret remove API_KEY
   ```
3. List all secrets:
   ```bash
   manus secret list
   ```

### `alias`

Manages command aliases for the `manus` CLI.

#### Usage
```bash
manus alias [COMMAND] [OPTIONS]
```

#### Subcommands
- `add <name> <command>`: Adds a new alias.
- `remove <name>`: Removes an existing alias.
- `list`: Lists all aliases.

#### Examples
1. Add a new alias:
   ```bash
   manus alias add d "deploy -e development"
   ```
2. Remove an alias:
   ```bash
   manus alias remove d
   ```
3. List all aliases:
   ```bash
   manus alias list
   ```

### `completion`

Generates shell completion scripts.

#### Usage
```bash
manus completion [OPTIONS] <shell>
```

#### Options
- `-o, --output <file>`: Outputs the completion script to a file.

#### Examples
1. Generate a Bash completion script:
   ```bash
   manus completion bash
   ```
2. Generate a Zsh completion script:
   ```bash
   manus completion zsh
   ```
3. Output the script to a file:
   ```bash
   manus completion bash -o ~/bash_completion.sh
   ```

### `help`

Displays help information for commands.

#### Usage
```bash
manus help [COMMAND]
```

#### Examples
1. Display general help information:
   ```bash
   manus help
   ```
2. Display help for a specific command:
   ```bash
   manus help deploy
   ```

This comprehensive reference provides detailed insights into the `manus` CLI. Ensure you're familiar with the configuration and best practices to make the most out of its extensive capabilities.