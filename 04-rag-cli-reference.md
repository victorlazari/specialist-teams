# RAG CLI Command Reference

This document provides a comprehensive reference for the `rag` command-line interface (CLI). The `rag` CLI is a powerful tool designed for managing, deploying, and interacting with Recurrent Augmentation Generators (RAGs) in various computing environments. This reference includes a deep dive into each command, its associated flags and arguments, and detailed examples of usage to help both new and experienced users effectively utilize the tool.

## Table of Contents

- [Installation](#installation)
- [Getting Started](#getting-started)
- [Command Syntax](#command-syntax)
- [Commands](#commands)
  - [init](#init)
  - [deploy](#deploy)
  - [status](#status)
  - [update](#update)
  - [rollback](#rollback)
  - [logs](#logs)
  - [config](#config)
  - [cleanup](#cleanup)
- [Examples](#examples)

## Installation

To install the `rag` CLI, download the latest release from its official repository or install it via a package manager. For example, to install via Homebrew:

```bash
brew install rag-cli
```

## Getting Started

Once installed, you can verify the installation by running:

```bash
rag --version
```

This should return the current version of the `rag` CLI, confirming a successful installation.

## Command Syntax

The `rag` CLI commands follow the general syntax:

```bash
rag [command] [subcommand] [flags] [arguments]
```

Each command may have specific subcommands, flags, and arguments that modify its behavior.

## Commands

### init

Initializes a new RAG project in the current directory.

#### Usage

```bash
rag init [options]
```

#### Options

- `-n, --name <project-name>`: Specifies the name of the project. Defaults to the current directory name.
- `-t, --template <template-name>`: Initializes the project using a specific template.
- `-d, --directory <path>`: Specifies a custom directory to initialize the project.

#### Examples

```bash
rag init --name my-rag-project
rag init -t basic-template
rag init -d /path/to/initialize
```

### deploy

Deploys the RAG to a specified environment.

#### Usage

```bash
rag deploy [environment] [options]
```

#### Arguments

- `environment`: The target environment for deployment (e.g., `staging`, `production`).

#### Options

- `-c, --config <file>`: Specifies the configuration file to use for deployment.
- `-f, --force`: Forces the deployment, even if there are warnings or errors.
- `--dry-run`: Simulates the deployment process without making any changes.

#### Examples

```bash
rag deploy production -c config.yaml
rag deploy staging --dry-run
```

### status

Displays the current status of the RAG in a specified environment.

#### Usage

```bash
rag status [environment]
```

#### Arguments

- `environment`: The environment whose status you want to check.

#### Examples

```bash
rag status production
rag status staging
```

### update

Updates the RAG configuration or codebase in a specified environment.

#### Usage

```bash
rag update [environment] [options]
```

#### Arguments

- `environment`: The environment where the update will be applied.

#### Options

- `-c, --config <file>`: Specifies the configuration file to use for the update.
- `-r, --restart`: Restarts the RAG after the update.

#### Examples

```bash
rag update production -c new-config.yaml
rag update staging --restart
```

### rollback

Rolls back the RAG to a previous state in a specified environment.

#### Usage

```bash
rag rollback [environment] [options]
```

#### Arguments

- `environment`: The environment where the rollback will occur.

#### Options

- `-v, --version <version>`: Specifies the version to roll back to.
- `--force`: Forces the rollback, even if there are warnings or errors.

#### Examples

```bash
rag rollback production -v 1.2.3
rag rollback staging --force
```

### logs

Fetches and displays logs for the RAG from a specified environment.

#### Usage

```bash
rag logs [environment] [options]
```

#### Arguments

- `environment`: The environment from which to fetch logs.

#### Options

- `-n, --lines <number>`: Specifies the number of lines to display from the logs.
- `-f, --follow`: Continuously outputs new log entries as they are written.

#### Examples

```bash
rag logs production -n 100
rag logs staging --follow
```

### config

Manages configuration settings for the RAG.

#### Usage

```bash
rag config [subcommand] [options]
```

#### Subcommands

- `set <key> <value>`: Sets a configuration key to a specified value.
- `get <key>`: Retrieves the value of a specified configuration key.
- `list`: Lists all configuration settings.

#### Examples

```bash
rag config set max_retries 5
rag config get max_retries
rag config list
```

### cleanup

Cleans up resources associated with the RAG in a specified environment.

#### Usage

```bash
rag cleanup [environment] [options]
```

#### Arguments

- `environment`: The environment where the cleanup will occur.

#### Options

- `--all`: Cleans up all resources, including logs and temporary files.
- `--dry-run`: Simulates the cleanup process without making any changes.

#### Examples

```bash
rag cleanup production --all
rag cleanup staging --dry-run
```

## Examples

### Example 1: Initialize and Deploy

Initialize a new RAG project and deploy it to the production environment.

```bash
rag init --name my-new-project
cd my-new-project
rag deploy production -c deploy-config.yaml
```

### Example 2: Update and Check Status

Update the RAG configuration in the staging environment and check its status.

```bash
rag update staging -c update-config.yaml --restart
rag status staging
```

### Example 3: Rollback and Retrieve Logs

Rollback the RAG in the production environment to a previous version and retrieve the latest 50 log lines.

```bash
rag rollback production -v 1.1.0
rag logs production -n 50
```

This comprehensive reference should serve as a useful guide to understanding and using the `rag` CLI effectively. For any further details, users are encouraged to explore the `--help` option available for each command to view additional information directly from the CLI.