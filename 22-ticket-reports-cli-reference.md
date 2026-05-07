# ticket-reports CLI Reference

Welcome to the comprehensive command-line interface (CLI) reference guide for the `ticket-reports` tool. This document provides an in-depth look at the available commands, flags, arguments, and usage examples for `ticket-reports`, enabling you to effectively utilize this tool for generating, managing, and analyzing ticket reports.

## Table of Contents

1. [Overview](#overview)
2. [Installation](#installation)
3. [General Syntax](#general-syntax)
4. [Commands](#commands)
   - [init](#init)
   - [generate](#generate)
   - [list](#list)
   - [show](#show)
   - [delete](#delete)
   - [export](#export)
5. [Global Flags](#global-flags)
6. [Examples](#examples)

## Overview

The `ticket-reports` CLI tool is designed to facilitate the generation and management of reports related to ticketing systems. This tool supports various operations such as initializing a report, generating detailed ticket reports, listing available reports, displaying report details, and exporting reports to different formats.

## Installation

To install `ticket-reports`, you can use a package manager like `npm`:

```bash
npm install -g ticket-reports
```

Alternatively, you can download and install the binary from the official repository.

## General Syntax

The general syntax for using `ticket-reports` is as follows:

```bash
ticket-reports [command] [options] [arguments]
```

- **command**: The operation you wish to perform (e.g., `init`, `generate`).
- **options**: Flags that modify the behavior of the command.
- **arguments**: Additional parameters that provide input to the command.

## Commands

### init

The `init` command initializes a new ticket report configuration in the current directory.

#### Syntax

```bash
ticket-reports init [options]
```

#### Options

- `-n`, `--name <report-name>`: Specifies the name of the report configuration.
- `-t`, `--type <ticket-type>`: Defines the type of tickets to be included (e.g., `bug`, `feature`).
- `-p`, `--project <project-id>`: Associates the report with a specific project ID.

#### Example

```bash
ticket-reports init -n "Weekly Bug Report" -t bug -p 12345
```

### generate

The `generate` command creates a ticket report based on the initialized configuration.

#### Syntax

```bash
ticket-reports generate [options]
```

#### Options

- `-f`, `--filter <criteria>`: Applies filtering criteria to the tickets (e.g., `status:open`).
- `-s`, `--start-date <date>`: Sets the start date for the report period.
- `-e`, `--end-date <date>`: Sets the end date for the report period.
- `--force`: Forces report generation even if a report already exists.

#### Example

```bash
ticket-reports generate -f "priority:high" -s "2023-01-01" -e "2023-01-31"
```

### list

The `list` command displays all available ticket reports.

#### Syntax

```bash
ticket-reports list [options]
```

#### Options

- `--all`: Lists all reports, including archived ones.
- `--format <format>`: Specifies the output format (e.g., `table`, `json`).

#### Example

```bash
ticket-reports list --format json
```

### show

The `show` command displays detailed information about a specific report.

#### Syntax

```bash
ticket-reports show [options] <report-id>
```

#### Options

- `--details`: Includes detailed ticket information in the output.

#### Example

```bash
ticket-reports show --details 67890
```

### delete

The `delete` command removes a specific ticket report.

#### Syntax

```bash
ticket-reports delete [options] <report-id>
```

#### Options

- `--force`: Forces deletion without confirmation.

#### Example

```bash
ticket-reports delete --force 67890
```

### export

The `export` command exports a report to a specified format.

#### Syntax

```bash
ticket-reports export [options] <report-id>
```

#### Options

- `-o`, `--output <file-path>`: Specifies the file path for the exported report.
- `--format <format>`: Specifies the export format (e.g., `pdf`, `csv`).

#### Example

```bash
ticket-reports export -o "report.pdf" --format pdf 67890
```

## Global Flags

These flags can be used with any command:

- `--help`: Displays help information for the command.
- `--version`: Shows the version of the `ticket-reports` tool.
- `--verbose`: Enables verbose output for debugging purposes.

## Examples

Below are some detailed examples and use cases for the `ticket-reports` CLI tool.

### Example 1: Initializing and Generating a Report

1. **Initialize a report configuration for a monthly feature ticket report:**

   ```bash
   ticket-reports init -n "Monthly Feature Report" -t feature -p 67890
   ```

2. **Generate the report for the month of February 2023:**

   ```bash
   ticket-reports generate -s "2023-02-01" -e "2023-02-28"
   ```

### Example 2: Listing and Showing Reports

1. **List all reports in JSON format:**

   ```bash
   ticket-reports list --format json
   ```

2. **Show details of a specific report with ID 10101:**

   ```bash
   ticket-reports show --details 10101
   ```

### Example 3: Deleting and Exporting Reports

1. **Delete a report with ID 10101 without confirmation:**

   ```bash
   ticket-reports delete --force 10101
   ```

2. **Export a report to CSV format:**

   ```bash
   ticket-reports export --format csv -o "report.csv" 67890
   ```

In conclusion, the `ticket-reports` CLI tool offers a versatile and efficient way to manage and analyze ticket reports. With commands tailored for initialization, generation, listing, detailing, deletion, and exporting, users can streamline their ticket management workflows with ease. Make sure to explore the different options and flags to fully leverage the capabilities of `ticket-reports`.