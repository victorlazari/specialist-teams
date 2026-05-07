# Claude CLI Command Reference

The Claude CLI is a command-line interface for interacting with the Claude system, providing a comprehensive set of commands for managing and operating Claude environments. This documentation provides an exhaustive reference for each command, flag, and argument available in the Claude CLI.

## Table of Contents

1. [Installation](#installation)
2. [Configuration](#configuration)
3. [Usage](#usage)
4. [Commands](#commands)
    - [help](#help)
    - [version](#version)
    - [init](#init)
    - [start](#start)
    - [stop](#stop)
    - [status](#status)
    - [deploy](#deploy)
    - [logs](#logs)
    - [config](#config)
5. [Examples](#examples)

## Installation

To install the Claude CLI, you need to have Python 3.6 or higher installed on your system. Use pip to install the CLI:

```bash
pip install claude-cli
```

Verify the installation by running:

```bash
claude --version
```

## Configuration

Before using the Claude CLI, you must configure it with your Claude environment details. By default, Claude uses a configuration file located at `~/.claude/config.yaml`. You can specify a different configuration file using the `--config` flag with each command.

### Configuration File Format

The configuration file should be in YAML format:

```yaml
api_key: YOUR_API_KEY
endpoint: https://api.claude.example.com
default_environment: production
```

- **api_key**: Your personal or organizational API key for accessing Claude services.
- **endpoint**: The base URL for the Claude API.
- **default_environment**: The default environment to use if none is specified in commands.

## Usage

The basic syntax for the Claude CLI is:

```bash
claude [command] [options]
```

Use the `help` command to get a list of all available commands or to get help on a specific command.

## Commands

### help

Provides help information for Claude CLI commands.

#### Syntax

```bash
claude help [command]
```

#### Flags

- **-h, --help**: Show help message and exit.

#### Examples

- To view general help:

  ```bash
  claude help
  ```

- To view help for a specific command, e.g., `deploy`:

  ```bash
  claude help deploy
  ```

### version

Displays the current version of the Claude CLI.

#### Syntax

```bash
claude version
```

#### Examples

- To check the version of Claude CLI:

  ```bash
  claude version
  ```

### init

Initializes a new Claude project in the current directory.

#### Syntax

```bash
claude init [options]
```

#### Flags

- **-d, --directory**: Specify the directory to initialize the project in. Defaults to the current directory.
- **-t, --template**: Use a specific template for initialization. Options include `basic`, `web`, `service`.

#### Examples

- Initialize a project in the current directory with the basic template:

  ```bash
  claude init --template basic
  ```

- Initialize a project in a specified directory with the web template:

  ```bash
  claude init --directory /path/to/project --template web
  ```

### start

Starts the Claude environment.

#### Syntax

```bash
claude start [environment] [options]
```

#### Arguments

- **environment**: The environment to start. If omitted, the default environment is used.

#### Flags

- **-f, --foreground**: Run in the foreground.
- **-d, --daemon**: Run as a background process.

#### Examples

- Start the default environment in the foreground:

  ```bash
  claude start --foreground
  ```

- Start a specific environment as a daemon:

  ```bash
  claude start production --daemon
  ```

### stop

Stops the Claude environment.

#### Syntax

```bash
claude stop [environment] [options]
```

#### Arguments

- **environment**: The environment to stop. If omitted, the default environment is used.

#### Examples

- Stop the default environment:

  ```bash
  claude stop
  ```

- Stop a specific environment:

  ```bash
  claude stop production
  ```

### status

Checks the status of the Claude environment.

#### Syntax

```bash
claude status [environment] [options]
```

#### Arguments

- **environment**: The environment to check the status of. If omitted, the default environment is used.

#### Flags

- **-v, --verbose**: Display detailed status information.

#### Examples

- Check the status of the default environment:

  ```bash
  claude status
  ```

- Check the status of a specific environment with detailed information:

  ```bash
  claude status production --verbose
  ```

### deploy

Deploys the latest code to the Claude environment.

#### Syntax

```bash
claude deploy [environment] [options]
```

#### Arguments

- **environment**: The environment to deploy to. If omitted, the default environment is used.

#### Flags

- **-b, --branch**: Specify the branch to deploy from. Defaults to `main`.
- **-f, --force**: Force deployment even if there are uncommitted changes.

#### Examples

- Deploy the main branch to the default environment:

  ```bash
  claude deploy
  ```

- Deploy a specific branch to a specific environment:

  ```bash
  claude deploy staging --branch feature/new-feature
  ```

### logs

Fetches logs from the Claude environment.

#### Syntax

```bash
claude logs [environment] [options]
```

#### Arguments

- **environment**: The environment to fetch logs from. If omitted, the default environment is used.

#### Flags

- **-t, --tail**: Tail logs in real-time.
- **-n, --lines**: Specify the number of lines to display. Defaults to 100.

#### Examples

- Fetch the latest 100 lines of logs from the default environment:

  ```bash
  claude logs
  ```

- Tail logs in real-time from a specific environment:

  ```bash
  claude logs production --tail
  ```

### config

Manages the Claude CLI configuration.

#### Syntax

```bash
claude config [options]
```

#### Flags

- **-s, --set [key=value]**: Set a configuration key to a specific value.
- **-g, --get [key]**: Get the value of a specific configuration key.
- **-r, --reset [key]**: Reset a specific configuration key to its default value.

#### Examples

- Set the API key:

  ```bash
  claude config --set api_key=NEW_API_KEY
  ```

- Get the current endpoint configuration:

  ```bash
  claude config --get endpoint
  ```

- Reset the default environment configuration:

  ```bash
  claude config --reset default_environment
  ```

## Examples

### Initialize and Deploy a New Project

1. Initialize a new project with the service template:

   ```bash
   claude init --template service
   ```

2. Start the development environment:

   ```bash
   claude start development
   ```

3. Deploy to the staging environment:

   ```bash
   claude deploy staging --branch develop
   ```

### Monitor and Manage Environments

1. Check the status of the production environment with verbose output:

   ```bash
   claude status production --verbose
   ```

2. Fetch and tail logs from the production environment:

   ```bash
   claude logs production --tail
   ```

3. Stop the production environment:

   ```bash
   claude stop production
   ```

This documentation provides a comprehensive overview of the Claude CLI, detailing each command, flag, and argument to empower users to effectively manage and operate their Claude environments. For further support, consult the official Claude documentation or reach out to the support team.